# PostgreSQL schema & seed (TesProject)

This container uses PostgreSQL running on port **5000** with database **myapp** and user **appuser**.

Connection string (authoritative): see `db_connection.txt`:
- `psql postgresql://appuser:dbuser123@localhost:5000/myapp`

## Domain

A minimal **Items** domain to support a simple CRUD backend/frontend.

### Extension: `pgcrypto`
Used for UUID generation:
- `CREATE EXTENSION IF NOT EXISTS pgcrypto;`

### Function: `set_updated_at()`
A shared trigger function keeps `updated_at` current:

- `set_updated_at()` sets `NEW.updated_at = now()` on UPDATE.

> Note: when applying via shell, `$$` must be escaped as `\$\$` to avoid the shell expanding `$$` into a PID.

### Table: `items`

Columns:
- `id` UUID PK, default `gen_random_uuid()`
- `title` TEXT, **NOT NULL**
- `description` TEXT, nullable
- `created_at` TIMESTAMPTZ, **NOT NULL**, default `now()`
- `updated_at` TIMESTAMPTZ, **NOT NULL**, default `now()` (maintained by trigger)

Constraints:
- `items_title_len` CHECK `char_length(title) BETWEEN 1 AND 200`

Indexes:
- `idx_items_created_at` on `items(created_at DESC)` (supports typical “newest first” listing)

Triggers:
- `trg_items_updated_at` BEFORE UPDATE ON `items` calling `set_updated_at()`

## Seed data

Three minimal rows inserted as individual statements:
- `First item` (with description)
- `Second item` (NULL description)
- `Editable item` (with description)

## How schema/seed was applied

Per container rules, statements were executed **one at a time** using the CLI command from `db_connection.txt`:

- `CONN=$(cat db_connection.txt)`
- `$CONN -v ON_ERROR_STOP=1 -c "SQL_STATEMENT"`

No `.sql` migration files were added.

### Applied changes / notes

- Previous demo tables (`projects`, `tasks`) were dropped to align the DB with the frontend’s expected simple `items` CRUD shape.
- `pgcrypto` was enabled so `gen_random_uuid()` can be used as the `id` default.
- `updated_at` is automatically maintained via trigger to avoid relying on application code for timestamp updates.
