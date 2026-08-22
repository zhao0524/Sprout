# Sprout Backend (FastAPI) — LEGACY

> **Not the live backend.** Sprout's backend is now **n8n + Supabase** with no
> custom API server — see [`../docs/backend.md`](../docs/backend.md). This FastAPI
> service is an earlier implementation kept for reference and is not deployed. The
> notes below describe that legacy server.

Plan generation, estimates, weather proxy, World Labs (Marble) 3D generation, and
the n8n webhook. Data access is through the Supabase client (secret key,
server-side only).

## Quick start

```bash
cd backend
python -m venv .venv
# Windows: .venv\Scripts\activate   |   macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env        # fill in Supabase values
uvicorn app.main:app --reload
```

- API docs: http://localhost:8000/docs
- Health: http://localhost:8000/health (reports `supabase_configured`, `mock_weather`, `mock_world_labs`)

`MOCK_WEATHER=true` (default) mocks the forecast. The **3D preview needs a real
World Labs key** and `MOCK_WORLD_LABS=false`; there is no built-in demo world, so
in mock mode the preview has nothing to render. Real data (gardens, plans,
notifications) needs the Supabase env vars set.

## Endpoints

Base path `/api`. All non-webhook endpoints require `Authorization: Bearer <supabase JWT>`.

- **Gardens** — `POST /gardens`, `GET /gardens`, `GET /gardens/{id}`, `PATCH /gardens/{id}`, `DELETE /gardens/{id}` (owner-checked; related rows cascade), `POST /gardens/{id}/obstacles`.
- **Crops** — `GET /crops`, `GET /crops/{id}`.
- **Plans** — `POST /gardens/{id}/plans/generate`, `GET /plans/{id}`.
- **Weather** — `GET /gardens/{id}/weather`.
- **World (Marble)** — `POST /gardens/{id}/world` (start generation from the garden photo + planned crops), `GET /gardens/{id}/world/status`, `GET /gardens/{id}/world/assets` (splat/pano URLs + scale metadata), `GET /gardens/{id}/world/pano` (same-origin panorama proxy).
- **Marketplace** — `POST /marketplace/listings`, `GET /marketplace/listings` (browse others' published/unreserved, filters `crop_id`/`city`/`exchange_type`), `GET /marketplace/listings/mine`, `GET /marketplace/listings/reserved`, `PATCH`/`DELETE /marketplace/listings/{id}`, `POST`/`DELETE /marketplace/listings/{id}/reserve`. Reserving notifies the seller. Backend-mediated because seller name/city come from `profiles`, which RLS hides from other users.
- **Webhook** — `POST /webhooks/n8n/weather-notification` (shared-secret header `X-N8N-Secret`, not a user token).

## Auth

`auth.py` verifies Supabase JWTs via JWKS (RS256/ES256), falling back to an HS256
shared secret. It allows a small clock-skew leeway so freshly issued tokens whose
`iat` is a few seconds ahead of the server clock still verify.

## World Labs (Marble)

`services/world_labs.py` uploads the garden photo as a Marble media asset,
starts a generation with an image + text prompt built from the plan's crops,
polls the operation, and exposes the finished world's assets. The hosted Marble
viewer cannot be embedded (`X-Frame-Options: DENY`), so the frontend renders the
returned Gaussian-splat scene itself and links out to the full world.

## Tests

```bash
pytest
```

Covers the optimizer invariants, the yield/savings ranges, JWT verification
(including clock-skew tolerance), the World Labs response parsing, and the n8n
webhook secret check.

## Layout

```
app/
  main.py          FastAPI app, CORS, error shape, router mounting
  config.py        pydantic-settings env loading (SUPABASE_URL / SUPABASE_SECRET_KEY, aliases accepted)
  auth.py          Supabase JWT verification (JWKS + HS256 fallback, clock-skew leeway)
  supabase_client  server-side Supabase client (secret key)
  crops_repo.py    loads the curated crop dataset
  models/          pydantic request/response schemas
  routes/          gardens, crops, plans, weather, world, webhooks
  services/        optimizer, estimates, weather, world_labs
  data/            crop_seed_data.json
tests/
```
