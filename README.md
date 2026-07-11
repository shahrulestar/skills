![shahrulestar/skills — The Open Agent Skills Ecosystem](https://www.skills.sh/shahrulestar/skills/opengraph-image?195d45ea74f8736c)

# skills

**Agent skills for UX writing, conversion analytics, and A/B testing — built for coding agents by shahrulestar.com**

[skills.sh](https://skills.sh/shahrulestar/skills) · [License: MIT](LICENSE)

---

## About

A markdown-only collection of installable agent skills by [Shahrul Estar](https://github.com/shahrulestar). Each skill teaches coding agents (Cursor, Claude Code, Codex, Copilot, and [68+ more](https://github.com/vercel-labs/skills#supported-agents)) how to handle a specific product workflow — from UX copy audits to conversion tracking to A/B experiments.

This repo contains only `.md` skill files — no scripts, manifests, or tooling. Install with the [skills CLI](https://github.com/vercel-labs/skills).

---

## Skills

| Skill | Version | What it does | Example prompt |
| --- | --- | --- | --- |
| [ux-writer](https://skills.sh/shahrulestar/skills/ux-writer) | 1.3.1 | Audit and rewrite UI copy — headings, labels, buttons, microcopy, i18n, a11y | `Review all user-facing copy on the pricing page` |
| [conversion-analytics](https://skills.sh/shahrulestar/skills/conversion-analytics) | 1.0.0 | Map funnels, design event taxonomy, wire tracking, verify events for experiments | `Set up conversion tracking for the signup flow with our existing Plausible setup` |
| [ab-testing](https://skills.sh/shahrulestar/skills/ab-testing) | 1.1.0 | Design, implement, run, and analyze A/B tests on any agent-built project | `Should we A/B test the new hero headline? Check if we have enough traffic.` |

---

## Recommended workflow

```
ux-writer  →  conversion-analytics  →  ab-testing
(copy)         (events + baseline)      (experiment)
```

1. **ux-writer** — audit copy before you ship or test variants.
2. **conversion-analytics** — wire events and establish baseline rates.
3. **ab-testing** — run the experiment loop with honest statistics.

---

## Example prompts

### ux-writer

```
Review all user-facing copy on the pricing page
```

```
Audit SaaS onboarding and billing settings copy
```

### conversion-analytics

```
Set up conversion tracking for the signup flow with our existing Plausible setup
```

```
Verify our PostHog events fire correctly and give me baseline conversion rates for A/B testing
```

### ab-testing

```
Should we A/B test the new hero headline on the landing page? Check if we have enough traffic.
```

```
We ran the pricing table test for two weeks — analyze results and tell me if we should ship variant B.
```

---

## Install

```bash
npx skills add shahrulestar/skills --skill <skill-name>
```

| Skill | Command |
| --- | --- |
| ux-writer | `npx skills add shahrulestar/skills --skill ux-writer` |
| conversion-analytics | `npx skills add shahrulestar/skills --skill conversion-analytics` |
| ab-testing | `npx skills add shahrulestar/skills --skill ab-testing` |

Install all skills in this bundle:

```bash
npx skills add shahrulestar/skills --skill '*'
```

### Options

| Option | Description |
| --- | --- |
| `-g` / `--global` | Install to your user directory (all projects) |
| `-y` / `--yes` | Skip confirmation prompts |
| `--copy` | Copy files instead of symlinking |

### Where files go

| Agent | Project path | Global path (`-g`) |
| --- | --- | --- |
| Cursor, Codex, Copilot, Cline, … | `.agents/skills/<skill>/` | `~/.cursor/skills/<skill>/` or agent home |
| Claude Code | `.claude/skills/<skill>/` | `~/.claude/skills/<skill>/` |

Full matrix: [vercel-labs/skills — Supported Agents](https://github.com/vercel-labs/skills#supported-agents)

### Verify

```bash
npx skills add shahrulestar/skills --list
npx skills list
```

### Update & remove

```bash
npx skills update ux-writer
npx skills update conversion-analytics
npx skills update ab-testing
npx skills check
npx skills remove ux-writer
```

---

## License

MIT © [Shahrul Estar](https://github.com/shahrulestar) — see [LICENSE](LICENSE).
