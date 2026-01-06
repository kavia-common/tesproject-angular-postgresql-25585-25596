# PostgreSQL schema & seed (TesProject)

This container uses PostgreSQL running on port **5000** with database **myapp** and user **appuser**.

Connection string (authoritative): see `db_connection.txt`:
- `psql postgresql://appuser:dbuser123@localhost:5000/myapp`

## Domain

A minimal **Projects / Tasks** domain to support CRUD.

### Table: `projects`
Columns:
- `id` UUID PK, default `gen_random_uuid()`
- `name` TEXT, required, length 1..200
- `description` TEXT, required, default `''`
- `created_at` TIMESTAMPTZ, default `now()`
- `updated_at` TIMESTAMPTZ, default `now()` (maintained by trigger)

Constraints:
- `projects_name_len` (name length)

### Table: `tasks`
Columns:
- `id` UUID PK, default `gen_random_uuid()`
- `project_id` UUID FK -> `projects(id)` ON DELETE CASCADE
- `title` TEXT, required, length 1..200
- `description` TEXT, required, default `''`
- `status` TEXT, required, default `'todo'` (enum-like check)
- `due_date` DATE nullable
- `created_at` TIMESTAMPTZ, default `now()`
- `updated_at` TIMESTAMPTZ, default `now()` (maintained by trigger)

Constraints:
- `tasks_title_len` (title length)
- `tasks_status_chk` status IN ('todo','in_progress','done')

Indexes:
- `idx_tasks_project_id` on `tasks(project_id)`
- `idx_tasks_status` on `tasks(status)`

### Updated timestamps
A shared trigger function keeps `updated_at` current:

- function: `set_updated_at()`
- triggers:
  - `trg_projects_updated_at` on `projects`
  - `trg_tasks_updated_at` on `tasks`

## Seed data

One project and three tasks:

- Project: `TesProject Demo`
- Tasks:
  - `Set up backend API` (in_progress)
  - `Wire up frontend` (todo)
  - `Verify end-to-end` (todo)

Seed inserts were executed as individual `INSERT` statements. The project insert is written to be idempotent with `ON CONFLICT DO NOTHING`; tasks are inserted via `INSERT ... SELECT` from the seeded project.

## How schema/seed was applied

Per container rules, statements were executed **one at a time** using:

- `CONN=$(cat db_connection.txt)`
- `$CONN -c "SQL_STATEMENT"`

No `.sql` migration files were added.
