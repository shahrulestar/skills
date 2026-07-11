---
name: ab-testing
description: >
  General-purpose A/B testing (split testing) for any project built or maintained by coding agents:
  websites, web apps, mobile apps, dashboards, and edge/serverless stacks. Covers hypothesis design,
  sample-size math, stack-aware variant implementation, event tracking, low-traffic alternatives,
  pre-flight review, honest analysis, and ship-or-no-ship decisions. Use whenever the user wants to
  test a UI/UX change, compare two versions of a page, feature, button, headline, onboarding flow,
  pricing display, or copy; run an experiment or split test; do conversion rate optimization; or
  asks "which version performs better" or "should I change this" — even without saying A/B test.
  Also covers agent-specific advantages: reading the actual codebase to produce mergeable variants,
  persona pre-flight before spending traffic, wiring bucketing logic, and writing statistical analysis
  when results arrive. Works across Next.js, React, Vue, Workers, React Native, and greenfield repos
  with no prior experiment setup.
metadata:
  author: Shahrul Estar
  github: https://github.com/shahrulestar/skills
  version: "1.1.0"
---

# A/B Testing

A/B testing answers one question: does this specific change actually make things better for real users, or does it just feel like it should? This skill covers the full loop — deciding whether a test is even the right tool, forming a real hypothesis, designing variants, implementing the split, running it without poisoning the data, and reading the results honestly — plus the parts of that loop an agent can do that a human running tests by hand usually can't.

This is a **general skill** for **agent-built projects**. It does not assume a specific framework, audience, or analytics vendor. Read the repo first, adapt patterns to the stack, and reuse whatever instrumentation already exists.

## Works with any agent-built project

Use this skill on projects the agent (or team) created or maintains — greenfield or legacy:

| Project type | Example tests |
| --- | --- |
| Marketing / landing page | Hero headline, CTA label, social proof layout, pricing table |
| Web app / SaaS | Onboarding steps, dashboard empty states, settings UX, upgrade prompts |
| E-commerce / checkout | Cart layout, trust badges, shipping copy, payment step order |
| Mobile app | Tab labels, permission dialogs, paywall copy, signup flow |
| Internal / admin tools | Default filters, table density, bulk-action placement |
| API / backend (when UI-adjacent) | Error message wording surfaced to users, email template variants |
| Edge / serverless | Middleware-based splits, cached page variants, worker routing |

**Stack examples:** Next.js App Router, Pages Router, Remix, Vite + React, Vue/Nuxt, SvelteKit, Cloudflare Workers/Pages, Express, React Native, Expo, static sites with analytics.

If the repo has no experiment infrastructure yet, start with a cookie- or user-ID-based split and existing analytics — don't introduce a heavy platform for a one-off test.

## Why this is different with an agent in the loop

Most A/B testing advice (and most A/B testing tools) was written for a human marketer working through a dashboard. Read this once before jumping into the steps below:

- **The codebase is the source of truth, not a screenshot.** An agent can open the actual component, see design tokens, copy conventions, and accessibility patterns, and write a variant that's a real, mergeable diff — not a mockup someone still has to translate into code. Always read the project before designing a variant (Step 3).
- **You can pre-flight a variant before it ever touches a real visitor.** Before committing real traffic, walk through both variants role-playing the personas who use the project, and flag confusing copy, broken flows, or accessibility issues. See Step 7.
- **One agent, one continuous loop.** Hypothesis, implementation, and analysis stay in context — the agent remembers *why* the test was designed a certain way when interpreting results.
- **Low-traffic reality.** Most agent-built projects — indie tools, MVPs, niche audiences, internal apps — never hit enterprise visitor counts. Step 4 exists to catch tests that mathematically cannot conclude.

What an agent should *not* pretend to do: fabricate traffic, fabricate results, or declare a winner from an undersized sample because the user wants an answer. Statistics here are guardrails, not obstacles.

## The loop

1. Decide if A/B testing is even the right tool for this question
2. Write a real hypothesis, not just "let's try this version too"
3. Read the actual project before designing anything
4. Check whether there's enough traffic to test this at all — pick a method accordingly
5. Design the variant(s)
6. Implement the split (assignment, tracking, rollback path)
7. Pre-flight with a persona walkthrough before spending real traffic
8. Run it without peeking it to death
9. Analyze honestly — statistical *and* practical significance
10. Decide, document, and either ship or learn

---

