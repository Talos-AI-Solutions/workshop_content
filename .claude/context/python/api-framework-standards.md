---
description: FastAPI framework standards with complete implementation patterns for Python REST APIs
globs: ["**/*.py", "**/pyproject.toml", "**/requirements.txt", "**/alembic.ini"]
alwaysApply: false
---

# FastAPI Framework Standards for Modern Python APIs

Complete implementation patterns for building production-ready FastAPI applications with PostgreSQL, SQLAlchemy, and Alembic.

## Prerequisites

- Python 3.11+ with type hints support
- FastAPI for async REST API framework
- Uvicorn as ASGI server
- SQLAlchemy 2.0+ for database ORM
- Alembic for database migrations
- Pydantic for data validation
- PostgreSQL database (Docker-hosted recommended)

## Core Principles

**FastAPI is our standardized choice for Python REST APIs:**
- Async-native architecture for high performance (20,000+ RPS)
- Type-safe with Python type hints and Pydantic validation
- Automatic OpenAPI documentation generation
- Perfect integration with SQLAlchemy + Alembic + PostgreSQL
- Excellent developer experience with IDE support

## Project Structure

### Recommended FastAPI Project Organization

```
project/
├── app/
│   ├── __init__.py
│   ├── main.py              # FastAPI application entry point
│   ├── config.py            # Configuration and settings
│   ├── database.py          # Database connection and session
│   ├── models/              # SQLAlchemy ORM models
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── post.py
│   ├── schemas/             # Pydantic schemas for validation
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── post.py
│   ├── routers/             # API route handlers
│   │   ├── __init__.py
│   │   ├── users.py
│   │   ├── posts.py
│   │   └── health.py
│   ├── dependencies.py      # Dependency injection functions
│   ├── middleware.py        # Custom middleware
│   └── exceptions.py        # Custom exceptions and handlers
├── alembic/                 # Database migrations
│   ├── versions/
│   └── env.py
├── tests/                   # Test suite
│   ├── __init__.py
│   ├── conftest.py
│   ├── test_users.py
│   └── test_posts.py
├── scripts/                 # Utility scripts
│   └── entrypoint.sh
├── alembic.ini
├── pyproject.toml
├── .env.example
├── Dockerfile
└── docker-compose.yml
```

## FastAPI Application Setup

### Main Application (app/main.py)

```python
"""FastAPI application entry point with configuration and routing."""
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from fastapi.middleware.trustedhost import TrustedHostMiddleware
from fastapi.responses import JSONResponse
from contextlib import asynccontextmanager

from app.config import settings
from app.database import engine
from app import models
from app.routers import users, posts, health
from app.exceptions import CustomException, custom_exception_handler


@asynccontextmanager
async def lifespan(app: FastAPI):
    """Application lifespan events for startup and shutdown."""
    # Startup
    print("Starting up FastAPI application...")
    # Could add: cache warmup, external service connections, etc.
    yield
    # Shutdown
    print("Shutting down FastAPI application...")
    await engine.dispose()


app = FastAPI(
    title="My FastAPI Application",
    description="Production-ready FastAPI with PostgreSQL, SQLAlchemy, and Alembic",
    version="1.0.0",
    docs_url="/docs",
    redoc_url="/redoc",
    openapi_url="/openapi.json",
    lifespan=lifespan,
)

# Configure CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.cors_origins,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Add security middleware
app.add_middleware(
    TrustedHostMiddleware,
    allowed_hosts=settings.allowed_hosts
)

# Register custom exception handlers
app.add_exception_handler(CustomException, custom_exception_handler)

# Include routers with prefix and tags
app.include_router(health.router, prefix="/health", tags=["health"])
app.include_router(users.router, prefix="/api/v1/users", tags=["users"])
app.include_router(posts.router, prefix="/api/v1/posts", tags=["posts"])


@app.get("/", tags=["root"])
async def root():
    """Root endpoint with API information."""
    return {
        "message": "Welcome to FastAPI Application",
        "docs": "/docs",
        "redoc": "/redoc",
        "version": "1.0.0"
    }
```

### Configuration Management (app/config.py)

