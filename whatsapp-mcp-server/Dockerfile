# ==============================================================================
# WhatsApp MCP Server Dockerfile
# Python application with uv package manager and ffmpeg for audio processing
# ==============================================================================

FROM python:3.11-slim

# Install system dependencies
# - ffmpeg: Required for audio format conversion (voice messages)
# - sqlite3: For database operations
# - curl: For health checks and debugging
RUN apt-get update && apt-get install -y --no-install-recommends \
    ffmpeg \
    sqlite3 \
    curl \
    && rm -rf /var/lib/apt/lists/* \
    && apt-get clean

# Install uv package manager (fast Python package installer)
RUN pip install --no-cache-dir uv

# Create non-root user for security best practice
RUN groupadd -g 1000 appgroup && \
    useradd -u 1000 -g appgroup -m -d /app appuser

WORKDIR /app

# Create store directory (will be mounted from volume)
RUN mkdir -p /app/store && chown -R appuser:appgroup /app

# Copy dependency files first for better layer caching
COPY --chown=appuser:appgroup pyproject.toml uv.lock .python-version ./

# Switch to non-root user before installing dependencies
USER appuser

# Install Python dependencies using uv
RUN uv sync --frozen

# Copy application source code
COPY --chown=appuser:appgroup *.py ./

# Environment variables for container networking
# WHATSAPP_BRIDGE_URL: URL to WhatsApp Bridge API (set in docker-compose)
# MESSAGES_DB_PATH: Path to SQLite database (mounted volume)
ENV WHATSAPP_BRIDGE_URL="http://whatsapp-bridge:8080"
ENV MESSAGES_DB_PATH="/app/store/messages.db"

# Volume mount point (shared with bridge container)
VOLUME ["/app/store"]

# Expose MCP HTTP port for SSE transport
EXPOSE 8000

# MCP Server runs via SSE transport for remote connections
# Use --transport stdio for local/stdio mode
CMD ["uv", "run", "main.py", "--transport", "sse", "--host", "0.0.0.0", "--port", "8000"]
