---
description: Comprehensive Docker security patterns including non-root users, secrets management, cloud integration, and network isolation
globs: Dockerfile, docker-compose.yml, docker-compose.*.yml
alwaysApply: false
---

# Docker Security Standards

Comprehensive security patterns for containerized applications with modern Docker features, cloud secrets integration, and defense-in-depth strategies.

## Core Security Principles

1. **Never run as root** - Use non-root users in all production containers
2. **No secrets in layers** - Use BuildKit secrets or runtime secrets, never ENV/ARG
3. **Minimal attack surface** - Use slim/distroless base images
4. **Network isolation** - Segment networks by security tier
5. **Resource constraints** - Prevent DoS with limits
6. **Least privilege** - Grant minimum necessary permissions
7. **Cloud secrets** - Use managed secrets services for production

## Non-Root User Patterns

### Python (uv) Applications

```dockerfile
# syntax=docker/dockerfile:1
FROM python:3.12-slim AS base

# Create non-root user early
RUN groupadd -r appuser && useradd -r -g appuser -u 1000 appuser

WORKDIR /app

# Install uv as root
COPY --from=ghcr.io/astral-sh/uv:latest /uv /usr/local/bin/uv

# Copy dependency files
COPY --chown=appuser:appuser pyproject.toml uv.lock ./

# Install dependencies with cache mount (still as root for system packages if needed)
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --frozen --no-dev

# Copy application code with proper ownership
COPY --chown=appuser:appuser . .

# Switch to non-root user BEFORE CMD
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/health')" || exit 1

CMD ["uv", "run", "uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Node.js Applications

```dockerfile
# syntax=docker/dockerfile:1
FROM node:20-slim AS base

# Create non-root user with specific UID/GID
RUN groupadd -g 1001 -r nodejs && \
    useradd -r -u 1001 -g nodejs nodejs

WORKDIR /app

# Dependencies stage (as root for system packages)
FROM base AS deps