```python
"""Application configuration using Pydantic settings."""
from pydantic_settings import BaseSettings, SettingsConfigDict
from typing import List
from functools import lru_cache


class Settings(BaseSettings):
    """Application settings loaded from environment variables."""

    # Application
    app_name: str = "FastAPI Application"
    environment: str = "development"
    debug: bool = False

    # Database
    database_url: str
    db_pool_size: int = 5
    db_max_overflow: int = 10
    db_echo: bool = False

    # Security
    secret_key: str
    algorithm: str = "HS256"
    access_token_expire_minutes: int = 30

    # CORS
    cors_origins: List[str] = ["http://localhost:3000"]
    allowed_hosts: List[str] = ["localhost", "127.0.0.1"]

    # API
    api_v1_prefix: str = "/api/v1"

    # Pagination
    default_page_size: int = 50
    max_page_size: int = 100

    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
        case_sensitive=False,
    )


@lru_cache()
def get_settings() -> Settings:
    """Get cached settings instance."""
    return Settings()


settings = get_settings()
```

### Database Configuration (app/database.py)

```python
"""Database connection and session management with async support."""
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker
from sqlalchemy.orm import declarative_base
from typing import AsyncGenerator

from app.config import settings

# Create async engine
engine = create_async_engine(
    settings.database_url,
    echo=settings.db_echo,
    pool_size=settings.db_pool_size,
    max_overflow=settings.db_max_overflow,
    pool_pre_ping=True,  # Verify connections before using
    pool_recycle=3600,   # Recycle connections after 1 hour
)

# Async session factory
AsyncSessionLocal = async_sessionmaker(
    engine,
    class_=AsyncSession,
    expire_on_commit=False,
    autocommit=False,
    autoflush=False,
)

# Declarative base for models
Base = declarative_base()


async def get_db() -> AsyncGenerator[AsyncSession, None]:
    """Dependency for getting database sessions.

    Usage in routes:
        async def get_users(db: AsyncSession = Depends(get_db)):
            # Use db session here
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

## SQLAlchemy Models

### User Model Example (app/models/user.py)

```python
"""User SQLAlchemy model."""
from sqlalchemy import Column, Integer, String, Boolean, DateTime, Text
from sqlalchemy.orm import relationship
from sqlalchemy.sql import func
from datetime import datetime

from app.database import Base


class User(Base):
    """User model with authentication and profile information."""

    __tablename__ = "users"

    id = Column(Integer, primary_key=True, index=True)
    email = Column(String(255), unique=True, index=True, nullable=False)
    username = Column(String(50), unique=True, index=True, nullable=False)
    hashed_password = Column(String(255), nullable=False)
    full_name = Column(String(255), nullable=True)
    bio = Column(Text, nullable=True)
    is_active = Column(Boolean, default=True)
    is_superuser = Column(Boolean, default=False)
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())

    # Relationships
    posts = relationship("Post", back_populates="author", cascade="all, delete-orphan")

    def __repr__(self) -> str:
        return f"<User(id={self.id}, username={self.username})>"
```

### Post Model Example (app/models/post.py)

```python
"""Post SQLAlchemy model."""
from sqlalchemy import Column, Integer, String, Text, Boolean, DateTime, ForeignKey
from sqlalchemy.orm import relationship
from sqlalchemy.sql import func

from app.database import Base


class Post(Base):
    """Post model for blog posts or articles."""

    __tablename__ = "posts"

    id = Column(Integer, primary_key=True, index=True)
    title = Column(String(255), nullable=False, index=True)
    content = Column(Text, nullable=False)
    published = Column(Boolean, default=False)
    author_id = Column(Integer, ForeignKey("users.id"), nullable=False)
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())

    # Relationships
    author = relationship("User", back_populates="posts")

    def __repr__(self) -> str:
        return f"<Post(id={self.id}, title={self.title})>"
```

## Pydantic Schemas

### User Schemas (app/schemas/user.py)

```python
"""Pydantic schemas for User request/response validation."""
from pydantic import BaseModel, EmailStr, Field, ConfigDict
from datetime import datetime
from typing import Optional


class UserBase(BaseModel):
    """Shared User properties."""
    email: EmailStr
    username: str = Field(..., min_length=3, max_length=50)
    full_name: Optional[str] = None
    bio: Optional[str] = None


class UserCreate(UserBase):
    """Properties required for creating a User."""
    password: str = Field(..., min_length=8, max_length=100)


class UserUpdate(BaseModel):
    """Properties that can be updated for a User."""
    email: Optional[EmailStr] = None
    username: Optional[str] = Field(None, min_length=3, max_length=50)
    full_name: Optional[str] = None
    bio: Optional[str] = None
    password: Optional[str] = Field(None, min_length=8, max_length=100)


