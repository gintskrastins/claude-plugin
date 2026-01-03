# Dockerfile Patterns by Project Type

## Project Detection

Detect project type by checking for these files:

| File | Project Type |
|------|-------------|
| `package.json` | Node.js |
| `requirements.txt`, `pyproject.toml`, `setup.py` | Python |
| `go.mod` | Go |
| `Cargo.toml` | Rust |
| `pom.xml`, `build.gradle` | Java |
| `*.csproj`, `*.sln` | .NET |
| `Gemfile` | Ruby |
| `composer.json` | PHP |

## Node.js Pattern

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

FROM node:20-alpine
WORKDIR /app

RUN addgroup --system --gid 1001 nodejs && \
    adduser --system --uid 1001 nodejs

COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules

USER nodejs
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s CMD wget -qO- http://localhost:3000/health || exit 1
CMD ["node", "dist/index.js"]
```

## Python Pattern

```dockerfile
FROM python:3.12-slim-bookworm AS builder
WORKDIR /app

RUN pip install --no-cache-dir pip-tools
COPY requirements*.txt ./
RUN pip wheel --no-cache-dir --wheel-dir /wheels -r requirements.txt

FROM python:3.12-slim-bookworm
WORKDIR /app

RUN useradd --create-home --shell /bin/bash appuser

COPY --from=builder /wheels /wheels
RUN pip install --no-cache-dir /wheels/* && rm -rf /wheels

COPY --chown=appuser:appuser . .

USER appuser
EXPOSE 8000
HEALTHCHECK --interval=30s --timeout=3s CMD curl -f http://localhost:8000/health || exit 1
CMD ["python", "-m", "uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## Go Pattern

```dockerfile
FROM golang:1.22-alpine AS builder
WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-w -s" -o /app/server ./cmd/server

FROM gcr.io/distroless/static-debian12
WORKDIR /app

COPY --from=builder /app/server .

USER nonroot:nonroot
EXPOSE 8080
ENTRYPOINT ["/app/server"]
```

## Rust Pattern

```dockerfile
FROM rust:1.75-slim-bookworm AS builder
WORKDIR /app

COPY Cargo.toml Cargo.lock ./
RUN mkdir src && echo "fn main() {}" > src/main.rs
RUN cargo build --release && rm -rf src

COPY . .
RUN touch src/main.rs && cargo build --release

FROM gcr.io/distroless/cc-debian12
WORKDIR /app

COPY --from=builder /app/target/release/myapp .

USER nonroot:nonroot
EXPOSE 8080
ENTRYPOINT ["/app/myapp"]
```

## Java (Maven) Pattern

```dockerfile
FROM eclipse-temurin:21-jdk-alpine AS builder
WORKDIR /app

COPY pom.xml .
COPY .mvn .mvn
COPY mvnw .
RUN ./mvnw dependency:go-offline

COPY src ./src
RUN ./mvnw package -DskipTests

FROM eclipse-temurin:21-jre-alpine
WORKDIR /app

RUN addgroup --system --gid 1001 java && \
    adduser --system --uid 1001 java

COPY --from=builder /app/target/*.jar app.jar
RUN chown java:java app.jar

USER java
EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=3s CMD wget -qO- http://localhost:8080/actuator/health || exit 1
ENTRYPOINT ["java", "-jar", "app.jar"]
```

## Generic Pattern

For unknown project types, create a minimal image:

```dockerfile
FROM alpine:3.19
WORKDIR /app

RUN addgroup --system --gid 1001 appgroup && \
    adduser --system --uid 1001 --gid 1001 appuser

COPY --chown=appuser:appuser . .

USER appuser
EXPOSE 8080
CMD ["/app/start.sh"]
```
