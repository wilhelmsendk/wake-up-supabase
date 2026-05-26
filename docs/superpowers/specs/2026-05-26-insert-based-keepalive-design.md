# Insert-based keepalive design

**Date:** 2026-05-26
**Status:** Approved, ready for implementation

## Problem

The current workflow keeps Supabase free-tier projects alive by issuing a daily
`GET /rest/v1/<table>?select=*&limit=1` against PostgREST. In practice, projects
configured this way are still getting paused. A read against an empty or
RLS-restricted table can return 401/403, which is not a successful DB query and
does not reset Supabase's inactivity timer. Even successful reads may not be a
strong enough activity signal.

A write is unambiguous: an `INSERT` is a real DB transaction, it cannot be
served from any cache, and Supabase clearly counts it as activity.

## Goal

Replace the daily SELECT with a daily INSERT of a single row into a dedicated
`wake_up_supabase` table on each project.

## Constraints

- Continue to use **anon keys only**. No service_role key, no Postgres
  connection string in CI secrets.
- PostgREST does not expose DDL, so the workflow cannot run `CREATE TABLE`.
  Table creation is handled out-of-band: the user runs a one-time SQL snippet
  per project, copied from the README into the Supabase Dashboard's SQL Editor.
- A failure on one project must not abort the loop. Preserve existing
  fault isolation (`set +e`/`set -e` + `continue`).
- Never print key values to logs.

## Schema (one-time per project)

```sql
create table if not exists public.wake_up_supabase (
  id bigserial primary key,
  value text not null
);

alter table public.wake_up_supabase enable row level security;

create policy "anon insert wake_up_supabase"
  on public.wake_up_supabase
  for insert
  to anon
  with check (true);
```

Design notes:

- `value text` (not `timestamptz`) per user preference — stored as an ISO 8601
  Zulu string, e.g. `2026-05-26T06:00:00Z`. Human-readable in the dashboard.
- Only an `insert` policy is granted to `anon`. No `select` policy means anon
  cannot read the rows back, which is the desired posture for a write-only
  heartbeat table.
- `bigserial` covers ~9e18 rows; one row/day is harmless forever.

## Workflow change

For each project entry in `SUPABASE_PROJECTS_JSON`:

- Resolve `TABLE`: if the JSON entry omits `table`, default to
  `wake_up_supabase`. This keeps existing configs working and removes the
  field as a required setup step.
- Generate timestamp: `NOW=$(date -u +"%Y-%m-%dT%H:%M:%SZ")`.
- POST to `<url>/rest/v1/<table>` with:
  - headers: `apikey: <key>`, `Authorization: Bearer <key>`,
    `Content-Type: application/json`, `Prefer: return=minimal`
  - body: `{"value":"<NOW>"}`
- Success = HTTP 201. Anything else gets a targeted message:
  - 401/403 → "anon role can't insert — run the SQL from the README in the
    Supabase SQL Editor"
  - 404 → "table not found — check the table name or run the README SQL"
  - other → generic error with the HTTP code

Same `set +e` / `CURL_EXIT` / `continue` pattern as the existing code — no
change to fault isolation.

## Documentation updates

- `README.md`:
  - Reframe from "daily SELECT keeps the DB alive" to "daily INSERT logs a
    heartbeat row, which is a real DB write".
  - Add the one-time SQL setup block under a clear "Set up each Supabase
    project" heading.
  - Mark `table` as optional in the `SUPABASE_PROJECTS_JSON` example.
- `README_zh-CN.md`: mirror the English changes.
- `CLAUDE.md`: update the key invariant from
  *"must hit /rest/v1/<table> with SELECT"* to
  *"must POST a row to /rest/v1/<table>"*, and add a new invariant:
  *"the target table needs an RLS policy granting INSERT to anon — see the
  README setup SQL"*.

## Explicitly out of scope

- No table cleanup / TTL. ~365 rows/year is negligible.
- No extra columns (runner ID, commit SHA, etc.). `value` alone proves activity.
- No alternate credentials (service_role, direct Postgres connection string).
  Keeps the existing security posture.
- No client-side retry. The workflow already runs daily; a single missed day
  isn't worth complicating the script.
