# macOS Setup & Deployment Guide

## Overview

This document provides comprehensive guidance for running the AI Resume Analyzer monorepo on macOS, supporting both Intel (x86_64) and Apple Silicon (ARM64) architectures.

## Quick Start (One Command)

```bash
git clone <repo>
cd AI-Resume-Analyser
docker compose up --build
```

That's it! Wait for all services to show as "healthy" (typically 2-3 minutes on first run).

## System Requirements

### Minimum
- macOS 10.15 or later
- Docker Desktop 4.0+
- 8GB RAM
- 20GB free disk space

### Recommended
- macOS 12.0+ (Monterey or later)
- Docker Desktop latest version
- 16GB RAM
- 60GB free disk space

## Architecture Support

### Intel Macs (x86_64)
✅ **Fully supported** — All images run natively for best performance.

Build time: ~3-5 minutes (first build)
Subsequent builds: ~2-3 minutes (with layer caching)

### Apple Silicon Macs (M1/M2/M3/M4)
✅ **Fully supported** — All base images have native ARM64 builds.

**How it works:**
- Dockerfile `ARG TARGETARCH` detects ARM architecture
- Dependencies compile for ARM64 automatically
- No emulation needed (native performance)

Build time: ~3-5 minutes (first build, same as Intel)
Subsequent builds: ~2-3 minutes (with layer caching)

## Docker Desktop Configuration

### Installation

