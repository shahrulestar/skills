---
name: conversion-analytics
description: >
  Designs and implements conversion tracking for agent-built projects — websites, web apps,
  mobile apps, and edge stacks. Detects existing analytics vendors, maps funnels, designs
  event taxonomy, wires stack-aware tracking, verifies events fire correctly, and establishes
  baseline rates for experiments. Use when setting up analytics, conversion tracking, funnel
  events, goal tracking, event naming, PostHog/GA4/Plausible/Mixpanel instrumentation, or
  preparing metrics before A/B tests. Trigger on "track conversions," "set up analytics events,"
  "event taxonomy," "funnel tracking," "baseline conversion rate," or "wire tracking for experiments."
metadata:
  author: Shahrul Estar
  github: https://github.com/shahrulestar/skills
  version: "1.0.0"
---

# Conversion Analytics

Conversion analytics answers: can we **measure** what users do on the path to value, reliably and without leaking PII? This skill covers detecting your stack, mapping funnels, designing events, implementing tracking, verifying instrumentation, and establishing baselines — the foundation **before** A/B tests or optimization.

This is a **general skill** for **agent-built projects**. Reuse existing analytics; do not bolt on a second system for one funnel unless the repo has none.

---

## Quick Start

1. **Detect** — stack and existing analytics (Step 1)
2. **Map funnel** — surface → action → outcome (Step 2)
3. **Design taxonomy** — event names and properties (Step 3)
4. **Output table** — event plan; **no file edits** (Step 4)
5. **Implement** — wire tracking after user approval (Step 5)
6. **Verify** — events fire, properties present, baseline captured (Step 6)

---

## Step 1 — Detect stack and existing analytics

Before designing events, read the repo:

1. **Dependencies** — `package.json`, mobile config, `wrangler.toml`, framework (Next.js, Vite, Expo, etc.).
2. **Existing SDK** — GA4 (`gtag`, `@next/third-parties`), Plausible, PostHog, Mixpanel, Amplitude, Segment, Cloudflare Web Analytics, Firebase Analytics.
3. **Current events** — search for `track(`, `capture(`, `plausible(`, `gtag('event'`, custom analytics wrappers.
4. **Consent / privacy** — cookie banners, CMP, `beforeSend` hooks, DNT handling.
5. **Identity** — anonymous visitor ID, logged-in user ID, device ID (mobile).

If analytics exists, **extend it**. If none exists, recommend the lightest fit for the stack (see [references/vendor-patterns.md](references/vendor-patterns.md)).

---

## Step 2 — Map the funnel

For each conversion goal the user cares about, trace:

```
Surface (page/screen) → User action → Business outcome
```

**Examples:**

| Goal | Surface | Action event | Outcome event |
|------|---------|--------------|---------------|
| Signup | Landing hero | `cta_click` | `signup_complete` |
| Trial start | Onboarding step 3 | `onboarding_step_complete` | `trial_started` |
| Purchase | Checkout | `checkout_started` | `purchase_complete` |

Tag **priority** — P0 events block experiment readiness; P1 nice-to-have; P2 defer.

For naming conventions and anti-patterns, see [references/event-taxonomy.md](references/event-taxonomy.md).

---

## Step 3 — Design event taxonomy

Every event needs:

- **Stable name** — `object_action` (e.g., `signup_cta_click`, not `click`)
- **Trigger** — exact UI moment (button click, form submit success, route change)
- **Properties** — `surface`, `variant` (slot for A/B), `source`, non-PII context
- **No PII** — no raw email, name, or free-text user input in properties

Reserve `variant` or `experiment_id` + `variant` for future A/B tests even if not running one yet.

Read [references/privacy-compliance.md](references/privacy-compliance.md) before implementing.

---

## Step 4 — Output format (no edits on first pass)

**Do not edit files on the first pass** unless the user explicitly says "implement all" or "apply without review."

### Executive summary

- Analytics vendor detected (or recommendation if none)
- Funnel goal(s)
- Event count by priority (P0 / P1 / P2)
- Gaps blocking A/B readiness (missing primary metric, no visitor ID, etc.)

### Event plan table

| Surface | Event name | Trigger | Properties | Priority |
|---------|------------|---------|------------|----------|

**Priority:** `P0` — required for conversion/experiment readiness · `P1` — useful context · `P2` — defer

### Baseline note

State how baseline rates will be captured once events are live (e.g., "7-day `signup_complete` / `landing_view` rate").

### Partial approval

User may say "implement P0 only" or "implement rows 1 and 3" — follow exactly.

---

## Step 5 — Implement (after approval)

1. Edit approved rows only
2. Reuse existing analytics SDK and naming patterns in the repo
3. Attach `variant` property at capture sites used in experiments
4. Do not duplicate tracking (one event per user action)
5. Respect consent — no tracking before consent when CMP is present

Stack-specific snippets: [references/vendor-patterns.md](references/vendor-patterns.md).

---

## Step 6 — Verify

Before calling instrumentation done:

- [ ] P0 events fire in dev/staging (document how to trigger each)
- [ ] Properties match the approved table (no missing `surface`)
- [ ] No PII in payload (spot-check network tab or vendor live view)
- [ ] Duplicate fires ruled out (double-click, strict mode, re-renders)
- [ ] Baseline window stated (e.g., collect 7–14 days before sample-size math)

Hand off to **ab-testing** when the user wants to run experiments — baseline rate + primary metric must exist first.

---

## Related skills

| Skill | When |
|-------|------|
| [ux-writer](https://skills.sh/shahrulestar/skills/ux-writer) | CTA/label copy before naming events tied to UI text |
| [ab-testing](https://skills.sh/shahrulestar/skills/ab-testing) | After P0 events and baseline exist |

```bash
npx skills add shahrulestar/skills --skill ux-writer
npx skills add shahrulestar/skills --skill ab-testing
```

---

---

## Additional resources

| File | Contents |
|------|----------|
| [references/event-taxonomy.md](references/event-taxonomy.md) | Naming, properties, anti-patterns |
| [references/vendor-patterns.md](references/vendor-patterns.md) | GA4, Plausible, PostHog, Workers, React Native |
| [references/privacy-compliance.md](references/privacy-compliance.md) | Consent, PII, GDPR-conscious defaults |

---

## Rules

- Reuse existing analytics — never add a second vendor for one funnel without asking.
- Table first, implement after approval — same as ux-writer.
- No PII in event properties.
- Reserve `variant` for future A/B tests.
- State baseline collection window before sample-size math.
- If no analytics exists, recommend one lightweight option — don't install three.
