# Docker

Containerization platform for building, shipping, and running applications.

## Dockerfile Best Practices

### General Guidelines

1. **Use specific base image tags** with SHA256 digest for reproducibility
2. **Copy dependency files first** to leverage layer caching
3. **Don't run containers as root** - use non-privileged users
4. **Use multi-stage builds** to reduce final image size

---

## Node.js Dockerfile

```dockerfile
FROM node:20.9.0-bullseye-slim

# Set production environment
ENV NODE_ENV production

WORKDIR /usr/src/app

# Copy package files first (layer caching)
COPY package*.json ./

# Clean install production dependencies only
RUN npm ci --only=production

# Copy app source with proper ownership
COPY --chown=node:node . .

# Run as non-root user
USER node

# Use node directly (not npm start) for proper signal handling
CMD ["node", "server.js"]
```

> ⚠️ **Important:** Don't use `CMD "npm" "start"` - npm doesn't forward SIGTERM signals properly, causing ungraceful shutdowns.

### Why Avoid npm/yarn in CMD?
- When using `npm start`, npm becomes PID 1
- Docker sends SIGTERM to PID 1 on shutdown
- npm doesn't forward signals to your app
- Results in forced SIGKILL after timeout (data loss risk)

---

## Python Dockerfile

```dockerfile
FROM python:3.10-slim

WORKDIR /app

# Copy requirements first (layer caching)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Use production WSGI server
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "manage:app"]
```

### Production Servers

| Server | Type | Use Case |
|--------|------|----------|
| **Gunicorn** | WSGI | Flask, Django (sync) |
| **Uvicorn** | ASGI | FastAPI, Starlette (async) |
| **Gunicorn + Uvicorn** | Hybrid | Production async apps |

```dockerfile
# FastAPI with Uvicorn workers
CMD ["gunicorn", "-k", "uvicorn.workers.UvicornWorker", "-b", "0.0.0.0:8000", "main:app"]
```

---

## Java Dockerfile (Multi-Stage)

```dockerfile
# Stage 1: Build with JDK
FROM maven:3.9.6-eclipse-temurin-17 AS builder
WORKDIR /app

# Copy pom.xml first (dependency caching)
COPY pom.xml .
RUN mvn dependency:go-offline -B

# Copy source and build
COPY src ./src
RUN mvn clean package -DskipTests

# Stage 2: Run with lightweight JRE
FROM eclipse-temurin:17-jre
WORKDIR /app

COPY --from=builder /app/target/*.jar app.jar

ENTRYPOINT ["java", "-jar", "app.jar"]
```

### JDK vs JRE

| Component | Purpose | Use In |
|-----------|---------|--------|
| **JDK** | Development (compiler, debugger) | Build stage |
| **JRE** | Runtime only (JVM, libraries) | Run stage |

---

## Security Best Practices

```dockerfile
# Use non-root user
RUN useradd -m appuser
USER appuser

# Use image digest for immutability
FROM node:20-alpine@sha256:abc123...

# Don't store secrets in image
# Use build args or runtime secrets instead
```

---

## Common Commands

```bash
docker build -t myapp:latest .
docker run -d -p 3000:3000 myapp:latest
docker images
docker ps
docker logs <container_id>
docker exec -it <container_id> /bin/sh
```