1. Download [Docker Desktop for macOS](https://www.docker.com/products/docker-desktop)
2. Double-click the installer and follow prompts
3. Authorize installation with your password
4. Verify installation:

```bash
docker --version
docker compose version
docker run hello-world
```

### Resource Allocation

For optimal performance, configure Docker Desktop resources:

1. Open **Docker** menu → **Preferences**
2. Navigate to **Resources** tab
3. Set the following:

**Memory:** 16GB (minimum 8GB)
**CPU:** 4 cores (minimum 2)
**Disk Image Size:** 60GB (minimum 30GB)
**Swap:** 1GB

4. Click **Apply & Restart**

### Network Settings

**Important:** Docker Desktop on macOS uses a virtual Linux VM for containers. By default:
- Container-to-container networking: Uses container DNS names (e.g., `postgres:5432`)
- Host-to-container networking: Use `localhost` (e.g., `http://localhost:8080`)
- Container-to-host networking: Not directly supported (use Docker Desktop networking)

This is already configured correctly in our `docker-compose.yml`.

## Startup Procedures

### First-Time Setup

```bash
# Clone the repository
git clone <repo-url>
cd AI-Resume-Analyser

# Start all services (includes building images)
docker compose up --build

# Wait for this output:
# Container ai-resume-analyser-nginx-1 ... Started
# All services should show "Healthy" status
```

**What happens:**
1. Pulls base images from Docker Hub
2. Builds 8 custom images (takes ~3-5 minutes)
3. Starts all 11 services
4. Runs database migrations automatically
5. All services reach "Healthy" status

### Subsequent Starts

```bash
# If services are stopped but images are built
docker compose up

# If you made code changes
docker compose up --build

# If you want fresh volumes
docker compose down -v && docker compose up --build
```

### Stopping Services

```bash
# Stop all services (preserves data)
docker compose down

# Stop and remove all data
docker compose down -v

# Restart specific service
docker compose restart <service-name>
```

## Verification

### Check Services Status

```bash
docker compose ps

# Expected output: All services should show status "Up X seconds" or "Up X seconds (healthy)"
```

### Test API Endpoints

```bash
# Register a new user
curl -X POST http://localhost:8080/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email":"testuser@example.com",
    "password":"Test123!",
    "full_name":"Test User",
    "role":"student"
  }'

# Expected: 201 Created with access_token in response
```

### Access Frontend

```bash
# Open in browser
open http://localhost:8080

# Or use curl
curl -sS http://localhost:8080/ | head -20
```

### View Logs

```bash
# All services
docker compose logs

# Specific service
docker compose logs auth-service

# Follow logs (live)
docker compose logs -f api-gateway

# Last 50 lines
docker compose logs --tail=50
```

## Architecture Detection (Advanced)

The system automatically detects your macOS architecture and builds the correct images.

### How It Works

1. **Dockerfiles use `ARG TARGETARCH`:**
   ```dockerfile
   ARG TARGETARCH
   RUN if [ "$TARGETARCH" = "arm64" ]; then
     # Download aarch64 binary
   else
     # Download x86_64 binary
   fi
   ```

2. **Docker automatically sets `TARGETARCH` based on your platform**

3. **If you need to force amd64 emulation:**
   ```bash
   DOCKER_DEFAULT_PLATFORM=linux/amd64 docker compose up --build
   ```

## Environment Variables

### Docker Compose Environment

The system loads environment variables from:
1. `.env` (if present, takes priority)
2. `.env.example` (fallback)
3. Hardcoded defaults in `docker-compose.yml`

### Setting Custom Values

Create a `.env` file in the project root:

```bash
# Copy the example as a starting point
cp .env.example .env

# Edit as needed
nano .env
```

**Common variables:**
- `POSTGRES_USER`: Database user (default: resume)
- `POSTGRES_PASSWORD`: Database password (default: resume_pass)
- `POSTGRES_DB`: Database name (default: resume_ai)
- `OPENROUTER_API_KEY`: Optional, for AI features
- `MINIO_ACCESS_KEY`: MinIO admin user
- `MINIO_SECRET_KEY`: MinIO admin password

### Loading Custom Environment

```bash
# Use specific env file
docker compose --env-file .env.custom up

# Or set variables before running
export POSTGRES_PASSWORD=my_secure_pass
docker compose up
```

## Troubleshooting

### Services Won't Start

**Symptoms:** Services show "Exited" or "Error" status

**Solutions:**

```bash
# View logs
docker compose logs <service-name>

# Restart service
docker compose restart <service-name>

# Full restart
docker compose down && docker compose up --build
```

### Permission Denied Errors

**Symptoms:** `permission denied` or `cannot connect to Docker daemon`

**Solutions:**

```bash
# Verify Docker is running
docker ps

# Restart Docker Desktop
# Menu > Restart Docker Desktop

# If persistent, reinstall Docker Desktop
```

### Memory/Performance Issues

**Symptoms:** Services slow to start, containers crash, builds take >10 minutes

**Solutions:**

1. Check Docker Desktop resource allocation:
   - Increase Memory to 16GB
   - Increase CPU to 4 cores
   - Increase Disk to 60GB

2. Clean up unused Docker resources:
   ```bash
   docker system prune -a --volumes
   ```

3. Check system resources:
   ```bash
   # macOS Activity Monitor
   # or
   docker stats
   ```

### Port Already in Use

**Symptoms:** `bind: address already in use`

**Solutions:**

```bash
# Find what's using the port
lsof -i :8080

# Kill the process
kill -9 <PID>

# Or change ports in docker-compose.yml
# Change:
# ports:
#   - "8081:8080"  # Use 8081 instead
```

### Database Won't Initialize

**Symptoms:** Auth service can't connect to postgres

**Solutions:**

```bash
# Check postgres status
docker compose logs postgres

# Wait longer (migrations take time on first run)
docker compose up

# If stuck, reset postgres
docker compose down -v
docker compose up --build
```

### Frontend Shows Blank Page

**Symptoms:** `http://localhost:8080` loads but shows nothing

**Solutions:**

```bash
# Check nginx is running
docker compose ps nginx

# View nginx logs
docker compose logs nginx

# Check frontend built successfully
docker compose logs frontend

# Rebuild frontend
docker compose build --no-cache frontend
docker compose up
```

### API Returns 502 Bad Gateway

**Symptoms:** API calls fail with 502 error

**Solutions:**

```bash
# Check API service health
docker compose ps api-gateway auth-service

# View API logs
docker compose logs api-gateway

# Ensure migrations ran
docker compose ps migrations

# Restart API
docker compose restart api-gateway
```

### "no matching manifest" Error

**Symptoms:** `no matching manifest for linux/arm64` (on Apple Silicon)

This is rare but can happen with third-party images that don't support ARM64.

**Solution:**

```bash
# Force Intel/x86_64 emulation (slower)
DOCKER_DEFAULT_PLATFORM=linux/amd64 docker compose up --build
```

## Performance Tips

### For Apple Silicon (M1/M2/M3)

1. **Use native ARM64 images** (default, no action needed)
   - Much faster than amd64 emulation
   - Build time: ~3-5 minutes

2. **Allocate sufficient resources:**
   - Memory: 16GB minimum
   - CPU: 4+ cores recommended

3. **Use local volumes** (default, no action needed)
   - `/var/lib/postgresql` is on macOS filesystem
   - File access is optimized

### For Intel Macs

1. **Similar setup to Apple Silicon**
   - No special considerations needed
   - Native x86_64 performance

2. **Performance is generally equivalent to Linux**

## Deployment Notes

### Docker Hub Authentication

If you need to push custom images:

```bash
docker login

# Build and tag
docker build -t your-username/ai-resume-analyzer:latest .

# Push
docker push your-username/ai-resume-analyzer:latest
```

### Kubernetes Deployment (Future)

The `infra/k8s/` directory contains Kubernetes manifests for production deployment.

**Note:** These are for reference only. The ConfigMaps need to be updated with your actual domain/URLs before deploying.

See [infra/k8s/README.md](infra/k8s/README.md) for details.

## Development Workflow

### Making Code Changes

```bash
# Edit code locally
# ...

# Rebuild affected service
docker compose build api-gateway

# Restart service
docker compose restart api-gateway

# View logs
docker compose logs -f api-gateway
```

### Database Schema Changes

```bash
# Changes are in services/auth-service/alembic/versions/

# Create migration
docker compose exec migrations alembic revision --autogenerate -m "your message"

# Apply migration
docker compose exec migrations alembic upgrade head
```

### Adding Dependencies

```bash
# Edit services/*/pyproject.toml

# Rebuild
docker compose build <service-name>

# Restart
docker compose restart <service-name>
```

## Monitoring & Debugging

### View Real-Time Metrics

```bash
docker stats

# Shows memory, CPU, network usage per container
```

### Inspect Container

```bash
# Run shell in container
docker compose exec auth-service /bin/bash

# Run command in container
docker compose exec postgres psql -U resume -d resume_ai -c "SELECT * FROM user;"
```

### Save Logs

```bash
# Export all logs
docker compose logs > application.log

# Export specific service
docker compose logs postgres > postgres.log
```

## Production Deployment

### Before Going Live

1. **Update `.env.example` with all required variables**
2. **Create production `.env` with secrets**
3. **Update Kubernetes CORS_ORIGINS** in `infra/k8s/api-gateway.yaml`
4. **Use production database** (not SQLite or local postgres)
5. **Set up SSL/TLS** with proper certificates
6. **Configure backups** for postgres and minio volumes
7. **Set resource limits** on all containers

### Environment Variables for Production

```bash
# Create secure .env
POSTGRES_PASSWORD=<generate-strong-password>
MINIO_ACCESS_KEY=<generate-strong-key>
MINIO_SECRET_KEY=<generate-strong-secret>
JWT_SECRET=<generate-strong-secret>
OPENROUTER_API_KEY=<your-api-key>
```

### Backing Up Data

```bash
# Backup postgres
docker compose exec postgres pg_dump -U resume resume_ai > backup.sql

# Backup minio
docker compose exec minio mc mirror --remove local/resumes /backups/resumes
```

## Support & Resources

- [Docker Desktop for Mac Documentation](https://docs.docker.com/desktop/mac/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Docker Multi-Architecture Builds](https://docs.docker.com/build/building/multi-platform/)

## Checklist

- [ ] Docker Desktop installed and running
- [ ] Git cloned the repository
- [ ] Ran `docker compose up --build`
- [ ] All services show "Healthy" or "Up"
- [ ] Frontend accessible at `http://localhost:8080`
- [ ] API registration endpoint works
- [ ] Created `.env` with custom values (optional)

## Summary

The AI Resume Analyzer is fully compatible with macOS and supports both Intel and Apple Silicon architectures natively. The system is designed to work out-of-the-box with a single command.

**Total setup time:** 3-5 minutes (first run includes image builds)
**Subsequent starts:** 30 seconds (if volumes preserved)

For questions or issues, refer to the troubleshooting section or check the main [README.md](README.md).
