# Pesa Wallet

A personal finance web application for tracking money across several accounts at once.
Eric keeps money in a bank account, mobile money wallets and cash — Pesa Wallet pulls all
of those transactions into one ledger, classifies them, keeps spending inside budgets, and
produces reports for any period.

**Stack:** React + Vite + Tailwind CSS · Node.js + Express · SQLite

---

## 1. What to expect

### Sign-in and private wallets

- Register with a name, email and password, then sign in from `/login`.
- Passwords are hashed with scrypt and a random per-user salt; the API never stores or
  returns the plain password.
- Sessions are opaque random tokens in an `HttpOnly`, `SameSite=Lax` cookie. Only a SHA-256
  hash of each token is kept in the database, so a stolen database file cannot be replayed.
- Every account, category, transaction and budget belongs to the user who created it. All
  queries are filtered by the signed-in user, so two people on the same server never see each
  other's money. A brand new account starts with the default category tree and no transactions.
- Demo data has its own account, so registering shows an empty wallet.

### Accounts (bank, mobile money, cash, savings)

- Create as many accounts as you need, each with a type, institution, colour and opening balance.
- Balances are calculated live from the opening balance plus every transaction, so they never drift.
- Every account tracks total money in, money out and its transaction count.
- Deleting an account removes its transactions with it (with a clear confirmation first).

### Transactions

- Record income and expenses with an amount, date, account, description and optional category.
- Filter by free text (description, account or category), type, account, category (subcategories
  included automatically), date range, with sorting and pagination.
- Totals for the current filter are shown above the table, so "how much did I spend on transport
  between two dates?" is answered without exporting anything.
- Edit or delete any transaction; balances and reports update immediately.

### Categories and subcategories

- Two levels: a category such as **Food & Dining** holds **Groceries**, **Restaurants** and
  **Coffee & Snacks**.
- Separate income and expense trees.
- Every transaction can be linked to either a top level category or a subcategory, and each row
  shows "Food & Dining › Groceries" so the link is always visible.
- Categories expose transaction counts and totals; deleting a category keeps transaction history
  (the transaction becomes uncategorised).

### Budgets and notifications

- Set an **overall** monthly budget and/or a budget per expense category, for any month.
- Category budgets include every subcategory, so a "Food & Dining" budget covers groceries,
  restaurants and snacks.
- Each budget shows spent, remaining, a progress bar and a status: on track / close to the limit
  (80%+) / exceeded (100%+).
- Notifications when a limit is crossed:
  - The bell in the header lists every breached or nearly breached budget.
  - A banner appears on the dashboard and the budgets page.
  - The moment a transaction pushes a budget over its limit, a toast notification appears with the
    amounts — that check runs on the server when the transaction is saved.

### Reports

- Generate a report for any date range: this month, last month, last 30 days, last 3 months, this
  year, or a custom range.
- Includes income, expenses, net, average expense, cash flow per month, expenses by category
  (with subcategory breakdown and share bars), income by category, per-account opening/in/out/
  closing balances and the five largest expenses.
- **Export to CSV** or print the report.

### Visualisations

- Cash flow bar chart (income vs expenses, last 6 months).
- Spending-by-category donut with percentages.
- Account balance bars and budget progress bars.
- All money amounts are formatted in RWF.

---

## 2. Prerequisites

| Requirement | Version |
| --- | --- |
| Node.js | 20.19 or newer (built and tested on Node 24) |
| npm | 10 or newer |
| Build tools for `better-sqlite3` | Prebuilt binaries are downloaded automatically for Windows, macOS and Linux. If your platform has no prebuilt binary, a C++ toolchain (Visual Studio Build Tools / Xcode / `build-essential`) and Python 3 are needed. |

No database server is required — SQLite is embedded and the file is created automatically.

---

## 3. Getting started

```bash
# 1. Install dependencies for the API, the client and the root scripts
npm install

# 2. Start the API (port 4000) and the Vite dev server (port 5173) together
npm run dev
```

Open <http://localhost:5173>. You will land on the sign-in screen.

**Demo account:** `demo@pesawallet.com` / `demo1234` — or create your own account, which starts
with the default categories and an empty wallet.

