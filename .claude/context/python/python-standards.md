# Python Standards and Implementation Patterns

This document provides comprehensive code examples and implementation patterns for Python 3.11+ development.

## Modern Python Patterns

### Dataclasses for Data Structures

```python
from dataclasses import dataclass, field
from typing import List, Optional
from datetime import datetime

@dataclass
class User:
    """User data model with validation."""
    id: int
    username: str
    email: str
    created_at: datetime = field(default_factory=datetime.now)
    roles: List[str] = field(default_factory=list)
    metadata: dict = field(default_factory=dict)

    def __post_init__(self):
        """Validate data after initialization."""
        if not self.email or "@" not in self.email:
            raise ValueError(f"Invalid email: {self.email}")
```

### Pydantic for Data Validation

```python
from pydantic import BaseModel, EmailStr, Field, validator
from typing import Optional
from datetime import datetime

class UserCreate(BaseModel):
    """User creation model with automatic validation."""
    username: str = Field(..., min_length=3, max_length=50)
    email: EmailStr
    password: str = Field(..., min_length=8)
    age: Optional[int] = Field(None, ge=18, le=120)

    @validator('username')
    def username_alphanumeric(cls, v: str) -> str:
        """Ensure username is alphanumeric."""
        if not v.isalnum():
            raise ValueError('Username must be alphanumeric')
        return v

    class Config:
        str_strip_whitespace = True
```

### Dependency Injection Pattern

```python
from typing import Protocol, runtime_checkable
from abc import abstractmethod

@runtime_checkable
class Repository(Protocol):
    """Repository protocol for dependency injection."""

    @abstractmethod
    async def get(self, id: int) -> Optional[dict]:
        """Get item by ID."""
        ...

    @abstractmethod
    async def create(self, data: dict) -> dict:
        """Create new item."""
        ...

class UserService:
    """Service with injected dependencies."""

    def __init__(self, repo: Repository, cache: CacheProtocol):
        self._repo = repo
        self._cache = cache

    async def get_user(self, user_id: int) -> Optional[dict]:
        """Get user with caching."""
        cached = await self._cache.get(f"user:{user_id}")
        if cached:
            return cached

        user = await self._repo.get(user_id)
        if user:
            await self._cache.set(f"user:{user_id}", user, ttl=300)
        return user
```

### Custom Context Managers

```python
from contextlib import contextmanager, asynccontextmanager
from typing import Generator, AsyncGenerator
import time

@contextmanager
def timer(operation: str) -> Generator[None, None, None]:
    """Context manager for timing operations."""
    start = time.perf_counter()
    try:
        yield
    finally:
        elapsed = time.perf_counter() - start
        print(f"{operation} took {elapsed:.3f}s")

@asynccontextmanager
async def database_transaction(
    connection: Connection
) -> AsyncGenerator[Connection, None]:
    """Async context manager for database transactions."""
    async with connection.transaction():
        try:
            yield connection
        except Exception:
            await connection.rollback()
            raise
        else:
            await connection.commit()

# Usage
with timer("data processing"):
    process_data()

async with database_transaction(conn) as tx:
    await tx.execute("INSERT INTO users ...")
```

### Generators for Large Data Processing

```python
from typing import Iterator, Generator
import csv

def read_large_file(filepath: str) -> Generator[dict, None, None]:
    """Process large files efficiently with generators."""
    with open(filepath, 'r') as f:
        reader = csv.DictReader(f)
        for row in reader:
            yield row

def process_batches(
    items: Iterator[dict],
    batch_size: int = 1000
) -> Generator[list[dict], None, None]:
    """Yield batches of items from iterator."""
    batch = []
    for item in items:
        batch.append(item)
        if len(batch) >= batch_size:
            yield batch
            batch = []
    if batch:
        yield batch

# Usage: Memory-efficient processing
for batch in process_batches(read_large_file('data.csv')):
    process_batch(batch)
```

## Web Development Patterns

### FastAPI Application Structure

```python
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel
from typing import List, Optional

app = FastAPI(
    title="My API",
    version="1.0.0",
    docs_url="/docs",
    redoc_url="/redoc"
)

# CORS middleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

class UserResponse(BaseModel):
    id: int
    username: str
    email: str

@app.get("/users/{user_id}", response_model=UserResponse)
async def get_user(
    user_id: int,
    service: UserService = Depends(get_user_service)
) -> UserResponse:
    """Get user by ID."""
    user = await service.get_user(user_id)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"User {user_id} not found"
        )
    return UserResponse(**user)
```

### SQLAlchemy Async Pattern

```python
from sqlalchemy.ext.asyncio import (
    create_async_engine,
    AsyncSession,
    async_sessionmaker
)
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column
from sqlalchemy import select
from typing import Optional

class Base(DeclarativeBase):
    pass

class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    username: Mapped[str] = mapped_column(unique=True, index=True)
    email: Mapped[str] = mapped_column(unique=True)
    is_active: Mapped[bool] = mapped_column(default=True)

# Database setup
engine = create_async_engine(
    "postgresql+asyncpg://user:pass@localhost/db",
    echo=True,
    pool_size=20,
    max_overflow=0
)

AsyncSessionLocal = async_sessionmaker(
    engine,
    class_=AsyncSession,
    expire_on_commit=False
)

async def get_user_by_email(email: str) -> Optional[User]:
    """Query user by email."""
    async with AsyncSessionLocal() as session:
        stmt = select(User).where(User.email == email)
        result = await session.execute(stmt)
        return result.scalar_one_or_none()
```

