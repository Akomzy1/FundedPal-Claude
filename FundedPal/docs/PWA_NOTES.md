# PWA Notes — FundedPal

Reference notes for the Progressive Web App layer. Covers iOS quirks, Web Push setup, caching strategy, and common gotchas. Keep this in `/docs/` and update it as you learn more about real-world device behaviour.

---

## Why PWA (and the limits to know about)

FundedPal ships as a PWA because:

- Traders live on desktop during sessions — the dashboard is desktop-first
- Mobile use is mostly notifications + one-tap signal execution — Web Push handles this
- No App Store / Play Store review delays for trading-rule updates, which need to ship fast
- Single codebase keeps build velocity high

The honest limits:

- **iOS Web Push requires PWA install** — Safari only delivers push notifications if the user has added FundedPal to their home screen. About 30% of iOS users won't do this. Mitigation: email notification fallback (Phase 8) and a clear "Install to receive signals" prompt for iOS users.
- **iOS background execution is unreliable** — service workers wake up briefly for push events, then are killed. Don't rely on them for anything beyond receiving notifications.
- **No `beforeinstallprompt` on iOS** — Apple never implemented it. iOS users must use the Safari share sheet → "Add to Home Screen". Show an instructional card instead of a programmatic prompt.

The reason none of this kills the product: **auto-execute runs server-side via the MT5 EA bridge.** The browser doesn't need to be open for trades to happen. The PWA is the interface; the bridge does the executing. So even if a user's phone is off, the system still trades their account within the rules.

---

## Architecture

```
┌─────────────────┐         ┌──────────────────┐
│  Browser / PWA  │◄────────│  Service Worker  │
│  (Next.js app)  │         │  (Serwist)       │
└────────┬────────┘         └──────────────────┘
         │                            ▲
         │ HTTPS                      │ Push Event
         ▼                            │
┌─────────────────┐         ┌──────────────────┐
│  Vercel Edge    │         │  Web Push        │
│  (Next.js SSR)  │         │  Server (VAPID)  │
└────────┬────────┘         └──────────────────┘
         │                            ▲
         ▼                            │
┌─────────────────┐                   │
│   Supabase      │───────────────────┘
│   (DB + Auth)   │   (signals trigger
└────────┬────────┘    push send via
         │             webhook)
         ▼
┌─────────────────┐
│  MT5 EA Bridge  │
│  (Railway)      │
└─────────────────┘
```

Key separation: **the PWA never executes trades directly.** Trades flow EA bridge → server → broker. The PWA only displays state and accepts taps that send signed requests to the bridge.

---

## Service worker caching strategy

We use `@serwist/next` (modern, actively maintained replacement for next-pwa). Configured in `next.config.ts`.

### Routes by strategy

| Route pattern | Strategy | Reasoning |
|---|---|---|
| `/api/*` | `NetworkOnly` | Trading data must be fresh. Stale data is worse than no data. |
| `/actions/*` | `NetworkOnly` | Server actions never cached. |
| `/app/*` (dashboard) | `NetworkFirst` (3s timeout) | Try fresh, fall back to last-cached only if offline. |
| `/_next/static/*` | `CacheFirst` | Hashed assets, immutable. |
| `/_next/image` | `StaleWhileRevalidate` | Images can be slightly stale. |
| Fonts (`*.woff2`) | `CacheFirst` (1yr) | Fonts never change. |
| `/icons/*` | `CacheFirst` (1yr) | App icons never change. |
| `/` (marketing) | `StaleWhileRevalidate` | Marketing can be slightly stale. |
| `/firms/*` (public) | `StaleWhileRevalidate` | Public firm directory. |

### Critical: never cache trading state

Equity, positions, P&L, rule usage, and compliance decisions must NEVER be served from cache. This is enforced two ways:

1. `NetworkOnly` strategy on `/api/*` and `/actions/*`
2. `export const dynamic = 'force-dynamic'` on every dashboard page

