# Event Taxonomy

Extended reference for Step 3 of the conversion-analytics skill.

## Naming convention

Use `object_action` in snake_case:

| Good | Bad | Why |
|------|-----|-----|
| `signup_cta_click` | `click` | Describes what was clicked |
| `checkout_started` | `button1` | Stable across UI refactors |
| `trial_started` | `success` | Names the business outcome |
| `pricing_page_view` | `pageview` | Distinguishes surfaces |

## Required properties (minimum)

| Property | When | Example |
|----------|------|---------|
| `surface` | Always | `landing`, `checkout`, `onboarding` |
| `variant` | A/B or multivariate | `a`, `b`, or `control` |
| `source` | Attributed traffic | `organic`, `paid`, `direct` |
| `experiment_id` | During A/B test | `hero-cta-v1` |

Add domain-specific properties only when they aid analysis (e.g., `plan_tier`, `step_index`).

## Anti-patterns

| Anti-pattern | Fix |
|--------------|-----|
| Vague `click`, `submit`, `view` | Name the object and action |
| PII in properties (`email`, `name`) | Hash or omit; use internal user ID only if already in app |
| One mega-event with `type` enum | Split into named events for cleaner funnels |
| Different names for same action across platforms | Align web + mobile event names |
| Tracking before consent | Gate behind CMP / consent callback |

## Funnel event sets

**Landing → signup:**

1. `landing_view` (or use vendor pageview + `surface`)
2. `signup_cta_click`
3. `signup_form_start`
4. `signup_complete`

**Trial → paid:**

1. `trial_started`
2. `upgrade_cta_click`
3. `checkout_started`
4. `purchase_complete`

Mark the **primary metric** for experiments explicitly (usually the last step or highest-signal proxy).

## Variant slot for A/B

Even without a live test, include in capture calls:

```js
capture('signup_cta_click', { surface: 'landing', variant: getVariant() });
```

`getVariant()` returns `control` when no experiment runs — avoids schema churn later.
