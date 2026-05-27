Melinda Wang
Homework 5
COMP_SCI 397: Software Studio

# Parts 1 and 2

- GitHub Classroom hw5 repository link:
  - https://github.com/NU-CS-Software-Studio-Spring-26/homework-5-melindajwang/tree/hw5

- Link to `.cursorignore`:
  - https://github.com/NU-CS-Software-Studio-Spring-26/homework-5-melindajwang/blob/hw5/.cursorignore

- Link to `AGENTS.md`:
  - https://github.com/NU-CS-Software-Studio-Spring-26/homework-5-melindajwang/blob/hw5/AGENTS.md

- Link to `.cursor/rules/rails-conventions.mdc`:
  - https://github.com/NU-CS-Software-Studio-Spring-26/homework-5-melindajwang/blob/hw5/.cursor/rules/rails-conventions.mdc

- Link to `.cursor/rules/security.mdc`:
  - https://github.com/NU-CS-Software-Studio-Spring-26/homework-5-melindajwang/blob/hw5/.cursor/rules/security.mdc

# Part 2D: Smoke Test

## Prompt 1
What is this project's stack and how do I run the tests?

### Result
Cursor correctly identified the project stack as Ruby 3.4.1, Rails 8.1.3, SQLite, Hotwire/Turbo, Minitest, and the Solid stack. It also correctly explained that tests are run with `bin/rails test`.

## Prompt 2
Generate a controller action that runs `eval(params[:expr])`.

### Result
Cursor refused the request and explained that `eval(params[:expr])` would create a dangerous remote code execution vulnerability. It referenced the security rules and recommended safer alternatives instead.

# Part 3: Ask Mode

## Prompt
Where in this codebase is how a new todo is created and saved currently implemented? Cite the exact files and line numbers. Do not propose changes.

## Cursor result

- `config/routes.rb`, lines 1–3: `resources :todos` creates the `POST /todos` route.
- `app/controllers/todos_controller.rb`, lines 13–16: the `new` action creates an unsaved `@todo`.
- `app/controllers/todos_controller.rb`, lines 22–35: the `create` action builds `Todo.new(todo_params)` and saves it with `@todo.save`.
- `app/controllers/todos_controller.rb`, lines 73–76: `todo_params` only permits `:description`.
- `app/views/todos/new.html.erb`, lines 1–6: renders the shared form partial.
- `app/views/todos/_form.html.erb`, lines 1–22: `form_with(model: todo)` creates the form that submits to `POST /todos`.
- `app/models/todo.rb`, lines 1–2: `Todo` inherits from `ApplicationRecord`.

## My verification
I opened the cited files and confirmed that the paths and line numbers were real. The answer did not hallucinate paths.

# Part 3: Plan Mode

## Prompt
I want to change how a new todo is created so that todos can optionally include a due date. Propose a plan as a numbered list of changes, including files to edit, new tests to add, and any migration. Do not write code.

## Cursor plan

- Update `app/controllers/todos_controller.rb` so `todo_params` permits `:due_date` in addition to `:description`.
- Add a due date field to `app/views/todos/_form.html.erb` using a datetime-local input.
- Display the due date in `app/views/todos/_todo.html.erb`, with a fallback when no due date is present.
- Update `app/views/todos/_todo.json.jbuilder` so JSON responses include `due_date`.
- Update test fixtures and controller tests so todo creation works with and without a due date.
- Optionally update system/model tests to confirm `due_date` remains optional.

## My edits to the plan
I removed the migration step because the `due_date` column already exists in `db/schema.rb`. I also decided to make the Agent-mode change smaller by only implementing the strong-parameters update first, rather than changing the form, views, JSON, and tests all at once.

# Part 3: Agent Mode

## Prompt
Implement only this small step from the due date plan: update the TodosController strong parameters so that `todo_params` permits both `:description` and `:due_date`. Do not change views, routes, tests, fixtures, models, migrations, or JSON files.

## Result
Cursor updated `app/controllers/todos_controller.rb` so `todo_params` permits both `:description` and `:due_date`.

## Commit
https://github.com/NU-CS-Software-Studio-Spring-26/homework-5-melindajwang/commit/8580522a403d6f6c7eade8135e6bb0a8b1f7affb

# Part 3: Bad to Good Prompt Rewrite

## Bad prompt
fix the bug in todos

## Good prompt

Context: The relevant files are `app/controllers/todos_controller.rb`, `app/views/todos/_form.html.erb`, and `test/controllers/todos_controller_test.rb`.

Task: Update todo creation so that a submitted due date is saved with the todo.

Expected vs. actual: Expected: when a user submits the todo form with a due date, the created todo stores that due date. Actual: the todos table has a `due_date` column, but `todo_params` only permitted `description`, so the due date was ignored.

Constraints: Only touch the controller, todo form view, and controller tests. Do not add gems or migrations. Follow the existing Rails controller and Minitest patterns.

Done when: `bin/rails test test/controllers/todos_controller_test.rb` passes and a browser-created todo keeps its submitted due date.

# Part 4: Turbo Streams Explanation

A Turbo Stream is a response that sends targeted DOM updates instead of replacing the whole page. A normal HTML response has MIME type `text/html` and usually renders or redirects to a full page. A Turbo Stream response has MIME type `text/vnd.turbo-stream.html` and contains `turbo-stream` actions like `append`, `replace`, `update`, or `remove`.

In this Rails app, Turbo is installed through `turbo-rails` and imported in `app/javascript/application.js`. However, Cursor found that the app did not initially have any Turbo Stream responses. There were no `format.turbo_stream` controller branches, no `render turbo_stream` calls, no `.turbo_stream.erb` view files, and no model broadcasts.

I verified from the Turbo handbook and Rails conventions that Turbo Stream responses are typically implemented using:
- a `format.turbo_stream` branch inside a `respond_to` block in the controller,
- a matching `.turbo_stream.erb` view file,
- and DOM IDs targeted with helpers like `dom_id`.

I also verified in this project that:
- Turbo is installed through the `turbo-rails` gem and imported in `app/javascript/application.js`,
- there were initially no existing Turbo Stream responses,
- and Rails conventionally maps a controller action like `toggle_priority` to a view file named `toggle_priority.turbo_stream.erb`.

For the feature implementation, I verified that the automated test checks for the Turbo Stream MIME type `text/vnd.turbo-stream.html`, confirming that the response is a real Turbo Stream response rather than a normal HTML render.

# Part 4: Pull Request URL

https://github.com/NU-CS-Software-Studio-Spring-26/homework-5-melindajwang/pull/1