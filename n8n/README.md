# Sprout Automation Workflows (n8n Engine)

n8n powers Sprout's event-driven and scheduled automation backend. All workflow logic executes on a managed n8n engine tightly integrated with Supabase (PostgreSQL) — orchestrating dynamic garden operations, intelligent crop replanning, environmental impact modeling, seed marketplace matching, and closed-loop agronomic learning.

**Webhook Base URL:** `https://davidzhao0524.app.n8n.cloud/webhook/`

---

## Event-Driven Workflows (Client-Triggered)

- `POST /sprout/weather` — **Weather Condition Automation**: Evaluates real-time environmental conditions and generates prioritized garden care tasks.
- `POST /sprout/failed-crop` — **Dynamic Crop Replanning**: Re-optimizes the remaining space-time grid upon crop loss, selecting viable companion successors.
- `POST /sprout/harvest` — **Environmental Impact Tracking**: Dynamically computes cumulative water savings, carbon offsets, and organic food yields upon harvest logging.
- `POST /sprout/seed-match` — **Smart Seed Marketplace Matching**: Ranks and pairs garden crop requirements with available active community listings.
- `POST /sprout/seed-redistribute` — **Leftover Seed Redistribution**: Publishes community surplus seed listings and notifies compatible local growers.

---

## Scheduled Workflows (Automated Engine)

Executed daily at **07:00 America/Toronto**:

1. **Planting Reminders** — Monitors upcoming plant dates across all active garden plans and emits personalized notifications.
2. **Daily Action Engine** — Dynamically scans active gardens and evaluates care requirements, generating prioritized daily tasks.
3. **Closed-Loop Agronomic Learning** — Aggregates historical garden outcome events, dynamically computing localized crop success rates and yield benchmarks per sunlight and seasonal condition.

---

## Documentation Reference

For comprehensive request schemas, database operations, error handling, and integration contracts, see [`../docs/backend.md`](../docs/backend.md).
