# AGENTS.md

## Stack
- Ruby 3.4.1, Rails ~> 8.1.3 (`.ruby-version`, `Gemfile`).
- SQLite via the `sqlite3` gem; development/test databases live under `storage/`.
- Puma web server with Thruster in front; Propshaft asset pipeline.
- Hotwire (`turbo-rails`, `stimulus-rails`) + `importmap-rails`; JSON views via `jbuilder`.
- Background/cache/cable use the Solid stack: `solid_queue`, `solid_cache`, `solid_cable`.
- Deploys via Kamal + Docker (`Dockerfile`, `config/deploy.yml`, `.kamal/`).

## Commands
- `bin/setup` — install dependencies and prepare the app.
- `bin/dev` — run the local development server.
- `bin/rails test` — run the Minitest suite under `test/`.
- `bin/rubocop` — lint using the Omakase config.
- `bin/brakeman` — run security checks.
- `bin/jobs` — run the Solid Queue worker; `bin/kamal` — deploy.

## Conventions
- Style follows `rubocop-rails-omakase` from `.rubocop.yml`.
- Routes include `resources :todos`, `GET /hello`, and the `/up` health check.
- Controllers use `before_action :set_todo` and `params.expect(...)` for strong params.
- Todo strong params currently permit only `:description`.
- JSON responses use Jbuilder views/partials under `app/views/todos/`.
- Current `Todo` schema has `description:string`, `due_date:datetime`, and timestamps.

## Don'ts
- Don't read, commit, log, or expose `.env`, `config/master.key`, or credential files.
- Don't hand-edit `db/schema.rb`; change the database through migrations in `db/migrate/`.
- Don't read files listed in `.cursorignore`, including secrets, logs, temp files, storage, and build artifacts.
- Don't add Sidekiq, Redis cache, or Redis cable gems; this app already uses the Solid stack.