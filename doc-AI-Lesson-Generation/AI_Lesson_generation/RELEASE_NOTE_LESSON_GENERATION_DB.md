# Release Note - Lesson Generation DB Changes

## Scope

This release introduces database support for asynchronous AI lesson generation job tracking and hardens data integrity constraints.

## Included Migrations

- `V60__Create_lesson_generation_job_table.sql`
  - Adds `lesson_generation_job` table for session-based async job lifecycle.
  - Captures request payload, status, retries, error details, and generated lesson reference.
  - Adds query indexes for workspace/status/time and session lookup.

- `V61__Add_constraints_to_lesson_generation_job.sql`
  - Normalizes invalid status values before constraint enforcement.
  - Cleans orphan references before adding foreign keys.
  - Adds status check constraint.
  - Adds foreign keys to `workspaces`, `users`, and optional `lessons`.

## Deployment Order

1. Deploy application code compatible with `lesson_generation_job`.
2. Run DB migration `V60`.
3. Run DB migration `V61`.
4. Verify lesson generation APIs:
   - `POST /lesson/generation/content`
   - `POST /lesson/generation/generate`
   - `POST /lesson/generation/retry/{sessionId}`
   - `GET /lesson/generation/jobs`

## Operational Impact

- Adds one new table and additional indexes.
- Enforces referential integrity for async lesson generation jobs.
- Deletes invalid orphan rows in `lesson_generation_job` during `V61` to keep constraints valid.

## Rollback Guidance (Manual)

- Rollback scripts are provided under:
  - `course-forge-backend/src/main/resources/db/manual-rollback/RB_V61__Add_constraints_to_lesson_generation_job.sql`
  - `course-forge-backend/src/main/resources/db/manual-rollback/RB_V60__Create_lesson_generation_job_table.sql`

Rollback order:
1. `RB_V61...`
2. `RB_V60...`

## Compatibility Notes

- Existing APIs and tables are not modified.
- New constraints apply only to `lesson_generation_job`.
- If historical bad rows exist in `lesson_generation_job`, `V61` will sanitize/remove them before constraints are added.

