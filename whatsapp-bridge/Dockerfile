# ==============================================================================
# WhatsApp Bridge Dockerfile
# Multi-stage build for Go application with CGO and SQLite support
# ==============================================================================

# ------------------------------------------------------------------------------
# Stage 1: Builder - Compile Go binary with CGO enabled
# ------------------------------------------------------------------------------
FROM golang:1.24-alpine AS builder

# Install build dependencies for CGO and SQLite
RUN apk add --no-cache \
    gcc \
    musl-dev \
    sqlite-dev \
    ca-certificates

WORKDIR /build

# Copy dependency files first for better layer caching
COPY go.mod go.sum ./

# Update and download dependencies (regenerates go.sum if needed)
RUN go mod tidy && go mod download

# Copy source code
COPY . .

# Build with CGO enabled for SQLite support
# Let Go auto-detect architecture (works for both amd64 and arm64)
ENV CGO_ENABLED=1

RUN go build -ldflags="-s -w" -o whatsapp-bridge .

# ------------------------------------------------------------------------------
# Stage 2: Runtime - Minimal Alpine image with only runtime dependencies
# ------------------------------------------------------------------------------
FROM alpine:3.21

# Install runtime dependencies
RUN apk add --no-cache \
    sqlite-libs \
    ca-certificates \
    tzdata

# Create non-root user for security best practice
RUN addgroup -g 1000 appgroup && \
    adduser -u 1000 -G appgroup -h /app -D appuser

WORKDIR /app

# Create store directory for persistent data
RUN mkdir -p /app/store && chown -R appuser:appgroup /app

# Copy binary from builder stage
COPY --from=builder /build/whatsapp-bridge /app/whatsapp-bridge

# Set ownership
RUN chown appuser:appgroup /app/whatsapp-bridge

# Switch to non-root user
USER appuser

# Volume mount point for persistent SQLite databases
# - whatsapp.db: Session/authentication data
# - messages.db: Message history
VOLUME ["/app/store"]

# Expose port for internal API (not exposed to public)
EXPOSE 8080

# Health check for container orchestration
HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:8080/api/send 2>/dev/null || exit 1

# Run the bridge
CMD ["./whatsapp-bridge"]