class UserInDB(UserBase):
    """Properties stored in database."""
    id: int
    is_active: bool
    is_superuser: bool
    created_at: datetime
    updated_at: Optional[datetime] = None

    model_config = ConfigDict(from_attributes=True)


class UserResponse(UserInDB):
    """User response without sensitive data."""
    pass


class UserListResponse(BaseModel):
    """Paginated list of users."""
    users: list[UserResponse]
    total: int
    page: int
    page_size: int

    model_config = ConfigDict(from_attributes=True)
```

### Post Schemas (app/schemas/post.py)

```python
"""Pydantic schemas for Post request/response validation."""
from pydantic import BaseModel, Field, ConfigDict
from datetime import datetime
from typing import Optional

from app.schemas.user import UserResponse


class PostBase(BaseModel):
    """Shared Post properties."""
    title: str = Field(..., min_length=1, max_length=255)
    content: str = Field(..., min_length=1)
    published: bool = False


class PostCreate(PostBase):
    """Properties required for creating a Post."""
    pass


class PostUpdate(BaseModel):
    """Properties that can be updated for a Post."""
    title: Optional[str] = Field(None, min_length=1, max_length=255)
    content: Optional[str] = Field(None, min_length=1)
    published: Optional[bool] = None


class PostInDB(PostBase):
    """Properties stored in database."""
    id: int
    author_id: int
    created_at: datetime
    updated_at: Optional[datetime] = None

    model_config = ConfigDict(from_attributes=True)


class PostResponse(PostInDB):
    """Post response with author information."""
    author: UserResponse

    model_config = ConfigDict(from_attributes=True)


class PostListResponse(BaseModel):
    """Paginated list of posts."""
    posts: list[PostResponse]
    total: int
    page: int
    page_size: int

    model_config = ConfigDict(from_attributes=True)
```

## API Routers

### User Router (app/routers/users.py)

```python
"""User API endpoints with CRUD operations."""
from fastapi import APIRouter, Depends, HTTPException, status, Query
from sqlalchemy import select, func
from sqlalchemy.ext.asyncio import AsyncSession
from typing import List

from app.database import get_db
from app import models, schemas
from app.dependencies import get_current_user
from app.config import settings

router = APIRouter()


@router.post(
    "/",
    response_model=schemas.UserResponse,
    status_code=status.HTTP_201_CREATED
)
async def create_user(
    user_in: schemas.UserCreate,
    db: AsyncSession = Depends(get_db)
):
    """Create a new user.

    - **email**: Valid email address (unique)
    - **username**: Username between 3-50 characters (unique)
    - **password**: Password with minimum 8 characters
    - **full_name**: Optional full name
    - **bio**: Optional biography
    """
    # Check if user already exists
    result = await db.execute(
        select(models.User).where(
            (models.User.email == user_in.email) |
            (models.User.username == user_in.username)
        )
    )
    existing_user = result.scalar_one_or_none()

    if existing_user:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="User with this email or username already exists"
        )

    # Hash password (use proper hashing in production)
    from passlib.context import CryptContext
    pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
    hashed_password = pwd_context.hash(user_in.password)

    # Create user
    db_user = models.User(
        email=user_in.email,
        username=user_in.username,
        hashed_password=hashed_password,
        full_name=user_in.full_name,
        bio=user_in.bio
    )

    db.add(db_user)
    await db.commit()
    await db.refresh(db_user)

    return db_user


@router.get(
    "/",
    response_model=schemas.UserListResponse
)
async def list_users(
    page: int = Query(1, ge=1),
    page_size: int = Query(settings.default_page_size, ge=1, le=settings.max_page_size),
    db: AsyncSession = Depends(get_db)
):
    """Get paginated list of users.

    - **page**: Page number (default: 1)
    - **page_size**: Number of items per page (default: 50, max: 100)
    """
    # Get total count
    count_result = await db.execute(select(func.count(models.User.id)))
    total = count_result.scalar()

    # Get paginated users
    offset = (page - 1) * page_size
    result = await db.execute(
        select(models.User)
        .offset(offset)
        .limit(page_size)
        .order_by(models.User.created_at.desc())
    )
    users = result.scalars().all()

    return {
        "users": users,
        "total": total,
        "page": page,
        "page_size": page_size
    }


@router.get(
    "/{user_id}",
    response_model=schemas.UserResponse
)
async def get_user(
    user_id: int,
    db: AsyncSession = Depends(get_db)
):
    """Get a specific user by ID."""
    result = await db.execute(
        select(models.User).where(models.User.id == user_id)
    )
    user = result.scalar_one_or_none()

    if not user:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"User with id {user_id} not found"
        )

    return user


