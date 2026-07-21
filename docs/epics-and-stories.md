# Epics and Stories - TODO App: Due Dates, Priorities & Filters Upgrade

## MVP

- Epic: Task Due Dates
  - Story: Add optional due date field to tasks
    - Acceptance Criteria: A task can be created or edited with a `dueDate` field in `YYYY-MM-DD` format; the field is optional and tasks can be saved without it.
    - Technical Requirements: The `due_date` column already exists on the `tasks` table (packages/backend/src/app.js) and is already accepted/persisted by the `POST /api/tasks` and `PUT /api/tasks/:id` handlers; the `due-date-input` `type="date"` field already exists in packages/frontend/src/TaskForm.js and is included in the payload passed to `onSave`. No further changes required for this story.
  - Story: Validate due date format on input
    - Acceptance Criteria: When a `dueDate` is provided, it must conform to ISO `YYYY-MM-DD` format to be accepted.
    - Technical Requirements: Add a regex check (`/^\d{4}-\d{2}-\d{2}$/`) against `due_date` in the `POST /api/tasks` and `PUT /api/tasks/:id` handlers in packages/backend/src/app.js before the `INSERT`/`UPDATE` statements run.
  - Story: Ignore invalid due date values
    - Acceptance Criteria: If a `dueDate` value is malformed or invalid, it is ignored and the task is saved as if no due date was provided (no error is thrown to block saving).
    - Technical Requirements: In the same handlers in packages/backend/src/app.js, when the format check fails, pass `null` for `due_date` to the `stmt.run(...)` call instead of returning a 400, following the existing `description || ''` fallback pattern already used for optional fields.

- Epic: Task Priority
  - Story: Add priority field to tasks
    - Acceptance Criteria: A task can be created or edited with a `priority` field accepting one of `P1`, `P2`, or `P3`.
    - Technical Requirements: Add a `priority TEXT DEFAULT 'P3'` column to the `CREATE TABLE IF NOT EXISTS tasks` statement in packages/backend/src/app.js; update the `POST /api/tasks` and `PUT /api/tasks/:id` handlers' destructuring, `INSERT`/`UPDATE` SQL, and `stmt.run(...)` calls to include `priority`; add a priority `<TextField select>`/MUI `Select` control to packages/frontend/src/TaskForm.js and include its value in the object passed to `onSave` in `handleSubmit`.
  - Story: Default priority to P3 when not specified
    - Acceptance Criteria: If no `priority` value is provided when creating a task, the task is saved with `priority` set to `P3`.
    - Technical Requirements: Rely on the `DEFAULT 'P3'` column constraint added above, and apply a `priority || 'P3'` fallback in the `POST`/`PUT` handlers in packages/backend/src/app.js (mirroring the existing `due_date || null` pattern) so omitted values resolve to `'P3'` explicitly in the response payload.
  - Story: Validate priority is one of P1, P2, or P3
    - Acceptance Criteria: Any `priority` value other than `P1`, `P2`, or `P3` is rejected or not accepted as valid input.
    - Technical Requirements: In the `POST /api/tasks` and `PUT /api/tasks/:id` handlers in packages/backend/src/app.js, add a validation check (e.g., `!['P1','P2','P3'].includes(priority)`) that returns a 400 response with an error message, following the existing title-required validation pattern; constrain the frontend priority field in packages/frontend/src/TaskForm.js to a dropdown with only those three options so invalid values can't be entered client-side.

- Epic: Task Filtering
  - Story: Add "All" filter tab showing completed and incomplete tasks
    - Acceptance Criteria: Selecting the "All" tab displays every task regardless of completion status.
    - Technical Requirements: Add a MUI `Tabs`/`Tab` control to packages/frontend/src/TaskList.js with `All`, `Today`, `Overdue` values tracked in component state (e.g., `useState('all')`); when `All` is active, call the existing `fetchTasks()` function with no additional query parameters against `GET /api/tasks` in packages/backend/src/app.js.
  - Story: Add "Today" filter tab showing incomplete tasks due today
    - Acceptance Criteria: Selecting the "Today" tab displays only incomplete tasks whose `dueDate` equals the current date; completed tasks are excluded.
    - Technical Requirements: Extend the `buildTaskQuery` helper and `GET /api/tasks` handler in packages/backend/src/app.js to accept a `due=today` query parameter, adding `due_date = date('now')` and `completed = 0` clauses to the generated `WHERE`; update `fetchTasks` in packages/frontend/src/TaskList.js to include this parameter in the `fetch('/api/tasks?...')` call when the "Today" tab is selected.
  - Story: Add "Overdue" filter tab showing incomplete tasks past due date
    - Acceptance Criteria: Selecting the "Overdue" tab displays only incomplete tasks whose `dueDate` is earlier than the current date; completed tasks are excluded.
    - Technical Requirements: Extend the same `buildTaskQuery` helper and `GET /api/tasks` handler in packages/backend/src/app.js to accept a `due=overdue` query parameter, adding `due_date < date('now')` and `completed = 0` clauses; update `fetchTasks` in packages/frontend/src/TaskList.js to include this parameter when the "Overdue" tab is selected.

