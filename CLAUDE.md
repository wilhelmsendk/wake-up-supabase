# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A single GitHub Actions workflow that keeps free-tier Supabase projects from being auto-paused. There is no application code, build step, or test suite — the entire repo is `.github/workflows/wake-up-supabase.yml`.

## Architecture

`wake-up-supabase.yml` runs daily (`cron: "0 6 * * *"`, 06:00 UTC) or on manual `workflow_dispatch`. It:

1. Validates the `SUPABASE_PROJECTS_JSON` repository secret exists.
2. Parses that secret with `jq` — it is a JSON array of `{ "url", "anon_key", "table"? }` objects. `table` is optional; if omitted, it defaults to `wake_up_supabase`.
3. For each project, `POST`s `{"value":"<UTC ISO timestamp>"}` to `<url>/rest/v1/<table>` with `apikey`, `Authorization: Bearer`, `Content-Type: application/json`, and `Prefer: return=minimal` headers. Success = HTTP 201.

Key invariants when editing the workflow:

- The ping **must be a write (`POST /rest/v1/<table>`)**, not a `SELECT` and not `/auth/v1/health`. Supabase's 7-day inactivity timer tracks *database* activity, and reads have proven unreliable in practice (caching layers, RLS-denied 401/403 responses). An `INSERT` is an unambiguous DB transaction that Postgres must execute, which is what keeps the project alive.
- The **target table needs an RLS policy granting `INSERT` to the `anon` role**. The README contains a one-time SQL snippet (`create table if not exists public.wake_up_supabase ...` + `create policy ... for insert to anon`) that users must run once per project in the Supabase SQL Editor. PostgREST does not expose DDL, so the workflow cannot create the table itself.
- Use **anon keys only**, never the `service_role` key.
- The job must never print key values to logs.
- A failure on one project (bad URL, curl error, non-2xx) must not abort the loop — `set +e`/`set -e` brackets the curl call and `continue` skips bad entries. Preserve this fault isolation.

## Testing changes

There is no local test harness. To verify a workflow change, push it and trigger a run from the **Actions** tab via "Run workflow" (`workflow_dispatch`). Confirm `SUPABASE_PROJECTS_JSON` is set on the repo's secrets first, or the run fails at the validation step by design. Each successful run also leaves a new row in the project's `wake_up_supabase` table that you can inspect via the Supabase dashboard.