@router.patch(
    "/{user_id}",
    response_model=schemas.UserResponse
)
async def update_user(
    user_id: int,
    user_in: schemas.UserUpdate,
    db: AsyncSession = Depends(get_db),
    current_user: models.User = Depends(get_current_user)
):
    """Update a user (requires authentication).

    Users can only update their own profile unless they are superusers.
    """
    result = await db.execute(
        select(models.User).where(models.User.id == user_id)
    )
    user = result.scalar_one_or_none()

    if not user:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"User with id {user_id} not found"
        )

    # Check permissions
    if user.id != current_user.id and not current_user.is_superuser:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Not enough permissions"
        )

    # Update user fields
    update_data = user_in.model_dump(exclude_unset=True)

    if "password" in update_data:
        from passlib.context import CryptContext
        pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
        update_data["hashed_password"] = pwd_context.hash(update_data.pop("password"))

    for field, value in update_data.items():
        setattr(user, field, value)

    await db.commit()
    await db.refresh(user)

    return user


@router.delete(
    "/{user_id}",
    status_code=status.HTTP_204_NO_CONTENT
)
async def delete_user(
    user_id: int,
    db: AsyncSession = Depends(get_db),
    current_user: models.User = Depends(get_current_user)
):
    """Delete a user (requires authentication and superuser permissions)."""
    if not current_user.is_superuser:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Not enough permissions"
        )

    result = await db.execute(
        select(models.User).where(models.User.id == user_id)
    )
    user = result.scalar_one_or_none()

    if not user:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"User with id {user_id} not found"
        )

    await db.delete(user)
    await db.commit()

    return None
```

### Health Check Router (app/routers/health.py)

```python
"""Health check endpoints for monitoring."""
from fastapi import APIRouter, Depends, status
from sqlalchemy import text
from sqlalchemy.ext.asyncio import AsyncSession

from app.database import get_db, engine

router = APIRouter()


@router.get("/", status_code=status.HTTP_200_OK)
async def health_check():
    """Basic health check endpoint."""
    return {
        "status": "healthy",
        "service": "fastapi-application"
    }


@router.get("/ready", status_code=status.HTTP_200_OK)
async def readiness_check(db: AsyncSession = Depends(get_db)):
    """Readiness check including database connectivity."""
    try:
        # Test database connection
        await db.execute(text("SELECT 1"))

        # Check connection pool status
        pool = engine.pool

        return {
            "status": "ready",
            "database": "connected",
            "pool_size": pool.size(),
            "checked_out": pool.checkedout(),
        }
    except Exception as e:
        return {
            "status": "not_ready",
            "database": "disconnected",
            "error": str(e)
        }


@router.get("/live", status_code=status.HTTP_200_OK)
async def liveness_check():
    """Liveness check for container orchestration."""
    return {
        "status": "alive"
    }
```

## Authentication and Authorization

### Dependencies (app/dependencies.py)

```python
"""Dependency injection functions for authentication and authorization."""
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from sqlalchemy import select
from sqlalchemy.ext.asyncio import AsyncSession
from jose import JWTError, jwt
from datetime import datetime, timedelta

from app.database import get_db
from app import models
from app.config import settings

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/api/v1/auth/login")


def create_access_token(data: dict) -> str:
    """Create JWT access token."""
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(minutes=settings.access_token_expire_minutes)
    to_encode.update({"exp": expire})

    encoded_jwt = jwt.encode(
        to_encode,
        settings.secret_key,
        algorithm=settings.algorithm
    )
    return encoded_jwt


async def get_current_user(
    token: str = Depends(oauth2_scheme),
    db: AsyncSession = Depends(get_db)
) -> models.User:
    """Get current authenticated user from JWT token."""
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )

    try:
        payload = jwt.decode(
            token,
            settings.secret_key,
            algorithms=[settings.algorithm]
        )
        user_id: int = payload.get("sub")
        if user_id is None:
            raise credentials_exception
    except JWTError:
        raise credentials_exception

    result = await db.execute(
        select(models.User).where(models.User.id == user_id)
    )
    user = result.scalar_one_or_none()

    if user is None:
        raise credentials_exception

    if not user.is_active:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Inactive user"
        )

    return user


