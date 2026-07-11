# Vendor Patterns

Stack-specific tracking patterns for Step 5 of the conversion-analytics skill. Adapt to project conventions.

## GA4 (gtag / Google Analytics 4)

```js
// Page view with surface
gtag('event', 'page_view', {
  page_location: window.location.href,
  surface: 'pricing',
});

// Conversion event
gtag('event', 'signup_complete', {
  surface: 'signup',
  method: 'email',
  variant: variant ?? 'control',
});
```

Next.js: prefer `@next/third-parties/google` or existing `gtag` wrapper in the repo.

## Plausible

```js
plausible('signup_cta_click', {
  props: { surface: 'landing', variant: variant ?? 'control' },
});
```

Custom events require Plausible plan support — verify `package.json` / docs before adding.

## PostHog

```js
posthog.capture('signup_complete', {
  surface: 'signup',
  variant: variant ?? 'control',
});
```

Identify only after consent and when needed for funnels:

```js
posthog.identify(userId); // never pass email as distinct ID without approval
```

Feature flags (for A/B): see ab-testing `implementation-patterns.md`.

## Mixpanel / Amplitude

```js
mixpanel.track('signup_complete', {
  surface: 'signup',
  variant: variant ?? 'control',
});
```

Align with existing `track` wrapper if the repo abstracts vendors.

## Cloudflare Web Analytics

Best for page-level metrics; limited custom events. For conversion events on Workers/Pages, use:

- Worker `analytics_engine` binding, or
- First-party endpoint + existing vendor

Do not assume CF Web Analytics alone supports full funnel custom events.

## React Native / Firebase

```js
import analytics from '@react-native-firebase/analytics';

await analytics().logEvent('signup_complete', {
  surface: 'signup',
  variant: variant ?? 'control',
});
```

## Cloudflare Workers (first-party)

```js
await fetch('https://your-domain.com/api/events', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    event: 'signup_cta_click',
    surface: 'landing',
    variant,
    timestamp: Date.now(),
  }),
});
```

Store in Analytics Engine, D1, or forward to PostHog — match existing backend patterns.

## Shared helper pattern

If the repo lacks a wrapper, suggest a thin `lib/analytics.ts`:

```ts
export function track(event: string, properties: Record<string, string | number | boolean>) {
  // delegate to detected vendor — single import site for components
}
```

One helper beats scattered vendor calls.

## Verification per vendor

| Vendor | How to verify |
|--------|----------------|
| GA4 | DebugView in GA4 admin |
| Plausible | Site dashboard → Goals / custom events |
| PostHog | Live events tab |
| Mixpanel | Live view |
| Firebase | DebugView (enable debug mode) |

Document which view the user should open in the verify checklist.
