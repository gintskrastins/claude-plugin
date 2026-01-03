# Docker Compose Patterns

## Basic Structure

```yaml
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    depends_on:
      - db
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  db:
    image: postgres:16-alpine
    volumes:
      - db_data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password

volumes:
  db_data:

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

## Development vs Production

### Development (docker-compose.dev.yml)
```yaml
services:
  app:
    build:
      context: .
      target: development
    volumes:
      - .:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - DEBUG=*
    ports:
      - "3000:3000"
      - "9229:9229"  # Debug port
    command: npm run dev
```

### Production (docker-compose.prod.yml)
```yaml
services:
  app:
    build:
      context: .
      target: production
    restart: unless-stopped
    environment:
      - NODE_ENV=production
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

## Common Service Patterns

### Database Services

#### PostgreSQL
```yaml
postgres:
  image: postgres:16-alpine
  volumes:
    - postgres_data:/var/lib/postgresql/data
  environment:
    POSTGRES_DB: myapp
    POSTGRES_USER: myapp
    POSTGRES_PASSWORD_FILE: /run/secrets/db_password
  healthcheck:
    test: ["CMD-SHELL", "pg_isready -U myapp"]
    interval: 10s
    timeout: 5s
    retries: 5
```

#### MySQL
```yaml
mysql:
  image: mysql:8.0
  volumes:
    - mysql_data:/var/lib/mysql
  environment:
    MYSQL_DATABASE: myapp
    MYSQL_ROOT_PASSWORD_FILE: /run/secrets/mysql_root_password
  healthcheck:
    test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
    interval: 10s
    timeout: 5s
    retries: 5
```

#### MongoDB
```yaml
mongo:
  image: mongo:7.0
  volumes:
    - mongo_data:/data/db
  environment:
    MONGO_INITDB_ROOT_USERNAME: root
    MONGO_INITDB_ROOT_PASSWORD_FILE: /run/secrets/mongo_password
```

### Cache Services

#### Redis
```yaml
redis:
  image: redis:7-alpine
  volumes:
    - redis_data:/data
  command: redis-server --appendonly yes
  healthcheck:
    test: ["CMD", "redis-cli", "ping"]
    interval: 10s
    timeout: 5s
    retries: 5
```

### Message Queues

#### RabbitMQ
```yaml
rabbitmq:
  image: rabbitmq:3-management-alpine
  ports:
    - "5672:5672"
    - "15672:15672"
  volumes:
    - rabbitmq_data:/var/lib/rabbitmq
  environment:
    RABBITMQ_DEFAULT_USER: myapp
    RABBITMQ_DEFAULT_PASS_FILE: /run/secrets/rabbitmq_password
```

### Reverse Proxy

#### Nginx
```yaml
nginx:
  image: nginx:alpine
  ports:
    - "80:80"
    - "443:443"
  volumes:
    - ./nginx.conf:/etc/nginx/nginx.conf:ro
    - ./certs:/etc/nginx/certs:ro
  depends_on:
    - app
```

#### Traefik
```yaml
traefik:
  image: traefik:v3.0
  command:
    - "--api.insecure=true"
    - "--providers.docker=true"
    - "--entrypoints.web.address=:80"
  ports:
    - "80:80"
    - "8080:8080"
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock:ro
```

## Networking

```yaml
services:
  app:
    networks:
      - frontend
      - backend

  db:
    networks:
      - backend

  nginx:
    networks:
      - frontend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true  # No external access
```

## Volume Best Practices

```yaml
volumes:
  db_data:
    driver: local

  # Named volumes for persistence
  app_uploads:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /data/uploads
```
