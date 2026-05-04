# Horse Race Tipping — Resume Here

**Live site:** https://horseracetipping.com  
**Worker:** `racetipping-api` (Cloudflare, account `a6f47c17811ee2f8b6caeb8f38768c20`)  
**D1:** `racetipping-db` UUID `42d14cda-3b37-46fe-8f2c-e6d1aea7d0c3`  
**GitHub:** `PaddyGallivan/racetipping-api` (source: `index.js`)  
**Platform handover:** https://raw.githubusercontent.com/LuckDragonAsgard/asgard-source/main/docs/HANDOVER.md

## What it is
Self-hosted race day tipping comp platform. Pub/family groups scan a QR code, pick horses, watch live leaderboard. Admin manages race days, enters results, tracks payments. Stripe subscription ($20 AUD/month) after 2 free race days.

## Current state (2026-05-05)

### Live & working ✅
- Homepage + punter UI: `horseracetipping.com/{slug}`
- Admin panel: PIN login, race day management, venue/horses, results entry, payments
- Leaderboard: `/api/{slug}/leaderboard/{VENUE_CODE}`
- Punter register/login (email + password)
- Stripe checkout + webhook (subscription)
- **Resend email** (via `hello@luckdragon.io`):
  - Welcome email on registration
  - Results notification to all tippers when results entered
- **CF Email Routing**: `hello@horseracetipping.com` → `paddy@luckdragon.io` (Active)
- Terms: `/terms` — includes Luck Dragon Pty Ltd, subscription section, paddy@luckdragon.io
- Privacy: `/privacy`

### Worker bindings (D1 + secrets)
- `DB` → D1 `42d14cda-3b37-46fe-8f2c-e6d1aea7d0c3`
- `RESEND_API_KEY` → Resend (sends from `hello@luckdragon.io`)
- `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`
- `SUPER_PIN`, `ANTHROPIC_API_KEY`, `CF_ACCOUNT_ID`, `CF_API_TOKEN`, `ASGARD_VAULT_ID`

### Live data
- Org: `Gallivan Family` (slug: `family`, admin PIN: `1016`)
- Active race day: Gallivan Cup — May 2026 (Hawkesbury NSW + Eagle Farm QLD)
- Tippers: active, no test data

## Key route patterns
```
GET  /api/{slug}/raceday/current          → active race day + venues
POST /api/{slug}/auth/login               → {pin} or {email,password} → {ok, token}
POST /api/{slug}/punter/register          → {name, email, password, motto?} → {ok, token, tipper}
POST /api/{slug}/punter/login             → {email, password} → {ok, token, tipper}
GET  /api/{slug}/leaderboard/{CODE}       → leaderboard for venue
POST /api/{slug}/admin/raceday            → create race day (admin)
POST /api/{slug}/admin/venue              → add venue
POST /api/{slug}/admin/horses             → set horses for race
POST /api/{slug}/results                  → enter result + triggers email notification
POST /api/{slug}/stripe/checkout          → get Stripe checkout URL (admin only)
POST /api/stripe/webhook                  → Stripe webhook handler
GET  /{slug}                              → punter UI (HTML)
GET  /terms                               → T&C
GET  /privacy                             → Privacy policy
```

## Deploy
```bash
# Fetch token from vault
CF_TOKEN=$(curl -s -H "X-Pin: PIN" https://asgard-vault.pgallivan.workers.dev/secret/CF_API_TOKEN)

# Deploy (always include D1 binding or it silently drops)
# Metadata must include: {"main_module":"worker.js","bindings":[{"name":"DB","type":"d1","id":"42d14cda-3b37-46fe-8f2c-e6d1aea7d0c3"},...]}
```

## Known issues / next actions
- `info@horseracetipping.com` and `bookings@horseracetipping.com` still route to missing `asgard-email` worker — fix or delete those CF Email Routing rules
- No automated TAB results import (TAB API drops data after race day; must scrape same-day)
- Resend domain for `horseracetipping.com` added (id `200c2d41`) but DNS not verified — sending from `luckdragon.io` instead
- No GitHub Actions CI (token lacks org write for workflows)

## Credentials
All in `asgard-vault.pgallivan.workers.dev` — fetch with PIN:
- `CF_API_TOKEN` — Cloudflare (Workers + D1 + Stripe)
- `RESEND_API_KEY` — Resend email
- `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET` — Stripe live keys
- `GITHUB_TOKEN` — GitHub PaddyGallivan user
