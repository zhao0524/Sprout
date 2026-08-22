# Sprout Backend — n8n + Supabase

Sprout has **no custom API server**. All backend automation runs on a hosted
**n8n** instance backed by **Supabase (Postgres)**. **No external APIs are used
anywhere.**

The frontend interacts with the backend in exactly two ways:

1. **POST JSON to n8n webhook URLs** for event-driven flows.
2. **Read result rows directly from Supabase tables** (via the Supabase client,
   protected by RLS).

There is no polling of n8n — by the time the frontend reads a table, n8n has
already written the rows.

**n8n webhook base URL:** `https://davidzhao0524.app.n8n.cloud/webhook/`

Every event-driven path below is relative to that base, e.g. the weather flow is
`https://davidzhao0524.app.n8n.cloud/webhook/sprout/weather`.

---

## Event-driven workflows (frontend POSTs to these)

### 1. Weather Condition Automation — `POST /sprout/weather`

- **Body:** a `weather_status` of `normal | heavy_rain | heat | frost`, plus garden context.
- **Reads:** `gardens`.
- **Writes:** `garden_actions`.
- **Errors:** missing required fields → **HTTP 400 with no writes**.

### 2. Failed Crop Replanning — `POST /sprout/failed-crop`

- **Reads:** `planting_plans`, `plot_assignments`, `crops`, `garden_preferences`, `gardens`.
- **Behaviour:** picks a replacement crop and updates the relevant `plot_assignments` row.
- **Writes:** updates `plot_assignments`; writes `garden_actions`.

### 3. Environmental Impact Tracking — `POST /sprout/harvest`

- **Reads:** `crops` (for estimate factors).
- **Writes:** upserts cumulative metrics into `garden_impact` using the
  fetch-existing-then-branch pattern (see below). All metrics are labelled **estimates**.

### 4. Smart Seed Marketplace Matching — `POST /sprout/seed-match`

- **Behaviour:** derives the required crops from `plot_assignments`, matches
  `marketplace_listings` that are `status = active` and **not reserved**, and ranks
  by crop match, same-city, and quantity.
- **Reads:** `plot_assignments`, `marketplace_listings`.
- **Writes:** inserts the ranked rows into `seed_matches`.

### 5. Leftover Seed Redistribution — `POST /sprout/seed-redistribute`

- **Behaviour:** creates a giveaway `marketplace_listings` row, finds other users
  who need the crop (the owner is excluded), and matches them.
- **Writes:** the giveaway `marketplace_listings` row; up to **5** `seed_matches`
  rows; an owner `garden_actions` row.
- **Errors:** unknown crop → **HTTP 400 with no writes**.

---

## Scheduled workflows (no frontend call)

These run **daily at 07:00 America/Toronto**. They require no frontend
involvement — their output appears in Supabase after each run.

### 6. Planting Reminders

- **Driven from:** `plot_assignments.plant_date`, joined via `planting_plans` to
  `gardens` (for `user_id`) and `crops` (for names).
- **Writes:** `garden_actions`. De-duplicates so a reminder is not repeated.

### 7. Daily Action Engine

- **Active gardens:** those with `planting_plans.status = 'active'`.
- **Reads:** `planting_plans`, `plot_assignments`, `crops`, `gardens`.
- **Writes:** `garden_actions`, **capped at 3 per garden per day**, with de-duplication.

### 8. Closed-Loop Learning

- **Reads:** `garden_outcome_events`.
- **Behaviour:** groups by crop plus condition (sunlight or planting month) and
  computes `success_rate` and average yield.
- **Writes:** upserts into `crop_performance` (fetch-existing then branch). Only
  writes a group once it has **3 or more samples** (`MIN_SAMPLE = 3`).

---

## Data model notes

### `garden_actions`

Columns are exactly: `title`, `description`, `priority`, `garden_id`, `user_id`,
`created_at`. **There is no `category` or `source` column** — category and source
context is folded into the `description` text.

### Tables added for these workflows

Four tables now exist in Supabase for the workflows above:

- **`garden_impact`** — cumulative environmental-impact estimates (workflow 3).
- **`seed_matches`** — ranked seed/listing matches, shared by workflows 4 and 5.
- **`garden_outcome_events`** — raw outcome samples consumed by workflow 8.
- **`crop_performance`** — learned success rate / average yield per crop+condition (workflow 8).

### Upsert = fetch-existing then branch

The Supabase node version in use has **no native upsert**. Accumulating records
therefore use a two-step pattern: **fetch the existing row, then branch to an
UPDATE if it exists or an INSERT if it does not.** Expect this double-step write in
the Environmental Impact (`garden_impact`) and Closed-Loop Learning
(`crop_performance`) workflows. Keep it in mind when reading or extending them.

---

## Frontend integration guidance

- **To trigger an event flow:** POST JSON to the matching webhook URL above.
- **To display results:** read the relevant Supabase tables directly —
  `garden_actions`, `garden_impact`, `seed_matches`, `crop_performance`. Do **not**
  poll n8n; n8n has already written the rows.
- **Scheduled workflows** (6–8) need no frontend calls. Their output shows up in
  Supabase after each 07:00 America/Toronto run.

## Security follow-up

Webhook triggers are currently **unauthenticated by default**. This is acceptable
for the hackathon, but before the app goes any further it should require
authentication (e.g. a shared secret header or signed request) on every webhook —
track this as a security follow-up.
