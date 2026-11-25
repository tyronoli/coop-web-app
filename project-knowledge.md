# Project Knowledge: Golden Egg Yolk Cooperatives Management System

> **How to use this file:** it is part of the `coop-web-app` skill in Gemini, together with `TASKS.md`. You also attach `STATUS.md` at the start of every chat. This file is the source of truth for scope, rules, stack, security and data model (it rarely changes). `TASKS.md` is the full task checklist. `STATUS.md` says what is done and what comes next (it changes every session). Always follow all three. If a request conflicts with them, say so and ask before deviating.
>
> **Working in a Gemini chat:** the assistant here cannot run code, install packages, edit the user's folders or run git. So it should (1) give **complete, copy-paste-ready files** with the full path above each one, (2) say which commands to run and what output to expect, (3) work **one task at a time** from `TASKS.md`, (4) after each step or finished task, show the updated checkbox lines and the exact git commands (from "Git: finishing a task" in `TASKS.md`, filled in with the task ID, branch name and commit message), and (5) at the end of each session, give a complete updated `STATUS.md` for the user to save. Chats do not share memory, so anything that must carry over belongs in `STATUS.md`.

---

## Overview

Web-based cooperative management system for **Golden Egg Yolk Cooperatives**. It handles membership, share capital, loans, collections, accounting, dividends and patronage refunds, documents, reports and administration, all on one central database with role-based access. Members can log in themselves through a **member portal** (v1).

Reference design: the Sikhay Yaman Cooperative Management System flow chart and UI board. We follow its structure, **minus the Agriculture module**.

**Scope**
- **v1:** everything below, including the member portal.
- **v2 (planned, do not build now):** Agriculture module (Farmer Registration, Farm Records, Production, Procurement, Inventory, Sales/Buyers, Payments, Logistics) and other member services.
- Design v1 so v2 plugs in without rewrites: one Django app per module, a module registry and feature flags (see "Extensibility"), and no agriculture-specific assumptions in core models.

**Task workflow:** work is done one task at a time from `TASKS.md`. Read it first, do only the requested task, check its dependencies and Definition of Done. **Every commit ticks the `TASKS.md` checkboxes it covers, in the same commit**, and the commit message starts with the task ID (for example `T-013: add scoped throttles`). Only tick a box when the step is finished and its tests pass. Use one branch per task (`task/T-XXX-short-name`). **When a task is finished, follow "Git: finishing a task" in `TASKS.md`:** run the tests and lint, commit, push the task branch and open a pull request. Never push to `main`, never force push, and never commit `.env`, dumps, uploaded member files or secrets.

**Golden rule: nothing business-specific is hard-coded.** Interest rates, loan products, fees, terms, eligibility rules, share capital rules, dividend rates, number formats and similar values are set by an admin at runtime. See "Admin-configurable settings".

Tagline: *Stronger Together, Growing Better.* (placeholder, confirm with the cooperative)
Core values shown on the emblem: **Unity, Integrity, Sustainability, Prosperity**.

---

## Tech stack

| Layer | Choice |
|---|---|
| Frontend | React + TypeScript (Vite), React Router, TanStack Query, React Hook Form + Zod |
| Backend | Python 3.12+, Django 5, Django REST Framework |
| Database | PostgreSQL 16 (hosted on **Supabase** as plain Postgres, see "Hosting") |
| Auth | Django auth with OTP (TOTP/email); session or JWT in httpOnly cookies (see Security). Supabase Auth is **not** used. |
| Background jobs | Celery + Redis (reports, notifications, scheduled jobs such as due-date reminders and delinquency aging) |
| Cache / rate-limit store | Redis (a managed Redis from the backend host); the DB cache backend is an acceptable fallback early on |
| Frontend hosting | **Netlify** (static React build) |
| Backend hosting | Not decided yet, see "Hosting". Keep the backend containerized and configured only through environment variables so it can move anywhere. |
| Local dev | Docker Compose (Django, Postgres, Redis, frontend) |

