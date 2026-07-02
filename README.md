# Async FastAPI + PostgreSQL

A backend template using **FastAPI**, **async SQLAlchemy/SQLModel**, **PostgreSQL** (with pgvector), and **Alembic** for migrations. The project follows patterns from [fastapi-best-practices](https://github.com/zhanymkanov/fastapi-best-practices).

## Features

- Async database access with SQLAlchemy 2.x and SQLModel
- User CRUD and JWT-based auth (`/auth/signup`, `/auth/login`)
- Alembic migrations
- Docker Compose for local PostgreSQL (pgvector image)
- Pytest test suite with async HTTP client support

## Tech Stack

| Layer | Technology |
|-------|------------|
| API | FastAPI, Uvicorn |
| ORM | SQLModel, SQLAlchemy (async) |
| Database | PostgreSQL 16 + pgvector |
| Migrations | Alembic |
| Auth | bcrypt, PyJWT |
| Config | pydantic-settings |

## Project Structure

```
├── alembic/              # Database migrations
├── docker/pgvector/      # Custom PostgreSQL image with pgvector
├── src/
│   ├── main.py           # Application entry point
│   ├── config/           # Settings and environment config
│   ├── database/         # Async engine, session, and init
│   ├── server/
│   │   ├── auth/         # User and auth routes, models, service
│   │   └── utils/        # Utility routes
│   └── services/         # Shared services (e.g. JWT)
├── tests/
├── docker-compose.yml
├── Dockerfile
├── Makefile
└── requirements.txt
```

Each server module follows this layout:

| File | Purpose |
|------|---------|
| `router.py` | API endpoints |
| `schemas.py` | Pydantic request/response models |
| `models.py` | SQLModel database models |
| `service.py` | Business logic |
| `constants.py` | Route paths and constants |

## Prerequisites

- Python 3.12+
- Docker and Docker Compose (for local PostgreSQL)
- `make` (optional, for convenience commands)

## Quick Start

### 1. Clone and configure environment

```bash
cp .env.example .env
```

Edit `.env` with your database credentials. Defaults work with the included Docker Compose setup:

```env
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_DB=async_db
```

The app builds the async connection string automatically from these values (`postgresql+asyncpg://…` on port `5433`).

### 2. Start PostgreSQL

```bash
make up
```

Or without Make:

```bash
docker-compose up -d
```

PostgreSQL is exposed on **localhost:5433**.

### 3. Install dependencies

```bash
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

### 4. Run migrations

```bash
alembic upgrade head
```

### 5. Start the API

```bash
python -m src.main
```

Or with Uvicorn directly:

```bash
uvicorn src.server:app --host 0.0.0.0 --port 8080 --reload
```

The API is available at [http://localhost:8080](http://localhost:8080). Interactive docs at [http://localhost:8080/docs](http://localhost:8080/docs).

## API Endpoints

### Health

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/` | Health check |

### Users

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/api/create-user` | Create a user |
| `GET` | `/api/get-users` | List all users |
| `GET` | `/api/get-user/{user_id}` | Get user by ID |
| `PUT` | `/api/update-user/{user_id}` | Update a user |
| `DELETE` | `/api/delete-user/{user_id}` | Delete a user |

### Auth

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/auth/signup` | Register and receive an access token |
| `POST` | `/auth/login` | Log in and receive an access token |

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `POSTGRES_USER` | `postgres` | Database username |
| `POSTGRES_PASSWORD` | `postgres` | Database password |
| `POSTGRES_DB` | `async_db` | Database name |
| `POSTGRES_HOST` | `localhost` | Database host |
| `POSTGRES_PORT` | `5433` | Database port |

Settings are loaded from `.env` via `src/config/config.py`.

## Alembic Migrations

Alembic is pre-configured in this project:

- `alembic/env.py` uses `SQLModel.metadata` as `target_metadata`
- `alembic.ini` contains the sync database URL for migration commands

Common commands:

```bash
# Generate a new migration from model changes
alembic revision --autogenerate -m "describe your change"

# Apply all pending migrations
alembic upgrade head

# Roll back one revision
alembic downgrade -1
```

When adding new models, import them in `alembic/env.py` so autogenerate can detect them.

## Testing

Ensure PostgreSQL is running, then:

```bash
pytest --disable-warnings -s
```

Tests use the async test client and override the database session dependency. See `tests/test_auth.py` and `tests/test_auth_asgi.py`.

## Make Commands

| Command | Description |
|---------|-------------|
| `make up` | Start PostgreSQL via Docker Compose |
| `make down` | Stop services and remove containers/images |
| `make build` | Build Docker images |
| `make logs` | Tail service logs |
| `make print-env` | Print `DATABASE_URL` from `.env` |

## Docker

The root `Dockerfile` builds a production image running Uvicorn on port 8080. PostgreSQL uses a custom image based on [pgvector/pgvector](https://github.com/pgvector/pgvector) (see `docker/pgvector/`).

## References

- [FastAPI Best Practices](https://github.com/zhanymkanov/fastapi-best-practices)
- [The Ultimate FastAPI Project Setup](https://medium.com/@lawsontaylor/the-ultimate-fastapi-project-setup-fastapi-async-postgres-sqlmodel-pytest-and-docker-ed0c6afea11b)