RUN apt-get update && apt-get install -y --no-install-recommends \
    python3 make g++ && \
    rm -rf /var/lib/apt/lists/*

COPY package.json package-lock.json ./
RUN --mount=type=cache,target=/root/.npm \
    npm ci --only=production

# Production stage
FROM base AS production

# Copy node_modules with proper ownership
COPY --from=deps --chown=nodejs:nodejs /app/node_modules ./node_modules

# Copy application code with ownership
COPY --chown=nodejs:nodejs . .

# Switch to non-root user
USER nodejs

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s \
  CMD node healthcheck.js || exit 1

CMD ["node", "server.js"]
```

### Java (Spring Boot) Applications

```dockerfile
# syntax=docker/dockerfile:1
FROM eclipse-temurin:21-jdk-jammy AS builder

WORKDIR /app

COPY gradle gradle
COPY gradlew build.gradle settings.gradle ./
RUN --mount=type=cache,target=/root/.gradle \
    ./gradlew dependencies --no-daemon

COPY src src
RUN --mount=type=cache,target=/root/.gradle \
    ./gradlew bootJar --no-daemon && \
    java -Djarmode=layertools -jar build/libs/*.jar extract

# Runtime stage with non-root user
FROM eclipse-temurin:21-jre-jammy

# Create spring user
RUN groupadd -r spring && useradd -r -g spring -u 1000 spring

WORKDIR /app

# Copy layers with ownership
COPY --from=builder --chown=spring:spring /app/dependencies/ ./
COPY --from=builder --chown=spring:spring /app/spring-boot-loader/ ./
COPY --from=builder --chown=spring:spring /app/snapshot-dependencies/ ./
COPY --from=builder --chown=spring:spring /app/application/ ./

# Switch to non-root user
USER spring

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=3s --start-period=40s \
  CMD curl -f http://localhost:8080/actuator/health || exit 1

ENTRYPOINT ["java", "org.springframework.boot.loader.launch.JarLauncher"]
```

### Go Applications (Static Binaries)

```dockerfile
# syntax=docker/dockerfile:1
FROM golang:1.21-alpine AS builder

WORKDIR /src

COPY go.mod go.sum ./
RUN --mount=type=cache,target=/root/.cache/go-build \
    --mount=type=cache,target=/go/pkg/mod \
    go mod download

COPY . .

# Build static binary with CGO disabled
RUN --mount=type=cache,target=/root/.cache/go-build \
    --mount=type=cache,target=/go/pkg/mod \
    CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o /app .

# Minimal runtime with scratch (no shell, no package manager, no root user)
FROM scratch

# Copy CA certificates for HTTPS
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

# Copy binary
COPY --from=builder /app /app

# Use non-root UID (no user creation possible in scratch)
USER 65534:65534

EXPOSE 8080

# Health check not possible in scratch without shell - use in compose instead

ENTRYPOINT ["/app"]
```

## BuildKit Secrets (Build-Time)

### Using Secrets During Build (Not in Layers)

```dockerfile
# syntax=docker/dockerfile:1
FROM python:3.12-slim

# Access secret during build without persisting in layer
RUN --mount=type=secret,id=pip_config \
    pip config set global.index-url $(cat /run/secrets/pip_config)

# Install from private repository using secret
RUN --mount=type=secret,id=github_token \
    pip install git+https://$(cat /run/secrets/github_token)@github.com/private/repo.git

# Secret is NOT in any layer
```

**Build command:**
```bash
docker build \
  --secret id=pip_config,src=~/.pip/pip.conf \
  --secret id=github_token,env=GITHUB_TOKEN \
  .
```

### SSH Mount for Private Repositories

```dockerfile
# syntax=docker/dockerfile:1
FROM node:20-slim

# Use SSH agent forwarding (no keys in image)
RUN --mount=type=ssh \
    npm install git+ssh://git@github.com/private/repo.git
```

**Build command:**
```bash
docker buildx build --ssh default .
```

## Docker Compose Secrets (Runtime)

### Development (File-Based Secrets)

```yaml
services:
  api:
    build: .
    secrets:
      - db_password
      - jwt_secret
    environment:
      DB_HOST: postgres
      DB_USER: appuser
      # Password comes from secret, not env var
      DB_PASSWORD_FILE: /run/secrets/db_password
      JWT_SECRET_FILE: /run/secrets/jwt_secret

secrets:
  db_password:
    file: ./secrets/db_password.txt
  jwt_secret:
    file: ./secrets/jwt_secret.txt
```

**Application code (Python example):**
```python
import os
from pathlib import Path

def get_secret(secret_name: str) -> str:
    """Read secret from file or environment variable."""
    secret_file = os.getenv(f"{secret_name}_FILE")
    if secret_file:
        return Path(secret_file).read_text().strip()
    return os.getenv(secret_name, "")

db_password = get_secret("DB_PASSWORD")
jwt_secret = get_secret("JWT_SECRET")
```

### Production (External Secrets)

```yaml
services:
  api:
    secrets:
      - db_password
      - jwt_secret
    environment:
      DB_PASSWORD_FILE: /run/secrets/db_password
      JWT_SECRET_FILE: /run/secrets/jwt_secret

secrets:
  db_password:
    external: true
    name: prod_db_password
  jwt_secret:
    external: true
    name: prod_jwt_secret
```

**Create external secrets (Docker Swarm):**
```bash
echo "my_db_password" | docker secret create prod_db_password -
echo "my_jwt_secret" | docker secret create prod_jwt_secret -
```

## Cloud Secrets Integration

### AWS Secrets Manager (ECS Task Definition)

```json
{
  "family": "app-task",
  "containerDefinitions": [
    {
      "name": "app",
      "image": "myapp:latest",
      "secrets": [
        {
          "name": "DB_PASSWORD",
          "valueFrom": "arn:aws:secretsmanager:region:account:secret:db-password"
        },
        {
          "name": "JWT_SECRET",
          "valueFrom": "arn:aws:secretsmanager:region:account:secret:jwt-secret"
        }
      ]
    }
  ],
  "executionRoleArn": "arn:aws:iam::account:role/ecsTaskExecutionRole",
  "taskRoleArn": "arn:aws:iam::account:role/ecsTaskRole"
}
```

**IAM Policy (ecsTaskExecutionRole):**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetSecretValue",
        "kms:Decrypt"
      ],
      "Resource": [
        "arn:aws:secretsmanager:region:account:secret:db-password*",
        "arn:aws:secretsmanager:region:account:secret:jwt-secret*"
      ]
    }
  ]
}
```

### Azure Key Vault (Container Instances)

```yaml
apiVersion: 2019-12-01
location: eastus
name: app-container-group
properties:
  containers:
  - name: app
    properties:
      image: myapp:latest
      environmentVariables:
      - name: DB_PASSWORD
        secureValue:
          vaultName: my-keyvault
          secretName: db-password
      - name: JWT_SECRET
        secureValue:
          vaultName: my-keyvault
          secretName: jwt-secret
      resources:
        requests:
          cpu: 1
          memoryInGB: 1.5
  identity:
    type: UserAssigned
    userAssignedIdentities:
      /subscriptions/{subscription-id}/resourceGroups/{rg}/providers/Microsoft.ManagedIdentity/userAssignedIdentities/{identity-name}: {}
```

**Grant Key Vault Access:**
```bash
az keyvault set-policy \
  --name my-keyvault \
  --object-id {managed-identity-principal-id} \
  --secret-permissions get
```

### GCP Secret Manager (Cloud Run)

```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: app
spec:
  template:
    metadata:
      annotations:
        run.googleapis.com/execution-environment: gen2
    spec:
      serviceAccountName: app-sa@project.iam.gserviceaccount.com
      containers:
      - image: gcr.io/project/myapp:latest
        env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-password
              key: latest
        - name: JWT_SECRET
          valueFrom:
            secretKeyRef:
              name: jwt-secret
              key: latest
```

**Grant Secret Access:**
```bash
gcloud secrets add-iam-policy-binding db-password \
  --member="serviceAccount:app-sa@project.iam.gserviceaccount.com" \
  --role="roles/secretmanager.secretAccessor"
```

## Network Security and Isolation

### Three-Tier Architecture

```yaml
services:
  # Public tier - External access
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    networks:
      - frontend
    depends_on:
      - api

  # Application tier - Internal only
  api:
    build: .
    networks:
      - frontend
      - backend
    depends_on:
      database:
        condition: service_healthy
    # No ports published - only accessible via nginx

  # Data tier - Most restricted
  database:
    image: postgres:16-alpine
    networks:
      - backend  # Only backend network, no frontend access
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password
    # No ports published - internal only

networks:
  frontend:
    driver: bridge
    driver_opts:
      com.docker.network.bridge.host_binding_ipv4: "127.0.0.1"

  backend:
    driver: bridge
    internal: true  # No external access - critical for security

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

### Port Binding Security

```yaml
services:
  # BAD - Binds to all interfaces (0.0.0.0)
  app_insecure:
    ports:
      - "8080:8080"  # Accessible from anywhere

  # GOOD - Binds to localhost only
  app_secure:
    ports:
      - "127.0.0.1:8080:8080"  # Only accessible from host

  # BEST - No published ports, use reverse proxy
  app_best:
    expose:
      - "8080"  # Internal only, via nginx
    networks:
      - internal
```

## Resource Limits (DoS Prevention)

```yaml
services:
  api:
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          cpus: '0.5'
          memory: 512M
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s

  database:
    deploy:
      resources:
        limits:
          cpus: '4'
          memory: 4G
        reservations:
          cpus: '1'
          memory: 1G
```

## File Permissions and Ownership

```dockerfile
# BAD - Overly permissive
COPY app.py /app/
RUN chmod 777 /app/app.py  # World writable - security risk

# GOOD - Proper ownership and permissions
COPY --chown=appuser:appuser app.py /app/
RUN chmod 644 /app/app.py  # Owner write, group/others read

# BEST - Set ownership during COPY
COPY --chown=appuser:appuser --chmod=644 app.py /app/
```

## Security Scanning and Best Practices

### .dockerignore for Security

```
# Prevent secrets from being copied
.env
.env.local
.env.*.local
*.pem
*.key
*.crt
id_rsa*
.ssh/

# Prevent sensitive files
**/credentials.json
**/secrets.yml
**/config/secrets.yml
.aws/
.gcp/
.azure/

