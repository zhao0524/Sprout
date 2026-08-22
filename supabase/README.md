# Supabase Database & Security Architecture

Sprout uses Supabase (PostgreSQL) for authentication, relational data persistence, real-time access control via Row Level Security (RLS), and secure asset storage.

---

## Database Setup & Migrations

Execute the SQL migration scripts in order:

1. `migrations/0001_initial_schema.sql` — Core relational tables, foreign key constraints, indexes, and the automated new-user profile creation trigger.
2. `migrations/0002_marketplace.sql` — Peer-to-peer marketplace extensions (crop linkage, pricing, localization, reservation workflow).
3. `seed.sql` — Comprehensive agronomic crop dataset, spacing footprints, growth timelines, yield distributions, and companion/conflict relationships.
4. `rls-policies.sql` — Row-Level Security policies and storage bucket access rules.

---

## Dynamic Workflow Tables

The backend workflow engine interacts with dedicated tables in Supabase to maintain dynamic state:

- **`garden_actions`** — Action and notification stream for active gardens (`title`, `description`, `priority`, `garden_id`, `user_id`, `created_at`).
- **`garden_impact`** — Dynamically computed cumulative environmental metrics (food weight harvested, water saved, CO2 avoided).
- **`seed_matches`** — Ranked candidate listings connecting growers with compatible surplus seed offerings.
- **`garden_outcome_events`** — Historical harvest and growth records capturing real-world crop outcomes.
- **`crop_performance`** — Dynamic agronomic intelligence model storing computed success rates and yield metrics across environmental conditions.

### State Accumulation
Accumulating tables (`garden_impact`, `crop_performance`) use an atomic fetch-and-branch pattern to update existing records or insert new entries as events occur.

---

## Storage Architecture

The private storage bucket **`garden-images`** manages garden site photos:
- Storage access policies enforce strict user isolation, allowing users to upload and read assets located exclusively under their authenticated `auth.uid()` path (e.g. `user-id/garden-id/original.jpg`).
- Uploaded garden images are directly processed by the 3D Gaussian Splatting engine to generate spatial garden representations.

---

## Security & Access Control

- **Frontend (`VITE_SUPABASE_PUBLISHABLE_KEY`)**: Uses the public publishable key with authenticated user sessions; all queries are strictly constrained by Row-Level Security.
- **Backend & Automation Engine (`SUPABASE_SECRET_KEY`)**: Uses the service key within secured server environments to perform cross-table aggregations, scheduled workflow updates, and automated notifications.

The agronomic crop dataset is defined in `backend/app/data/crop_seed_data.json` and synchronized into `supabase/seed.sql`.
