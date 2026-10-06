---
description: Security-scan, push to GitHub, and set up README, Pages, CI/CD and repo About
argument-hint: <github-repo-url>
allowed-tools: Bash, Read, Write, Edit, Grep, Glob
---

Publish this project to the GitHub repo at: $ARGUMENTS

If `$ARGUMENTS` is empty or not a GitHub URL (`https://github.com/<owner>/<repo>` or `git@github.com:<owner>/<repo>.git`), ask the user for it and stop. Parse `<owner>` and `<repo>` from it and reuse them below.

Respect the constraints in CLAUDE.md (vanilla HTML/CSS/JS, no build tooling, no persistence, no `!important`). Do not change app behaviour while doing this.

Work through the steps in this order. The security scan runs BEFORE anything is pushed, because pushed secrets are effectively public even if deleted later.

## 1. Preflight
- Run `gh auth status`. If not logged in, tell the user to run `gh auth login` and stop.
- Run `git status` and `git remote -v`. Note the current branch and any existing `origin`.

## 2. Security scan (blocking)
Scan everything that would be pushed: the working tree AND full git history (`git log -p --all` or `git grep` across `git rev-list --all`).

Look for:
- Secret files: `.env*`, `*.pem`, `*.key`, `id_rsa*`, `*.p12`, `credentials*`, `*.keystore`, `.DS_Store`, `.claude/settings.local.json`.
- Secret patterns: API keys/tokens (`AKIA[0-9A-Z]{16}`, `ghp_`, `gho_`, `github_pat_`, `xox[baprs]-`, `sk-`, `AIza`), `-----BEGIN .* PRIVATE KEY-----`, `password|passwd|secret|token|api[_-]?key` assignments with literal values, bearer tokens, connection strings with credentials.
- Personal data: email addresses, phone numbers, internal hostnames/IPs, absolute local paths such as `/Users/<name>`.
- Known exception for this repo: the FormSubmit email address is allowed ONLY inside `FORMSUBMIT_ENDPOINT` in `index.html`, but it is still visible to anyone viewing a public repo or the Pages site. Flag it to the user and recommend using FormSubmit's random-string alias endpoint (obtainable after the first activation email) instead of the raw address, and ask whether to proceed. Any other occurrence of that email address is a failure.

Handling:
- Ensure a `.gitignore` exists covering `.env*`, `*.pem`, `*.key`, `.DS_Store`, `.claude/settings.local.json`, `node_modules/`. Create or extend it.
- If anything sensitive is found in the working tree, remove it, add it to `.gitignore`, and report it.
- If anything sensitive is found in git HISTORY, STOP. Do not push. Tell the user what was found and where, and that history must be rewritten and any credential rotated. Do not rewrite history without explicit approval.
- Finish with a short report: what was scanned, what was found, what was fixed. Only continue if the result is clean or the user explicitly accepts the remaining items.

## 3. README
Create or update `README.md` (edit existing content rather than overwriting it). Base it on the actual code in `index.html`, and include:
- Title and one-line description (IT PMO Kanban board demo for a bank).
- Live demo link: `https://<owner>.github.io/<repo>/`.
- Features (Kanban columns, drag-and-drop plus keyboard "Move ▸", filters, summary strip, overdue highlighting, Add Task with email notification via FormSubmit).
- How to run locally (`open index.html`).
- Constraints/notes: single file, no dependencies, no persistence (refresh resets to seed data), only network call is FormSubmit.
- Deployment: GitHub Actions to GitHub Pages.
Do not put the notification email address in the README.

## 4. GitHub Actions CI/CD
Create or update `.github/workflows/` (an existing Pages deploy workflow may already be there; read it first and improve rather than duplicate). One workflow, triggered on push to the default branch and `workflow_dispatch`, with:
- A `validate` job: checkout, extract the `<script>` body from `index.html` and run `node --check`; fail if `localStorage`, `sessionStorage`, `indexedDB`, `document.cookie`, `!important`, or `http(s)://` CDN references (other than the FormSubmit endpoint) appear; a secret-pattern grep over the repo.
- A `deploy` job (needs `validate`): `actions/configure-pages`, `actions/upload-pages-artifact` (publish only `index.html`, not the whole repo), `actions/deploy-pages`.
- Permissions: `contents: read`, `pages: write`, `id-token: write`. Concurrency group `pages`. Pin actions to current major versions.

## 5. Commit and push
- If there is no `origin`, `git remote add origin <repo-url>`; if `origin` differs from the given URL, ask before changing it.
- Stage specific files (never `git add -A` blindly; re-check `git status` against the scan results), commit with a clear message ending with the Co-Authored-By line from the session's attribution rules, then `git push -u origin <branch>`.
- Never force-push. If the push is rejected, report why and ask.

## 6. GitHub Pages
- Enable Pages with the Actions source: `gh api -X POST repos/<owner>/<repo>/pages -f build_type=workflow` (if it already exists, use `-X PUT` instead).
- Watch the deploy: `gh run list --limit 1` then `gh run watch <id>`. If it fails, read `gh run view <id> --log-failed`, fix, and re-push.
- Confirm the URL with `gh api repos/<owner>/<repo>/pages --jq .html_url` and check it responds (`curl -sI`).

## 7. Repo About section
Set description, homepage (the Pages URL) and topics:
`gh repo edit <owner>/<repo> --description "IT PMO Kanban board demo for a bank: single-file vanilla HTML/CSS/JS, drag-and-drop, no dependencies" --homepage "<pages-url>" --add-topic kanban --add-topic pmo --add-topic vanilla-js --add-topic github-pages`
If an About description already exists, show it and keep it unless it is empty or clearly stale.

## 8. Final report
Give the user a short summary: repo URL, Pages URL, workflow status, security scan result, and anything they must do manually (for example, confirming the FormSubmit activation email, or a Pages setting that needed owner permissions).
