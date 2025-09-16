# PROJECT_NAME — ML Model API (FastAPI)

Production-ready template for serving a Machine Learning model as a REST API using **FastAPI**.  
Includes local development, testing, Docker packaging, and deployment guidancee.

---

## Table of Contents
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Repository Structure](#repository-structure)
- [Prerequisites](#prerequisites)
- [Quickstart (Local)](#quickstart-local)
- [Configuration](#configuration)
- [API Reference](#api-reference)
- [Examples](#examples)
- [Testing & Quality](#testing--quality)
- [Docker](#docker)
- [Deployment](#deployment)
- [Monitoring](#monitoring)
- [Model Management](#model-management)
- [Security](#security)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

---

## Features
- 🚀 **FastAPI** service with `/health` and `/predict` endpoints
- ✅ **Pydantic** request/response validation + auto-generated **OpenAPI/Swagger** docs
- 🧪 **pytest** tests, **ruff/black** lint & format, **mypy** type checks
- 🐳 **Docker** image with production server (Gunicorn + Uvicorn workers)
- 📦 Optional **MLflow** integration for model versioning & artifacts
- 📈 Prometheus-ready **/metrics** (optional) for latency and error monitoring

---

## Tech Stack
- **Runtime:** Python 3.10+  
- **Web:** FastAPI, Uvicorn/Gunicorn  
- **Validation:** Pydantic v2  
- **ML:** scikit-learn (example) / Replace with your framework  
- **Tests:** pytest  
- **Lint/Format:** ruff, black  
- **Typing:** mypy  
- **Packaging:** Docker

---

## Repository Structure
```
.
├── app/
│   ├── main.py                 # FastAPI application entrypoint
│   ├── api.py                  # Routes and handlers
│   ├── models/                 # ML model loading & inference code
│   │   ├── __init__.py
│   │   └── predictor.py
│   ├── schemas.py              # Pydantic request/response models
│   ├── config.py               # Settings from env vars
│   └── utils/                  # Helpers (logging, metrics)
├── models/
│   └── model.joblib            # Example model artifact (git-ignored in real projects)
├── tests/
│   ├── test_api.py
│   └── test_model.py
├── scripts/
│   ├── train.py                # Example: trains & saves model.joblib
│   └── smoke_curl.sh           # Example: curl calls for local testing
├── .env.example
├── .gitignore
├── Dockerfile
├── requirements.txt
├── pyproject.toml              # tool configs (ruff/black/mypy/pytest) if you prefer
├── Makefile                    # convenience commands
└── README.md
```

---

## Prerequisites
- **Python** 3.10+  
- **pip** or **uv**/**poetry** (optional)  
- **Docker** (optional but recommended)  
- **Git**

---

## Quickstart (Local)
```bash
# 1) Clone
git clone https://github.com/YOUR_NAME/PROJECT_NAME.git
cd PROJECT_NAME

# 2) Create and activate venv
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate

# 3) Install deps
pip install -r requirements.txt

# 4) Configure env (copy example and edit)
cp .env.example .env

# 5) (Optional) Train a sample model
python scripts/train.py

# 6) Run API (dev)
uvicorn app.main:app --reload --port 8000

# Open docs
# http://localhost:8000/docs  (Swagger UI)
# http://localhost:8000/redoc (ReDoc)
```

---

## Configuration
Environment variables are loaded from `.env`. Example:

```env
# .env
APP_NAME=PROJECT_NAME
APP_ENV=local
LOG_LEVEL=INFO
MODEL_PATH=./models/model.joblib
ENABLE_METRICS=true
```

`app/config.py` reads these via `pydantic-settings` (or os.environ).

---

## API Reference

### `GET /health`
- **200 OK**: `{"status": "ok", "model_loaded": true}`

### `POST /predict`
- **Request (application/json)**:
```json
{
  "age": 42,
  "income": 75000.0,
  "utilization": 0.23
}
```
- **Response (200)**:
```json
{
  "risk_probability": 0.1374,
  "model_version": "v1"
}
```

### `GET /metrics` (optional)
- Prometheus text metrics (latency, request count, errors)

---

## Examples

**cURL**
```bash
curl -s http://localhost:8000/health

curl -s -X POST http://localhost:8000/predict \
  -H 'Content-Type: application/json' \
  -d '{"age": 42, "income": 75000, "utilization": 0.23}'
```

**Python client**
```python
import requests
payload = {"age": 42, "income": 75000, "utilization": 0.23}
res = requests.post("http://localhost:8000/predict", json=payload, timeout=10)
print(res.json())
```

---

## Testing & Quality
```bash
# Run unit tests
pytest -q

# Lint & format
ruff check .
black --check .

# Type-check
mypy app
```

> Tip: Add a pre-commit hook to run ruff/black/mypy before each commit.

---

## Docker
```bash
# Build image
docker build -t your-dockerhub-username/project-name:latest .

# Run container
docker run --rm -p 8000:8000 \
  --env-file .env \
  your-dockerhub-username/project-name:latest

# Multi-arch push (example)
docker push your-dockerhub-username/project-name:latest
```

**Dockerfile** uses Gunicorn + Uvicorn workers for production:
- Health: `/health`
- Port: `8000`

---

## Deployment
- **AWS ECS/Fargate** or **AWS App Runner** (simple)
- **Kubernetes (EKS/AKS/GKE)** with:
  - Liveness: `GET /health`
  - Readiness: `GET /health`
  - Resources: requests/limits tuned by load tests
- **SageMaker Real-Time Endpoint** if you want managed model registry & autoscaling

Minimal K8s readiness/liveness probes:
```yaml
livenessProbe:
  httpGet: { path: /health, port: 8000 }
  initialDelaySeconds: 15
  periodSeconds: 10
readinessProbe:
  httpGet: { path: /health, port: 8000 }
  initialDelaySeconds: 5
  periodSeconds: 5
```

---

## Monitoring
- **Logs**: structured JSON logs (level, path, latency, request-id)
- **Metrics**: request count, error rate, p95 latency (Prometheus + Grafana)
- **Tracing**: OpenTelemetry (optional) to trace calls to DB/feature stores

---

## Model Management
- Track experiments and register models with **MLflow**
- Store artifacts in **S3/Blob/GCS**
- Version API (`/v1`, `/v2`) when changing contracts
- Add input validation & drift checks (whylogs/evidently) in a background task

---

## Security
- **Auth**: API keys or OAuth2/JWT for protected endpoints
- **CORS**: restrict allowed origins
- **Rate limiting** (ingress or gateway)
- Never commit real secrets—use env vars or a secrets manager (AWS Secrets Manager/Azure Key Vault)

---

## Troubleshooting
- `422 Unprocessable Entity`: input schema mismatch → check request JSON types
- Model not found: verify `MODEL_PATH` and model artifact presence
- High latency: increase workers (`WEB_CONCURRENCY`), enable async I/O for I/O-bound ops

---

## Contributing
1. Fork & create feature branch: `git checkout -b feature/xyz`
2. Run tests & linters locally
3. Submit PR with a clear description and screenshots (if UI)

---

## License
MIT © YEAR YOUR_NAME
