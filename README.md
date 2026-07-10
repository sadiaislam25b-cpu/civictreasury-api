# CivicTreasury API — Individual MVP

Deployed, functional FastAPI application built for the capstone individual MVP requirement.

## 🔗 Live Deployment

**https://civictreasury-api.onrender.com**

Interactive API docs (Swagger UI) are available at `/docs` on the deployed URL: https://civictreasury-api.onrender.com/docs

## Features

- **User authentication**: register, login (JWT), logout, profile view/edit
- **Scenario CRUD**: full create/read/update/delete for UBI policy scenarios, scoped per user
- Stable `main` branch, no broken builds

## Tech Stack

- Python 3.12 + FastAPI
- SQLAlchemy ORM
- SQLite (dev) — swap to Postgres via `DATABASE_URL` env var for production
- JWT auth via `python-jose`, password hashing via `passlib` + `bcrypt`

## API Endpoints

| Method | Path | Description |
|---|---|---|
| POST | `/auth/register` | Register a new user |
| POST | `/auth/login` | Log in, returns JWT |
| POST | `/auth/logout` | Log out (auth required) |
| GET | `/auth/profile` | View own profile (auth required) |
| PATCH | `/auth/profile` | Update own profile (auth required) |
| POST | `/scenarios` | Create a scenario (auth required) |
| GET | `/scenarios` | List your scenarios (auth required) |
| GET | `/scenarios/{id}` | Get one scenario (auth required) |
| PUT | `/scenarios/{id}` | Update a scenario (auth required) |
| DELETE | `/scenarios/{id}` | Delete a scenario (auth required) |

## Running Locally

```bash
pip install -r requirements.txt
uvicorn app.main:app --reload
```

Visit `http://localhost:8000/docs` for the interactive API explorer.

## Environment Variables

| Variable | Purpose | Default |
|---|---|---|
| `SECRET_KEY` | JWT signing secret — **set a real one on Render** | dev-only fallback |
| `DATABASE_URL` | Postgres connection string (optional) | falls back to local SQLite |

---

## Product Specification

# UBI Budgeting & Policy Simulation Platform
### Product Specification — Team TBD

## Part I — Project Proposal

### Project Description

The UBI Budgeting & Policy Simulation Platform is a three-sided web application that allows everyday citizens to simulate the personal financial impact of a Universal Basic Income program, connect their real bank accounts to see how UBI would affect their actual expenses and income, and receive AI-guided budgeting support. A separate government administration dashboard gives program operators visibility into disbursements, recipient demographics, geographic distribution, and funding sources.

**The problem it solves:** UBI is widely discussed as a policy idea, but most people have no concrete sense of what it would mean for their own financial life. This platform makes the concept tangible by grounding it in real fiscal math (policy simulator) and the user's own bank data (personal finance dashboard). It also surfaces existing real-world assistance programs users may qualify for today, since UBI does not yet exist.

**Impact:** Users walk away with a clear, personalized understanding of how UBI would change their financial situation — and what government programs are available to them right now. Administrators gain a data-driven view of what a real UBI rollout would look like at scale.

### User Personas / Audience

**Persona 1 — The Curious Citizen**
Adults (18–65) who have heard about UBI in the news and want to understand what it would actually mean for them personally. They care about financial security, fairness, and understanding policy without needing an economics degree. They've tried reading articles or watching videos about UBI but find them too abstract — no tool helps them see their own numbers.

**Persona 2 — The Budget-Conscious User**
Working adults living paycheck to paycheck or managing tight budgets who want a clear picture of their finances. They care about knowing where their money goes, reducing expenses, and finding financial relief. They've tried basic budgeting apps (Mint, YNAB) but haven't found anything that connects their finances to the broader policy conversation or helps them discover assistance programs they may qualify for.

**Persona 3 — The Government Administrator**
Policy analysts or program officers responsible for overseeing a UBI pilot or simulation. They care about fiscal accountability, geographic equity, and understanding how funds are being used. They lack a centralized dashboard that shows recipient data, disbursement totals, state-level distribution, and funding source breakdowns in one place.

### User Stories

**MVP (Without these features, the application will not be useful)**

