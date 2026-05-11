FROM python:3.12-slim

ARG UV_VERSION=0.11.13

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
ENV PYTHONPATH=/app/libs/shared
ENV VIRTUAL_ENV=/app/.venv
ENV PATH="/app/.venv/bin:$PATH"

WORKDIR /app

RUN apt-get update && apt-get install -y --no-install-recommends \
	build-essential \
	curl \
	ca-certificates \
	&& rm -rf /var/lib/apt/lists/*

RUN set -eux; \
	arch="$(uname -m)"; \
	case "$arch" in \
		x86_64) uv_arch="x86_64" ;; \
		aarch64|arm64) uv_arch="aarch64" ;; \
		*) echo "Unsupported architecture: $arch" >&2; exit 1 ;; \
	esac; \
	curl -LsSf -o /tmp/uv.tar.gz "https://github.com/astral-sh/uv/releases/download/${UV_VERSION}/uv-${uv_arch}-unknown-linux-gnu.tar.gz"; \
	tar -xzf /tmp/uv.tar.gz -C /tmp; \
	mv "/tmp/uv-${uv_arch}-unknown-linux-gnu/uv" /usr/local/bin/uv; \
	mv "/tmp/uv-${uv_arch}-unknown-linux-gnu/uvx" /usr/local/bin/uvx; \
	rm -rf "/tmp/uv-${uv_arch}-unknown-linux-gnu" /tmp/uv.tar.gz

COPY services/ranking-service/pyproject.toml services/ranking-service/uv.lock /app/services/ranking-service/

WORKDIR /app/services/ranking-service
RUN uv venv "$VIRTUAL_ENV" \
	&& uv sync --frozen --no-dev --active --no-install-project

WORKDIR /app
COPY libs/shared /app/libs/shared
COPY services/ranking-service /app/services/ranking-service

WORKDIR /app/services/ranking-service

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
