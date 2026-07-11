# Privacy & Compliance

Reference for consent, PII, and GDPR-conscious defaults in conversion-analytics work.

## Before adding events

1. Check for existing cookie banner, CMP (Cookiebot, OneTrust), or consent hook.
2. Check privacy policy / terms for analytics disclosure obligations.
3. Confirm whether the product serves EU/UK users (stricter consent defaults).

## Consent gating

```js
function trackIfConsented(event, properties) {
  if (!hasAnalyticsConsent()) return;
  analytics.capture(event, properties);
}
```

Never fire marketing/analytics pixels before consent when a CMP is present.

## PII rules

**Do not send as event properties:**

- Email, phone, full name
- Free-text form fields
- Government IDs, payment card data
- Precise location (lat/long)

**Acceptable when already in app identity layer:**

- Opaque `user_id` / `account_id` (if product already uses it server-side)
- Aggregated plan tier, country (if disclosed in privacy policy)

## Anonymization

- Prefer vendor anonymize-IP settings when available (GA4 default in many setups).
- Use session-scoped visitor IDs for anonymous funnels.
- Hash only when the product already documents it — don't invent hashing schemes without legal review.

## Data minimization

- Collect properties needed for funnel analysis only.
- Drop properties that don't change decisions (page title duplicates URL, etc.).
- Set retention in vendor admin when the team controls it — note in handoff doc.

## Experiment-specific

When wiring `variant` for A/B tests:

- Variant assignment is not PII.
- Do not log variant + email in the same custom dimension without approval.
- Document experiment ID in internal runbook, not in user-visible UI.

## When to escalate

Ask the user before implementing if:

- Tracking touches health, financial, or children's data
- Cross-border data transfer is unclear
- Legal/compliance copy on cookie banner must change

Leave legal text formal; flag UX issues only.