# Version control
.git
.gitignore

# Development
node_modules
.venv
__pycache__
*.pyc
.pytest_cache
coverage/
```

### Image Scanning Commands

```bash
# Scan with Trivy (open source)
trivy image myapp:latest

# Scan with Snyk
snyk container test myapp:latest

# Docker Scout (built-in)
docker scout cves myapp:latest

# Scan for secrets in image
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image --scanners secret myapp:latest
```

### Health Check Security

```yaml
services:
  api:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 3s
      retries: 3
      start_period: 40s

    # Health endpoint should NOT expose:
    # - Environment variables
    # - Secret values
    # - Internal service details
    # - Database connection strings

    # Health endpoint SHOULD expose:
    # - Service status (healthy/unhealthy)
    # - Dependency status (database connected)
    # - Version/build info (safe)
```

## Complete Secure Example (Python FastAPI)

```dockerfile
# syntax=docker/dockerfile:1
FROM python:3.12-slim AS base

ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PIP_NO_CACHE_DIR=1

# Builder stage
FROM base AS builder

WORKDIR /app

# Install uv
COPY --from=ghcr.io/astral-sh/uv:latest /uv /usr/local/bin/uv

# Copy dependency files
COPY pyproject.toml uv.lock ./

# Install dependencies with cache mount
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --frozen --no-dev

