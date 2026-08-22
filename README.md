# Sprout

Turn a yard, balcony, raised bed, or community plot into a season-long food garden with succession planting.

Sprout takes a garden's dimensions, sunlight level, and a list of vegetables you eat, then produces a season-long planting plan that reuses each garden section over time (lettuce in spring hands the same cells off to beans in summer). The plan renders on a cell grid with a timeline slider, and a set of automations keep the garden on track over the season — weather actions, failed-crop replanning, environmental-impact tracking, a seed marketplace, and daily reminders.

> **Live:**
> Frontend — https://sprout-1-qckn.onrender.com
> Backend (n8n) — https://davidzhao0524.app.n8n.cloud/webhook/

## Architecture

There is **no custom API server.** Backend automation runs entirely on a hosted
**n8n** instance backed by **Supabase (Postgres)**, and **no external APIs are
used anywhere.** The frontend talks to the backend in exactly two ways:

1. **POST JSON to n8n webhook URLs** to trigger event-driven flows.
2. **Read result rows directly from Supabase tables** (Supabase client, RLS-protected).

There is no polling of n8n — by the time the frontend reads a table, n8n has
already written the rows. Scheduled workflows run on their own daily cadence and
also just write to Supabase.

**See [`docs/backend.md`](./docs/backend.md) for the full backend reference** —
every workflow, its webhook path, request body, the tables it reads/writes, error
behaviour, and the frontend integration contract.

```
         POST /sprout/*  (webhooks)
Frontend ───────────────────────────▶  n8n (workflows)  ──writes──▶  Supabase
   │                                     ▲                              │
   │        read rows (Supabase client)  │  scheduled (07:00)           │
   └─────────────────────────────────────┴──────────────────◀──reads───┘
```

## What it does

- **Auth** — Supabase email/password; the browser talks to Supabase directly for auth and RLS-protected reads/writes.
- **Gardens** — create/list/delete gardens with dimensions, sunlight, city, and an optional photo (Supabase Storage).
- **Crop selection & plan** — a curated crop dataset with must-have/preferred/optional priorities; the plan renders on a garden grid with a dashed cell overlay, a legend, a timeline slider that swaps crops for successors by date, and yield/savings ranges.
- **Marketplace** — list surplus produce to sell/trade/give away, browse other growers' listings by crop and city, and reserve an item.
- **Automations (n8n)** — weather-driven garden actions, failed-crop replanning, environmental-impact estimates, smart seed-marketplace matching, leftover-seed redistribution, daily planting reminders, a daily action engine, and closed-loop learning of crop performance. Results land in Supabase tables the frontend reads (`garden_actions`, `garden_impact`, `seed_matches`, `crop_performance`).

## Structure

```
frontend/   React + Vite + TS + Tailwind (Supabase client)
supabase/   migrations, seed, RLS policies, storage bucket, workflow tables
docs/       backend.md — the n8n + Supabase backend reference
n8n/        exported workflow(s)
backend/    LEGACY FastAPI service — not the live backend (see note below)
```

> **Note on `backend/`.** The `backend/` folder is an earlier FastAPI
> implementation and is **not** the live backend. The live backend is n8n +
> Supabase as described above and in [`docs/backend.md`](./docs/backend.md).

## Frontend — quick start

```bash
cd frontend
npm install
cp .env.example .env    # fill in your Supabase values
npm run dev             # http://localhost:5173
```

The frontend requires `VITE_SUPABASE_URL` and `VITE_SUPABASE_PUBLISHABLE_KEY` — it fails fast at startup if they are missing rather than running against placeholders. See [`supabase/README.md`](./supabase/README.md) to provision the database, seed crops, and enable RLS.

## Environment

Frontend (`frontend/.env`):

- `VITE_SUPABASE_URL` — Supabase project URL.
- `VITE_SUPABASE_PUBLISHABLE_KEY` — Supabase publishable/anon key (safe in the browser with RLS on).
- `VITE_N8N_WEBHOOK_BASE_URL` — the n8n webhook base, `https://davidzhao0524.app.n8n.cloud/webhook/`, that event-driven flows POST to.

There is no `VITE_API_BASE_URL` in the live architecture — the frontend reads Supabase directly and POSTs to n8n webhooks; it does not call a custom API server.

## Deployment

- **Frontend** — static site on Render (`frontend/`), env vars above, with a rewrite rule `/*` → `/index.html` so client-side routes work on refresh.
- **Backend** — the n8n instance is hosted (n8n Cloud) and Supabase is managed; there is no server to deploy.
