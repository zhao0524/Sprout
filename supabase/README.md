# Supabase setup

Run these in the Supabase SQL editor (or via the CLI) in order:

1. `migrations/0001_initial_schema.sql` — tables + the new-user profile trigger.
2. `migrations/0002_marketplace.sql` — extends `marketplace_listings` (crop link, price, city, reservation fields).
3. `seed.sql` — curated crops and companion/conflict relationships.
4. `rls-policies.sql` — row level security and the storage bucket policy.

## Workflow tables (n8n backend)

The live backend is n8n writing into Supabase (see [`../docs/backend.md`](../docs/backend.md)).
Beyond the core schema, four tables back the automation workflows and now exist in
the project:

- **`garden_actions`** — the app's action/reminder feed. Columns are exactly
  `title`, `description`, `priority`, `garden_id`, `user_id`, `created_at`. There is
  **no `category` or `source` column**; that context is folded into `description`.
- **`garden_impact`** — cumulative environmental-impact estimates.
- **`seed_matches`** — ranked seed/listing matches (seed-match and redistribution flows).
- **`garden_outcome_events`** — raw outcome samples for closed-loop learning.
- **`crop_performance`** — learned success rate / average yield per crop + condition.

Accumulating tables (`garden_impact`, `crop_performance`) are written with a
**fetch-existing then branch (update vs insert)** pattern because the Supabase node
version in use has no native upsert.

## Storage

Create a **private** bucket named `garden-images`. The storage policy in
`rls-policies.sql` restricts each user to objects under a folder named after
their `auth.uid()` (e.g. `user-id/garden-id/original.jpg`).

## Keys

- Frontend uses the **publishable/anon** key (`VITE_SUPABASE_PUBLISHABLE_KEY`)
  for both auth and direct table reads.
- n8n connects with the **secret/service-role** key and may bypass RLS. Never
  expose it to the browser.

`seed.sql` is generated from `backend/app/data/crop_seed_data.json`; edit the
JSON and regenerate rather than hand-editing the SQL.
