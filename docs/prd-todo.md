# Product Requirements Document (PRD) - TODO App: Due Dates, Priorities & Filters Upgrade

## 1. Overview

We are upgrading the existing TODO app (currently limited to a `title` and `completed` status) to make task management more practical for users. This upgrade adds optional due dates, priority levels, and date-based filters so users can quickly identify what's urgent and organize their work without adding complexity like notifications, recurring tasks, or multi-user support. All data continues to be stored locally, with no backend or external storage changes required.

---

## 2. MVP Scope

- Add an optional `dueDate` field to each task
  - Format: ISO `YYYY-MM-DD`
  - Invalid values are ignored and treated as absent
- Add a `priority` field to each task
  - Enum: `P1 | P2 | P3`
  - Defaults to `P3` when not specified
- Add filter tabs for viewing tasks:
  - **All**: shows both completed and incomplete tasks
  - **Today**: shows only incomplete tasks due today
  - **Overdue**: shows only incomplete tasks past their due date
- Data validation:
  - `title` remains required
  - `priority` must be one of `P1`, `P2`, or `P3`
  - `dueDate` is optional; malformed values are ignored rather than rejected
- Continue using local storage only; no backend or external storage changes

---

## 3. Post-MVP Scope

- Visually highlight overdue tasks (e.g., red highlight) so they stand out in the list
- Color-coded priority badges (e.g., red for P1, orange for P2, gray for P3)
- Sorting rules applied to the task list:
  1. Overdue tasks first
  2. Then by priority (P1 → P2 → P3)
  3. Then by due date (ascending)
  4. Tasks without a due date sorted last

---

## 4. Out of Scope

- Notifications or reminders
- Recurring tasks
- Multi-user support
- Keyboard navigation / accessibility enhancements
- External or backend storage (app remains local-storage only)
