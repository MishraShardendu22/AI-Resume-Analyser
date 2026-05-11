# AI Resume Analyzer 

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
make up
```

The stack runs behind nginx on:

- Frontend: http://localhost:8080
- API: http://localhost:8080/api/v1

To enable AI-powered outputs, set `OPENROUTER_API_KEY` in `.env.example` (or your own env file).

## Default Environment

The project runs using `.env` if present; otherwise `.env.example`. Update your local `.env` if you want custom secrets or model choices.

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
