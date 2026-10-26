# Home Assistant's supported Debian base includes Bashio and S6.
# Debian/glibc lets NumPy, Numba and llvmlite install manylinux wheels.
ARG BASE_IMAGE=ghcr.io/home-assistant/base-debian:trixie
FROM ${BASE_IMAGE} AS builder

RUN apt-get update && apt-get install -y --no-install-recommends \
    python3 python3-venv python3-dev build-essential pkg-config \
    && rm -rf /var/lib/apt/lists/*

# Keep Poetry separate from the application dependencies.
RUN python3 -m venv /opt/poetry \
    && /opt/poetry/bin/pip install --no-cache-dir 'poetry==2.2.1'
RUN python3 -m venv /opt/venv

ENV VIRTUAL_ENV=/opt/venv \
    PATH="/opt/venv/bin:/opt/poetry/bin:${PATH}" \
    POETRY_VIRTUALENVS_CREATE=false

WORKDIR /build
COPY pyproject.toml poetry.lock ./
RUN poetry install --only main --no-root --no-interaction --no-ansi \
    && python -c 'import llvmlite, numba; from pipecat.services.openai.realtime.llm import OpenAIRealtimeLLMService'

FROM ${BASE_IMAGE}
RUN apt-get update && apt-get install -y --no-install-recommends \
    python3 ffmpeg libsndfile1 libgomp1 \
    && rm -rf /var/lib/apt/lists/*

COPY --from=builder /opt/venv /opt/venv
ENV VIRTUAL_ENV=/opt/venv \
    PATH="/opt/venv/bin:${PATH}" \
    PYTHONUNBUFFERED=1

COPY app/ /app/
COPY root/ /
RUN chmod +x /run.sh
WORKDIR /
CMD ["/run.sh"]