If you're ever tempted to cache trading data "just for offline state", don't. Showing yesterday's equity as if it's live is worse than showing nothing.

### Offline shell

When fully offline, show a designed `/offline.html` shell that:
- Displays the FundedPal wordmark in Fraunces
- Says "You're offline. Trading state is paused on this device."
- Reassures: "Your accounts continue trading via the bridge — sign in when you're back online to see live state."
- Has a retry button that pings `/api/health`

This page is precached on first install.

---

## Web Push setup

### VAPID keys

Generated once and stored as env vars:

```bash
# Generate locally with web-push CLI
npx web-push generate-vapid-keys

# .env.local
VAPID_PUBLIC_KEY=BL...
VAPID_PRIVATE_KEY=xy...
VAPID_SUBJECT=mailto:notifications@fundedpal.com
```

Public key ships to the client. Private key stays server-side.

### Subscription flow

```
User logs in
  └─ Service worker registered
      └─ After 30s of dashboard activity OR after 1st signal viewed:
          └─ Show permission prompt (custom UI, not native)
              └─ On accept → call Notification.requestPermission()
                  └─ On granted → register pushManager.subscribe()
                      └─ POST subscription to /api/push/subscribe
                          └─ Stored in push_subscriptions table
```

Don't trigger the permission prompt on first page load. iOS specifically penalises this and users dismiss reflexively. The custom UI explains what they'll receive ("High-confluence signals only — never spam") before triggering the native prompt.

### push_subscriptions schema

```sql
create table public.push_subscriptions (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references public.profiles(id) on delete cascade,
  endpoint text not null,
  p256dh text not null,
  auth text not null,
  user_agent text,
  platform text,                   -- 'ios','android','desktop'
  created_at timestamptz not null default now(),
  last_seen_at timestamptz not null default now(),
  unique (user_id, endpoint)
);
```

### Sending notifications

Server-side, via `web-push` package:

```typescript
// lib/push/send.ts
import webpush from 'web-push'

webpush.setVapidDetails(
  process.env.VAPID_SUBJECT!,
  process.env.VAPID_PUBLIC_KEY!,
  process.env.VAPID_PRIVATE_KEY!
)

export async function sendSignalPush(userId: string, signal: Signal) {
  const subs = await getActiveSubscriptions(userId)
  const payload = JSON.stringify({
    title: `${signal.symbol} ${signal.side.toUpperCase()} signal`,
    body: `Confluence ${signal.confluence_score}/100 · ${signal.killzone}`,
    icon: '/icons/icon-192.png',
    badge: '/icons/badge-72.png',
    tag: `signal-${signal.id}`,
    data: { url: `/strategies?signal=${signal.id}` },
    actions: [
      { action: 'execute', title: 'Execute' },
      { action: 'dismiss', title: 'Dismiss' }
    ]
  })

  await Promise.allSettled(
    subs.map(sub =>
      webpush.sendNotification(sub, payload).catch(err => {
        if (err.statusCode === 410) markSubscriptionExpired(sub.id)
      })
    )
  )
}
```

### Triggering pushes

Push sends are triggered server-side when:
- A signal with confluence ≥85 is generated (Pro+ tier)
- An account approaches a rule threshold (e.g. 80% of daily loss budget)
- A position is auto-closed by the Compliance Guardian
- Bridge disconnects for >5 minutes (Elite tier)

Tier-gate at the send layer — Trader tier gets only critical notifications (rule breach imminent), Pro gets signals + rule alerts, Elite gets everything plus bridge events.

### Service worker push handler

