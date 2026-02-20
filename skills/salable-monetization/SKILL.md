---
name: salable-monetization
description: Monetize apps on beta.salable.app 2.0 using Salable MCP tools and beta OpenAPI contracts. Use when creating or modifying products, plans, line items, prices, currency options, tiers, and entitlements, including flat-rate, per-seat, and metered packaging. For monetize/monetise requests, default scope includes public pricing views, entitlement-based feature gating, and subscription management views.
---

# Salable Monetization

Design Salable 2.0 monetization with a split path:
- MCP for catalog provisioning writes.
- REST for runtime app surfaces.

## Sources

- Primary contracts: `references/openapi.yaml`
- Hosted refresh source: `https://beta.salable.app/openapi.yaml`
- Runtime endpoint focus: `references/openapi-focus.md`
- MCP patterns: `references/mcp-tool-playbook.md`
- Pricing payload patterns: `references/pricing-model-templates.md`
- Auth recommendations: `references/auth-options.md`

At the start of every new session, fetch `https://beta.salable.app/openapi.yaml` and refresh `references/openapi.yaml` before implementation work. If hosted fetch is unavailable, continue with local references and state this assumption.

## Hard Rules

- Use only `beta.salable.app` contracts.
- Before provisioning, verify `mcp__salable__*` tools are available.
- If MCP is unavailable, stop and instruct user to set `SALABLE_API_KEY` and restart Codex.
- Never use raw email in identity fields (`owner`, `grantee`, `granteeId`).
- Require real app authentication before checkout, entitlement checks, or subscription management.
- Keep pricing/paywall routes public; keep checkout/account actions authenticated.
- Use MCP for provisioning; do not use REST fallback for catalog writes.
- Use REST for runtime surfaces; do not use MCP fallback for runtime operations.

## Default Monetize Deliverable

When asked to monetize/monetise an app or product, deliver:
1. Public pricing/paywall view.
2. Entitlement-based feature gating.
3. Authenticated subscription management view.

## Security Gate (Run First)

- Confirm app auth exists and identify non-email billing principal source (`org/tenant id` -> internal account/user id -> non-email `user.id`).
- If only email exists, derive deterministic non-email IDs with salted hash/HMAC and use consistently.
- Verify runtime config: `SALABLE_SECRET_KEY`, `SALABLE_API_BASE_URL=https://beta.salable.app`, `SALABLE_PRODUCT_ID`, and required plan ID env vars used by the app.
- If auth is missing or unclear, stop and return stack-specific auth recommendations (Auth.js first for Next.js).

## Provisioning Workflow

1. Confirm entitlement map with user: features, keys, plan mapping.
2. Confirm packaging: plan names, line item types, intervals, currencies, amounts, seat/usage limits.
3. Normalize all price inputs to decimal major units before writes (for example `"unitAmount": 15.0`, not `1500`). In `currencyOptions`, use decimal literals (for example `{ "currency": "USD", "unitAmount": 15.0 }`). Do not use `0.015`; minimum supported unit amount is `0.1`.
4. Build in dependency order: entitlements -> product -> plans.
5. Use `plans_save` as the default and required plan-write path unless operation is archive-only.
6. For updates, read existing plan first and preserve nested IDs.
7. Re-read persisted prices and confirm intended display amounts (avoid 100x errors).
8. Return created/updated IDs and a concise summary.

## Runtime Integration Workflow

1. Use REST endpoints for pricing, entitlements, checkout, and subscription actions.
2. Read exact request/response schema from `references/openapi-focus.md` before implementing each endpoint.
3. Handle wrapped responses via `body.data` for object and list payloads.
4. Keep pricing route unauthenticated with graceful fallback rendering if live fetch fails.
5. Ensure checkout follows create cart -> add cart item -> checkout sequence and uses authenticated non-email identity mapping.
6. Ensure account/subscription routes are auth-protected and include billing portal access.

## Verification

- Validate provisioning by reading back plans/prices and checking amounts, intervals, currencies, and entitlements.
- Validate app integration by running project build and relevant tests if available; fix failures before handoff.

## Guardrails

- Keep `lineItems` as structured objects in `plans_save` payloads (never serialized JSON strings).
- Preserve existing IDs on updates (`plan`, `lineItems`, `prices`, `currencyOptions`, `tiers`) to avoid duplicates.
- Enforce max one `per_seat` line item per plan.
- Ensure checkout interval/count exists on selected plan prices before cart item creation.
- Keep default currency consistent across line items used in the same checkout/cart.
- Never send `includeArchived=false`; omit flag by default and use `includeArchived=true` only when needed.
- Always send both `successUrl` and `cancelUrl` in checkout calls.
- Validate user ownership before subscription actions (`cancel`, `auto-renew`, `portal`, `invoices`).

## Known Quirks

- Some environments reject `includeArchived=false`; treat omission as active-only default.
- `plans_save` may fail with `Expected object, received string` if `lineItems` are stringified.
- Checkout may return `400` when `cancelUrl` is missing.
- Runtime auth failures may be Salable API config issues (`SALABLE_SECRET_KEY`, wrong base URL), not app session auth.
- `subscription.ownerId` may be an internal reference; do owner-scoped lookup checks instead of direct equality checks.
