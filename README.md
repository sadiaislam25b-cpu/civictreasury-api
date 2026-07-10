# CivicTreasury API — Individual MVP

Deployed, functional FastAPI application built for the capstone individual MVP requirement.

## 🔗 Live Deployment

**[TODO: paste your Render URL here once deployed, e.g. https://civictreasury-api.onrender.com]**

Interactive API docs (Swagger UI) are available at `/docs` on the deployed URL.

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

**[TODO: paste the full contents of your Product Specification document here]**