Assume mobile-first. The UI board shows phone screens with a bottom nav (Dashboard, Members, Loans, Collections, More), and the same app must work on desktop.

---

## Suggested repo layout

```
/
├── docker-compose.yml
├── backend/
│   ├── manage.py
│   ├── config/              # settings (base/dev/prod), urls, celery
│   └── apps/
│       ├── accounts/        # users, roles, OTP, login throttling
│       ├── configuration/   # admin-managed settings, loan products, fees, rules (see below)
│       ├── members/         # membership, applications, profiles
│       ├── share_capital/
│       ├── loans/
│       ├── collections/     # payments and official receipts
│       ├── accounting/      # chart of accounts, journal, ledger, statements
│       ├── dividends/       # dividends and patronage refund
│       ├── documents/       # certificates, agreements, clearances (PDF)
│       ├── reports/
│       ├── audit/           # audit trail, activity logs
│       └── core/            # shared base models, permissions, utils
└── frontend/
    └── src/
        ├── api/             # typed API client
        ├── components/
        ├── features/        # one folder per module below (staff app + member portal)
        ├── routes/
        └── types/
# v2 adds backend/apps/agriculture/ and frontend/src/features/agriculture/
```

---

## Modules (main windows and sub-windows)

Numbering follows the reference, with Agriculture deferred to v2 so Administration becomes #11. The Member Portal is added as #12.