# Runtime stage
FROM base AS runtime

# Create non-root user
RUN groupadd -r appuser && useradd -r -g appuser -u 1000 appuser

WORKDIR /app

# Copy virtual environment
COPY --from=builder --chown=appuser:appuser /app/.venv /app/.venv

# Copy application code
COPY --chown=appuser:appuser . .

# Switch to non-root user
USER appuser

ENV PATH="/app/.venv/bin:$PATH"

EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/health')" || exit 1

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**Corresponding docker-compose.yml:**

```yaml
services:
  api:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "127.0.0.1:8000:8000"
    networks:
      - frontend
      - backend
    environment:
      DB_HOST: postgres
      DB_NAME: myapp
      DB_USER: appuser
      DB_PASSWORD_FILE: /run/secrets/db_password
      JWT_SECRET_FILE: /run/secrets/jwt_secret
      REDIS_URL: redis://redis:6379
    secrets:
      - db_password
      - jwt_secret
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          cpus: '0.5'
          memory: 512M
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 3s
      retries: 3

  postgres:
    image: postgres:16-alpine
    networks:
      - backend
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password
    volumes:
      - postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U appuser"]
      interval: 10s
      timeout: 5s
      retries: 5
    deploy:
      resources:
        limits:
          memory: 1G

  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes --requirepass $(cat /run/secrets/redis_password)
    networks:
      - backend
    secrets:
      - redis_password
    volumes:
      - redis-data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "--raw", "incr", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3

networks:
  frontend:
    driver: bridge
    driver_opts:
      com.docker.network.bridge.host_binding_ipv4: "127.0.0.1"
  backend:
    driver: bridge
    internal: true

volumes:
  postgres-data:
    driver: local
  redis-data:
    driver: local

secrets:
  db_password:
    file: ./secrets/db_password.txt
  jwt_secret:
    file: ./secrets/jwt_secret.txt
  redis_password:
    file: ./secrets/redis_password.txt
```

## Security Checklist

**Pre-deployment verification:**
- [ ] All containers run as non-root users
- [ ] No secrets in ENV or ARG directives
- [ ] BuildKit secrets used for build-time secrets
- [ ] Cloud secrets manager configured for production
- [ ] Network isolation with internal networks
- [ ] Resource limits set on all services
- [ ] Health checks configured for all services
- [ ] .env files in .gitignore and .dockerignore
- [ ] Ports bound to 127.0.0.1 (not 0.0.0.0) for internal services
- [ ] Base images use specific tags (not :latest)
- [ ] Minimal base images (slim/distroless/scratch)
- [ ] File permissions are restrictive (no chmod 777)
- [ ] Security scanning completed with no critical issues
- [ ] All dependencies use lock files
- [ ] Health endpoints don't expose sensitive data

**Post-deployment monitoring:**
- [ ] Regular security scans scheduled
- [ ] Dependency updates automated
- [ ] Secret rotation procedures documented
- [ ] Incident response plan in place
- [ ] Audit logging enabled
- [ ] Network traffic monitored
- [ ] Resource usage within limits
- [ ] Health checks passing consistently

## Reference Commands

```bash
# Validate no secrets in layers
docker history myapp:latest | grep -i "password\|secret\|token\|key"

# Verify non-root user
docker run --rm myapp:latest whoami  # Should not be root

# Check container permissions
docker run --rm myapp:latest ls -la /app

# Scan for vulnerabilities
trivy image --severity HIGH,CRITICAL myapp:latest

# Test network isolation
docker network inspect backend  # Should show "internal": true

# Verify resource limits
docker inspect myapp | jq '.[0].HostConfig.Memory'
```

Always prioritize security at every layer: build, runtime, network, and cloud infrastructure.
