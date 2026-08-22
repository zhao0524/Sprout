# Sprout

Intelligent, season-long food garden planning with dynamic space-time optimization and automated care.

Sprout transforms any yard, balcony, raised bed, or community plot into a productive, continuous food garden. By analyzing exact dimensions, sunlight exposure, and user crop preferences, Sprout computes a multi-season succession planting schedule that maximizes spatial and temporal soil utilization. The resulting interactive plan features a dynamic cell grid, timeline scrubbing, photorealistic 3D Gaussian-splat visualization, and a suite of automated workflows managing weather alerts, failed-crop replanning, environmental impact metrics, and a peer-to-peer seed marketplace.

> **Live Deployments:**
> - **Frontend:** https://sprout-1-qckn.onrender.com
> - **Automation Backend:** https://davidzhao0524.app.n8n.cloud/webhook/

---

## Architecture & Data Flow

Sprout features an event-driven architecture powered by **Supabase (PostgreSQL, Auth, Storage, RLS)**, an **n8n Automation Engine**, and **World Labs Marble 3D Gaussian Splatting**.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                             Client Browser (React + TS)                     │
│  - Interactive Grid & Timeline Slider      - World Labs 3D Splat Viewer     │
│  - Dynamic Yield & Savings Projections    - Real-Time Action & Alert Feed  │
└──────────────┬──────────────────────────────────────────┬───────────────────┘
               │                                          │
       Auth & RLS Queries                          Webhook Events
               │                                          │
               ▼                                          ▼
┌───────────────────────────────┐          ┌──────────────────────────────────┐
│      Supabase (PostgreSQL)    │◀─Writes──│       n8n Automation Engine      │
│  - User Profiles & Gardens    │          │  - Dynamic Weather Alerts        │
│  - Spatial Plans & Assignments│          │  - Automated Crop Replanning     │
│  - Private Image Storage      │          │  - Impact & Yield Analytics      │
│  - Marketplace & Actions Feed │──Reads──▶│  - Daily Plant Care Engine       │
│  - Learned Crop Performance   │          │  - Closed-Loop Agronomic ML      │
└───────────────────────────────┘          └──────────────────────────────────┘
```

1. **Direct Authenticated Queries**: The frontend communicates securely with Supabase using Row Level Security (RLS) for authentication, garden management, spatial plan rendering, and real-time notification feeds.
2. **Event-Driven Workflow Automation**: Dynamic lifecycle events (weather changes, crop outcomes, seed requests) trigger targeted n8n webhooks that execute multi-step logic and write updates directly back to the database.
3. **Scheduled Agronomic Engine**: Automated workflows run on daily schedules to monitor frost alerts, evaluate daily garden tasks, and continuously train the crop performance model.

See [`docs/backend.md`](./docs/backend.md) for the complete backend specification.

---

## Key Features

- **Dynamic Space-Time Optimizer**: Solves 2D spatial allocation and 1D temporal scheduling simultaneously. When early spring crops (such as radishes or lettuce) reach harvest, the engine automatically schedules compatible summer successors (such as bush beans) into the exact same garden cells.
- **Interactive Spatial Plan & Timeline**: Visualizes physical garden beds on a sub-divided grid with real-world dimensions (cm/m), color-coded crop footprints, companion planting highlights, and an interactive date slider.
- **Photorealistic 3D Garden Reconstruction**: Integrates with the World Labs Marble API to synthesize a 3D Gaussian-splat representation from uploaded garden imagery and planned vegetation, rendered directly in-browser via WebGL.
- **Automated Garden Care & Weather Intelligence**: Continuous monitoring triggers localized care recommendations, frost alerts, and seasonal reminders directly in the user's action feed.
- **Dynamic Yield & Economic Projections**: Computes data-driven harvest weight ranges (kg) and grocery cost offsets based on crop footprint, yield curves, and regional retail pricing.
- **Peer-to-Peer Produce & Seed Marketplace**: Enables local growers to list surplus produce or seeds for sale, trade, or giveaway, with real-time reservation and notification workflows.
- **Closed-Loop Agronomic Learning**: Collects real-world harvest events to compute localized crop success rates and yield benchmarks across varying sunlight and microclimate conditions.

---

## Repository Structure

```text
frontend/     React + Vite + TypeScript + Tailwind CSS (Supabase Client, Three.js / Spark)
supabase/     PostgreSQL migrations, seed data, Row-Level Security policies, storage
docs/         Comprehensive backend and architectural specifications
n8n/          Event-driven and scheduled workflow definitions
backend/      FastAPI service (Space-time constraint optimizer, estimates, 3D processing)
```

---

## Quick Start (Frontend)

```bash
cd frontend
npm install
cp .env.example .env    # Configure your Supabase and API credentials
npm run dev             # Launches local server at http://localhost:5173
```

### Environment Configuration

Configure `frontend/.env` with your deployment variables:

- `VITE_SUPABASE_URL` — Supabase project API URL.
- `VITE_SUPABASE_PUBLISHABLE_KEY` — Supabase publishable anon key (safe in browser with RLS enabled).
- `VITE_N8N_WEBHOOK_BASE_URL` — Base endpoint for triggering backend automation webhooks (`https://davidzhao0524.app.n8n.cloud/webhook/`).

Refer to [`supabase/README.md`](./supabase/README.md) for database provisioning, seed data initialization, and RLS configuration.

---

## Production Deployment

- **Frontend**: Hosted on Render as a static site (`frontend/`), configured with client-side routing rewrites (`/*` → `/index.html`).
- **Database & Storage**: Managed Supabase PostgreSQL with secure Row Level Security policies and isolated storage buckets.
- **Automation Engine**: Hosted n8n Cloud executing live event webhooks and daily scheduled agronomic routines.