async def get_current_active_superuser(
    current_user: models.User = Depends(get_current_user)
) -> models.User:
    """Verify that current user is a superuser."""
    if not current_user.is_superuser:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Not enough permissions"
        )
    return current_user
```

## Error Handling

### Custom Exceptions (app/exceptions.py)

```python
"""Custom exceptions and exception handlers."""
from fastapi import Request, status
from fastapi.responses import JSONResponse
from fastapi.exceptions import RequestValidationError
from sqlalchemy.exc import IntegrityError


class CustomException(Exception):
    """Base custom exception."""
    def __init__(self, message: str, status_code: int = status.HTTP_400_BAD_REQUEST):
        self.message = message
        self.status_code = status_code
        super().__init__(self.message)


class DatabaseException(CustomException):
    """Database operation exceptions."""
    def __init__(self, message: str = "Database operation failed"):
        super().__init__(message, status_code=status.HTTP_500_INTERNAL_SERVER_ERROR)


async def custom_exception_handler(request: Request, exc: CustomException) -> JSONResponse:
    """Handle custom exceptions."""
    return JSONResponse(
        status_code=exc.status_code,
        content={
            "error": {
                "message": exc.message,
                "type": exc.__class__.__name__
            }
        }
    )


async def validation_exception_handler(request: Request, exc: RequestValidationError) -> JSONResponse:
    """Handle Pydantic validation errors."""
    return JSONResponse(
        status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
        content={
            "error": {
                "message": "Validation error",
                "details": exc.errors()
            }
        }
    )


async def integrity_error_handler(request: Request, exc: IntegrityError) -> JSONResponse:
    """Handle SQLAlchemy integrity errors."""
    return JSONResponse(
        status_code=status.HTTP_409_CONFLICT,
        content={
            "error": {
                "message": "Database integrity error",
                "detail": "A record with this information already exists"
            }
        }
    )
```

## Testing

### Test Configuration (tests/conftest.py)

```python
"""Pytest configuration and fixtures."""
import pytest
import asyncio
from typing import AsyncGenerator
from httpx import AsyncClient
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker

from app.main import app
from app.database import Base, get_db
from app.config import settings

# Test database URL (use separate test database)
TEST_DATABASE_URL = "postgresql+asyncpg://testuser:testpass@localhost:5432/test_db"

# Create test engine
test_engine = create_async_engine(TEST_DATABASE_URL, echo=True)
TestSessionLocal = async_sessionmaker(
    test_engine,
    class_=AsyncSession,
    expire_on_commit=False
)


@pytest.fixture(scope="session")
def event_loop():
    """Create event loop for async tests."""
    loop = asyncio.get_event_loop_policy().new_event_loop()
    yield loop
    loop.close()


@pytest.fixture(scope="function")
async def db_session() -> AsyncGenerator[AsyncSession, None]:
    """Create database session for tests."""
    # Create tables
    async with test_engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)

    # Create session
    async with TestSessionLocal() as session:
        yield session

    # Drop tables
    async with test_engine.begin() as conn:
        await conn.run_sync(Base.metadata.drop_all)


@pytest.fixture(scope="function")
async def client(db_session: AsyncSession) -> AsyncGenerator[AsyncClient, None]:
    """Create test client with database session override."""
    async def override_get_db():
        yield db_session

    app.dependency_overrides[get_db] = override_get_db

    async with AsyncClient(app=app, base_url="http://test") as ac:
        yield ac

    app.dependency_overrides.clear()
```

### User API Tests (tests/test_users.py)

```python
"""Tests for user API endpoints."""
import pytest
from httpx import AsyncClient
from sqlalchemy.ext.asyncio import AsyncSession

from app import models


@pytest.mark.asyncio
async def test_create_user(client: AsyncClient):
    """Test creating a new user."""
    response = await client.post(
        "/api/v1/users/",
        json={
            "email": "test@example.com",
            "username": "testuser",
            "password": "testpassword123",
            "full_name": "Test User"
        }
    )

    assert response.status_code == 201
    data = response.json()
    assert data["email"] == "test@example.com"
    assert data["username"] == "testuser"
    assert "hashed_password" not in data
    assert "id" in data


@pytest.mark.asyncio
async def test_create_duplicate_user(client: AsyncClient, db_session: AsyncSession):
    """Test creating duplicate user returns error."""
    # Create first user
    await client.post(
        "/api/v1/users/",
        json={
            "email": "test@example.com",
            "username": "testuser",
            "password": "testpassword123"
        }
    )

    # Try to create duplicate
    response = await client.post(
        "/api/v1/users/",
        json={
            "email": "test@example.com",
            "username": "testuser",
            "password": "testpassword123"
        }
    )

    assert response.status_code == 400
    data = response.json()
    assert "already exists" in data["detail"].lower()


