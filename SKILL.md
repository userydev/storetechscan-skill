---
name: storetechscan
description: Bulk-check ecommerce store domains the user already has: storefront platform (Shopify, Shopify Plus, WooCommerce, Magento...), marketing/helpdesk/subscription apps and pixels, public Shopify catalog price range, deterministic or AI-calibrated prospect fit, and public role contacts. Use when the user asks which stores in a list run a given platform or app, what they charge, or how to reach them. Not a lead database: the user supplies the domains.
---

# StoreTechScan

Live inspection of user-supplied domains over REST (`POST https://storetechscan.ytriple.dev/api/v1/inspect`) or MCP (`https://storetechscan.ytriple.dev/mcp`, Streamable HTTP, no install). Unreachable, WAF-blocked or erroring domains are returned with `billable: false` and never cost credits.

## Decide which path to use

- No API key available: call anonymously. Free preview is 25 domains per client IP per UTC day (shared across REST and MCP), deterministic rules only, `icp_fit_score` is null.
- User has a key (`zsk_live_...`): send `Authorization: Bearer <key>`. Up to 100 domains per REST call (25 per MCP call), calibrated AI fit score 1.0–5.0, credits never expire.
- Never ask the user to paste a key into chat if an environment variable or MCP header already carries it.

## Quickstart

```bash
curl -X POST https://storetechscan.ytriple.dev/api/v1/inspect \
  -H 'content-type: application/json' \
  -d '{"domains":["fromourplace.com","hydromateusa.com"]}'
```

MCP client config: `{ "mcpServers": { "storetechscan": { "url": "https://storetechscan.ytriple.dev/mcp" } } }`
Tools: `inspect_domains`, `get_inspection_results`, `check_credits`, `get_ecommerce_benchmark_2026`, `buy_credits`, `claim_purchase`.
Sample response: https://storetechscan.ytriple.dev/samples/inspect-response.json · OpenAPI 3.1: https://storetechscan.ytriple.dev/openapi.json · Full context: https://storetechscan.ytriple.dev/llms-full.txt

## Handling limits and payment

- HTTP 400: fix the input (1–100 valid domains per call, deduplicated).
- HTTP 401: the Bearer key is invalid. Drop the header to fall back to the free preview, or ask the user to check the key.
- HTTP 402 with `error_code: FREE_PREVIEW_EXHAUSTED` or `FREE_PREVIEW_BATCH_TOO_LARGE`: the free allowance is used up or the batch is too large for anonymous use. Tell the user the allowance resets at `free_preview.resets_at` (UTC), and show `upgrade.pricing_url` so a human can buy credits. Packs:
  - Starter: $9 for 500 credits ($0.018 / domain)
  - Growth: $19 for 2,000 credits ($0.0095 / domain)
  - Agency: $49 for 10,000 credits ($0.0049 / domain)
  After purchase the human receives a `zsk_live_...` key; retry the same call with it.
- Buying without leaving the session (MCP): call `buy_credits` with `package_id`; show its `checkout_url` to the human and keep `session_id` and `claim_token` to yourself. Poll `claim_purchase` every 10–15 seconds (up to 15 minutes) until `ready` is true; it returns the key once. Reconnect with `Authorization: Bearer <key>` and continue. Over REST: `POST /api/billing/checkout` with `source: "agent"`, then `POST /api/billing/claim`.
- HTTP 402 with `credits_balance` on a keyed call: the account is out of credits. Report the balance and the pricing URL.
- HTTP 429 with `Retry-After`: wait that many seconds and retry once.

## Reading results

- `status` OK means the homepage answered; anything else (`DNS_ERROR`, `TIMEOUT`, `WAF_CHALLENGE`, `HTTP_4XX`...) is unreachable and unbilled.
- Platform: `primary_platform`, `is_shopify`, `is_shopify_plus`. Apps: `marketing_plugins`, `support_plugins`, `subscription_reviews_plugins`, `pixels`, `tech_stack`.
- Catalog: `shopify_probe_status` LIVE plus `shopify_products_count_sample`, `shopify_price_min/median/max` in `shopify_currency` (never converted).
- Fit: `icp_vertical`, `business_model`, `icp_one_liner`, `icp_fit_score` (paid only), `is_real_merchant`, `has_b2b_wholesale`.
- Contacts: `primary_email`, `generic_emails` (same-domain role addresses only, not SMTP-verified), social URLs.
- Say what cannot be inferred: no revenue or traffic estimates; server-side or sales-channel tags (Meta Pixel, GA4) leave no public marker.

## Common requests

### shopify-klaviyo-shortlist

User says: "Here are 40 store domains. Which ones run Shopify with Klaviyo, and what do they charge?"

1. Call `POST /api/v1/inspect` with body `{"domains":["<up to 25 anonymous / 100 with key>"]}`, or MCP tool `inspect_domains` with `{"domains":["..."],"platform":"Shopify"}`.
2. Read `is_shopify`, `is_shopify_plus`, `marketing_plugins`, `shopify_price_median`, `shopify_currency`, `shopify_products_count_sample`.
3. The MCP platform filter also matches detected tech, so platform: "Klaviyo" keeps only Klaviyo rows; on REST filter marketing_plugins client-side. Prices are the store's native currency, never converted; shopify_probe_status LIVE means the public catalog answered.

### reachable-contact-for-outreach

User says: "Find a public contact address for each of these brands so I can email their partnerships team."

1. Call `POST /api/v1/inspect` with body `{"domains":["..."]}`, or MCP tool `inspect_domains` with `{"domains":["..."],"real_merchant_only":true}`.
2. Read `primary_email`, `generic_emails`, `instagram_url`, `linkedin_url`, `status`, `billable`.
3. Only same-domain role addresses (info@, sales@, partnerships@ ...) are returned; personal emails are never extracted. Rows with billable:false were unreachable and cost nothing.

### segment-by-platform-and-tools

User says: "Group this list by ecommerce platform and tell me who uses a subscription app or a helpdesk."

1. Call `GET /api/v1/inspect?domains=a.com,b.com,c.com`, or MCP tool `inspect_domains` with `{"domains":["..."]}`.
2. Read `primary_platform`, `platforms`, `subscription_reviews_plugins`, `support_plugins`, `pixels`, `tech_stack`.
3. Server-side or sales-channel installs (common for Meta Pixel and GA4) leave no public marker, so absence is not proof of non-use.
