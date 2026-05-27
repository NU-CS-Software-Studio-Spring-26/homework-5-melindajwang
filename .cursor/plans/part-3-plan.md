---
name: Optional Due Date On Todo
overview: Allow new (and edited) todos to optionally accept a `due_date`. The DB column already exists, so the change is permitting the param, exposing the field in the form, rendering it in views, and extending tests.
todos:
  - id: permit-param
    content: Add :due_date to params.expect in TodosController#todo_params
    status: pending
  - id: form-field
    content: Add form.datetime_field :due_date to app/views/todos/_form.html.erb
    status: pending
  - id: html-partial
    content: Render due_date (with nil fallback) in app/views/todos/_todo.html.erb
    status: pending
  - id: json-partial
    content: Add :due_date to json.extract! in app/views/todos/_todo.json.jbuilder
    status: pending
  - id: fixtures
    content: Add a due_date to one of the todos fixtures, leave the other nil
    status: pending
  - id: controller-test
    content: Update controller create test for due_date and add a no-due-date case
    status: pending
  - id: system-test
    content: Extend system create test to fill in Due date and assert it on show
    status: pending
  - id: model-test-optional
    content: Optionally add a Todo model test covering nil and present due_date
    status: pending
isProject: false
---

### Context

- `todos.due_date` (datetime) already exists in [db/schema.rb](db/schema.rb) line 18 via [db/migrate/20260519180933_add_due_date_to_todo.rb](db/migrate/20260519180933_add_due_date_to_todo.rb). No new migration is required.
- Today's create path: [app/controllers/todos_controller.rb](app/controllers/todos_controller.rb) `create` at lines 23-35 calls `Todo.new(todo_params)` and `@todo.save`, where `todo_params` at lines 74-76 permits only `:description`.
- The form at [app/views/todos/_form.html.erb](app/views/todos/_form.html.erb) only renders a `:description` field; partials [app/views/todos/_todo.html.erb](app/views/todos/_todo.html.erb) and [app/views/todos/_todo.json.jbuilder](app/views/todos/_todo.json.jbuilder) don't surface `due_date`.

### Assumptions (flag any to change)

- `due_date` is **truly optional** — no presence or "must be in the future" validation. Empty submissions store `NULL`.
- Use an HTML `datetime-local` input (`form.datetime_field`) to match the `datetime` column. (If a date-only UX is preferred, the column type would need to change via a new migration — out of scope here.)
- Display format on the show/index pages uses a simple `l(todo.due_date, format: :long)` when present, with a "No due date" fallback. No I18n format additions.

### Numbered changes

1. **Permit the new attribute (controller)** — Update `todo_params` in [app/controllers/todos_controller.rb](app/controllers/todos_controller.rb) at lines 74-76 so the `params.expect(todo: [...])` list includes `:due_date` alongside `:description`. No other controller changes needed; `create` (lines 23-35) and `update` (lines 38-48) both already flow through `todo_params`.

2. **Add the field to the shared form** — In [app/views/todos/_form.html.erb](app/views/todos/_form.html.erb), add a new labeled `form.datetime_field :due_date` block below the description field (around line 17). Leaving the field blank must remain valid.

3. **Display the due date in the HTML show/index partial** — Update [app/views/todos/_todo.html.erb](app/views/todos/_todo.html.erb) to render `todo.due_date` (formatted via `l(...)`) inside a new `<p>`, with a "No due date" fallback when it is `nil`. This affects both `index` and `show` because both render the `_todo` partial.

4. **Expose due date in JSON responses** — Update [app/views/todos/_todo.json.jbuilder](app/views/todos/_todo.json.jbuilder) to add `:due_date` to the `json.extract!` list so the JSON API surfaces the new attribute consistently with `description`.

5. **Update the test fixture** — In [test/fixtures/todos.yml](test/fixtures/todos.yml), give at least `todos(:one)` a `due_date` (e.g. `<%= 1.week.from_now %>`), and leave `todos(:two)` with `due_date` unset to cover the "no due date" case used by `_todo.html.erb`.

6. **Extend the controller test** — In [test/controllers/todos_controller_test.rb](test/controllers/todos_controller_test.rb):
   - Modify the existing `should create todo` test (lines 18-24) to pass `due_date` in params and assert `Todo.last.due_date` is approximately equal to the submitted value (within ~1 second to tolerate datetime rounding).
   - Add a new test `should create todo without due date` that POSTs only a `description` and asserts the create succeeds and `Todo.last.due_date` is `nil`.
   - Optionally mirror the same two cases for `update`.

7. **Extend the system test** — In [test/system/todos_test.rb](test/system/todos_test.rb), extend `should create todo` (lines 13-22) to also `fill_in "Due date", with: ...` before clicking "Create Todo", then assert the formatted date appears on the resulting show page. Keep one path that omits the field to confirm it remains optional.

8. **(Optional) Add a model spec for the column** — In [test/models/todo_test.rb](test/models/todo_test.rb), add a small test that `Todo.new(description: "x").save` succeeds with `due_date` left `nil`, plus one confirming a `Todo` round-trips a non-nil `due_date`. Skip if you'd rather not expand the model suite.

### Things explicitly NOT in scope

- No new database migration (column already exists).
- No changes to the `Todo` model in [app/models/todo.rb](app/models/todo.rb) (kept empty unless validations are requested).
- No filtering/sorting by `due_date` on the index — say the word if you want it.
- No "due_date must be in the future" or "after created_at" validation — say the word if you want it.

### Verification

- Run `bin/rails test` for unit/controller tests and `bin/rails test:system` for the Capybara test.
- Manually exercise `bin/dev`: create a todo with and without a due date, confirm both render correctly and the JSON view at `/todos.json` includes `due_date`.