On the first start the API creates `server/data/wallet.db`, applies the schema and seeds the demo
account with six months of realistic demo data (5 accounts, 45 categories, ~200 transactions and 6 budgets,
including one deliberately breached budget so the notification is visible). Delete
`server/data/wallet.db` to start over, or run `npm run seed` to reset the demo data.

### Other scripts

| Command | What it does |
| --- | --- |
| `npm run dev` | API with hot reload + Vite dev server |
| `npm run dev:api` | API only (`http://localhost:4000`) |
| `npm run dev:web` | Client only (`http://localhost:5173`, proxies `/api`) |
| `npm run build` | Production build of the React client into `client/dist` |
| `npm start` | Express serves the API **and** the built client on one port |
| `npm run seed` | Reset the SQLite database to the demo data set |

### Production run (single service)

```bash
npm install
npm run build
npm start          # http://localhost:4000 serves the whole app
```

---

## 4. Configuration

Copy `.env.example` to `.env` in the project root if you want to change defaults.

| Variable | Default | Purpose |
| --- | --- | --- |
| `PORT` | `4000` | Port the Express server listens on |
| `DATABASE_PATH` | `server/data/wallet.db` | SQLite file location |
| `AUTO_SEED` | `true` | Seed the demo account when the database is empty; set to `false` for an empty database |
| `SESSION_DAYS` | `30` | How long a sign-in stays valid |
| `CORS_ORIGIN` | all origins | Comma separated list of allowed origins for the API |
| `VITE_PROXY_TARGET` | `http://localhost:4000` | API target used by the Vite dev proxy |
| `VITE_API_BASE_URL` | `/api` | API base URL compiled into the client (only needed if the API is on another host) |

---

## 5. Project structure

```
wallet-web/
├── client/                     # React client
│   ├── src/
│   │   ├── components/
│   │   │   ├── accounts/       # account icon + form
│   │   │   ├── budgets/        # alert banner + budget form
│   │   │   ├── categories/     # category form
│   │   │   ├── charts/         # recharts wrappers
│   │   │   ├── layout/         # sidebar, topbar, app shell
│   │   │   ├── transactions/   # transaction form
│   │   │   └── ui/             # button, card, modal, fields, badges…
│   │   ├── context/            # signed-in user, wallet data, toast notifications
│   │   ├── hooks/              # API query + debounce hooks
│   │   ├── lib/                # API client, formatting, constants
│   │   └── pages/              # Login/Register + Dashboard, Transactions, Accounts, Categories, Budgets, Reports
│   └── vite.config.js
├── server/                     # Express API
│   └── src/
│       ├── db/                 # connection, schema.sql, default categories, demo seed
│       ├── lib/                # auth (scrypt + sessions), errors, zod validation schemas
│       ├── routes/             # auth, accounts, categories, transactions, budgets, reports
│       ├── services/           # category tree helpers + analytics queries
│       ├── app.js              # middleware, static hosting, error handling
│       └── index.js            # entrypoint
└── package.json                # npm workspaces + dev/build/start scripts
```

---

## 6. API reference

All responses are JSON. Success payloads are `{ "data": … }`; list endpoints add `meta`.
Errors are `{ "error": { "message": "…", "details": [{ "field": "…", "message": "…" }] } }`.

Every endpoint except `/api/health` and `/api/auth/*` requires a signed-in session and returns
401 otherwise. The session cookie is set by register/login.

| Method | Endpoint | Description |
| --- | --- | --- |
| GET | `/api/health` | Health check (public) |
| POST | `/api/auth/register` | Create an account (`name`, `email`, `password`) and start a session |
| POST | `/api/auth/login` | Sign in with `email` and `password` |
| POST | `/api/auth/logout` | End the current session |
| GET | `/api/auth/me` | The signed-in user |
| GET | `/api/accounts` | Accounts with live balances and transaction counts |
| POST | `/api/accounts` | Create an account |
| PUT | `/api/accounts/:id` | Update an account |
| DELETE | `/api/accounts/:id` | Delete an account and its transactions |
| GET | `/api/categories` | Category trees (`expense`, `income`) with usage totals |
| POST | `/api/categories` | Create a category or subcategory |
| PUT | `/api/categories/:id` | Update a category |
| DELETE | `/api/categories/:id` | Delete a category (transactions become uncategorised) |
| GET | `/api/transactions` | Filtered, paginated transactions (`page`, `pageSize`, `type`, `accountId`, `categoryId`, `from`, `to`, `q`, `sort`) |
| POST | `/api/transactions` | Record a transaction; returns `meta.budget_alerts` when a budget is exceeded |
| PUT | `/api/transactions/:id` | Update a transaction |
| DELETE | `/api/transactions/:id` | Delete a transaction |
| GET | `/api/budgets?period=YYYY-MM` | Budgets for a month with spent, remaining, percent, status and alerts |
| POST | `/api/budgets` | Create or update a budget (`period`, `category_id`, `amount_cents`) |
| DELETE | `/api/budgets/:id` | Remove a budget |
| GET | `/api/summary?from&to` | Dashboard payload: totals, balances, 6 month series, category split, recent activity |
| GET | `/api/reports?from&to` | Full report for a range |
| GET | `/api/reports/export?from&to` | CSV export of the transactions in a range |