1. As a user, I can interact with the UBI policy simulator by adjusting sliders for monthly UBI amount, eligible population, phase-out income threshold, and funding sources (wealth tax, carbon tax, VAT on luxury goods, automation tax, and existing benefit offsets) — and immediately see a live breakdown of total program cost, revenue by source, fiscal surplus or deficit, and how much each income tax bracket would net gain or lose under the simulated policy.
2. As a user, I can connect my bank accounts via Plaid to view my real income and categorized expenses on a personal finance dashboard. I can toggle between weekly, monthly, and yearly views with a pie chart that updates to show my spending breakdown by category for the selected time range — and see my current surplus or deficit clearly.
3. As a user, I can see my estimated UBI payment displayed as a separate line item on my personal dashboard — calculated based on my actual income and the current policy simulator settings — so I can clearly understand what I would receive and how it would change my financial picture without it being mixed into my real income.
4. As a user, I can chat with an AI assistant that has context on both UBI policy and my financial data. It can answer questions about how UBI works, explain the simulator results, help me understand my budget, and recommend real existing government assistance programs (such as SNAP, EITC, LIHEAP, WIC, or housing vouchers) that I may qualify for today.
5. As a user, I can log into a separate admin dashboard and view total enrolled recipients, total monthly disbursements, per-recipient payment amounts, a state-by-state breakdown of who is receiving money and how much, and a funding source breakdown showing revenue contributions by tax type and whether the program is in surplus or deficit.

**Stretch Features (When time is running short, these features will get cut)**

1. As a user, I can manually edit or recategorize transactions pulled from Plaid so my expense breakdown accurately reflects my real spending habits.
2. As a user, I can see a side-by-side comparison of multiple UBI policy scenarios I've configured — so I can compare the fiscal impact and personal benefit of different program designs.
3. As a government administrator, I can filter the dashboard by date range, state, or income bracket to drill down into specific segments of the recipient population.

## Part II — Technical Specifications

### Schema Design

**users**

| Field | Constraints |
|---|---|
| user_id | SERIAL PRIMARY KEY |
| username | TEXT UNIQUE NOT NULL |
| email | TEXT UNIQUE NOT NULL |
| password_hash | TEXT NOT NULL |
| role | TEXT NOT NULL DEFAULT 'user' — 'user' or 'admin' |
| state | TEXT |
| annual_income | NUMERIC |
| created_at | TIMESTAMP DEFAULT NOW() |

**ubi_policy_settings**

| Field | Constraints |
|---|---|
| policy_id | SERIAL PRIMARY KEY |
| user_id | INTEGER REFERENCES users(user_id) ON DELETE CASCADE |
| monthly_ubi_amount | NUMERIC NOT NULL DEFAULT 1000 |
| eligible_population | BIGINT NOT NULL DEFAULT 258000000 |
| phase_out_income | NUMERIC NOT NULL DEFAULT 75000 |
| wealth_tax_rate | NUMERIC NOT NULL DEFAULT 2.0 |
| carbon_tax_per_ton | NUMERIC NOT NULL DEFAULT 50 |
| vat_luxury_rate | NUMERIC NOT NULL DEFAULT 5.0 |
| automation_tax_rate | NUMERIC NOT NULL DEFAULT 1.0 |
| benefit_offset_pct | NUMERIC NOT NULL DEFAULT 30 |
| created_at | TIMESTAMP DEFAULT NOW() |

**plaid_accounts**

| Field | Constraints |
|---|---|
| account_id | SERIAL PRIMARY KEY |
| user_id | INTEGER REFERENCES users(user_id) ON DELETE CASCADE |
| plaid_access_token | TEXT NOT NULL |
| plaid_item_id | TEXT NOT NULL |
| institution_name | TEXT |
| created_at | TIMESTAMP DEFAULT NOW() |

**transactions**

| Field | Constraints |
|---|---|
| transaction_id | SERIAL PRIMARY KEY |
| user_id | INTEGER REFERENCES users(user_id) ON DELETE CASCADE |
| plaid_transaction_id | TEXT UNIQUE NOT NULL |
| amount | NUMERIC NOT NULL |
| category | TEXT |
| merchant_name | TEXT |
| date | DATE NOT NULL |
| type | TEXT — 'income' or 'expense' |

**ai_conversations**

| Field | Constraints |
|---|---|
| conversation_id | SERIAL PRIMARY KEY |
| user_id | INTEGER REFERENCES users(user_id) ON DELETE CASCADE |
| role | TEXT NOT NULL — 'user' or 'assistant' |
| content | TEXT NOT NULL |
| created_at | TIMESTAMP DEFAULT NOW() |

### API Contract

**Auth**

`POST /api/auth/register` — Creates a new user account.
- Body: `{ username, email, password, role='user', state, annual_income }`
- 201: `{ id, username, email, role }`
- 409: Email already in use
- 400: Validation error

