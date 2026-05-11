# AI Resume Analyzer Monorepo

Production-style monorepo for an AI Resume Analyzer with Python microservices and a Next.js frontend. The platform supports resume uploads (PDF/DOCX/TXT), ATS scoring, resume-to-job matching, skill gap analysis, recruiter ranking, and AI-generated content.

## Architecture Overview

- **apps/frontend**: Next.js App Router UI for students and recruiters
- **services/api-gateway**: Entry point with auth, rate limiting, and routing
- **services/auth-service**: JWT auth, user management, refresh tokens
- **services/resume-service**: Resume upload, parsing, sectioning, storage
- **services/analysis-service**: ATS scoring, AI analysis, embeddings
- **services/ranking-service**: Candidate ranking and comparisons
- **libs/shared**: Shared SQLAlchemy models and utilities
- **infra/nginx**: Reverse proxy to frontend and API
- **infra/k8s**: Kubernetes manifests

## Quick Start (Docker Compose)

```bash
docker compose up --build
```

Optional shortcut:

```bash
make up
```

The stack runs behind nginx on:

- Frontend: http://localhost:8080
- API: http://localhost:8080/api/v1

To enable AI-powered outputs, set `OPENROUTER_API_KEY` in `.env.example` (or your own env file).

## Default Environment

The project runs using `.env` if present; otherwise `.env.example`. Update your local `.env` if you want custom secrets or model choices.

## macOS Setup (Intel + Apple Silicon)

### Prerequisites

**Docker Desktop:**
1. Download [Docker Desktop for macOS](https://www.docker.com/products/docker-desktop)
2. Install and start Docker Desktop
3. Verify installation: `docker --version && docker compose version`

**Other Requirements:**
- Git
- At least 8GB RAM available (16GB recommended for comfortable development)
- 20GB free disk space for images and volumes

### One-Command Startup

```bash
git clone <repo>
cd AI-Resume-Analyser
docker compose up --build
```

Wait for all services to show as "healthy" (typically 2-3 minutes on first run).

### Verify the Setup

Once all services are healthy, test the application:

```bash
# Test registration
curl -X POST http://localhost:8080/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"Test123!","full_name":"Test User","role":"student"}'

# Access frontend
open http://localhost:8080
```

### macOS Intel (x86_64)

Everything works out-of-the-box. All images are built for linux/amd64 and will run via Docker's native virtualization.

**Performance:** Native performance, fastest on Intel Macs.

### Apple Silicon (M1/M2/M3/M4 - arm64)

All services automatically detect Apple Silicon and pull/build arm64-compatible images.

**Dockerfile Support:**
- Python images: `python:3.12-slim` (supports both amd64 and arm64)
- Node images: `node:20-slim` (supports both amd64 and arm64)
- All dependencies compile for arm64 automatically

**If you encounter `no matching manifest` errors:**

Some third-party images may not support arm64. Force amd64 emulation (slower):

```bash
DOCKER_DEFAULT_PLATFORM=linux/amd64 docker compose up --build
```

**Performance Notes:**
- Apple Silicon runs arm64 images natively (much faster than amd64 emulation)
- Cold builds take ~3-5 minutes (subsequent builds are much faster due to Docker layer caching)
- If experiencing slowness, ensure Docker Desktop has sufficient resources:
  - Preferences → Resources → Memory: 8GB minimum, 16GB recommended
  - Preferences → Resources → Disk Image Size: 60GB recommended

## Services and Ports

- **Postgres**: 5432
- **Redis**: 6379
- **MinIO**: 9000 (console 9001)
- **Nginx**: 8080

## Useful Commands

```bash
make migrate
make logs
make test
make down
```

Equivalent Docker Compose commands:

```bash
docker compose build
docker compose up --build
docker compose ps
docker compose logs
docker compose down
```

## API Examples

```bash
curl -X POST http://localhost:8080/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"student@example.com","password":"Passw0rd!","full_name":"Student A","role":"student"}'
```

```bash
curl -X POST http://localhost:8080/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"student@example.com","password":"Passw0rd!"}'
```

## Repository Layout

```text
root/
├── apps/
│   └── frontend/
├── services/
│   ├── api-gateway/
│   ├── auth-service/
│   ├── resume-service/
│   ├── analysis-service/
│   └── ranking-service/
├── libs/
│   └── shared/
├── infra/
│   ├── docker/
│   ├── k8s/
│   └── nginx/
├── scripts/
├── docs/
├── docker-compose.yml
├── Makefile
└── .env.example
```

## Notes

- No OCR is used. Only text extraction from PDF/DOCX/TXT.
- OpenRouter is the AI provider with configurable model names.
- pgvector is enabled for embeddings in Postgres.

## Troubleshooting

### General: Service Health Check Failures

If services are stuck in an unhealthy state:

```bash
# View detailed logs
docker compose logs <service-name>

# Restart all services
docker compose restart

# Full reset (removes volumes)
docker compose down -v && docker compose up --build
```

### `Invalid credentials` after restarting

If you ran `docker compose down -v` or `make clean`, the database was reset. Register a new user and log in again.

### `no matching manifest` on Apple Silicon

Some third-party images may not publish arm64 builds. Force amd64 emulation (slower):

```bash
DOCKER_DEFAULT_PLATFORM=linux/amd64 docker compose up --build
```

### Ports already in use

If 8080/5432/6379 are already occupied, stop the conflicting services:

```bash
# Find what's using port 8080
lsof -i :8080

# Kill the process
kill -9 <PID>
```

Or change port mappings in `docker-compose.yml`:

```yaml
ports:
  - "8081:8080"  # Use 8081 instead of 8080
```

### macOS-Specific: Docker Desktop Memory/CPU Pressure

If services fail to start or are very slow:

1. Open **Docker Desktop** → **Preferences** → **Resources**
2. Increase memory to 16GB (minimum 8GB)
3. Increase CPU to at least 4 cores
4. Increase disk image size to 60GB
5. Click **Apply & Restart**

### macOS: Permission Denied on Socket

If you see "Permission denied" errors with Docker:

```bash
# Ensure your user is in the docker group (usually automatic with Docker Desktop)
docker ps  # Should work without sudo
```

### Frontend Not Loading

If `http://localhost:8080` shows an error:

```bash
# Check if nginx is running
docker compose ps nginx

# View nginx logs
docker compose logs nginx

# Verify frontend is building
docker compose logs frontend
```

### API Endpoint 502 Bad Gateway

If API requests fail with 502:

```bash
# Check if API services are healthy
docker compose ps api-gateway auth-service

# View logs
docker compose logs api-gateway

# The api-gateway service should be running and healthy
```

### Database Connection Errors

If you see `could not connect to server`:

```bash
# Check postgres is healthy
docker compose ps postgres

# View postgres logs
docker compose logs postgres

# Ensure database volume exists
docker volume ls | grep postgres_data

# If volume is gone, rebuild everything
docker compose down -v && docker compose up --build
```

### Certificate/SSL Errors (if using HTTPS proxies)

The application runs on HTTP locally. If you're behind a corporate proxy that intercepts HTTPS:

1. Add your proxy's certificate to Docker Desktop
2. Or bypass the proxy:

```bash
# Temporarily disable proxy
unset HTTP_PROXY HTTPS_PROXY ALL_PROXY
docker compose up --build
```
