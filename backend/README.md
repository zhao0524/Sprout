# Sprout Core Optimization & Spatial Services (FastAPI)

This service provides high-performance spatial-temporal garden optimization, yield and economic projection modeling, photorealistic 3D Gaussian-splat world generation via World Labs Marble, weather integration, and secure webhook processing.

Data access is managed through the Supabase client with authenticated token verification and server-side role enforcement.

---

## Quick Start

```bash
cd backend
python -m venv .venv
# Windows: .venv\Scripts\activate   |   macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env        # Configure Supabase and API credentials
uvicorn app.main:app --reload
```

- **Interactive API Docs:** http://localhost:8000/docs
- **System Health:** http://localhost:8000/health (reports `supabase_configured`, `weather_service`, `world_labs_service`)

---

## Service Endpoints

Base path: `/api`. All non-webhook endpoints enforce `Authorization: Bearer <supabase JWT>`.

- **Gardens:** `POST /gardens`, `GET /gardens`, `GET /gardens/{id}`, `PATCH /gardens/{id}`, `DELETE /gardens/{id}` (owner-verified; relational cascading), `POST /gardens/{id}/obstacles`.
- **Crops:** `GET /crops`, `GET /crops/{id}`.
- **Plans:** `POST /gardens/{id}/plans/generate` (2-stage space-time optimizer), `GET /plans/{id}`.
- **Weather:** `GET /gardens/{id}/weather` (real-time atmospheric forecast for garden coordinates).
- **World (Marble 3D):** `POST /gardens/{id}/world` (initiates 3D Gaussian-splat synthesis from garden photo and planned crop configuration), `GET /gardens/{id}/world/status`, `GET /gardens/{id}/world/assets` (splat scene URLs and scale metadata), `GET /gardens/{id}/world/pano` (equirectangular 360 panorama proxy).
- **Marketplace:** `POST /marketplace/listings`, `GET /marketplace/listings` (browse active listings with dynamic filtering by `crop_id`, `city`, `exchange_type`), `GET /marketplace/listings/mine`, `GET /marketplace/listings/reserved`, `PATCH`/`DELETE /marketplace/listings/{id}`, `POST`/`DELETE /marketplace/listings/{id}/reserve` (notifies seller and marks listing status).
- **Webhooks:** `POST /webhooks/n8n/weather-notification` (enforces shared secret header `X-N8N-Secret`).

---

## Authentication & Security

`auth.py` validates Supabase JWTs via JWKS (RS256/ES256) with fallback verification, incorporating clock-skew tolerance to validate newly minted tokens seamlessly.

---

## World Labs Marble 3D Engine

`services/world_labs.py` uploads user garden imagery as a Marble media asset, initiates 3D Gaussian-splat world generation with prompt conditioning derived from the spatial planting plan, and provides scene assets for real-time WebGL rendering via `@sparkjsdev/spark` and Three.js.

---

## Automated Test Suite

```bash
pytest
```

Validates:
- Spatial constraint satisfaction (boundary constraints, obstacle exclusion, non-overlap).
- Temporal interval scheduling and valid succession dates.
- Dynamic yield and economic savings range calculations.
- JWT authentication and security headers.
- World Labs asset response parsing.
- Webhook secret authorization.

---

## Directory Structure

```text
app/
  main.py          FastAPI application setup, CORS middleware, error handlers, routing
  config.py        Environment configuration and dynamic setting resolution
  auth.py          Supabase JWT authentication (JWKS verification, clock-skew leeway)
  supabase_client  Authenticated server-side Supabase client
  crops_repo.py    Dynamic loader and query interface for agronomic crop catalog
  models/          Pydantic schemas for requests, responses, and spatial objects
  routes/          Endpoints for gardens, crops, plans, weather, world 3D, and webhooks
  services/        Space-time optimizer, yield estimates, weather service, World Labs client
  data/            Agronomic seed dataset (crop_seed_data.json)
tests/             Automated verification tests for algorithms and API endpoints
```
