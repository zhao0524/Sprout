# n8n workflows

n8n is Sprout's backend. Automation runs on a hosted n8n instance backed by
Supabase — there is no custom API server and no external APIs.

**Webhook base URL:** `https://davidzhao0524.app.n8n.cloud/webhook/`

Event-driven flows the frontend POSTs to:

- `POST /sprout/weather` — Weather Condition Automation
- `POST /sprout/failed-crop` — Failed Crop Replanning
- `POST /sprout/harvest` — Environmental Impact Tracking
- `POST /sprout/seed-match` — Smart Seed Marketplace Matching
- `POST /sprout/seed-redistribute` — Leftover Seed Redistribution

Scheduled flows (daily 07:00 America/Toronto, no frontend call): Planting
Reminders, Daily Action Engine, Closed-Loop Learning.

**Full reference** — request bodies, tables read/written, error behaviour, the
fetch-then-branch upsert pattern, and the frontend integration contract — is in
[`../docs/backend.md`](../docs/backend.md).

`weather-workflow.json` is an exported workflow kept for reference; the live
workflows run on the hosted instance above.
