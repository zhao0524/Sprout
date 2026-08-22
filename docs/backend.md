# Sprout Backend — n8n + Supabase Architecture

Sprout operates on an event-driven automation backend where workflow logic executes on a hosted **n8n** engine integrated directly with **Supabase (PostgreSQL)**.

The frontend interacts with the backend through two streamlined pathways:

1. **POST JSON to n8n webhook endpoints** for event-driven workflows (weather evaluations, crop replanning, harvest logging, seed marketplace matching, and seed redistribution).
2. **Direct reactive queries to Supabase tables** (via the Supabase client, secured by PostgreSQL Row-Level Security).

Workflows write state changes directly to Supabase tables, allowing the frontend to immediately consume updated records without polling loops.

**n8n Webhook Base URL:** `https://davidzhao0524.app.n8n.cloud/webhook/`

Every event-driven path is relative to this base (for example, the weather automation is `https://davidzhao0524.app.n8n.cloud/webhook/sprout/weather`).

---

## Event-Driven Workflows (Client-Triggered)

### 1. Weather Condition Automation — `POST /sprout/weather`

- **Purpose:** Dynamically analyzes reported atmospheric conditions and translates them into actionable garden care tasks.
- **Request Body:** `{ "garden_id": "<uuid>", "weather_status": "normal | heavy_rain | heat | frost", "temperature_c": <number>, "precipitation_mm": <number> }`
- **Reads:** `gardens`
- **Writes:** Inserts prioritized action items into `garden_actions`.
- **Error Handling:** Missing required fields returns HTTP 400 with no database mutation.

### 2. Dynamic Crop Replanning — `POST /sprout/failed-crop`

- **Purpose:** Re-evaluates garden space and time constraints when a crop fails or is removed early, dynamically selecting a compatible successor crop to fill the newly opened window.
- **Request Body:** `{ "garden_id": "<uuid>", "failed_assignment_id": "<uuid>", "failure_date": "YYYY-MM-DD" }`
- **Reads:** `planting_plans`, `plot_assignments`, `crops`, `garden_preferences`, `gardens`.
- **Writes:** Updates the target `plot_assignments` row with the replacement crop and adjusted timeline; logs an informational task to `garden_actions`.

### 3. Environmental Impact Tracking — `POST /sprout/harvest`

- **Purpose:** Computes cumulative environmental impact metrics upon harvest logging, calculating avoided supply-chain transport emissions, water efficiency, and organic food mass.
- **Request Body:** `{ "garden_id": "<uuid>", "crop_id": "<string>", "harvest_weight_kg": <number>, "harvest_date": "YYYY-MM-DD" }`
- **Reads:** `crops` (for baseline agronomic conversion factors).
- **Writes:** Dynamically upserts cumulative metrics into `garden_impact` via stateful update branching.

### 4. Smart Seed Marketplace Matching — `POST /sprout/seed-match`

- **Purpose:** Analyzes the upcoming crop requirements from active `plot_assignments`, identifies matching active community listings from `marketplace_listings`, and scores them by geographic proximity, seed quantity, and compatibility.
- **Request Body:** `{ "garden_id": "<uuid>", "user_id": "<uuid>" }`
- **Reads:** `plot_assignments`, `marketplace_listings`.
- **Writes:** Inserts ranked recommendation rows into `seed_matches`.

### 5. Leftover Seed Redistribution — `POST /sprout/seed-redistribute`

- **Purpose:** Creates a community giveaway listing for surplus seeds and identifies nearby growers whose planned gardens benefit from the offering.
- **Request Body:** `{ "user_id": "<uuid>", "crop_id": "<string>", "quantity": <number>, "unit": "<string>", "city": "<string>" }`
- **Writes:** Inserts a giveaway `marketplace_listings` record, creates up to 5 targeted recommendation rows in `seed_matches`, and logs a confirmation item in `garden_actions`.
- **Error Handling:** Invalid crop identifier returns HTTP 400.

---

## Scheduled Workflows (Automated Engine)

Scheduled workflows execute automatically on a daily schedule (**07:00 America/Toronto**) to maintain system health and alert growers to time-sensitive operations.

### 6. Planting Reminders

- **Operation:** Evaluates `plot_assignments.plant_date` against current calendar dates, joining `planting_plans`, `gardens`, and `crops`.
- **Writes:** Emits dynamic, de-duplicated planting tasks into `garden_actions`.

### 7. Daily Action Engine

- **Operation:** Evaluates all active gardens (`planting_plans.status = 'active'`), synthesizing current weather conditions, growth stages, and care intervals.
- **Writes:** Populates `garden_actions` with prioritized, actionable recommendations (capped and de-duplicated to maintain high signal-to-noise).

### 8. Closed-Loop Agronomic Learning

- **Operation:** Consumes logged harvest outcomes from `garden_outcome_events`, clustering data by crop type, sunlight rating, and planting month.
- **Writes:** Dynamically updates `crop_performance` with empirical success rates and yield benchmarks once statistical sample thresholds (`MIN_SAMPLE >= 3`) are achieved.

---

## Data Architecture & Model Notes

### `garden_actions`
Standardized notification and action feed schema:
- `title` (text) — Concise headline of the required action or update.
- `description` (text) — Detailed context including source reasoning and agronomic recommendations.
- `priority` (text: `high` | `medium` | `low`) — Urgency level for dashboard prioritization.
- `garden_id` (uuid) — Target garden reference.
- `user_id` (uuid) — Owning user reference.
- `created_at` (timestamptz) — Event timestamp.

### Dynamic Workflow Tables

1. **`garden_impact`** — Tracks cumulative environmental sustainability indicators (`total_kg_yield`, `water_saved_liters`, `carbon_offset_kg`).
2. **`seed_matches`** — Stores ranked match scores pairing growers with active seed marketplace listings.
3. **`garden_outcome_events`** — Event stream recording actual harvest yields, growth durations, and crop health outcomes.
4. **`crop_performance`** — Dynamic empirical agronomic model mapping real-world success rates to environmental parameters.

### Stateful Upsert Pattern
For accumulating aggregate models (`garden_impact`, `crop_performance`), the workflow engine employs an atomic fetch-and-branch pattern (checking existing records, then executing an update with accumulated deltas or an initial insert).

---

## Frontend Integration Guidelines

- **Triggering Workflows:** Dispatch a `POST` request with JSON payload to the specific workflow endpoint.
- **Consuming Results:** Query Supabase tables directly using the standard Supabase client. Queries are automatically filtered by Row-Level Security based on the user's authenticated session.
- **Real-Time Data:** Table subscriptions or reactive query refetches provide instant UI updates when workflows complete.

---

## Security & Access Control

- **Client Queries:** Enforced via PostgreSQL Row-Level Security policies bound to authenticated user JWTs (`auth.uid()`).
- **Workflow Endpoints:** Webhook endpoints validate incoming payloads and can enforce secret key verification (`X-N8N-Secret` or HMAC signatures) for authorized system communication.
