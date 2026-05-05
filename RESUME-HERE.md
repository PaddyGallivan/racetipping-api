# Horse Race Tipping — Resume Here

**Live site:** https://horseracetipping.com  
**Worker:** `racetipping-api` (Cloudflare, account `a6f47c17811ee2f8b6caeb8f38768c20`)  
**D1:** `racetipping-db` UUID `42d14cda-3b37-46fe-8f2c-e6d1aea7d0c3`  
**GitHub:** `PaddyGallivan/racetipping-api` (source: `index.js`)  
**Platform handover:** https://raw.githubusercontent.com/LuckDragonAsgard/asgard-source/main/docs/HANDOVER.md

## What it is
Self-hosted race day tipping comp platform. Pub/family groups scan a QR code, pick horses, watch live leaderboard. Admin manages race days, enters results, tracks payments. Stripe subscription ($20 AUD/month) after 2 free race days.

## Current state (2026-05-05) — fully operational ✅

### Live & working
- Homepage + punter UI: `horseracetipping.com/{slug}`
- Admin panel: PIN login, race day management, venue/horses, results entry, payments
- Leaderboard: `/api/{slug}/leaderboard/{VENUE_CODE}`
- Punter register/login (email + password)
- Stripe checkout + webhook (subscription)
- **Resend email** (sends from `hello@horseracetipping.com`):
  - Welcome email on registration
  - Results notification to all tippers when results entered
- **Resend domain verified** for `horseracetipping.com` (id `200c2d41`)
- **CF Email Routing** (all active, all → paddy@luckdragon.io):
  - hello@, info@, bookings@, admin@
- **Subscription status card** in admin panel (trial/active/suspended)
- **Stripe billing portal** — "Manage / Cancel →" button in admin panel; portal config `bpc_1TTiRHAm8bVflPN0hFtUi7YN`
- **Super admin** `/api/super/orgs` + `/api/super/activate` — use `X-Admin-Pin: 1016`
- **Auto-results cron** `*/5 * * * *` — scrapes TAB same-day, writes results to D1
- Terms: `/terms` — Luck Dragon Pty Ltd, subscription section
- Privacy: `/privacy`
- Self-serve signup: `/signup` → `POST /api/signup`
- Free trial: 2 race days enforced, upgrade wall shown on expiry

### Worker bindings (full set — always include ALL on deploy)
- `DB` → D1 `42d14cda-3b37-46fe-8f2c-e6d1aea7d0c3`
- `RESEND_API_KEY` → `re_K1LYEyoD_9M43D6k5JUthWGefLvNhKfXV`
- `STRIPE_SECRET_KEY` → in vault as `STRIPE_SECRET_KEY`
- `STRIPE_WEBHOOK_SECRET` → in vault as `RACETIPPING_STRIPE_WEBHOOK_SECRET`
- `SUPER_PIN` → `1016` (also in vault as `RACETIPPING_SUPER_PIN`)
- `CF_ACCOUNT_ID`, `CF_API_TOKEN`, `ANTHROPIC_API_KEY`, `ASGARD_VAULT_ID`

> ⚠️ Always include ALL bindings on deploy or secrets silently disappear.

### Live data
- Org: `Gallivan Family` (slug: `family`, admin PIN: `1016`)
- subscription_status: `active` (manually set, no stripe_customer_id)
- Active race day: Gallivan Cup — May 2026

## Key routes
```
GET  /api/{slug}/raceday/current          → active race day + venues
POST /api/{slug}/auth/login               → {pin} or {email,password} → {ok, token}
POST /api/{slug}/punter/register          → register tipper
POST /api/{slug}/stripe/checkout          → get Stripe checkout URL (admin)
POST /api/{slug}/stripe/billing-portal    → get Stripe billing portal URL (admin)
POST /api/signup                          → create new org (self-serve)
GET  /api/super/orgs                      → list all orgs (X-Admin-Pin: 1016)
POST /api/super/activate                  → {org_slug, status} (X-Admin-Pin: 1016)
GET  /{slug}                              → punter UI
GET  /signup                              → self-serve signup page
GET  /terms, /privacy                     → legal
```

## Deploy
```bash
# Always include ALL bindings — see wrangler.toml for list
# Via asgard-tools (preferred):
curl -X POST https://asgard-tools.pgallivan.workers.dev/admin/deploy \
  -H "X-Pin: 535554" -H "Content-Type: application/json" \
  --data @deploy_payload.json
# payload: { worker_name, code_b64, bindings: [...all bindings...] }
```

## Credentials (all in vault — fetch with PIN 535554)
- `CF_API_TOKEN`, `CF_FULLOPS_TOKEN` — Cloudflare
- `RESEND_API_KEY` — Resend
- `STRIPE_SECRET_KEY`, `RACETIPPING_STRIPE_WEBHOOK_SECRET` — Stripe live
- `GITHUB_TOKEN` — GitHub PaddyGallivan user
- `RACETIPPING_SUPER_PIN` — Worker super admin PIN (1016)
