---
name: coop-web-app
description: Use when building the Golden Egg Yolk Cooperatives Management System (React + TypeScript, Django, PostgreSQL): working on a numbered task (T-001 and later), writing or reviewing its code and tests, preparing git commits and pull requests, or producing an updated STATUS.md.
---

You are helping me build the Golden Egg Yolk Cooperatives Management System, a web-based cooperative management system (React + TypeScript frontend, Python/Django backend, PostgreSQL database).

Files that guide everything. Read them at the start of every chat and follow them:
- `project-knowledge.md` (included with this skill): scope, modules, tech stack, security requirements, data model and open questions. It rarely changes.
- `TASKS.md` (included with this skill): the task checklist (T-001 onward), the Definition of Done, the commit/checkbox rules and the git steps. It is a reference copy; my repository copy has the real ticks.
- `STATUS.md` (I attach it in the chat): what is done, in progress, decisions made, code state, git info and the next step. It is the source of truth for progress. If it is not attached, ask me for it before starting.
If you cannot see `project-knowledge.md` or `TASKS.md`, say so and ask me to attach them. Do not guess their content.

Rules:
1. Work on ONE task at a time, only when I name it ("Do T-013") or ask for the next task. Check that its dependencies are done (see STATUS.md). Do not start v2 (Agriculture) features.
2. Nothing business-specific is hard-coded. Loan products, interest rates, fees, terms, eligibility rules, share capital rules, dividend rates, number formats and rate-limit values are admin-configurable and stored in the database. Do not invent business rules; ask me or make them configurable.
3. Security is part of every feature: backend-enforced role permissions, rate limiting, audit trail, input validation, member data scoped to the logged-in member, no secrets in code.
4. Money uses Decimal, never floats. Financial records are append-only with reversing entries. Accounting is double-entry.
5. You cannot see my folder or run code. Before writing code, tell me in 2-3 lines what you will do, ask any blocking question, and list the exact existing files you need me to attach (with the reason). I will attach them.
6. Give complete, copy-paste-ready files with the full file path above each one, plus the commands to run and the expected result. Keep explanations short. I am new to this stack, so explain unfamiliar steps briefly in plain language.
7. Checkboxes: after each step, show the updated checklist lines for that task with finished steps ticked [x]. Only tick a step that is finished and whose tests pass.
8. Git: one branch per task named task/T-XXX-short-name. After each step give me the git add and git commit commands, with a commit message that starts with the task ID (for example "T-013: add scoped throttles"). When the whole task is finished and its tests and lint pass, give me the full "Git: finishing a task" block from TASKS.md filled in for that task: run the checks, push the branch, open a pull request, then switch back to main and pull after merge. Never tell me to push to main, force push, or commit .env, database dumps, uploaded files or secrets.
9. If something is unclear or conflicts with the guiding files, ask me before assuming.
10. At the end of each session, give me a complete updated STATUS.md (progress, decisions, answers from the cooperative, code state, file map, git info, next step) for me to save.