```typescript
// app/sw.ts (or wherever Serwist mounts)
self.addEventListener('push', (event) => {
  if (!event.data) return
  const payload = event.data.json()

  event.waitUntil(
    self.registration.showNotification(payload.title, {
      body: payload.body,
      icon: payload.icon,
      badge: payload.badge,
      tag: payload.tag,
      data: payload.data,
      actions: payload.actions,
      vibrate: [200, 100, 200],
      requireInteraction: payload.tag?.startsWith('signal-')
    })
  )
})

self.addEventListener('notificationclick', (event) => {
  event.notification.close()
  const url = event.notification.data?.url || '/dashboard'

  event.waitUntil(
    clients.matchAll({ type: 'window' }).then(windowClients => {
      const existing = windowClients.find(c => c.url.includes(self.location.origin))
      if (existing) {
        existing.focus()
        existing.postMessage({ type: 'NAVIGATE', url })
        return
      }
      return clients.openWindow(url)
    })
  )
})
```

---

## iOS-specific quirks

### Install flow

iOS does not support `beforeinstallprompt`. Detection:

```typescript
const isIOS = /iPad|iPhone|iPod/.test(navigator.userAgent)
const isStandalone = window.matchMedia('(display-mode: standalone)').matches
const isPWAInstalled = isStandalone || (window.navigator as any).standalone

if (isIOS && !isPWAInstalled) {
  // Show iOS-specific install card with instructions:
  // 1. Tap the Share icon (square with arrow up) at the bottom of Safari
  // 2. Scroll down and tap "Add to Home Screen"
  // 3. Tap "Add" in the top right
}
```

### Status bar

`apple-mobile-web-app-status-bar-style` set to `black-translucent` lets the app's `--ink` background bleed under the status bar. Pair with safe-area-inset CSS:

```css
body {
  padding-top: env(safe-area-inset-top);
  padding-bottom: env(safe-area-inset-bottom);
  padding-left: env(safe-area-inset-left);
  padding-right: env(safe-area-inset-right);
}

/* Or per-element where layout demands it */
.app-topbar {
  padding-top: calc(0.75rem + env(safe-area-inset-top));
}
```

### Notch & Dynamic Island

The dashboard topbar must respect `safe-area-inset-top`. Otherwise the notch eats into the equity display. Test on actual hardware — simulators lie about safe area behaviour.

### Push permission timing

iOS Safari only allows `Notification.requestPermission()` in response to a direct user gesture (tap). Calling it on page load is silently rejected. Wire it to a button click in the custom permission UI.

### Push notification limits

iOS shows at most 1 push notification icon at a time per app — newer ones replace older ones unless they have different `tag` values. Use unique tags per signal (`signal-${id}`) so users see each one.

### No keep-alive

iOS aggressively kills service workers. Don't expect `setInterval` in the SW to fire reliably. Anything time-sensitive must be triggered server-side via push.

### Standalone mode bugs

In standalone mode (PWA installed):
- External links open in Safari, not in-app — usually fine, but `target="_blank"` to a Stripe Customer Portal session opens a new browser tab. Test the billing flow end-to-end on installed iOS.
- Some users report cookies behaving slightly differently between Safari and standalone — auth sessions may need to be checked separately. Mitigation: middleware does an auth check on every protected route load.

---

## Android quirks

Less painful than iOS but worth noting:

- Chrome on Android fully supports `beforeinstallprompt`. Use it.
- Notifications work without prior install (unlike iOS) but install rate is still worth optimising for.
- Some OEM browsers (Samsung Internet, Mi Browser) have inconsistent service worker support. Not worth supporting initially — show a "best on Chrome" hint if detected.
- Battery optimisation can kill background fetch on aggressive ROMs. Auto-execute via the bridge bypasses this.

---

## Testing matrix

Before any release, manually test:

| Surface | Tests |
|---|---|
| Chrome desktop | Install flow, push permission, push delivery, offline shell |
| Safari macOS | Install flow, push (macOS supports it), offline shell |
| iOS Safari (browser) | Install instructional card, basic browsing without push |
| iOS standalone (PWA installed) | Push permission, push delivery, safe-area layout, share targets |
| Chrome Android | beforeinstallprompt, push permission, push delivery |
| Android standalone | Status bar, navigation gestures, install prompt suppression after install |