### Error Handling Middleware

```python
from fastapi import Request, status
from fastapi.responses import JSONResponse
from fastapi.exceptions import RequestValidationError
from loguru import logger

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(
    request: Request,
    exc: RequestValidationError
) -> JSONResponse:
    """Handle validation errors."""
    logger.error(f"Validation error: {exc.errors()}")
    return JSONResponse(
        status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
        content={
            "detail": exc.errors(),
            "body": exc.body
        }
    )

@app.exception_handler(Exception)
async def general_exception_handler(
    request: Request,
    exc: Exception
) -> JSONResponse:
    """Handle unexpected errors."""
    logger.exception(f"Unexpected error: {exc}")
    return JSONResponse(
        status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
        content={"detail": "Internal server error"}
    )
```

## Performance Optimization

### Profiling with cProfile

```python
import cProfile
import pstats
from pstats import SortKey
from functools import wraps

def profile_function(func):
    """Decorator to profile function execution."""
    @wraps(func)
    def wrapper(*args, **kwargs):
        profiler = cProfile.Profile()
        profiler.enable()
        result = func(*args, **kwargs)
        profiler.disable()

        stats = pstats.Stats(profiler)
        stats.sort_stats(SortKey.CUMULATIVE)
        stats.print_stats(20)  # Top 20 functions
        return result
    return wrapper

@profile_function
def expensive_operation():
    """Function to profile."""
    # Your code here
    pass
```

### NumPy Vectorization

```python
import numpy as np
from typing import List

# BAD: Python loop
def calculate_distances_slow(
    points: List[tuple[float, float]]
) -> List[float]:
    """Slow distance calculation."""
    distances = []
    for x, y in points:
        distance = (x**2 + y**2) ** 0.5
        distances.append(distance)
    return distances

# GOOD: NumPy vectorization
def calculate_distances_fast(points: np.ndarray) -> np.ndarray:
    """Fast vectorized distance calculation."""
    return np.sqrt(np.sum(points**2, axis=1))

# Usage
points = np.random.rand(1_000_000, 2)
distances = calculate_distances_fast(points)  # Much faster!
```

### Caching Strategies

```python
from functools import lru_cache, cache
from typing import Dict, Any
import hashlib
import json

@lru_cache(maxsize=128)
def expensive_computation(n: int) -> int:
    """Cache results of expensive computation."""
    # Expensive operation
    return sum(i**2 for i in range(n))

class SmartCache:
    """Custom cache with expiration."""

    def __init__(self):
        self._cache: Dict[str, tuple[Any, float]] = {}

    def get(self, key: str, max_age: float = 300) -> Optional[Any]:
        """Get cached value if not expired."""
        if key in self._cache:
            value, timestamp = self._cache[key]
            if time.time() - timestamp < max_age:
                return value
        return None

    def set(self, key: str, value: Any):
        """Cache value with timestamp."""
        self._cache[key] = (value, time.time())

    @staticmethod
    def cache_key(**kwargs) -> str:
        """Generate cache key from kwargs."""
        serialized = json.dumps(kwargs, sort_keys=True)
        return hashlib.md5(serialized.encode()).hexdigest()
```

## Type System Examples

### Generic Types with TypeVar

```python
from typing import TypeVar, Generic, List, Callable, Protocol

T = TypeVar('T')
K = TypeVar('K')
V = TypeVar('V')

class Repository(Generic[T]):
    """Generic repository pattern."""

    def __init__(self, model_class: type[T]):
        self.model_class = model_class

    async def get(self, id: int) -> Optional[T]:
        """Get entity by ID."""
        # Implementation
        pass

    async def list(self) -> List[T]:
        """List all entities."""
        # Implementation
        pass

# Usage with type safety
user_repo: Repository[User] = Repository(User)
users: List[User] = await user_repo.list()
```

### Protocol Definitions for Duck Typing

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Serializable(Protocol):
    """Protocol for serializable objects."""

    def to_dict(self) -> dict:
        """Convert to dictionary."""
        ...

    @classmethod
    def from_dict(cls, data: dict) -> 'Serializable':
        """Create from dictionary."""
        ...

def save_to_json(obj: Serializable, filepath: str):
    """Save any serializable object to JSON."""
    import json
    with open(filepath, 'w') as f:
        json.dump(obj.to_dict(), f)
```

## Async Programming Patterns

### Async Context Managers

```python
from typing import AsyncContextManager
import aiohttp