### Data model

- **users** — name, unique email, scrypt password hash.
- **sessions** — hashed token, user, expiry (30 days by default).
- **accounts** — name, type (`bank` / `mobile_money` / `cash` / `savings` / `other`), institution,
  opening balance, colour, archived flag.
- **categories** — name, kind (`income` / `expense`), optional `parent_id` (one level), colour.
  Unique per kind + name + parent.
- **transactions** — account, optional category, type, amount, description, date (ISO `YYYY-MM-DD`).
- **budgets** — optional category (`NULL` = overall), period `YYYY-MM`, amount. Unique per
  period + category.

**Money is stored as integer cents** (`amount_cents`, `opening_balance_cents`) so sums never suffer
floating point drift; the UI converts to RWF for display. Dates are stored as ISO calendar dates so
range queries are timezone independent.

---

## 7. Design and security notes

- **Authentication:** passwords are hashed with scrypt; sign-in issues a random opaque token
  stored only as a SHA-256 hash. Sessions live in an `HttpOnly`, `SameSite=Lax` cookie, are
  removed on logout and checked on every protected request.
- **Authorization:** every wallet table has a `user_id`, and every read and write is filtered by
  the signed-in user — account ids supplied by other users return 404/422 rather than data.
- **Validation:** every request body and query string is validated with Zod; failures return a 422
  with per-field details that the forms display inline.
- **Parameterised SQL:** all queries use prepared statements with bound parameters, so user input is
  never interpolated into SQL.
- **Referential integrity:** SQLite foreign keys are enabled; deleting an account cascades to its
  transactions, deleting a category keeps history and nulls the link.
- **HTTP hardening:** Helmet with a strict content security policy, request body size limits,
  rate limiting on `/api`, a configurable CORS origin and no `X-Powered-By` header.
- **Modularity:** routes, validation, data access and analytics are separate modules; the client
  mirrors that with pages, feature components, UI primitives, hooks and contexts.
- **Single source of truth:** balances, totals, budget spending and reports are all derived by SQL
  from the transaction ledger rather than stored and kept in sync.

---

## 8. Deployment

The project is designed to deploy as a **single Node service** (the Express app serves the built
client), which works on Render, Fly.io, Railway and similar hosts. SQLite needs a persistent disk.

### Render

1. Create a **Web Service** from this repository.
2. Build command: `npm install && npm run build`
3. Start command: `npm start`
4. Add a **disk** mounted at `/data`, then set `DATABASE_PATH=/data/wallet.db`.
5. Add `NODE_ENV=production` (this marks the session cookie `Secure`). Leave `CORS_ORIGIN` unset
   when the client is served by the same host.
6. Set `AUTO_SEED=false` to skip the demo account in production, then register your own.

Render provides `PORT` automatically; the server reads it.

### Fly.io

`fly launch` will detect Node. Keep `npm run build` in the build step, run `npm start`, and attach a
volume mounted at `/data` with `DATABASE_PATH=/data/wallet.db`.

> **Deployment URL:** add your public link here once the app is live, for example
> `https://pesa-wallet.onrender.com`.

---

## 9. Submission notes

- Every required capability in the brief is implemented and reachable from the UI: multi-account
  transaction tracking, period reports, budgets with notifications, categories/subcategories with
  expense linking, and visual summaries.
- The repository includes this README with the feature description, prerequisites, setup steps and
  deployment instructions required by the submission guidelines.