`POST /api/auth/login` — Authenticates a user and returns a session token.
- Body: `{ email, password }`
- 200: `{ token, user: { id, username, role } }`
- 401: Invalid credentials

**UBI Policy Simulator**

`GET /api/simulator/default` — Returns default UBI policy settings.
- 200: `{ monthly_ubi_amount, eligible_population, phase_out_income, wealth_tax_rate, carbon_tax_per_ton, vat_luxury_rate, automation_tax_rate, benefit_offset_pct }`

`POST /api/simulator/calculate` — Calculates program cost, revenue breakdown, fiscal balance, and tax bracket impact.
- Body: `{ monthly_ubi_amount, eligible_population, phase_out_income, wealth_tax_rate, carbon_tax_per_ton, vat_luxury_rate, automation_tax_rate, benefit_offset_pct }`
- 200: `{ total_program_cost, revenue: { wealth_tax, carbon_tax, vat_luxury, automation_tax, benefit_offsets, total }, fiscal_balance, bracket_impact: [{ bracket, net_gain }] }`
- 400: Validation error

`POST /api/simulator/save` — Saves a user's custom policy configuration. Requires auth.
- Body: `{ monthly_ubi_amount, eligible_population, phase_out_income, ... }`
- 201: `{ policy_id, ...settings }`
- 401: Not authenticated

**Personal Finance Dashboard**

`POST /api/plaid/link` — Exchanges a Plaid public token for an access token and stores the linked account.
- Body: `{ public_token }`
- 201: `{ account_id, institution_name }`
- 400: Plaid error
- 401: Not authenticated

`GET /api/transactions` — Returns transactions for the authenticated user.
- Optional query: `?range=weekly|monthly|yearly` (default: monthly)
- 200: `[{ transaction_id, amount, category, merchant_name, date, type }, ...]`
- 401: Not authenticated

`GET /api/transactions/summary` — Returns income total, expense total, surplus/deficit, and category breakdown.
- Optional query: `?range=weekly|monthly|yearly`
- 200: `{ income_total, expense_total, surplus, categories: [{ name, amount, percentage }] }`
- 401: Not authenticated

`GET /api/ubi/estimate` — Returns the user's estimated monthly UBI payment based on income and saved/default policy settings.
- 200: `{ monthly_ubi_estimate, policy_used: { monthly_ubi_amount, phase_out_income } }`
- 401: Not authenticated

**AI Assistant**

`POST /api/ai/chat` — Sends a user message to the AI assistant with context on UBI policy and the user's financial data.
- Body: `{ message, conversation_history: [{ role, content }] }`
- 200: `{ reply: string }`
- 401: Not authenticated
- 500: AI service error

**Admin Dashboard**

`GET /api/admin/overview` — Returns high-level program stats. Admin role required.
- 200: `{ total_recipients, total_monthly_disbursement, avg_payment }`
- 401: Not authenticated / 403: Unauthorized

`GET /api/admin/by-state` — Returns recipient count and total disbursements grouped by state. Admin role required.
- 200: `[{ state, recipient_count, total_disbursement }, ...]`
- 401: Not authenticated / 403: Unauthorized

`GET /api/admin/funding` — Returns revenue breakdown by funding source. Admin role required.
- 200: `{ wealth_tax, carbon_tax, vat_luxury, automation_tax, benefit_offsets, total_revenue, total_program_cost, fiscal_balance }`
- 401: Not authenticated / 403: Unauthorized

### Core Technologies, 3rd-Party APIs and New Libraries

- **React** — Frontend user interface — component-based SPA with tab navigation, interactive sliders, pie charts, and chat UI.
- **Python and FastAPI** — Backend server — REST API handling auth, simulator calculations, Plaid integration, AI chat, and admin endpoints.
- **PostgreSQL** — Primary database — stores users, policy settings, linked accounts, transactions, and conversation history.
- **Plaid API** — Bank account integration — used to link accounts, pull transaction history, and detect income vs. expenses. Endpoints: `/link/token/create`, `/item/public_token/exchange`, `/transactions/get`
- **Anthropic Claude API (prompt engineering)** — Powers the AI budgeting and UBI assistant. System prompt includes UBI policy context, user financial summary, and a curated list of real assistance programs. Endpoint: `POST /v1/messages`
- **Recharts or Chart.js** — Frontend charting library for pie charts and bar charts in the personal dashboard and simulator results.
- **JWT (PyJWT)** — Authentication tokens for securing user and admin routes.