class HTTPClient:
    """Async HTTP client with context manager."""

    def __init__(self, base_url: str):
        self.base_url = base_url
        self._session: Optional[aiohttp.ClientSession] = None

    async def __aenter__(self) -> 'HTTPClient':
        """Enter async context."""
        self._session = aiohttp.ClientSession(base_url=self.base_url)
        return self

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        """Exit async context."""
        if self._session:
            await self._session.close()

    async def get(self, path: str) -> dict:
        """Make GET request."""
        async with self._session.get(path) as response:
            return await response.json()

# Usage
async with HTTPClient("https://api.example.com") as client:
    data = await client.get("/users")
```

### Async Generators

```python
from typing import AsyncGenerator
import asyncio

async def fetch_pages(
    base_url: str,
    page_count: int
) -> AsyncGenerator[dict, None]:
    """Async generator for paginated data."""
    async with aiohttp.ClientSession() as session:
        for page in range(1, page_count + 1):
            async with session.get(f"{base_url}?page={page}") as resp:
                data = await resp.json()
                yield data
                await asyncio.sleep(0.1)  # Rate limiting

# Usage
async for page_data in fetch_pages("https://api.example.com/data", 10):
    process_page(page_data)
```

## Testing Patterns

### Pytest Fixtures

```python
import pytest
from typing import AsyncGenerator
from httpx import AsyncClient

@pytest.fixture
async def db_session() -> AsyncGenerator[AsyncSession, None]:
    """Database session fixture."""
    async with AsyncSessionLocal() as session:
        yield session
        await session.rollback()

@pytest.fixture
async def client(db_session: AsyncSession) -> AsyncGenerator[AsyncClient, None]:
    """HTTP client fixture."""
    async with AsyncClient(app=app, base_url="http://test") as client:
        yield client

@pytest.fixture
def user_data() -> dict:
    """Sample user data fixture."""
    return {
        "username": "testuser",
        "email": "test@example.com",
        "password": "securepass123"
    }
```

### Parameterized Tests

```python
import pytest

@pytest.mark.parametrize("input,expected", [
    (1, 1),
    (2, 4),
    (3, 9),
    (4, 16),
    (-2, 4),
])
def test_square(input: int, expected: int):
    """Test square function with multiple inputs."""
    assert square(input) == expected

@pytest.mark.parametrize("email,valid", [
    ("user@example.com", True),
    ("invalid.email", False),
    ("user@", False),
    ("@example.com", False),
])
def test_email_validation(email: str, valid: bool):
    """Test email validation."""
    assert is_valid_email(email) == valid
```

### Async Testing

```python
import pytest

@pytest.mark.asyncio
async def test_create_user(client: AsyncClient, user_data: dict):
    """Test user creation endpoint."""
    response = await client.post("/users", json=user_data)
    assert response.status_code == 201
    data = response.json()
    assert data["username"] == user_data["username"]
    assert data["email"] == user_data["email"]
    assert "id" in data

@pytest.mark.asyncio
async def test_get_user_not_found(client: AsyncClient):
    """Test getting non-existent user."""
    response = await client.get("/users/99999")
    assert response.status_code == 404
```

### Mocking with pytest-mock

```python
import pytest
from unittest.mock import AsyncMock

@pytest.mark.asyncio
async def test_user_service_with_mock(mocker):
    """Test user service with mocked repository."""
    mock_repo = mocker.Mock()
    mock_repo.get = AsyncMock(return_value={
        "id": 1,
        "username": "testuser",
        "email": "test@example.com"
    })

    service = UserService(mock_repo, mock_cache)
    user = await service.get_user(1)

    assert user["username"] == "testuser"
    mock_repo.get.assert_called_once_with(1)
```

## Exception Handling

### Custom Exception Hierarchy

```python
class AppException(Exception):
    """Base application exception."""

    def __init__(self, message: str, code: str = "APP_ERROR"):
        self.message = message
        self.code = code
        super().__init__(message)

class ValidationError(AppException):
    """Validation error."""

    def __init__(self, message: str, field: Optional[str] = None):
        self.field = field
        super().__init__(message, code="VALIDATION_ERROR")

class NotFoundError(AppException):
    """Resource not found error."""

    def __init__(self, resource: str, identifier: Any):
        message = f"{resource} with ID {identifier} not found"
        super().__init__(message, code="NOT_FOUND")

class AuthenticationError(AppException):
    """Authentication error."""

    def __init__(self, message: str = "Authentication failed"):
        super().__init__(message, code="AUTH_ERROR")

# Usage
try:
    user = await get_user(user_id)
    if not user:
        raise NotFoundError("User", user_id)
except NotFoundError as e:
    logger.error(f"{e.code}: {e.message}")
    raise HTTPException(status_code=404, detail=e.message)
```

## Best Practices Summary

1. **Always use type hints** for function signatures and class attributes
2. **Use Pydantic** for data validation and serialization
3. **Implement async/await** for I/O-bound operations
4. **Write comprehensive tests** with pytest (>90% coverage)
5. **Profile before optimizing** - measure, don't guess
6. **Use protocols** for flexible, testable code
7. **Implement proper error handling** with custom exceptions
8. **Cache expensive operations** appropriately
9. **Use generators** for large data processing
10. **Follow Pythonic idioms** and PEP 8 style
