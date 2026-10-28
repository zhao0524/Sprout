# Sprout

Intelligent, season-long food garden planning with dynamic space-time optimization and automated care.

Sprout transforms any yard, balcony, raised bed, or community plot into a productive, continuous food garden. By analyzing exact dimensions, sunlight exposure, and user crop preferences, Sprout computes a multi-season succession planting schedule that maximizes spatial and temporal soil utilization. The resulting interactive plan features a dynamic cell grid, timeline scrubbing, photorealistic 3D Gaussian-splat visualization, and a suite of automated workflows managing weather alerts, failed-crop replanning, environmental impact metrics, and a peer-to-peer seed marketplace.

> **Live Deployments:**
> - **Frontend:** https://sprout-1-qckn.onrender.com
> - **Automation Backend:** https://davidzhao0524.app.n8n.cloud/webhook/

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

## Tech Stack

| Layer | Technology |
| --- | --- |
| Frontend | React 18, Vite, TypeScript, Tailwind CSS, React Router |
| 3D Visualization | Three.js, `@sparkjsdev/spark` (Gaussian-splat rendering), World Labs Marble API |
| API & Optimizer | Python, FastAPI, Pydantic, httpx |
| Database, Auth & Storage | Supabase (PostgreSQL, Auth, Storage, Row Level Security) |
| Automation | n8n Cloud (event-driven and scheduled workflows) |
| Weather | Open-Meteo |
| Hosting | Render (static frontend + FastAPI web service) |

---

## Architecture & Data Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         Client Browser (React + TS)                         │
│  - Interactive Grid & Timeline Slider      - World Labs 3D Splat Viewer     │
│  - Dynamic Yield & Savings Projections     - Real-Time Action & Alert Feed  │
└──────────────┬──────────────────────────────────────────┬───────────────────┘
               │                                          │
      Auth & RLS Queries                    Authenticated REST (Supabase JWT)
               │                                          │
               ▼                                          ▼
┌───────────────────────────────┐          ┌──────────────────────────────────┐
│      Supabase (PostgreSQL)    │◀─Writes──│        FastAPI Service           │
│  - User Profiles & Gardens    │          │  - Space-Time Optimizer          │
│  - Spatial Plans & Assignments│          │  - Yield & Savings Estimates     │
│  - Private Image Storage      │          │  - World Labs 3D Generation      │
│  - Marketplace & Actions Feed │          │  - Weather (Open-Meteo)          │
│  - Learned Crop Performance   │          └──────────────────────────────────┘
└───────────────▲───────────────┘          ┌──────────────────────────────────┐
                │                          │       n8n Automation Engine      │
                └──────────Reads/Writes────│  - Weather Alerts & Daily Tasks  │
                                           │  - Crop Replanning & Impact      │
                                           │  - Closed-Loop Agronomic Learning│
                                           └──────────────────────────────────┘
```

1. **Direct Authenticated Queries**: The frontend talks to Supabase with Row Level Security (RLS) for authentication, garden management, plan rendering, and real-time notification feeds.
2. **FastAPI Service**: Plan optimization, estimates, weather, marketplace operations, and World Labs 3D generation run in the FastAPI backend, which verifies the user's Supabase JWT on every request.
3. **Event-Driven & Scheduled Automation**: n8n workflows (weather changes, crop outcomes, seed requests, and a daily 07:00 America/Toronto schedule) execute multi-step logic and write results back to Supabase, or post notifications to the backend's secret-protected webhook.

See [`docs/backend.md`](./docs/backend.md) for the complete backend specification and [`n8n/README.md`](./n8n/README.md) for the workflow catalogue.

---

## Repository Structure

```text
frontend/     React + Vite + TypeScript + Tailwind CSS (Supabase client, Three.js / Spark)
backend/      FastAPI service (space-time optimizer, estimates, weather, marketplace, 3D generation)
supabase/     PostgreSQL migrations, seed data, Row Level Security policies
n8n/          Event-driven and scheduled workflow definitions
docs/         Backend and architectural specifications
render.yaml   Render blueprint for the frontend and API services
```

---

## Quick Start

### Frontend

```bash
cd frontend
npm install
cp .env.example .env    # Configure your Supabase and API credentials
npm run dev             # Launches local server at http://localhost:5173
```

### Backend

```bash
cd backend
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env        # Configure Supabase, World Labs, and n8n credentials
uvicorn app.main:app --reload   # http://localhost:8000 (docs at /docs)
pytest -q                       # Run the backend test suite
```

See [`backend/README.md`](./backend/README.md) for the endpoint reference and [`supabase/README.md`](./supabase/README.md) for database provisioning, seed data, and RLS configuration.

### Environment Configuration

**Frontend (`frontend/.env`)**

- `VITE_SUPABASE_URL` — Supabase project API URL.
- `VITE_SUPABASE_PUBLISHABLE_KEY` — Supabase publishable/anon key (safe in the browser with RLS enabled).
- `VITE_API_BASE_URL` — FastAPI base URL (defaults to `http://localhost:8000`).

**Backend (`backend/.env`)** — see `backend/.env.example` for details.

- `SUPABASE_URL`, `SUPABASE_SECRET_KEY`, `SUPABASE_JWKS_URL`, `SUPABASE_JWT_SECRET` — database access and token verification (server only).
- `WORLD_LABS_API_KEY`, `WORLD_LABS_BASE_URL`, `WORLD_LABS_MODEL` — 3D garden generation.
- `OPEN_METEO_BASE_URL` — weather forecasts.
- `N8N_WEBHOOK_SECRET` — shared secret for n8n callbacks.
- `FRONTEND_URL`, `BACKEND_URL`, `APP_ENV`, `MOCK_WORLD_LABS`, `MOCK_WEATHER` — app settings and mock flags.

---

## Production Deployment

- **Frontend**: Render static site (`frontend/`) with client-side routing rewrites (`/*` → `/index.html`).
- **API**: Render Python web service (`backend/`) running `uvicorn app.main:app`. Both services are defined in [`render.yaml`](./render.yaml); secrets are set in the Render dashboard.
- **Database & Storage**: Managed Supabase PostgreSQL with Row Level Security policies and isolated storage buckets.
- **Automation Engine**: Hosted n8n Cloud executing live event webhooks and daily scheduled agronomic routines.
