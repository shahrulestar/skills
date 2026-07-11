# Analysis Guide

Extended reference for Step 9 of the ab-testing skill.

## Analysis template

```markdown
## [Test name]

**Hypothesis:** Changing [X] to [Y] will [effect] [metric] because [reason].

**Variants:** A (control) — ... | B — ...

**Primary metric:** ... | **Guardrails:** bounce rate, LCP, support tickets

**Traffic:** ... visitors per variant over ... days

**Results:**
| Variant | Conversions | Rate | 95% CI |
|---------|-------------|------|--------|
| A       | ...         | x%   | [...]  |
| B       | ...         | y%   | [...]  |

**Lift:** z% relative (p = ..., significant: yes/no)

**Segments checked:** mobile, new vs returning (note if any segment drove result)

**Guardrails:** unchanged / degraded — [detail]

**Practical significance:** Is y%−x% worth shipping? (revenue, support, eng cost)

**Decision:** ship B / keep A / inconclusive — [next step]

**Learnings:** ...
```

## Statistical vs practical significance

- **Statistical:** p < 0.05 (or pre-committed threshold) — unlikely due to chance alone.
- **Practical:** Lift large enough to matter for the business (1% on 10 visitors ≠ 1% on 100k).

Report both. A "significant" 0.1% lift on a low-value metric may not be worth shipping.

## Confidence intervals

Prefer: "B is 4.2%–6.8% vs A 3.1%–5.0%" over "B wins."

If intervals overlap heavily → treat as inconclusive even if p < 0.05 on point estimate.

## Common mistakes

### Peeking

Stopping when interim results look good inflates false positives. Pre-commit duration or use sequential/Bayesian methods designed for monitoring.

### Simpson's paradox

Aggregate shows B wins; mobile shows A wins, desktop shows A wins — segment before deciding. Check device, source, new vs returning.

### Multiple comparisons

Testing 5 metrics and celebrating the one that hit p < 0.05 → Bonferroni or designate one primary metric upfront.

### Underpowered tests

"No significant difference" ≠ "no difference." May mean insufficient traffic. State power and MDE explicitly.

### External confounds

Holidays, outages, marketing campaigns, deploys during test window — note in learnings.

## Low-traffic reads

When in **directional** mode (see [sample-size.md](sample-size.md)):

- Label clearly: "Directional — not statistically conclusive."
- Do not declare a winner from < ~100 conversions per variant unless effect is huge and pre-committed as exploratory.
- Recommend next step: bigger change, longer window, or qualitative follow-up.

## Segment analysis rules

1. Pre-register segments of interest (mobile, new user) — don't fish every dimension.
2. If a segment is < 30% of traffic, treat subgroup results as hypothesis-generating only.
3. If one segment drives the win, document whether shipping to all users is appropriate.

## Handoff after decision

- Ship: remove experiment code or set flag to 100% winner; update analytics default.
- Keep control: archive hypothesis and learnings in repo or docs.
- Inconclusive: state what to change (MDE, metric, variant boldness) before retest.