- Epic: Data Validation & Storage
  - Story: Enforce required title field
    - Acceptance Criteria: A task cannot be saved without a non-empty `title`.
    - Technical Requirements: No change required — `POST /api/tasks` and `PUT /api/tasks/:id` in packages/backend/src/app.js already return a 400 with `{ error: 'Task title is required' }` when `title` is missing/blank, and packages/frontend/src/TaskForm.js already blocks submission client-side via its `if (!title.trim())` check in `handleSubmit`.
  - Story: Persist tasks using local storage only
    - Acceptance Criteria: All task data (including `dueDate` and `priority`) is saved to and loaded from local storage, with no calls to a backend or external storage service.
    - Technical Requirements: Continue using the existing in-memory `better-sqlite3` database instantiated via `new Database(':memory:')` in packages/backend/src/app.js as the sole data store for the new `priority` and filtering features; do not introduce external databases, third-party APIs, or cloud storage.

## Post-MVP

- Epic: Overdue Task Highlighting
  - Story: Visually highlight overdue tasks in the task list
    - Acceptance Criteria: Incomplete tasks with a `dueDate` earlier than the current date are visually distinguished (e.g., highlighted in red) in the task list.
    - Technical Requirements: In packages/frontend/src/TaskList.js, compute an `isOverdue` flag per task inside the `tasks.map(...)` render (`!task.completed && task.due_date && task.due_date < todayString`), and extend the existing conditional `background`/`borderColor` logic on the `ListItem` `sx` prop (currently based only on `task.completed`) to apply a red-tinted style when `isOverdue` is true.

- Epic: Priority Badges
  - Story: Add color-coded badge for P1 priority
    - Acceptance Criteria: Tasks with `priority` set to `P1` display a red badge indicating the priority level.
    - Technical Requirements: In packages/frontend/src/TaskList.js, render the already-imported MUI `Chip` component next to `task.title` in `ListItemText`, mapping `priority` to a color via a lookup object (e.g., `{ P1: 'error', P2: 'warning', P3: 'default' }`) so `P1` renders with the `error` (red) color.
  - Story: Add color-coded badge for P2 priority
    - Acceptance Criteria: Tasks with `priority` set to `P2` display an orange badge indicating the priority level.
    - Technical Requirements: Use the same `Chip`/color-lookup implementation in packages/frontend/src/TaskList.js, mapping `P2` to the `warning` (orange) MUI color.
  - Story: Add color-coded badge for P3 priority
    - Acceptance Criteria: Tasks with `priority` set to `P3` display a gray badge indicating the priority level.
    - Technical Requirements: Use the same `Chip`/color-lookup implementation in packages/frontend/src/TaskList.js, mapping `P3` (and the default) to the `default` (gray) MUI color.

- Epic: Task Sorting
  - Story: Sort overdue tasks before other tasks
    - Acceptance Criteria: When displaying the task list, overdue incomplete tasks appear before all non-overdue tasks.
    - Technical Requirements: Replace the `ORDER BY due_date IS NULL, due_date ASC, created_at ASC` clause in the `GET /api/tasks` handler in packages/backend/src/app.js with a composite `ORDER BY` that ranks first via `CASE WHEN completed = 0 AND due_date < date('now') THEN 0 ELSE 1 END`.
  - Story: Sort tasks by priority (P1 to P3)
    - Acceptance Criteria: Within the same overdue/non-overdue grouping, tasks are ordered by priority from `P1` to `P3`.
    - Technical Requirements: Add a secondary `ORDER BY` term to the same SQL query in packages/backend/src/app.js using `CASE priority WHEN 'P1' THEN 1 WHEN 'P2' THEN 2 WHEN 'P3' THEN 3 ELSE 4 END`.
  - Story: Sort tasks by due date ascending
    - Acceptance Criteria: Within the same priority grouping, tasks are ordered by `dueDate` from earliest to latest.
    - Technical Requirements: Retain `due_date ASC` as the next term in the composite `ORDER BY` clause in the `GET /api/tasks` handler in packages/backend/src/app.js, after the overdue and priority ranking terms.
  - Story: Place tasks without a due date last
    - Acceptance Criteria: Tasks with no `dueDate` are sorted after all tasks that have a `dueDate`, regardless of priority.
    - Technical Requirements: Keep the existing `due_date IS NULL` term as the final tiebreaker in the composite `ORDER BY` clause in packages/backend/src/app.js so `NULL` due dates sort last within each priority group.
