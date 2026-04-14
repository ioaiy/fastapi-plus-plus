# FastAPI++

Production-ready boilerplate for FastAPI applications built on Clean Architecture principles. Provides a structured foundation with dependency injection, async database layer, migrations, Docker setup, and clear separation of concerns — ready to extend with your own business logic.

Face++ integration included as a demo to showcase external service integration patterns.

## Stack

`Python 3.11+` `FastAPI` `PostgreSQL` `SQLAlchemy 2.0` `Alembic` `asyncpg` `httpx` `Punq` `Docker`

## Architecture

The project follows Clean Architecture with four layers, each with a single responsibility:

```
app/                              # Application factory (create_app)
core/                             # Domain layer — business logic, no framework dependencies
├── models/                       #   Pydantic domain models
└── services/                     #   Service layer (orchestration, image processing)
infrastructure/                   # Adapters — external world integration
├── cognitive_service.py          #   Face++ API client (demo integration)
├── container.py                  #   Punq DI container (singleton)
├── http.py                       #   HTTPX client with retry (tenacity)
├── lifespan.py                   #   FastAPI lifespan management
└── db/
    ├── sqlalchemy/                #   AsyncEngine + generic repository
    ├── repositories/              #   Concrete repositories
    ├── models/                    #   ORM models
    ├── session_manager.py         #   AsyncSession via ContextVar
    └── unitofwork.py              #   Unit of Work pattern
presentation/                     # Delivery layer — HTTP interface
└── asgi/
    ├── fastapi/                   #   App builder, exception handler
    ├── routes/                    #   API endpoints
    ├── requests/                  #   Request schemas
    └── responses/                 #   Response schemas
utils/                            # Cross-cutting: settings, logging, exceptions
migrations/                       # Alembic migrations
```

### Key patterns

- **Dependency Injection** — Punq container, all services registered as singletons
- **Repository + Unit of Work** — transactional database access via generic `SQLAlchemyRepository`
- **Protocol-based interfaces** — loose coupling between layers through Python Protocols
- **Factory pattern** — `create_app()` assembles the entire application
- **Async-first** — AsyncSession, asyncpg, httpx, FastAPI lifespan

## Install

### Poetry

https://python-poetry.org/docs/

```bash
git clone <repo-url>
cd fastapiplus
poetry shell
poetry install
```

### Environment

Create `.env` file in project root:

```env
DEBUG=false
DB_URL=postgresql+asyncpg://postgres:postgres@localhost:5432/postgres
FACEPLUSPLUS_API_KEY=your_api_key
FACEPLUSPLUS_API_SECRET=your_api_secret
```

Face++ credentials needed only for the demo integration — get them at [Face++ Console](https://console.faceplusplus.com/).

## Quickstart

### Docker (recommended)

```bash
docker compose up -d --build
```

Three services start:

- **postgres** — PostgreSQL on port `5432`
- **migrate** — runs `alembic upgrade head` and exits
- **app** — FastAPI on port `8000` (waits for migrate to complete)

### Local development

Start only the database:

```bash
docker compose -f docker-compose-dev.yml up -d
```

Run migrations and start the app:

```bash
alembic upgrade head
uvicorn app.main:create_app --factory --reload --host 0.0.0.0 --port 8000
```

## Demo API

The boilerplate ships with a Face++ demo under `/api/v1` to illustrate request handling, service orchestration, and external API integration.

Swagger UI: [http://localhost:8000/docs](http://localhost:8000/docs)

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/detect` | Upload image, detect faces via Face++ |
| `POST` | `/image/{id}` | Get image with optional face highlighting |
| `POST` | `/compare` | Compare two faces by tokens, returns confidence |
| `DELETE` | `/image/{id}` | Delete image by ID |

### Examples

```bash
# Detect faces in an image
curl -X POST http://localhost:8000/api/v1/detect -F "file=@photo.jpg"

# Get image with highlighted faces
curl -X POST http://localhost:8000/api/v1/image/{id} \
  -H "Content-Type: application/json" \
  -d '{"color": "green", "face_tokens": ["token1"]}' \
  --output result.jpg

# Compare two faces
curl -X POST http://localhost:8000/api/v1/compare \
  -H "Content-Type: application/json" \
  -d '{"face_token_1": "token1", "face_token_2": "token2"}'
```

## Database

PostgreSQL with async driver (asyncpg). Migrations managed by Alembic.

The demo creates two tables: `images` (stores uploaded data as bytea) and `faces` (detected face regions, FK to images with CASCADE delete).

```bash
# Create a new migration
alembic revision --autogenerate -m "description"

# Apply migrations
alembic upgrade head
```

## Code Quality

- **Pyright** — strict mode across all files
- **Black** — formatter (line-length: 88)
- **Ruff** — linter (F, E, W, N, UP, B, A, C4, C90)

## Dependencies

| Package | Purpose |
|---------|---------|
| `fastapi` | Web framework |
| `uvicorn` | ASGI server |
| `pydantic` + `pydantic-settings` | Validation & config |
| `sqlalchemy` + `asyncpg` | Async ORM & PostgreSQL driver |
| `alembic` | Database migrations |
| `httpx` + `tenacity` | HTTP client with retries |
| `opencv-python` | Image processing |
| `punq` | Dependency injection |
| `python-multipart` | File uploads |