## Step 1: Is this actually an A/B test?

A/B testing is the right tool when: there's a clear, single, randomizable point of difference; you have a metric you can measure per-visitor; and you can plausibly get enough independent visitors through both arms to tell signal from noise. It's the wrong tool when:

- **The traffic genuinely can't support it.** Go to Step 4 and pick an alternative instead.
- **The question is "why," not "which."** Use research (recordings, interviews, surveys) — not a split test on a guess.
- **The change is irreversible or risky for half your users** (pricing, legal/compliance, safety). Use staged rollout with monitoring, not 50/50.
- **It's an obvious fix.** Broken links, typos, invisible buttons — just fix them.

If it passes, prioritize with a quick ICE score (Impact × Confidence × Ease, 1–5 each) — favor high-traffic surfaces and visible changes.

## Step 2: Write a real hypothesis

A version label is not a hypothesis. A hypothesis names the change, expected effect, metric, and *why*.

**Template:** "Changing [X] to [Y] will [increase/decrease] [metric] for [user segment] because [reasoning grounded in something observed]."

**Weak:** "Let's try a different sign-up button."
**Strong:** "Changing the sign-up CTA from 'Sign Up' to 'Start free trial' will increase click-through for first-time visitors, because analytics show high bounce above the button and the current label doesn't state the offer."

If you can't fill in "because," find out why before treating the idea as test-ready.

## Step 3: Read the project before designing anything

Before drafting variants:

1. **Detect the stack** — `package.json`, `wrangler.toml`, `next.config`, `app/` vs `pages/`, mobile config, existing middleware.
2. **Open the component/page under test** — design tokens, component library, UX-writing or style guides in the repo.
3. **Find existing analytics** — GA4, Plausible, PostHog, Mixpanel, Amplitude, Cloudflare Web Analytics, custom events. Reuse them; don't bolt on a second system for one test. If events or baseline are missing, run the [conversion-analytics](https://skills.sh/shahrulestar/skills/conversion-analytics) skill first:
   `npx skills add shahrulestar/skills --skill conversion-analytics`
4. **Check routing, i18n, middleware, cache** — variants must not break architecture or serve the wrong arm from cache.
5. **Establish a baseline** — current conversion/click rate or proxy metric for sample-size math (Step 4). Requires a verified primary metric from conversion-analytics or equivalent instrumentation.

A variant designed from the repo ships as an idiomatic diff. A variant from a screenshot alone usually doesn't.

## Step 4: Do you actually have enough traffic? Pick a method.

This prevents running tests that can never conclude.

### Quick math

For conversion-rate tests at ~5% baseline, 95% confidence, 80% power, approximate visitors **per variant**:

| Minimum detectable lift (relative) | ~Visitors per variant |
| --- | --- |
| 5% | ~115,000 |
| 10% | ~29,000 |
| 20% | ~7,500 |
| 30%+ | ~3,000 or fewer |

Small changes need enormous traffic; bold changes need far less. When the user provides traffic and baseline numbers, calculate explicitly up front.

For extended tables, formulas, and low-traffic playbooks, see [references/sample-size.md](references/sample-size.md).

### If traffic is low (typical for MVPs and agent-built side projects)

1. **Make the change bigger** — test a different headline or flow, not a color tweak.
2. **Widen the test surface** — site-wide banner or nav vs. one deep page.
3. **Move the metric up-funnel** — CTA click, scroll, form start vs. rare final conversion.
4. **Use session count** when valid for single-session behavior.
5. **Bayesian / sequential testing** if tooling supports safe monitoring.
6. **Non-experimental fallback** — heatmaps, short surveys, before/after rollout (directional), usability tests (3–5 users).

Always state which mode you're in: **statistical test** vs. **directional read**.

## Step 5: Design the variant(s)

