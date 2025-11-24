---
description: 
globs: 
alwaysApply: false
---
# Docker Standards for Modern Web Applications

Best practices for containerizing Node.js applications with modern dependencies like Next.js, Prisma ORM, and CSS frameworks with native binaries.

## Prerequisites

- Docker 20.0+ and Docker Compose 2.0+
- Understanding of multi-stage Docker builds
- Node.js application with package.json
- Modern dependencies that may require native compilation

## Core Principle

**Use proven Docker patterns that handle modern JavaScript ecosystem dependencies correctly, prioritizing compatibility and build reliability over image size optimization.**

## Base Image Selection

### Use Debian-based Images for Modern Dependencies

```dockerfile
# ✅ CORRECT: Use Debian-based for native dependencies
FROM node:20-slim AS base

# ❌ AVOID: Alpine for apps with native dependencies
# FROM node:20-alpine AS base
```

**Why Debian over Alpine:**
- Better compatibility with native binaries (LightningCSS, sharp, etc.)
- glibc vs musl compatibility issues resolved
- More predictable builds with modern CSS frameworks

### Required Build Dependencies

```dockerfile
# Install build dependencies for native compilation
FROM base AS deps
RUN apt-get update && apt-get install -y --no-install-recommends \
    python3 \
    make \
    g++ \
    && rm -rf /var/lib/apt/lists/*
```

## Dependency Installation Strategy

### Install All Dependencies for Build Process

```dockerfile
# ✅ CORRECT: Install all dependencies (including dev)
COPY package.json package-lock.json* ./
COPY prisma ./prisma  # Copy schema before npm install
RUN npm ci  # Includes devDependencies needed for build

# ❌ INCORRECT: Production only misses build dependencies
# RUN npm ci --only=production
```

**Rationale:**
- Modern CSS frameworks (Tailwind CSS v4) are often in devDependencies
- Build tools are required during Docker build process
- Runtime optimization happens in final image layer

## File Copying Order Best Practices

### Critical: Copy Schema Files Before npm install

```dockerfile
# ✅ CORRECT ORDER
COPY package.json package-lock.json* ./
COPY prisma ./prisma  # Required for postinstall scripts
RUN npm ci

# ❌ INCORRECT ORDER - Causes postinstall failures
# COPY package.json package-lock.json* ./
# RUN npm ci  # Fails if postinstall needs schema files
# COPY prisma ./prisma
```

### Complete Multi-Stage Build Pattern

```dockerfile
# Use the official Node.js 20 image (Debian-based for compatibility)
FROM node:20-slim AS base

# Install dependencies only when needed
FROM base AS deps
RUN apt-get update && apt-get install -y --no-install-recommends \
    python3 \
    make \
    g++ \
    && rm -rf /var/lib/apt/lists/*
WORKDIR /app

# Copy package files and required schemas BEFORE npm install
COPY package.json package-lock.json* ./
COPY prisma ./prisma

# Install all dependencies (including dev dependencies needed for build)
RUN npm ci

# Build stage
FROM base AS builder
WORKDIR /app
RUN apt-get update && apt-get install -y --no-install-recommends \
    python3 \
    make \
    g++ \
    && rm -rf /var/lib/apt/lists/*

COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Generate database client if needed
RUN npx prisma generate

# Build the application
RUN npm run build

# Production image
FROM base AS runner
WORKDIR /app

ENV NODE_ENV=production

RUN groupadd --system --gid 1001 nodejs
RUN useradd --system --uid 1001 nextjs

# Copy built application
COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

# Copy database schema and generated client
COPY --from=builder /app/prisma ./prisma
COPY --from=builder /app/generated ./generated

USER nextjs

EXPOSE 3000

ENV PORT=3000
ENV HOSTNAME="0.0.0.0"

CMD ["node", "server.js"]
```

## Next.js Specific Configuration

### Required next.config.ts Settings

```typescript
const nextConfig: NextConfig = {
  // Enable standalone output for Docker
  output: 'standalone',
  
  // External packages for server components
  serverExternalPackages: ['@prisma/client'],
  
  // Security headers for production
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          { key: 'X-Frame-Options', value: 'DENY' },
          { key: 'X-Content-Type-Options', value: 'nosniff' },
          { key: 'Referrer-Policy', value: 'origin-when-cross-origin' },
          { key: 'X-XSS-Protection', value: '1; mode=block' },
        ],
      },
    ];
  },
};
```

