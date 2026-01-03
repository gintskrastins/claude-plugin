---
name: docker
description: |
  Generate production-ready Dockerfiles and docker-compose.yml files following 2025 best practices.
  Use when: (1) Creating or generating a Dockerfile, (2) Dockerizing or containerizing an application,
  (3) Setting up docker-compose for development or production, (4) Optimizing Docker images for size
  or security, (5) Creating multi-stage builds, (6) Questions about Docker best practices, layer
  caching, or security hardening.
---

# Docker

Generate optimized, secure Docker configurations following current best practices.

## Dockerfile Workflow

1. **Detect project type** by checking for:
   - `package.json` → Node.js
   - `requirements.txt` / `pyproject.toml` → Python
   - `go.mod` → Go
   - `Cargo.toml` → Rust
   - `pom.xml` / `build.gradle` → Java
   - See [dockerfile-patterns.md](references/dockerfile-patterns.md) for all patterns

2. **Generate Dockerfile** with these requirements:
   - Multi-stage build (builder + runtime stages)
   - Non-root user for runtime
   - Specific base image tags (never `latest`)
   - Minimal base images (alpine/slim/distroless)
   - Layer caching (copy dependencies before source)
   - HEALTHCHECK instruction
   - LABEL metadata

3. **Generate .dockerignore** using [dockerignore-template](assets/dockerignore-template) as base

## Docker Compose Workflow

1. **Identify services needed**:
   - Application service(s)
   - Database (postgres, mysql, mongo)
   - Cache (redis)
   - Reverse proxy if needed

2. **Generate docker-compose.yml** with:
   - Health checks for all services
   - Named volumes for data persistence
   - Secrets management (never plain text passwords)
   - Appropriate networking
   - Resource limits for production

3. For development, create separate `docker-compose.dev.yml` with:
   - Volume mounts for hot reload
   - Debug ports exposed
   - Development environment variables

See [compose-patterns.md](references/compose-patterns.md) for service templates.

## Quick Reference

### Security Checklist
- [ ] Non-root USER
- [ ] Pinned base image versions
- [ ] No secrets in image
- [ ] HEALTHCHECK defined
- [ ] Minimal base image

### Performance Checklist
- [ ] Multi-stage build
- [ ] Dependencies copied before source
- [ ] RUN commands combined
- [ ] .dockerignore present

See [best-practices.md](references/best-practices.md) for detailed guidance.
