# IT PMO Project Board

A single-file Kanban board demo for an IT PMO team at a fictitious bank. Built with vanilla HTML, CSS and JavaScript, with no dependencies.

**Live demo:** https://LowKhengLiang.github.io/clade_workshop/

![IT PMO Project Board screenshot](docs/screenshot.png)

## Features

- Four columns: Backlog, In Progress, Blocked, Done.
- Drag and drop cards between columns, or use the keyboard-accessible **Move ▸** menu on each card.
- Filter by project, assignee and priority. The summary strip always counts all tasks, regardless of filters.
- Overdue tasks (due date before today and not Done) are highlighted. Seed due dates are relative to today, so overdue examples always exist.
- Add Task form with validation. Each new task gets an ID like `UOB-ITPM-####` and triggers an email notification through [FormSubmit](https://formsubmit.co). If the email fails, the task is still added and a warning toast is shown.
- Delete with an inline "Delete? Yes/No" confirmation.

## Run locally

```sh
open index.html
```

## Notes

- Everything lives in `index.html` (markup, styles, script). There is no build step, bundler or package manager.
- Nothing is persisted (no localStorage, cookies or IndexedDB). Refreshing the page resets the board to its seed data.
- The only network call is the FormSubmit AJAX request. Set `FORMSUBMIT_ENDPOINT` at the top of the script to your own endpoint. Prefer FormSubmit's random-string alias over a raw email address, since the page source is public.

## Deployment

Pushes to `main` run `.github/workflows/pages.yml`: the `validate` job syntax-checks the script and enforces the project constraints, then the `deploy` job publishes `index.html` to GitHub Pages.