@pytest.mark.asyncio
async def test_list_users(client: AsyncClient):
    """Test listing users with pagination."""
    # Create test users
    for i in range(5):
        await client.post(
            "/api/v1/users/",
            json={
                "email": f"user{i}@example.com",
                "username": f"user{i}",
                "password": "password123"
            }
        )

    # Get user list
    response = await client.get("/api/v1/users/")

    assert response.status_code == 200
    data = response.json()
    assert len(data["users"]) == 5
    assert data["total"] == 5
    assert data["page"] == 1


@pytest.mark.asyncio
async def test_get_user(client: AsyncClient):
    """Test getting a specific user."""
    # Create user
    create_response = await client.post(
        "/api/v1/users/",
        json={
            "email": "test@example.com",
            "username": "testuser",
            "password": "testpassword123"
        }
    )
    user_id = create_response.json()["id"]

    # Get user
    response = await client.get(f"/api/v1/users/{user_id}")

    assert response.status_code == 200
    data = response.json()
    assert data["id"] == user_id
    assert data["email"] == "test@example.com"


@pytest.mark.asyncio
async def test_get_nonexistent_user(client: AsyncClient):
    """Test getting nonexistent user returns 404."""
    response = await client.get("/api/v1/users/999")

    assert response.status_code == 404
    data = response.json()
    assert "not found" in data["detail"].lower()
```

## Environment Configuration

### Example .env File

```bash
# Application
APP_NAME=FastAPI Application
ENVIRONMENT=development
DEBUG=true

# Database
DATABASE_URL=postgresql+asyncpg://appuser:devpassword@localhost:5432/appdb

# Security
SECRET_KEY=your-secret-key-change-in-production-use-openssl-rand-hex-32
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

# CORS
CORS_ORIGINS=http://localhost:3000,http://localhost:8000
ALLOWED_HOSTS=localhost,127.0.0.1

# API
API_V1_PREFIX=/api/v1

# Pagination
DEFAULT_PAGE_SIZE=50
MAX_PAGE_SIZE=100
```

## Docker Integration

### Complete Example: FastAPI + PostgreSQL + Alembic

See `.claude/context/orchestration/docker-standards.md` for:
- Complete docker-compose.yml with FastAPI and PostgreSQL
- Dockerfile for FastAPI application
- Entrypoint script with Alembic migrations
- Health checks and networking
- Environment variable configuration

## Best Practices Summary

### Code Organization
- Use modular structure with routers, models, schemas
- Separate concerns: database, config, dependencies
- Group related endpoints in routers with tags

### Async Patterns
- Always use `async def` for route handlers
- Use AsyncSession for all database operations
- Leverage FastAPI's async support for I/O operations

### Type Safety
- Use Python type hints everywhere
- Define Pydantic schemas for all request/response
- Enable strict typing in IDE/mypy

### Security
- Never store passwords in plain text (use bcrypt)
- Use JWT tokens for authentication
- Implement CORS properly (not allow_origins=["*"])
- Validate all inputs with Pydantic
- Use HTTPS in production

### Error Handling
- Use HTTPException for expected errors
- Implement custom exception handlers
- Return consistent error formats
- Log errors for debugging

### Testing
- Write tests for all endpoints
- Use TestClient for API testing
- Mock external dependencies
- Aim for 80%+ coverage

### Documentation
- Add docstrings to all endpoints
- Use Pydantic Field() for schema documentation
- Keep OpenAPI docs enabled in development

## Supporting Components

- **Implementation Guidance**: `.claude/agents/backend-developer.md`
  - When to use FastAPI
  - Why FastAPI for our stack
  - Integration philosophy
- **API Validation**: `/validate-api`
  - Verify FastAPI implementation
  - Check async patterns and security
  - Validate test coverage
- **Database Setup**: `.claude/context/orchestration/docker-standards.md`
  - PostgreSQL configuration
  - Alembic migrations
  - SQLAlchemy patterns
- **Code Quality**: `/lint-all`
  - Python linting and formatting
  - Type checking with mypy

---

**Note**: These patterns ensure production-ready FastAPI applications that integrate seamlessly with our PostgreSQL + SQLAlchemy + Alembic + Docker stack.