1. **Login & Security**: Login, Forgot Password, OTP Verification, Change Password.
2. **Main Dashboard**: member overview, summary cards (total members, paid-up share capital, loan portfolio, collections this month, outstanding loans, delinquent loans, cooperative fund, available loan fund), recent notifications, module shortcuts.
3. **Membership**: New Member Registration, Member List, Member Approval, Member Profile, Membership Certificate, Member Documents.
4. **Share Capital**: Share Capital Dashboard, New Contribution, Capital Build-up, Share Certificate, Capital Transaction History, Member Capital Statement, Redemption (if any).
5. **Loans**: Loan Dashboard, New Loan Application (pick any active admin-defined loan product), Eligibility Check / Loan Evaluation (driven by the product's rules), Loan Approval, Loan Release, Loan Statement, Amortization Schedule, Loan Payment, Delinquent Loans, Loan History.
6. **Collections**: Collection Dashboard, Record Payment, Official Receipt, Payment History, Due Loans, Delinquency List.
7. **Accounting**: Cash Receipts, General Ledger, Journal Entries, Trial Balance, Financial Statements (Balance Sheet, Income Statement, Cash Flow), Chart of Accounts.
8. **Dividends & Patronage**: Dividend Computation, Patronage Refund, Member Statement, History.
9. **Documents**: Membership Certificate, Share Certificate, Loan Certificate, Loan Agreements, Loan Applications, Amortization Schedule, Payment Receipts, Statements & Clearances, Certificate of Good Standing.
10. **Reports**: Report Menu and Generate Report. Membership, Share Capital, Loan Portfolio (with aging: current, 1-30, 31-60, over 60 days), Collection, Delinquency, Financial, and Management reports.
11. **Administration**: User Management, Roles & Permissions, Cooperative Settings, **Configuration Center** (Loan Products, Fees & Charges, Eligibility Rules, Share Capital Rules, Dividend & Patronage Rules, Number Formats, Aging Buckets, Notification Templates), Audit Trail, Backup, System Security.
12. **Member Portal** (self-service, role `Member`): Login and OTP, My Profile (view, request updates), My Share Capital (balance, transactions, statement), My Loans (status of applications and active loans, balance, amortization schedule, payment history, next due date), Apply for a Loan, My Documents (certificates, statements), Notifications, and a "Services" area where future services appear (v2 Agriculture). Members see only their own data. Profile changes by members go to a staff approval queue before they apply.

### Cross-cutting features (available in every module)
Member QR code, notifications, search and filters, audit trail, data security, backup and recovery, user activity logs.

---

## Key workflows

- **Membership:** Applicant submits online application → Membership Officer verifies → Approval → Member account created (member ID like `GEYC-000001`).
- **Loan process:** Application (member or staff, against an active loan product) → Eligibility check (product rules) → Loan Officer review → Loan Committee → Approval → Disbursement (release). The loan stores a snapshot of the product terms at approval.
- **Payment process:** Payment → Official Receipt → Update loan balance → Next payment due.
- **Accounting flow:** Transactions (share, loan, payment, expenses, etc.) → Accounting records (auto-posted journal entries) → Reports.
- **System integration:** Members → Transactions (share/loan/payment) → Accounting → Reports and Dashboards → Management decisions.

---

## User roles

Member, Membership Officer, Secretary, Loan Officer, Loan Committee, Cashier, Treasurer, Accountant, Manager, Board Member, System Administrator.

Enforce roles on the backend (DRF permission classes), never only in the UI. Roles map to **permissions stored in the database**, so an admin can adjust what each role can do in Roles & Permissions. Seed the defaults below, to be confirmed with the cooperative:

| Role | Can do |
|---|---|
| Member | Member portal: view own profile, capital, loans (status, balance, schedule), payments, statements, documents; apply for a loan; request profile updates |
| Membership Officer | Register and verify members, manage member documents |
| Secretary | Documents, certificates, meeting records, member records (read) |
| Loan Officer | Take applications, run eligibility checks, prepare loans, view delinquency |
| Loan Committee | Approve or reject loans |
| Cashier | Record payments and capital contributions, issue official receipts |
| Treasurer | Review collections, cash position, approve releases, view financial reports |
| Accountant | Journal entries, ledger, financial statements, dividend computation |
| Manager | All operational read access, reports, dashboards |
| Board Member | Read-only dashboards and reports, approvals where assigned |
| System Administrator | Users, roles, settings, backups, audit logs (no financial posting by default) |

Separation of duties: the person who records a transaction cannot be the one who approves or voids it.

---

## Core data model (starting point)

`User`, `Role`, `Permission`, `Member` (profile, status, member ID, QR token), `MembershipApplication`, `ProfileChangeRequest`, `MemberDocument`, `ShareCapitalAccount`, `CapitalTransaction`, `ShareCertificate`, `LoanProduct` (+ `LoanProductVersion`), `FeeCharge`, `EligibilityRule`, `ConfigSetting` (+ history), `NumberSequence`, `NotificationTemplate`, `LoanApplication`, `Loan` (stores a snapshot of the approved product version), `AmortizationSchedule` (rows), `LoanPayment`, `OfficialReceipt` (with unique OR number series), `Account` (chart of accounts), `JournalEntry`, `JournalLine`, `FiscalPeriod`, `DividendRun`, `DividendAllocation`, `PatronageRefund`, `Document`, `Notification`, `AuditLog`, `ActivityLog`, `CooperativeSettings`.

Money rules (important):
- Use `DecimalField` / `Decimal` for all money. Never floats. Currency is PHP (₱).
- Financial records are **append-only**. Corrections are reversing entries, not edits or deletes. Use soft-void with reason, actor and timestamp.
- Accounting is **double-entry**. Every posted transaction creates balanced journal lines (debits = credits, enforced in the DB layer and in service code).
- Wrap posting logic in `transaction.atomic()` and use `select_for_update()` on balances and loans to prevent double-posting under concurrency.
- Official receipt and certificate numbers are generated server-side, sequential and unique.
- Keep business logic in service modules (`services.py`), not in views or serializers.

---

## Admin-configurable settings (fully dynamic, nothing hard-coded)

Everything the cooperative may want to change later lives in the database and is edited by an admin in **Administration → Configuration Center**. Code contains only the *mechanisms* (how interest is computed, how rules are evaluated), never the *values*. No rate, term, fee, limit, threshold, label list or number format may appear as a constant in backend or frontend code. Migrations may seed sensible **example defaults** (clearly marked as editable), but the app must work with whatever the admin sets.

### Loan products
The admin creates, edits, activates, deactivates and retires any number of loan products (for example Regular, Emergency, Salary, Business: names are the admin's choice). Each product defines:
- name, code, description, active flag, availability dates
- **interest rate** and **rate basis** (per month or per annum), and **interest method** chosen from the computation methods the code supports (flat, diminishing balance, and others added later as new strategy classes)
- min and max amount, min and max term, allowed **payment frequencies** (weekly, semi-monthly, monthly, and so on)
- **fees and charges** (service fee, processing fee, insurance, notarial, capital build-up deduction): each as a fixed amount or a percentage, with when it is charged (upfront, spread, at release)
- **penalty rules**: late rate or fixed amount, grace period in days, compounding on or off
- **eligibility rules** (see below), **required documents**, **number of co-makers or collateral requirements**
- **approval workflow**: which roles approve, and in what order, and amount thresholds that require extra approval (for example above a set amount the Board must approve)
- GL accounts the product posts to (so accounting follows the config)

### Other admin-managed settings
- **Share capital:** par value, minimum initial subscription, minimum periodic contribution and frequency, redemption rules and notice period, certificate numbering
- **Membership:** membership types, membership fee, required documents, minimum age, member ID format
- **Eligibility rules:** a rule builder over named fields (for example months as member, paid-up share capital, loan-to-capital multiple, maximum active loans, no delinquent loans, minimum net take-home). Each rule is data: field, operator, value, message shown to the user
- **Dividends and patronage:** dividend rate, patronage refund rate or formula, fiscal year start and end, computation basis (for example average capital balance), reserve and statutory allocations
- **Collections and accounting:** accepted payment methods, payment allocation order (penalty → interest → principal, configurable), chart of accounts, fiscal periods, receipt and voucher number formats and series
- **Delinquency:** aging buckets and day ranges, when a loan counts as delinquent
- **System:** cooperative profile (name, address, registration numbers, logo), contact details, notification templates and channels, session timeout, password policy, OTP settings, rate-limit values, document templates (certificates, receipts, agreements)
- **Roles and permissions:** editable (see User roles)

### Rules for implementing configuration
- **Versioning and effective dates:** a change never rewrites history. Editing a product or rule creates a new version with an effective date. A loan **snapshots the exact product version and computed terms at approval**, so later edits only affect *new* loans. Show the admin a clear "applies to new loans only" notice.
- **Validation:** server-side validation of every setting (ranges, min ≤ max, rates ≥ 0, required fields, no overlapping aging buckets). Reject unsafe values, for example an interest rate outside a configured sanity ceiling.
- **Permissions and approval:** only roles with the configuration permission can edit. Optionally require a second approver (maker-checker) for changes to rates and fees.
- **Audit:** every config change is written to the audit trail with old and new values, actor, time, IP and reason.
- **Delivery to the UI:** the frontend reads products, fees, labels and limits from the API (cached with TanStack Query) and renders forms from that data. Loan forms, calculators and amortization previews must call the backend calculation service, never duplicate formulas in the client.
- **Calculation engine:** a single backend service computes amortization, fees, penalties and eligibility from the product version. It is pure and deterministic given (product version, inputs), so it is easy to unit test and to preview from the admin screen ("simulate a ₱20,000 loan over 6 months").
- **Seeding and environments:** seed data lives in a data migration or fixture, never in application code paths. Settings are exportable and importable as JSON so they can be promoted from staging to production.
- **Safe defaults:** if a required setting is missing, fail closed with a clear admin-facing error (for example "no active loan products"), never fall back to a hidden hard-coded value.
- Performance: cache settings in Redis or in-process with invalidation on change.

---

## Extensibility (v2 Agriculture and future services)

- Each module is a self-contained Django app with its own models, services, API routes and permissions. Core apps (`accounts`, `members`, `accounting`, `audit`, `configuration`, `core`) must not import from feature apps.
- Keep a **module registry** in the database (module key, name, enabled flag, required permission, icon, route). The staff menu and the member portal "Services" area are built from it, so enabling Agriculture in v2 adds a menu entry without changing the shell.
- Use **feature flags** per module so unfinished v2 work can ship dark.
- Other modules post to accounting only through the accounting service (journal entries), so Agriculture sales and payments can reuse the same ledger.
- Agriculture's data (farmers are members, production, inventory) must link to `Member`; do not add agriculture fields to `Member`.

---

## Hosting (decide fully after the project is built; constraints known now)

Current intent: **Supabase** for the database and **Netlify** for the frontend. Important constraints found while checking:

- **Netlify cannot run the Django backend.** Netlify hosts static sites and short serverless functions, not long-running Python/Django servers (see [Netlify support guide](https://answers.netlify.com/t/support-guide-can-i-run-a-web-server-http-listener-and-or-database-at-netlify/3078)). It will host the React build only.
- **Supabase provides the Postgres database, not a place to run Django.** So the Django API, plus Celery workers and Redis, need a separate backend host (a container host or VPS). Choosing that host is part of the later hosting decision. Keep the backend Dockerized and 12-factor so any host works.
- **Use Supabase only as Postgres.** Django owns users, sessions and permissions. Do not use Supabase Auth or the client-side Supabase SDK from the frontend. All data access goes through the Django API.
- **Lock down Supabase's auto-generated Data API.** Supabase can expose tables over its REST API using a public key. Enable Row Level Security on every table (or disable the Data API / keep app tables out of exposed schemas) and verify the current project settings, so no table is reachable except through Django. See [Supabase: Securing your API](https://supabase.com/docs/guides/api/securing-your-api). Treat this as a launch blocker.
- **Connections:** use Supabase's direct or session-mode connection for migrations. If the app uses the transaction-mode pooler, set Django's `DISABLE_SERVER_SIDE_CURSORS = True` and mind `CONN_MAX_AGE`. See [Supabase Supavisor FAQ](https://supabase.com/docs/guides/troubleshooting/supavisor-faq-YyP5tI). Keep `sslmode=require`. Pick the Supabase region closest to the Philippines and put the backend host in the same region to keep latency low.
- **Redis:** Supabase does not provide it. Use the backend host's managed Redis, or the database cache backend temporarily (slower, fine at small scale) until then.
- **Cookies across domains:** if the Netlify site and the API sit on unrelated domains, browsers block third-party cookies and break `SameSite` sessions. Prefer a custom domain with `app.<domain>` for the frontend and `api.<domain>` for the backend (same site), or proxy `/api/*` through a Netlify rewrite so requests are same-origin. Set `CSRF_TRUSTED_ORIGINS`, CORS allow-list and secure cookie flags accordingly.
- **Netlify config:** SPA fallback redirect to `index.html`, security headers (CSP, HSTS and others) in `netlify.toml`, build-time env vars for the API base URL only, **no secrets in the frontend build**.
- **Backups:** confirm the Supabase plan's backup and point-in-time recovery options. Free-tier projects have limited backups and may pause when inactive. Do not run a live cooperative on a free tier. Keep our own scheduled `pg_dump` backups as well.
- **Data privacy:** member financial and personal data hosted outside the Philippines still falls under RA 10173. Choose region and processor agreements with that in mind.
- All hosting details are **environment variables** (`DATABASE_URL`, `REDIS_URL`, `ALLOWED_HOSTS`, `CORS_ALLOWED_ORIGINS`, `CSRF_TRUSTED_ORIGINS`, `SECRET_KEY`, email settings). Nothing host-specific in code.

---

## Security requirements

Treat this as a financial system holding personal data (Philippine Data Privacy Act of 2012 / RA 10173 applies). Security is a requirement of every feature, not a later phase.

### Rate limiting and brute-force protection
- Use DRF throttling (`AnonRateThrottle`, `UserRateThrottle`, and `ScopedRateThrottle` for sensitive endpoints), backed by Redis so limits hold across workers. Add Nginx `limit_req` as an outer layer.
- Rate-limit values are **admin-configurable settings** like everything else (stored in the Configuration Center with safe floor and ceiling values); the numbers below are only the seeded defaults.
- Suggested starting limits (tune later):
  - Login: 5 attempts/min per IP and per username
  - OTP verify / request: 5 per 10 min per account
  - Forgot password: 3 per hour per account and per IP
  - Public registration / application: 5 per hour per IP
  - Authenticated API default: 120/min per user
  - Report generation and PDF export: 10/min per user
- Account lockout after repeated failures (for example 5 failures → 15-minute lock, with progressive backoff) using `django-axes` or equivalent. Return generic error messages so usernames cannot be enumerated.
- Return `429` with a `Retry-After` header, and log every throttled or locked event.
- Optionally add CAPTCHA on public forms.

### Authentication and sessions
- Passwords hashed with Argon2 (`argon2-cffi`); enforce Django password validators and a minimum length of 12.
- OTP / 2FA for login (staff roles mandatory, members optional or email-OTP). Short-lived, single-use, hashed OTP codes.
- Prefer httpOnly + Secure + SameSite cookies for session or JWT. Do **not** store tokens in `localStorage`. Use short access lifetimes, rotate and blacklist refresh tokens, and idle timeout (for example 15 min for staff).
- Forced password change on first login and after admin reset.

### API and application hardening
- Deny by default: every endpoint requires authentication and an explicit permission class. Add object-level checks so a member can only read their own records. Member portal endpoints are scoped server-side to `request.user`'s member record; never accept a member ID from the client for "my" data. Test for IDOR (changing an ID in the URL must not reveal another member's data).
- Validate all input with DRF serializers (and Zod on the client for UX only). Use the ORM or parameterized queries only, with no raw string-built SQL.
- Secure headers: HSTS, CSP, `X-Content-Type-Options`, `Referrer-Policy`, `X-Frame-Options`. Strict CORS allow-list, CSRF protection on cookie-authenticated requests.
- `DEBUG=False` and a strict `ALLOWED_HOSTS` in production. Secrets only from environment variables, never in git. Provide `.env.example`.
- File uploads: allow-list types, enforce size limits, check content type, randomize filenames, store outside the web root or in private object storage, serve via authorized endpoints.
- Pagination on every list endpoint with a maximum page size, to prevent bulk scraping.
- Escape output; never render user-supplied HTML. Sanitize filenames and anything that goes into PDFs.

### Data protection
- HTTPS everywhere. Database access over a private network, with a least-privilege DB user for the app (no superuser), and migrations run with a separate role.
- Encrypt sensitive fields at rest where practical (government IDs, TIN, contact details) and mask them in the UI and logs. Never log passwords, OTPs, tokens or full ID numbers.
- Automated encrypted backups (daily, tested restore), with retention policy and documented recovery steps.

### Auditability
- **Audit trail** records who, what, when, from which IP, and before/after values for create/update/void/approve actions on every financial and membership record. Use `django-simple-history` or a custom audit app. Audit records cannot be edited or deleted through the app.
- **User activity log** records logins, failed logins, permission changes, exports and report generation.
- Dependency scanning (`pip-audit`, `npm audit`) in CI; pin versions.

---

## UI and branding

- Emblem: circular seal with a golden egg yolk in a cracked shell on a nest, hen, rice stalks, lettuce rows and a greenhouse, with a mountain and sunrise behind. Gold ring border, dark green banner.
- Palette (approximate, sampled by eye from the emblem and reference board; confirm exact values):
  - Primary: deep forest green (`~#14532D`) with mid green (`~#2E7D32`) for buttons and headers
  - Accent: golden yellow (`~#F2B705`) for highlights and key numbers
  - Background: warm off-white (`~#FAF8F2`); text: near-black green-grey
  - Status: green = active/paid, amber = pending, red = delinquent/overdue
- Reference UI patterns (from the UI board): green top app bar with title and bell icon; card-based dashboard with summary tiles; menu-style module landing pages (icon + title + one-line description + chevron); bottom tab bar on mobile; forms with labelled fields, Cancel/Next or Cancel/Save; official receipt preview layout with logo and amount in words; donut chart for loan aging.
- Peso amounts formatted `₱1,234.56`. Dates in a consistent local format.
- Accessibility: keyboard navigable, visible focus, sufficient contrast, labelled inputs.
- Brand assets to add: `frontend/src/assets/logo.png` (the emblem).

---

## Coding conventions

- **Backend:** PEP 8, `ruff` + `black`, type hints, `pytest` + `pytest-django`. Thin views, logic in services, one app per module. Every model has `created_at`, `updated_at`, `created_by`. API versioned under `/api/v1/`.
- **Frontend:** strict TypeScript (no `any`), ESLint + Prettier, one folder per feature, API types shared or generated from the OpenAPI schema (`drf-spectacular`). Handle loading, empty and error states on every screen.
- **Tests:** must cover loan amortization math for every supported interest method, payment allocation, balanced journal posting, permissions per role, member-portal data isolation (IDOR), throttling/lockout behavior, and configuration behavior (changing a product creates a new version, existing loans keep their snapshot, invalid settings are rejected, missing settings fail closed).
- Small, focused commits. Do not commit `.env`, database dumps or uploaded member files.

### Commands (fill in as the project is created)
```
# backend
python manage.py runserver
python manage.py migrate
pytest

# frontend
npm run dev
npm run build
npm run lint
```

---

## Suggested build order

1. Project scaffold: Docker Compose, Django + DRF, Postgres, Redis, React + TS, CI.
2. Accounts: custom user, roles and DB-stored permissions, login, OTP, throttling and lockout, audit/activity logging.
3. **Configuration Center foundation:** versioned settings model, validation, audit, admin API and screens, module registry. Build this before the modules that depend on it.
4. Members: application → verification → approval → member account; profile and certificate.
5. Share Capital: contributions, statements, certificate (driven by config).
6. Accounting core: chart of accounts, double-entry journal, ledger (so later modules post into it).
7. Loans: loan products admin, calculation engine, application, eligibility, approval, release, amortization.
8. Collections: payments, official receipts, loan balance updates, delinquency aging.
9. **Member portal:** member login, profile, capital, loans and status, apply for a loan, documents, notifications.
10. Dashboard and Reports.
11. Dividends and patronage refund.
12. Documents (PDF generation) and remaining Administration screens.
13. Hardening pass: security review, Supabase Data API lockdown check, backup/restore test, load test the rate limits.
14. Hosting setup (Netlify frontend, Supabase database, backend host) and go-live.

**v2:** Agriculture module and other member services.

---

## Open questions (confirm with the cooperative before building)

Decided: the name is **Golden Egg Yolk Cooperatives**, members have a self-service portal in v1, Agriculture is v2, loan products and all rates and rules are admin-configurable, and hosting intent is Supabase (database) + Netlify (frontend).

Still open:
- **Logo:** the emblem still reads "Cavite Golden Yolk Agriculture Cooperative". Will it be updated to the new name?
- Which **interest methods** must the engine support at launch (flat, diminishing balance, others)? The admin picks from the supported list, so the code needs each method implemented.
- Example loan products and values to seed as editable defaults (or start empty).
- Does config editing need **maker-checker** (a second person approves rate and fee changes)?
- Dividend and patronage refund formulas (the engine needs the formula types), and the fiscal year.
- Required regulatory reports (e.g. CDA) and document templates.
- Member onboarding: do members self-register online (then get verified), or does staff create their portal accounts?
- Which **backend host** will run Django, Celery and Redis (to be decided at hosting time), and the expected number of members and users.
- Email/SMS provider for OTP and notifications.
