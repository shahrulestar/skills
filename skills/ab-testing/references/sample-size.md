# Sample Size & Low-Traffic Playbook

Extended reference for Step 4 of the ab-testing skill.

## Conversion-rate visitors per variant

Approximate visitors needed **per variant** to detect a relative lift at 95% confidence and 80% power.

### Baseline conversion rate ~5%

| Minimum detectable lift (relative) | ~Visitors per variant |
| --- | --- |
| 5% | ~115,000 |
| 10% | ~29,000 |
| 20% | ~7,500 |
| 30%+ | ~3,000 or fewer |

### Baseline conversion rate ~2%

| Minimum detectable lift (relative) | ~Visitors per variant |
| --- | --- |
| 10% | ~72,000 |
| 20% | ~19,000 |
| 30% | ~8,500 |
| 50%+ | ~3,000 or fewer |

### Baseline conversion rate ~10%

| Minimum detectable lift (relative) | ~Visitors per variant |
| --- | --- |
| 5% | ~63,000 |
| 10% | ~16,000 |
| 20% | ~4,200 |
| 30%+ | ~1,800 or fewer |

Exact numbers vary by calculator and assumptions. The **shape** matters: subtle changes need far more traffic than bold ones.

## Key terms

- **Confidence (95%):** Probability the result isn't a false positive if you declare a winner. Common threshold: p < 0.05.
- **Power (80%):** Probability of detecting a real effect if one exists.
- **MDE (minimum detectable effect):** Smallest lift you can reliably detect with your traffic budget.
- **Relative lift:** Percentage change from baseline (e.g., 5% → 5.5% = 10% relative lift).

## Quick feasibility check

```
weekly_visitors_to_test_surface × weeks_available / 2 = visitors per variant (50/50 split)
```

Compare that number to the table row for your expected lift. If you're short by an order of magnitude, change the test design — don't run it anyway.

## Low-traffic decision tree

```
Can you reach ~3,000+ visitors per variant in a reasonable window?
├── Yes → Standard A/B test (pre-commit sample size)
└── No
    ├── Can you make the variant MUCH bolder (headline/flow, not color)?
    │   └── Recalculate — large effects need far less traffic
    ├── Can you widen the surface (site-wide element)?
    │   └── More eligible traffic per week
    ├── Can you use an earlier funnel metric?
    │   └── CTA click, scroll, form start vs. rare conversion
    └── Still not viable
        ├── Bayesian/sequential tooling (if available)
        └── Non-experimental methods (see below)
```

## Non-experimental alternatives

Use when traffic cannot support significance. Always label results as **directional**, not proof.

| Method | Best for | Caveat |
| --- | --- | --- |
| Heatmaps / session recordings | Finding friction on existing UI | Observational — no variant comparison |
| On-page survey (1–2 questions) | Understanding intent or confusion | Self-selected respondents |
| Before/after rollout | Directional signal when you must ship | Confounded by seasonality, marketing, other changes |
| Usability test (3–5 users) | Qualitative friction in both variants | Small n — not statistical |
| Internal dogfooding | Obvious broken flows | Not representative of real users |

## When to say "don't test"

- Obvious bugs or fixes (just ship)
- Traffic math shows months to conclude and the change can't be made bolder
- Question is "why" not "which" — use research first
- Risky/irreversible changes — staged rollout instead