Lighthouse PWA audit must pass on every release. Add it to the CI:

```bash
pnpm dlx lhci autorun --collect.url=https://fundedpal.com --assert.preset=lighthouse:recommended
```

---

## Manifest reference

The full `public/manifest.webmanifest`:

```json
{
  "name": "FundedPal",
  "short_name": "FundedPal",
  "description": "Built for prop accounts. Engineered for compliance.",
  "start_url": "/dashboard",
  "scope": "/",
  "display": "standalone",
  "orientation": "any",
  "theme_color": "#0B0E14",
  "background_color": "#0B0E14",
  "categories": ["finance", "business", "productivity"],
  "lang": "en-GB",
  "dir": "ltr",
  "icons": [
    {
      "src": "/icons/icon-192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/icons/icon-512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/icons/icon-512-maskable.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "maskable"
    }
  ],
  "screenshots": [
    {
      "src": "/screenshots/dashboard-wide.png",
      "sizes": "1280x800",
      "type": "image/png",
      "form_factor": "wide",
      "label": "FundedPal dashboard"
    },
    {
      "src": "/screenshots/dashboard-narrow.png",
      "sizes": "375x812",
      "type": "image/png",
      "form_factor": "narrow",
      "label": "FundedPal on mobile"
    }
  ],
  "shortcuts": [
    {
      "name": "Dashboard",
      "url": "/dashboard",
      "icons": [{ "src": "/icons/shortcut-dashboard.png", "sizes": "96x96" }]
    },
    {
      "name": "Signals",
      "url": "/strategies",
      "icons": [{ "src": "/icons/shortcut-signals.png", "sizes": "96x96" }]
    }
  ]
}
```

The `screenshots` block enables the rich install UI on Chrome Android. The `shortcuts` block creates long-press app icon shortcuts on Android and macOS.

---

## Common gotchas

**The service worker doesn't update.** Browsers cache the SW itself for up to 24h. During development, manually unregister it via DevTools → Application → Service Workers → Unregister. In production, Serwist handles SW versioning automatically.

**Push notifications work locally but not in production.** Check that `https://` is enforced (push requires secure context), VAPID keys are set in production env, and the SW is being served with the correct MIME type (`application/javascript`, not `text/html`).

**Notifications appear briefly then vanish on iOS.** This is iOS's "throttling for spammy apps" behaviour. Make sure you're not sending more than ~3 pushes per hour per user. Tier-gate aggressively.

**Manifest is ignored.** Verify it's linked in `<head>` with `<link rel="manifest" href="/manifest.webmanifest" />` and the file is served with `Content-Type: application/manifest+json`. Vercel handles this automatically if the file extension is `.webmanifest`.

**App opens in browser instead of standalone after install.** The `start_url` in manifest must match the `scope`. If `start_url: "/dashboard"` but `scope: "/app"`, the install fails silently.

**Service worker registers on marketing site too.** This is fine — but make sure the `/` route doesn't aggressively prefetch dashboard data for unauthenticated users. Use `if (session)` guards.

---

## Future Phase 8 enhancements

When the time comes to polish the PWA further:

- **Background sync** for journal entries — write offline, sync when reconnected (Android only — iOS doesn't support it)
- **Share target API** — let users share trade screenshots from MT5 directly into FundedPal journal
- **Periodic background sync** — refresh rule profiles in the background (Android only, requires user permission)
- **File system access** — for backtest CSV exports
- **Web Share API** — share equity curve images to social
- **Email notification fallback** — for iOS users who haven't installed the PWA, deliver signal alerts via email (via Resend or similar)

---

## When to revisit native

Consider native (Expo or Swift/Kotlin) when:

- You have >5,000 paying users
- iOS notification install rate is below 50% and is materially hurting retention
- You need features impossible on PWA (Live Activities, widgets, deep biometric auth)
- Revenue justifies the maintenance cost of a second codebase

Until then, PWA is the right call.

---

*Last updated: track when you make changes*
