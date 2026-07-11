# Implementation Patterns by Stack

Stack-specific patterns for Step 6 of the ab-testing skill. Adapt to the project's conventions.

## Next.js App Router (middleware)

Best for SSR/SEO pages where variant affects above-the-fold content.

```ts
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

const EXPERIMENT_ID = 'hero-cta-v1';

function getVariant(request: NextRequest): 'a' | 'b' {
  const cookieName = `exp_${EXPERIMENT_ID}`;
  const existing = request.cookies.get(cookieName)?.value;
  if (existing === 'a' || existing === 'b') return existing;
  return Math.random() < 0.5 ? 'a' : 'b';
}

export function middleware(request: NextRequest) {
  const variant = getVariant(request);
  const response = NextResponse.next();
  response.cookies.set(`exp_${EXPERIMENT_ID}`, variant, {
    path: '/',
    maxAge: 60 * 60 * 24 * 30,
    sameSite: 'lax',
  });
  response.headers.set('x-experiment-variant', variant);
  return response;
}

export const config = { matcher: ['/'] };
```

Read variant in Server Components via headers or cookies. Pass to client components as props — avoid client-only split for SEO-critical content.

## Next.js — client-only fallback

Acceptable for below-the-fold or non-SEO UI where a brief flash is tolerable:

```tsx
'use client';

import { useEffect, useState } from 'react';

function useVariant(experimentId: string) {
  const [variant, setVariant] = useState<'a' | 'b' | null>(null);

  useEffect(() => {
    const key = `exp_${experimentId}`;
    let v = localStorage.getItem(key) as 'a' | 'b' | null;
    if (v !== 'a' && v !== 'b') {
      v = Math.random() < 0.5 ? 'a' : 'b';
      localStorage.setItem(key, v);
    }
    setVariant(v);
  }, [experimentId]);

  return variant;
}
```

## Cloudflare Workers / Pages

Use middleware or a Worker in front of the origin. Store experiment config in KV for kill-switch without redeploy.

```js
export async function onRequest(context) {
  const { request, env } = context;
  const experimentId = 'hero-cta-v1';
  const cookieName = `exp_${experimentId}`;

  let variant = parseCookie(request.headers.get('Cookie'), cookieName);
  if (variant !== 'a' && variant !== 'b') {
    variant = Math.random() < 0.5 ? 'a' : 'b';
  }

  const response = await context.next();
  response.headers.append(
    'Set-Cookie',
    `${cookieName}=${variant}; Path=/; Max-Age=2592000; SameSite=Lax`
  );
  return response;
}
```

**Cache:** Include variant in cache key or use `Cache-Control: private` for experiment pages.

## React (Vite, CRA, SPA)

- **Logged-in users:** Assign by `user.id` — stable, no cookie needed.
- **Anonymous:** Cookie or localStorage on first visit.
- **Routing:** Same URL, conditional render — don't create separate routes unless needed for analytics clarity.

Track events with existing analytics SDK; always attach `variant` property.

## React Native / Expo

```tsx
import AsyncStorage from '@react-native-async-storage/async-storage';

async function getVariant(experimentId: string): Promise<'a' | 'b'> {
  const key = `exp_${experimentId}`;
  const existing = await AsyncStorage.getItem(key);
  if (existing === 'a' || existing === 'b') return existing;
  const variant = Math.random() < 0.5 ? 'a' : 'b';
  await AsyncStorage.setItem(key, variant);
  return variant;
}
```

Send variant with analytics events (Segment, PostHog, Firebase Analytics, etc.).

## Static sites + edge function

For Astro, Hugo, Eleventy, etc.:

1. Edge function (Netlify, Cloudflare, Vercel) sets cookie and injects variant.
2. Or: client-side split for non-SEO pages with existing analytics.

Prefer edge when landing pages must be crawlable and LCP-sensitive.

## PostHog feature flags

When PostHog is already in the repo:

```tsx
import { useFeatureFlagVariantKey } from 'posthog-js/react';

function HeroCta() {
  const variant = useFeatureFlagVariantKey('hero-cta-v1');
  const label = variant === 'b' ? 'Start free trial' : 'Sign up';

  return (
    <button
      onClick={() =>
        posthog.capture('signup_cta_click', {
          surface: 'landing',
          experiment_id: 'hero-cta-v1',
          variant: variant ?? 'control',
        })
      }
    >
      {label}
    </button>
  );
}
```

Configure flag in PostHog UI; use multivariate or 50/50 rollout. Align event properties with [conversion-analytics](../../conversion-analytics/references/event-taxonomy.md).

## GrowthBook

Self-hosted or cloud — good for several concurrent experiments:

```tsx
import { useFeatureValue } from '@growthbook/growthbook-react';

function HeroCta() {
  const variant = useFeatureValue('hero-cta-v1', 'control');
  // render variant; track with same experiment_id + variant properties
}
```

Wire `trackingCallback` to your analytics SDK so exposure and conversion share `experiment_id`.

## Feature-flag platforms

| When | Recommendation |
| --- | --- |
| One-off test, simple UI split | Roll your own cookie/user split |
| Several concurrent experiments | GrowthBook (self-hosted) or PostHog |
| Team already uses a platform | Use what's there — don't duplicate |
| Privacy-sensitive / no third-party | Self-hosted GrowthBook or cookie split + first-party analytics |

Minimum event payload: `(visitor_id, experiment_id, variant, event_name, timestamp)`.

Event naming aligns with conversion-analytics — see `skills/conversion-analytics/references/vendor-patterns.md` in this repo.

## Kill switch checklist

- [ ] Experiment ID and split ratio in config (env, KV, or feature-flag UI) — not hardcoded
- [ ] Single flag to disable assignment (default all users to control)
- [ ] Documented rollback: remove variant code or flip flag, redeploy if needed
- [ ] Analytics events stop tagging variant after kill (or tag as `experiment_ended`)
