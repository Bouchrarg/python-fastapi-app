# FastAPI Starter

A minimal, production‑leaning FastAPI template with testing (coverage), linting/formatting, Docker, and CI. Ships with a root endpoint and a health check.

## Features
- FastAPI + Uvicorn
- Pydantic Settings for configuration management
- Custom logging with TRACE level support
- Testing with pytest + pytest-cov (HTML/XML coverage, 85% threshold)
- Ruff for linting and formatting; optional mypy typing
- Dockerfile and GitHub Actions CI (Python 3.13)

## Endpoints
- `GET /` → `{ "message": "the server is now active" }`
- `GET /health` → `{ "ok": true }`

## Quick Start
```bash
python -m venv .venv && source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install -r requirements-dev.txt
cp .env.example .env  # Optional: customize your environment variables
uvicorn app.main:app --reload --port 8000
```

Open [http://localhost:8000](http://localhost:8000) and [http://localhost:8000/health](http://localhost:8000/health)

## Testing with Coverage

```bash
pytest
# Outputs: htmlcov/ (HTML), coverage.xml (XML), threshold enforced via pyproject.toml
```

## Linting and Formatting

```bash
ruff check .
ruff format .
# Optional typing
mypy app
```

## Docker

```bash
docker build -t fastapi-starter .
docker run -p 8000:8000 fastapi-starter
```

## CI

GitHub Actions workflow runs linting and tests with coverage on Python 3.13. See `.github/workflows/ci.yml`.

## License

MIT. See [LICENSE](./LICENSE).
test
Q2 test
