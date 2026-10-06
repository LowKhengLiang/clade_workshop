# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file IT PMO Kanban board demo for a fictitious bank: everything lives in `index.html` (markup, `<style>`, `<script>`). There is no build, lint, or test tooling and no package manager. To run it, open the file in a browser (`open index.html`). A quick syntax check of the script: extract the `<script>` body to a temp `.js` file and run `node --check` on it.

## Hard constraints (from the original brief — do not violate)

- Vanilla HTML/CSS/JS only: no frameworks, bundlers, npm, CDNs, web fonts, or image files. Icons are Unicode/inline SVG; system font stack.
- No persistence of any kind (no localStorage, sessionStorage, IndexedDB, cookies). Refreshing resets the board to seed data; the header note says so.
- The only network call is the FormSubmit AJAX endpoint (`FORMSUBMIT_ENDPOINT`, top of the script). The email address must not appear anywhere else.
- CSS uses custom properties for palette/spacing, no `!important`. Visual style (revamped): kid-friendly "sticker" look, pastel palette (each pastel `--x` has a deeper `--x-deep` partner for outlines/non-text marks), 3px plum `--ink` borders, rounded shapes, emoji icons. Colour never carries meaning alone (text/icon accompanies it).
- Security: a CSP `<meta>` restricts the page to inline script/style and `connect-src https://formsubmit.co`; keep it in sync if the endpoint changes. Never put user strings into `innerHTML`/attributes without `escapeHtml()`; tooltips/toasts use `textContent`; filter specs from chart marks are whitelisted in `applyFilterSpec`; drops only act on the card being dragged.

## Architecture

- `state = { tasks, filters, confirmDeleteId, draggingId, nextId }` is the single source of truth. All card/board DOM is produced by `renderBoard()` → `renderCard()` from state; never mutate card contents directly. Transient UI (which card shows "Delete? Yes/No", which is being dragged) is also kept in `state` so a re-render preserves it.
- Dashboard: `renderDashboard()` (called from `renderBoard()`) builds KPI tiles and four charts (status donut SVG; project stacked bars, priority bars, due-date columns in HTML/CSS) from state. Tableau-style cross-filtering: marks carry `data-filter="dim=value;…"`, clicking toggles `state.filters` (project/status/priority/due/assignee); each chart calls `applyFilters(tasks, skipDims)` skipping its own dimensions so selected marks highlight and siblings dim. Hover/focus tooltips are delegated on `.dashboard`. KPI counts use the full `state.tasks`.
- Mutations go through `addTask`, `moveTask`, `deleteTask`, each of which re-renders. Filtering (`applyFilters`) only affects what the columns show; the summary strip counts always use the full `state.tasks`.
- Events are delegated on `#board` (click for delete confirm, `change` for the keyboard "Move ▸" select, HTML5 drag events for DnD). After a re-render replaces the focused element, `refocus()` restores keyboard focus.
- Add Task flow (`handleSubmit`): `validateForm` → `addTask` (optimistic) → `notifyNewTask` (FormSubmit, in try/catch so failure never breaks the board) → success or warning toast → reset form and close modal. The submit button is disabled with "Sending…" during the request.
- Task IDs are `UOB-ITPM-####` from `state.nextId`; seed tasks consume the first 8. Seed due dates are relative to today (`isoDate(offset)`) so overdue examples always exist.
- Overdue = `dueDate < today` (local-date ISO string comparison) and status ≠ Done.
- All user-supplied strings must pass through `escapeHtml()` before going into `innerHTML`; toasts use `textContent`.
- Form fields are read via `document.getElementById("t-…")`, not `form.title` etc. (names like `title` collide with built-in element properties).