## Prisma Integration

### Schema and Client Generation

```dockerfile
# Copy schema before npm install (for postinstall scripts)
COPY prisma ./prisma

# Generate client after copying all files
RUN npx prisma generate

# Copy generated client to runtime image
COPY --from=builder /app/generated ./generated
```

### Custom Output Path Handling

If using custom Prisma output:

```prisma
generator client {
  provider = "prisma-client-js"
  output   = "../generated/prisma"
}
```

Ensure both `prisma` and `generated` directories are copied to runtime image.

## Docker Compose Configuration

```yaml
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://user:password@db:5432/dbname
    depends_on:
      - db
      - redis

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: dbname
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./scripts/init-db.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "5432:5432"

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

## Troubleshooting Common Issues

### Problem: "Cannot find module" during build

**Symptoms:**
- Build fails with missing module errors
- Errors related to native binaries (lightningcss, sharp, etc.)

**Solution:**
```dockerfile
# Ensure build tools are installed
RUN apt-get update && apt-get install -y --no-install-recommends \
    python3 make g++ && rm -rf /var/lib/apt/lists/*

# Use full npm ci (not --only=production)
RUN npm ci
```

### Problem: Prisma postinstall script failures

**Symptoms:**
- "Schema not found" during npm install
- Prisma generate fails in Docker build

**Solution:**
```dockerfile
# Copy schema BEFORE npm install
COPY package.json package-lock.json* ./
COPY prisma ./prisma  # Critical: Before npm ci
RUN npm ci
```

### Problem: Alpine/musl compatibility issues

**Symptoms:**
- Native binary not found errors
- Architecture-specific module missing

**Solution:**
```dockerfile
# Switch to Debian-based image
FROM node:20-slim AS base  # Not node:20-alpine
```

## Validation Steps

After implementing Docker configuration:

```powershell
# Build successfully
docker-compose build

# Start all services
docker-compose up -d

# Verify all containers are healthy
docker-compose ps

# Check application accessibility
# App should be available at http://localhost:3000
```

## Success Criteria

Docker implementation is complete when:
- [x] Build completes without errors on first attempt
- [x] All services start successfully in Docker Compose
- [x] Application is accessible on configured ports
- [x] Database connections work correctly
- [x] No native dependency compilation errors

---

# Python Backend Applications with PostgreSQL and Alembic

Best practices for containerizing Python backend applications with PostgreSQL databases and Alembic migrations.

## Prerequisites

- Docker 20.0+ and Docker Compose 2.0+
- Python 3.11+ application with requirements or pyproject.toml
- SQLAlchemy 2.0+ and Alembic for database operations
- Understanding of database migrations and ORM patterns

## Core Principles

**Use proven patterns for Python backends with Docker-hosted databases:**
- PostgreSQL in containers for development/staging environment consistency
- Alembic for version-controlled database schema migrations
- SQLAlchemy for type-safe database operations
- Health checks for database readiness before application startup

## PostgreSQL Docker Configuration

### Basic PostgreSQL Container with Health Checks

```yaml
services:
  postgres:
    image: postgres:15-alpine
    container_name: app_postgres
    environment:
      POSTGRES_USER: ${DB_USER:-appuser}
      POSTGRES_PASSWORD: ${DB_PASSWORD:-devpassword}
      POSTGRES_DB: ${DB_NAME:-appdb}
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./scripts/init-db.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER:-appuser}"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

volumes:
  postgres_data:
    driver: local
```

**Key configuration details:**
- **Image selection**: `postgres:15-alpine` for smaller image size
- **Health checks**: Verify PostgreSQL accepts connections before app starts
- **Volume persistence**: Data survives container restarts
- **Environment variables**: Configurable credentials (never hardcode in production)
- **Init scripts**: Optional SQL scripts run on first container start

### Production PostgreSQL Configuration

```yaml
services:
  postgres:
    image: postgres:15-alpine
    container_name: app_postgres_prod
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_INITDB_ARGS: "--encoding=UTF8 --lc-collate=C --lc-ctype=C"
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s
    restart: unless-stopped
    secrets:
      - db_password
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          cpus: '1'
          memory: 1G

secrets:
  db_password:
    file: ./secrets/db_password.txt

volumes:
  postgres_data:
    driver: local
```

**Production enhancements:**
- **Secrets management**: Use Docker secrets instead of environment variables
- **Resource limits**: Prevent database from consuming all system resources
- **Start period**: Allow extra time for database initialization
- **Character encoding**: Explicit UTF8 encoding for international support

## Alembic Integration

### Project Directory Structure

```
project/
├── alembic/
│   ├── versions/
│   │   ├── 001_initial_schema.py
│   │   └── 002_add_users_table.py
│   ├── env.py
│   ├── script.py.mako
│   └── README
├── alembic.ini
├── app/
│   ├── __init__.py
│   ├── models/
│   │   ├── __init__.py
│   │   └── user.py
│   ├── database.py
│   └── main.py
├── scripts/
│   ├── entrypoint.sh
│   └── wait-for-db.sh
├── Dockerfile
├── docker-compose.yml
└── pyproject.toml
```

### Alembic Configuration (alembic.ini)

```ini
[alembic]
# Path to migration scripts
script_location = alembic

# Template used to generate migration files
file_template = %%(year)d%%(month).2d%%(day).2d_%%(hour).2d%%(minute).2d_%%(rev)s_%%(slug)s

# Prepend sys.path with the project directory
prepend_sys_path = .

# Separate multiple paths with semicolon
version_path_separator = os

# Database URL (overridden by environment variable in env.py)
sqlalchemy.url = driver://user:pass@localhost/dbname

# Logging configuration
[loggers]
keys = root,sqlalchemy,alembic

[handlers]
keys = console

[formatters]
keys = generic

[logger_root]
level = WARN
handlers = console
qualname =

[logger_sqlalchemy]
level = WARN
handlers =
qualname = sqlalchemy.engine

[logger_alembic]
level = INFO
handlers =
qualname = alembic

[handler_console]
class = StreamHandler
args = (sys.stderr,)
level = NOTSET
formatter = generic

[formatter_generic]
format = %(levelname)-5.5s [%(name)s] %(message)s
datefmt = %H:%M:%S

# Post-write hooks (optional - auto-format migrations)
[post_write_hooks]
hooks = ruff
ruff.type = console_scripts
ruff.entrypoint = ruff
ruff.options = format REVISION_SCRIPT_FILENAME
```

### Alembic Environment Configuration (env.py)

```python
"""Alembic environment configuration with Docker support."""
from logging.config import fileConfig
from sqlalchemy import engine_from_config, pool
from alembic import context
import os
import sys

# Add project root to path
sys.path.insert(0, os.path.dirname(os.path.dirname(__file__)))

# Import your models for autogenerate support
from app.models import Base  # Import your declarative base
from app import models  # Import all model modules

# Alembic Config object
config = context.config

# Override sqlalchemy.url from environment variable
database_url = os.getenv(
    'DATABASE_URL',
    'postgresql://appuser:devpassword@localhost:5432/appdb'
)
config.set_main_option('sqlalchemy.url', database_url)

# Interpret the config file for Python logging
if config.config_file_name is not None:
    fileConfig(config.config_file_name)

# Target metadata for autogenerate support
target_metadata = Base.metadata


def run_migrations_offline() -> None:
    """Run migrations in 'offline' mode.

    This configures the context with just a URL
    and not an Engine, though an Engine is acceptable
    here as well. By skipping the Engine creation
    we don't even need a DBAPI to be available.
    """
    url = config.get_main_option("sqlalchemy.url")
    context.configure(
        url=url,
        target_metadata=target_metadata,
        literal_binds=True,
        dialect_opts={"paramstyle": "named"},
        compare_type=True,
        compare_server_default=True,
    )

    with context.begin_transaction():
        context.run_migrations()


def run_migrations_online() -> None:
    """Run migrations in 'online' mode.

    In this scenario we need to create an Engine
    and associate a connection with the context.
    """
    connectable = engine_from_config(
        config.get_section(config.config_ini_section, {}),
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,  # Don't use connection pooling for migrations
    )

    with connectable.connect() as connection:
        context.configure(
            connection=connection,
            target_metadata=target_metadata,
            compare_type=True,
            compare_server_default=True,
            # Include schemas if using PostgreSQL schemas
            include_schemas=True,
        )

        with context.begin_transaction():
            context.run_migrations()


if context.is_offline_mode():
    run_migrations_offline()
else:
    run_migrations_online()
```

### SQLAlchemy Connection Management

#### Database Configuration (app/database.py)

```python
"""SQLAlchemy database configuration with connection pooling."""
from sqlalchemy import create_engine, event
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from sqlalchemy.pool import QueuePool
from typing import Generator
import os

# Database URL from environment
DATABASE_URL = os.getenv(
    'DATABASE_URL',
    'postgresql://appuser:devpassword@localhost:5432/appdb'
)

# Create engine with connection pooling
engine = create_engine(
    DATABASE_URL,
    poolclass=QueuePool,
    pool_size=5,          # Number of persistent connections
    max_overflow=10,      # Additional connections when pool is full
    pool_pre_ping=True,   # Verify connections before using
    pool_recycle=3600,    # Recycle connections after 1 hour
    echo=False,           # Set to True for SQL logging during dev
    future=True,          # Use SQLAlchemy 2.0 style
)

# Session factory
SessionLocal = sessionmaker(
    autocommit=False,
    autoflush=False,
    bind=engine,
    future=True,
)

# Declarative base for models
Base = declarative_base()


def get_db() -> Generator:
    """Dependency for getting database sessions.

    Usage with FastAPI:
        @app.get("/items")
        def get_items(db: Session = Depends(get_db)):
            return db.query(Item).all()
    """
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()


# Optional: Connection event listeners for debugging
@event.listens_for(engine, "connect")
def receive_connect(dbapi_conn, connection_record):
    """Log when new database connections are established."""
    print(f"New DB connection: {id(dbapi_conn)}")


@event.listens_for(engine, "checkout")
def receive_checkout(dbapi_conn, connection_record, connection_proxy):
    """Log when connections are checked out from pool."""
    print(f"Connection checkout: {id(dbapi_conn)}")
```

#### Async SQLAlchemy Support (app/database_async.py)

```python
"""Async SQLAlchemy configuration for high-performance applications."""
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker
from sqlalchemy.orm import declarative_base
from typing import AsyncGenerator
import os

# Async database URL (note: postgresql+asyncpg)
DATABASE_URL = os.getenv(
    'DATABASE_URL',
    'postgresql+asyncpg://appuser:devpassword@localhost:5432/appdb'
)

# Create async engine
engine = create_async_engine(
    DATABASE_URL,
    pool_size=5,
    max_overflow=10,
    pool_pre_ping=True,
    pool_recycle=3600,
    echo=False,
    future=True,
)

# Async session factory
AsyncSessionLocal = async_sessionmaker(
    engine,
    class_=AsyncSession,
    expire_on_commit=False,
)

Base = declarative_base()


async def get_db() -> AsyncGenerator[AsyncSession, None]:
    """Async dependency for getting database sessions.

    Usage with FastAPI:
        @app.get("/items")
        async def get_items(db: AsyncSession = Depends(get_db)):
            result = await db.execute(select(Item))
            return result.scalars().all()
    """
    async with AsyncSessionLocal() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise
        finally:
            await session.close()
```

## Docker Integration

### Python Application Dockerfile

```dockerfile
# Use Python 3.11 slim image
FROM python:3.11-slim AS base

# Set working directory
WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    postgresql-client \
    build-essential \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Install uv for fast dependency management
RUN pip install --no-cache-dir uv

# Copy dependency files
COPY pyproject.toml uv.lock* ./

# Install Python dependencies
RUN uv pip install --system --no-cache -r pyproject.toml

# Copy application code
COPY . .

# Copy entrypoint script
COPY scripts/entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

# Create non-root user
RUN groupadd --system --gid 1001 appuser && \
    useradd --system --uid 1001 --gid appuser --create-home appuser

# Change ownership
RUN chown -R appuser:appuser /app

USER appuser

EXPOSE 8000

ENTRYPOINT ["/entrypoint.sh"]
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Docker Entrypoint Script (scripts/entrypoint.sh)

```bash
#!/bin/bash
set -e

echo "Waiting for PostgreSQL to be ready..."

# Wait for PostgreSQL using pg_isready
until pg_isready -h "${DB_HOST:-postgres}" -p "${DB_PORT:-5432}" -U "${DB_USER:-appuser}"; do
  echo "PostgreSQL is unavailable - sleeping"
  sleep 2
done

echo "PostgreSQL is up - executing command"

# Run database migrations
echo "Running Alembic migrations..."
alembic upgrade head

# Execute the main command
exec "$@"
```

### Alternative: Python-based Wait Script (scripts/wait_for_db.py)

```python
"""Wait for PostgreSQL to be ready before starting application."""
import time
import sys
from sqlalchemy import create_engine, text
from sqlalchemy.exc import OperationalError
import os

DATABASE_URL = os.getenv(
    'DATABASE_URL',
    'postgresql://appuser:devpassword@postgres:5432/appdb'
)

def wait_for_db(max_retries: int = 30, retry_interval: int = 2) -> bool:
    """Wait for database to be ready.

    Args:
        max_retries: Maximum number of connection attempts
        retry_interval: Seconds to wait between attempts

    Returns:
        True if database is ready, False otherwise
    """
    engine = create_engine(DATABASE_URL)

    for attempt in range(max_retries):
        try:
            with engine.connect() as conn:
                conn.execute(text("SELECT 1"))
                print(f"Database is ready! (attempt {attempt + 1}/{max_retries})")
                return True
        except OperationalError as e:
            print(f"Database not ready (attempt {attempt + 1}/{max_retries}): {e}")
            time.sleep(retry_interval)

    print(f"Failed to connect to database after {max_retries} attempts")
    return False


if __name__ == "__main__":
    if not wait_for_db():
        sys.exit(1)
    print("Starting application...")
```

### Complete Docker Compose Configuration

```yaml
services:
  # PostgreSQL Database
  postgres:
    image: postgres:15-alpine
    container_name: backend_postgres
    environment:
      POSTGRES_USER: ${DB_USER:-appuser}
      POSTGRES_PASSWORD: ${DB_PASSWORD:-devpassword}
      POSTGRES_DB: ${DB_NAME:-appdb}
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./scripts/init-db.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER:-appuser}"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - backend_network
    restart: unless-stopped

  # Redis (optional - for caching/sessions)
  redis:
    image: redis:7-alpine
    container_name: backend_redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - backend_network
    restart: unless-stopped

  # Python Backend Application
  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: backend_app
    environment:
      - DATABASE_URL=postgresql://${DB_USER:-appuser}:${DB_PASSWORD:-devpassword}@postgres:5432/${DB_NAME:-appdb}
      - REDIS_URL=redis://redis:6379/0
      - ENVIRONMENT=${ENVIRONMENT:-development}
      - LOG_LEVEL=${LOG_LEVEL:-INFO}
    ports:
      - "8000:8000"
    volumes:
      - ./app:/app/app:ro  # Mount source code for development
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - backend_network
    restart: unless-stopped
    command: ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000", "--reload"]

networks:
  backend_network:
    driver: bridge

volumes:
  postgres_data:
    driver: local
  redis_data:
    driver: local
```

## Health Check Endpoints

### FastAPI Health Checks

```python
"""Health check endpoints for monitoring."""
from fastapi import APIRouter, status, Depends
from sqlalchemy import text
from sqlalchemy.orm import Session
from app.database import get_db, engine
import redis
import os

router = APIRouter(prefix="/health", tags=["health"])


@router.get("/", status_code=status.HTTP_200_OK)
async def health_check():
    """Basic health check endpoint."""
    return {
        "status": "healthy",
        "service": "backend-api",
        "environment": os.getenv("ENVIRONMENT", "unknown")
    }


@router.get("/db", status_code=status.HTTP_200_OK)
async def database_health(db: Session = Depends(get_db)):
    """Database connectivity health check."""
    try:
        # Execute simple query
        db.execute(text("SELECT 1"))

        # Check connection pool status
        pool = engine.pool
        return {
            "status": "healthy",
            "database": "connected",
            "pool_size": pool.size(),
            "checked_out": pool.checkedout(),
            "overflow": pool.overflow(),
        }
    except Exception as e:
        return {
            "status": "unhealthy",
            "database": "disconnected",
            "error": str(e)
        }


@router.get("/redis", status_code=status.HTTP_200_OK)
async def redis_health():
    """Redis connectivity health check."""
    try:
        redis_client = redis.from_url(
            os.getenv("REDIS_URL", "redis://redis:6379/0")
        )
        redis_client.ping()
        info = redis_client.info("memory")

        return {
            "status": "healthy",
            "redis": "connected",
            "memory_used": info.get("used_memory_human"),
            "memory_peak": info.get("used_memory_peak_human"),
        }
    except Exception as e:
        return {
            "status": "unhealthy",
            "redis": "disconnected",
            "error": str(e)
        }


@router.get("/full", status_code=status.HTTP_200_OK)
async def full_health_check(db: Session = Depends(get_db)):
    """Comprehensive health check for all services."""
    db_status = "unknown"
    redis_status = "unknown"

    # Check database
    try:
        db.execute(text("SELECT 1"))
        db_status = "healthy"
    except Exception:
        db_status = "unhealthy"

    # Check Redis
    try:
        redis_client = redis.from_url(
            os.getenv("REDIS_URL", "redis://redis:6379/0")
        )
        redis_client.ping()
        redis_status = "healthy"
    except Exception:
        redis_status = "unhealthy"

    overall_status = "healthy" if all([
        db_status == "healthy",
        redis_status == "healthy"
    ]) else "unhealthy"

    return {
        "status": overall_status,
        "checks": {
            "database": db_status,
            "redis": redis_status,
        }
    }
```

## Environment Variable Configuration

### Example .env File

```bash
# Database Configuration
DB_HOST=postgres
DB_PORT=5432
DB_USER=appuser
DB_PASSWORD=devpassword
DB_NAME=appdb
DATABASE_URL=postgresql://${DB_USER}:${DB_PASSWORD}@${DB_HOST}:${DB_PORT}/${DB_NAME}

# Redis Configuration
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_URL=redis://${REDIS_HOST}:${REDIS_PORT}/0

# Application Configuration
ENVIRONMENT=development
LOG_LEVEL=DEBUG
SECRET_KEY=your-secret-key-here-change-in-production

# API Configuration
API_V1_PREFIX=/api/v1
CORS_ORIGINS=http://localhost:3000,http://localhost:8000
```

### Environment Variable Loading (app/config.py)

```python
"""Application configuration with environment variables."""
from pydantic_settings import BaseSettings
from typing import List
from functools import lru_cache


class Settings(BaseSettings):
    """Application settings loaded from environment."""

    # Database
    database_url: str
    db_pool_size: int = 5
    db_max_overflow: int = 10

    # Redis
    redis_url: str

    # Application
    environment: str = "development"
    log_level: str = "INFO"
    secret_key: str

    # API
    api_v1_prefix: str = "/api/v1"
    cors_origins: List[str] = ["http://localhost:3000"]

    class Config:
        env_file = ".env"
        case_sensitive = False


@lru_cache()
def get_settings() -> Settings:
    """Get cached settings instance."""
    return Settings()
```

## Alembic Migration Workflow

### Creating a New Migration

```bash
# Auto-generate migration from model changes
alembic revision --autogenerate -m "Add users table"

# Create empty migration for data operations
alembic revision -m "Populate default data"

# Review generated migration file before applying
cat alembic/versions/20240115_1430_abc123_add_users_table.py
```

### Applying Migrations

```bash
# Upgrade to latest version
alembic upgrade head

# Upgrade one version
alembic upgrade +1

# Downgrade one version
alembic downgrade -1

# Downgrade to specific version
alembic downgrade abc123

# Show current version
alembic current

# Show migration history
alembic history --verbose
```

### Migration in Docker

```bash
# Run migrations inside container
docker-compose exec app alembic upgrade head

# Create new migration inside container
docker-compose exec app alembic revision --autogenerate -m "Migration name"

# Check migration status
docker-compose exec app alembic current
```

## Troubleshooting

### Problem: "Relation does not exist" errors

**Symptoms:**
- Application fails with "relation 'table_name' does not exist"
- Database tables not created

**Solution:**
```bash
# Verify migrations ran
docker-compose logs app | grep -i alembic

# Manually run migrations
docker-compose exec app alembic upgrade head

# Check database tables
docker-compose exec postgres psql -U appuser -d appdb -c "\dt"
```

### Problem: Connection refused to PostgreSQL

**Symptoms:**
- Application cannot connect to database
- "Connection refused" or "could not connect" errors

**Solution:**
```bash
# Check PostgreSQL health
docker-compose ps postgres

# Check PostgreSQL logs
docker-compose logs postgres

# Verify network connectivity
docker-compose exec app pg_isready -h postgres -p 5432 -U appuser

# Ensure depends_on with healthcheck is configured
```

### Problem: Migrations fail with "Target database is not up to date"

**Symptoms:**
- Alembic reports version mismatch
- Cannot apply migrations

**Solution:**
```bash
# Check current version
docker-compose exec app alembic current

# Check migration history
docker-compose exec app alembic history

# Stamp database to specific version (use with caution)
docker-compose exec app alembic stamp head

# Reset and reapply all migrations (development only)
docker-compose down -v
docker-compose up -d
```

## Go Application Docker Patterns

### Go Multi-Stage Dockerfile with Static Binary

Go applications are ideal for containerization due to static binary compilation and minimal runtime requirements.

```dockerfile
# syntax=docker/dockerfile:1
FROM golang:1.21-alpine AS builder

WORKDIR /src

# Enable Go modules caching
ENV GOMODCACHE=/root/.cache/go-build

# Copy go mod files first for layer caching
COPY go.mod go.sum ./
RUN --mount=type=cache,target=/root/.cache/go-build \
    --mount=type=cache,target=/go/pkg/mod \
    go mod download

# Copy source code
COPY . .

# Build with cache mounts and static linking
RUN --mount=type=cache,target=/root/.cache/go-build \
    --mount=type=cache,target=/go/pkg/mod \
    CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -ldflags="-s -w" -o /app .

# Minimal runtime with scratch (ultimate minimalism)
FROM scratch

# Copy CA certificates for HTTPS
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

# Copy static binary
COPY --from=builder /app /app

# Use non-root UID (no user creation in scratch)
USER 65534:65534

EXPOSE 8080

ENTRYPOINT ["/app"]
```

### Go with Alpine Runtime (If Shell Needed)

```dockerfile
FROM golang:1.21-alpine AS builder

WORKDIR /src

COPY go.mod go.sum ./
RUN --mount=type=cache,target=/root/.cache/go-build \
    --mount=type=cache,target=/go/pkg/mod \
    go mod download

COPY . .

RUN --mount=type=cache,target=/root/.cache/go-build \
    --mount=type=cache,target=/go/pkg/mod \
    CGO_ENABLED=0 go build -ldflags="-s -w" -o /app .

# Minimal Alpine runtime
FROM alpine:latest

RUN apk --no-cache add ca-certificates && \
    addgroup -g 1000 -S appuser && \
    adduser -S appuser -u 1000 -G appuser

WORKDIR /

COPY --from=builder /app /app

USER appuser

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget --spider http://localhost:8080/health || exit 1

ENTRYPOINT ["/app"]
```

### Go with Docker Compose

```yaml
services:
  api:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "127.0.0.1:8080:8080"
    environment:
      DATABASE_URL: postgresql://user:password@postgres:5432/dbname
      REDIS_ADDR: redis:6379
      LOG_LEVEL: info
    depends_on:
      postgres:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "/app", "health"]  # Custom health command
      interval: 30s
      timeout: 3s
      retries: 3
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 256M
    networks:
      - backend

  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: dbname
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - backend

networks:
  backend:
    driver: bridge
    internal: true

volumes:
  postgres-data:
```

### Go Build Optimization Tips

**Build flags:**
- `-ldflags="-s -w"`: Strip debug info and symbol table (smaller binary)
- `-a`: Force rebuild of all packages
- `-installsuffix cgo`: Allow multiple builds with different CGO settings
- `CGO_ENABLED=0`: Static linking, no C dependencies

**Cache mount benefits:**
- First build: ~30s
- Subsequent builds with cache: ~2s (95% faster)
- Shared cache across developers with BuildKit registry cache

## Java Application Docker Patterns

### Spring Boot Multi-Stage with Layer Extraction

```dockerfile
# syntax=docker/dockerfile:1
FROM eclipse-temurin:21-jdk-jammy AS builder

WORKDIR /app

# Copy Gradle wrapper and configuration
COPY gradle gradle
COPY gradlew build.gradle settings.gradle ./

# Download dependencies with cache mount
RUN --mount=type=cache,target=/root/.gradle \
    ./gradlew dependencies --no-daemon

# Copy source and build
COPY src src
RUN --mount=type=cache,target=/root/.gradle \
    ./gradlew bootJar --no-daemon

# Extract Spring Boot layers (optimal caching)
RUN java -Djarmode=layertools -jar build/libs/*.jar extract

# Runtime stage
FROM eclipse-temurin:21-jre-jammy

# Create non-root user
RUN groupadd -r spring && useradd -r -g spring -u 1000 spring

WORKDIR /app

# Copy layers in order of least to most frequently changing
COPY --from=builder --chown=spring:spring /app/dependencies/ ./
COPY --from=builder --chown=spring:spring /app/spring-boot-loader/ ./
COPY --from=builder --chown=spring:spring /app/snapshot-dependencies/ ./
COPY --from=builder --chown=spring:spring /app/application/ ./

# Switch to non-root user
USER spring

EXPOSE 8080

# JVM container awareness
ENV JAVA_OPTS="-XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0"

HEALTHCHECK --interval=30s --timeout=3s --start-period=40s \
  CMD curl -f http://localhost:8080/actuator/health || exit 1

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS org.springframework.boot.loader.launch.JarLauncher"]
```

### Maven Alternative

```dockerfile
FROM maven:3.9-eclipse-temurin-21 AS builder

WORKDIR /app

# Copy POM first for dependency caching
COPY pom.xml .
RUN --mount=type=cache,target=/root/.m2 \
    mvn dependency:go-offline

# Copy source and build
COPY src src
RUN --mount=type=cache,target=/root/.m2 \
    mvn package -DskipTests

# Runtime stage
FROM eclipse-temurin:21-jre-jammy

RUN groupadd -r spring && useradd -r -g spring spring

WORKDIR /app

COPY --from=builder --chown=spring:spring /app/target/*.jar app.jar

USER spring

EXPOSE 8080

ENV JAVA_OPTS="-XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0"

HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:8080/actuator/health || exit 1

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

### Java with Docker Compose

```yaml
services:
  api:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "127.0.0.1:8080:8080"
    environment:
      SPRING_PROFILES_ACTIVE: production
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/appdb
      SPRING_DATASOURCE_USERNAME: appuser
      SPRING_DATASOURCE_PASSWORD_FILE: /run/secrets/db_password
      SPRING_REDIS_HOST: redis
      JAVA_OPTS: -Xmx1g -XX:+UseContainerSupport
    secrets:
      - db_password
    depends_on:
      postgres:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/actuator/health"]
      interval: 30s
      timeout: 3s
      start_period: 40s
      retries: 3
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          cpus: '1'
          memory: 1G
    networks:
      - backend

  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: appdb
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
    networks:
      - backend

networks:
  backend:
    driver: bridge
    internal: true

volumes:
  postgres-data:

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

### Java JVM Container Tuning

**Modern JVM flags for containers:**
```bash
# Container-aware memory management
-XX:+UseContainerSupport
-XX:MaxRAMPercentage=75.0
-XX:InitialRAMPercentage=50.0

# GC optimization for containers
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200

# Diagnostic options (development only)
-XX:+PrintFlagsFinal
-XshowSettings:vm
```

**application.properties for Spring Boot:**
```properties
# Actuator health checks
management.endpoint.health.enabled=true
management.endpoints.web.exposure.include=health,info,metrics

# Graceful shutdown
server.shutdown=graceful
spring.lifecycle.timeout-per-shutdown-phase=30s

# Container optimizations
spring.jmx.enabled=false
```

### Java Build Optimization

**Layer extraction benefits:**
- Dependencies layer: Changes rarely (~5% of builds)
- Spring Boot loader: Never changes
- Snapshot dependencies: Changes occasionally
- Application layer: Changes frequently

**Result:** 90% faster rebuilds by caching dependency layers

## Supporting Components

- **Docker Architect Agent**: `.claude/agents/orchestration/docker-architect-agent.md`
  - Comprehensive Docker guidance for all languages
  - Security-first approach
  - Cloud secrets integration
- **Docker Commands**:
  - `/docker:generate-dockerfile` - Generate optimized Dockerfiles
  - `/docker:generate-compose` - Create docker-compose.yml
  - `/docker:validate-security` - Security validation
  - `/validate-docker` - Configuration validation
- **Implementation Guidance**: `.claude/agents/backend-developer.md`
  - When to use Docker-hosted databases
  - Why Alembic for Python projects
  - Technology stack recommendations
- **Code Quality**: `/lint-all`
  - Python code quality checks
  - Type checking and formatting validation

---

**Note**: These patterns ensure reliable containerized applications across Python, Node.js, Java, and Go with proper database management, migrations, and Docker integration.
