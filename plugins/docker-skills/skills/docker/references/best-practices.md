# Docker Best Practices Reference

## Security

### Non-root User
Always run containers as non-root:
```dockerfile
RUN addgroup --system --gid 1001 appgroup && \
    adduser --system --uid 1001 --gid 1001 appuser
USER appuser
```

### Specific Image Tags
Never use `latest`. Pin to specific versions:
```dockerfile
# Good
FROM node:20.11-alpine3.19
FROM python:3.12.1-slim-bookworm

# Bad
FROM node:latest
FROM python
```

### Minimal Base Images
Prefer in order: distroless > alpine > slim > full
- `gcr.io/distroless/static-debian12` - Static binaries (Go, Rust)
- `gcr.io/distroless/base-debian12` - Dynamic binaries
- `*-alpine` - ~5MB, uses musl libc
- `*-slim` - ~80MB, uses glibc

### No Secrets in Images
- Use build-time secrets with `--mount=type=secret`
- Use runtime environment variables
- Never `COPY` credential files

### HEALTHCHECK
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1
```

## Performance

### Multi-stage Builds
Separate build and runtime:
```dockerfile
# Build stage
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Runtime stage
FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/index.js"]
```

### Layer Caching
Copy dependencies before source code:
```dockerfile
# Dependencies change less frequently
COPY package*.json ./
RUN npm ci

# Source changes frequently
COPY . .
```

### Combine RUN Commands
```dockerfile
# Good - single layer
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/*

# Bad - multiple layers
RUN apt-get update
RUN apt-get install -y curl
```

### .dockerignore
Always include to exclude:
- `node_modules/`, `__pycache__/`, `venv/`
- `.git/`, `.env`, `*.log`
- Test files, documentation

## Maintainability

### LABEL Metadata
```dockerfile
LABEL org.opencontainers.image.title="My App" \
      org.opencontainers.image.version="1.0.0" \
      org.opencontainers.image.description="Application description" \
      org.opencontainers.image.source="https://github.com/org/repo"
```

### EXPOSE Documentation
```dockerfile
EXPOSE 8080/tcp
EXPOSE 9090/tcp
```

### ARG and ENV
```dockerfile
ARG NODE_ENV=production
ENV NODE_ENV=${NODE_ENV}
ENV PORT=8080
```

### WORKDIR
Always use absolute paths:
```dockerfile
WORKDIR /app
```