- **One variable per test** where possible; multivariate needs more traffic — call it out.
- **Control stays unchanged** — any control drift contaminates results.
- **Match accessibility and responsive behavior** across variants.
- **UX writing:** follow project voice/style guides. For copy-heavy variants (headlines, CTAs, labels, errors), optionally run the [ux-writer](https://skills.sh/shahrulestar/skills/ux-writer) skill before launch:
  `npx skills add shahrulestar/skills --skill ux-writer`
- **Pre-define primary metric and guardrails** (e.g., sign-ups up, bounce and LCP unchanged).

## Step 6: Implement the split

Needs: **deterministic assignment**, **event tracking with variant ID**, **kill switch**.

**Default assignment pattern** (edge middleware / worker — adapt to stack):

```js
function getVariant(request, experimentId, splitRatio = 0.5) {
  const cookieName = `exp_${experimentId}`;
  const existing = getCookie(request, cookieName);
  if (existing === 'a' || existing === 'b') return existing;
  return Math.random() < splitRatio ? 'a' : 'b';
}
```

**Rules that matter:**

- Assign once; persist via cookie or logged-in user ID for the test duration.
- **SSR/SEO pages:** split at edge/server, not client-only (avoids flash + crawler bias).
- **Cache:** variant must be in the cache key for above-the-fold content.
- **Config:** prefer KV/env/feature-flag store over hardcoded deploy-to-stop.
- **Platform choice:** roll-your-own for one-off tests; GrowthBook or PostHog when many concurrent experiments — prefer self-hosted/open-source when values conflict with third-party tracking.
- **Log** `(visitor_id, variant, event, timestamp)` minimum.

For stack-specific patterns (Next.js middleware, React conditional render, Workers, React Native, static sites), see [references/implementation-patterns.md](references/implementation-patterns.md).

## Step 7: Pre-flight before spending real traffic

Walk both variants as real personas for this product (infer from README, copy, or ask). Flag confusion, broken flows, a11y issues, mobile layout breaks.

For copy-heavy variants, an optional [ux-writer](https://skills.sh/shahrulestar/skills/ux-writer) audit adds a cheap quality gate — not a substitute for the live test.

Fix or drop variants that fail pre-flight.

## Step 8: Run it without poisoning the data

- Pre-commit: sample size or duration, primary metric, significance threshold.
- Don't stop early on promising interim results unless using sequential/Bayesian methods designed for it.
- Run ~2+ weeks when possible (day-of-week effects); avoid atypical launch windows.
- Avoid overlapping tests on the same users/pages without planning.

## Step 9: Analyze honestly

- **Statistical significance** ≠ practical importance.
- Report **confidence intervals**, not just point estimates.
- Check **guardrail metrics**.
- **Segment** (device, new vs returning, source) when data allows — avoid fishing.
- **Multiple comparisons:** adjust threshold (e.g., Bonferroni) when testing many variants/metrics.
- Losing variants still teach — document learnings.

For analysis templates, Simpson's paradox, peeking, and low-traffic reads, see [references/analysis-guide.md](references/analysis-guide.md).

## Step 10: Decide, document, ship or learn

Close every test with a written record:

```
## [Test name]
Hypothesis: ...
Variants: A (control) — ..., B — ...
Metric: ... | Guardrails: ...
Traffic / duration: ... visitors per variant over ... days
Result: A x% vs B y%, lift z% (95% CI: [..., ...]), p = ...
Statistically significant: yes/no | Practically worth shipping: yes/no
Decision: ship B / keep A / inconclusive, retest with [change]
What we learned: ...
```

Inconclusive is valid — say so plainly.

---

## Related skills

| Skill | When |
|-------|------|
| [conversion-analytics](https://skills.sh/shahrulestar/skills/conversion-analytics) | Before Step 4 — wire events, verify tracking, establish baseline |
| [ux-writer](https://skills.sh/shahrulestar/skills/ux-writer) | Copy-heavy variants — audit before traffic spend |

```bash
npx skills add shahrulestar/skills --skill conversion-analytics
npx skills add shahrulestar/skills --skill ux-writer
```

## Pitfall checklist

- [ ] Pre-committed sample size/duration, or eyeballing until it looks good?
- [ ] Stopped early on interim results?
- [ ] Control stayed unchanged?
- [ ] Lift statistically *and* practically significant?
- [ ] Guardrail metrics checked?
- [ ] Single segment drove the result?
- [ ] Multiple comparisons without threshold adjustment?
- [ ] External events skewed the window?
- [ ] Assignment sticky, not re-randomized per visit?
- [ ] Low traffic — stated honestly instead of coin-flip as proof?

## Quick reference

- **Thresholds:** 95% confidence (p < 0.05), 80% power unless user specifies otherwise.
- **Min test length:** ~2 weeks when feasible.
- **Small change → big traffic; big change → small traffic.**
- **One variable per test** unless explicitly multivariate.
- **Agent edge:** better variants (Step 3), free pre-flight (Step 7), honest analysis (Step 9) — statistics work the same for AI and humans.
