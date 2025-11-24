---
description: Validates FastAPI implementation against modern API development standards
allowed-tools: Bash(pytest:*), Bash(uv run pytest:*), Bash(uv run uvicorn:*), Bash(alembic:*), Bash(uv run alembic:*), Bash(curl:*), Bash(pkill:*), Bash(kill:*)
---

# FastAPI Implementation Validation

Perform comprehensive validation of FastAPI implementation against modern API development standards and best practices.

## Validation Steps

### 1. Project Type Verification
First, verify this is a Python API project by checking for Python dependency files. If neither found, report this doesn't appear to be a Python project.
- Check for @pyproject.toml (modern Python projects with uv)
- Check for requirements.txt (traditional Python projects)
- ❌ **Not a Python project**: Exit validation if no Python project files found

### 2. FastAPI Dependency Validation
Check for required FastAPI stack dependencies in @pyproject.toml or requirements.txt:
- ✅ **FastAPI**: Verify fastapi is present in dependencies
- ✅ **Uvicorn**: Confirm uvicorn server is available for async ASGI
- ✅ **Pydantic**: Check for pydantic (usually included with FastAPI)
- ✅ **SQLAlchemy**: Verify sqlalchemy for database ORM
- ✅ **Alembic**: Confirm alembic for database migrations
- ✅ **Database Driver**: Look for psycopg2 or asyncpg for PostgreSQL
- ⚠️ **Missing dependencies**: Report any missing core dependencies

### 3. Project Structure Validation
Validate proper FastAPI project structure:
- ✅ **Main application**: Look for `app/main.py` or `src/main.py` or `main.py`
- ✅ **Router organization**: Check for `routers/` directory for modular endpoints
- ✅ **Models**: Verify `models/` directory for SQLAlchemy models
- ✅ **Schemas**: Check for `schemas/` directory for Pydantic models
- ✅ **Database config**: Look for `database.py` or `db.py` for connection setup
- ⚠️ **Flat structure**: Recommend modular structure if everything in single file

### 4. FastAPI Application Configuration
Check the main FastAPI application file for proper setup:
- ✅ **FastAPI instance**: Verify `app = FastAPI()` declaration exists
- ✅ **Title and version**: Check if API includes title, description, version in FastAPI()
- ✅ **Router inclusion**: Look for `app.include_router()` calls for modular routing
- ✅ **CORS middleware**: Check for CORSMiddleware configuration
- ⚠️ **Missing metadata**: Recommend adding API metadata for better documentation

### 5. Async Pattern Validation
Verify proper async implementation:
- ✅ **Async endpoints**: Check for `async def` in route handlers
- ✅ **AsyncSession**: Verify use of AsyncSession for database operations (not sync Session)
- ✅ **Async engine**: Check database.py uses `create_async_engine` if async
- ⚠️ **Sync patterns**: Alert if using synchronous database patterns with FastAPI async

### 6. Pydantic Schema Validation
Check for proper request/response validation:
- ✅ **BaseModel schemas**: Verify Pydantic schemas inherit from BaseModel
- ✅ **Type hints**: Check schemas use proper Python type hints
- ✅ **Validation**: Look for Field() validators or custom validators
- ✅ **Response models**: Verify endpoints specify `response_model` parameter
- ⚠️ **No schemas**: Recommend creating Pydantic schemas if using raw dicts

### 7. Database Integration Validation
Verify proper SQLAlchemy and Alembic integration:
- ✅ **Alembic config**: Check for `alembic.ini` and `alembic/` directory
- ✅ **Migration versions**: Verify `alembic/versions/` contains migration files
- ✅ **SQLAlchemy models**: Check models inherit from declarative base
- ✅ **Dependency injection**: Look for database session dependency (Depends(get_db))
- ⚠️ **No migrations**: Recommend initializing Alembic if missing

### 8. OpenAPI Documentation
Validate automatic API documentation:
- ✅ **Docs available**: Attempt to verify FastAPI generates docs at `/docs` and `/redoc`
- ✅ **Schema generation**: Confirm FastAPI can generate OpenAPI schema
- ✅ **Tags organization**: Check if routers use tags for documentation grouping
- ⚠️ **Docs disabled**: Alert if docs are disabled in production (common but note it)

### 9. Security Implementation
Check for security best practices:
- ✅ **Authentication**: Look for authentication dependencies or middleware
- ✅ **Password hashing**: Check for password hashing (bcrypt, passlib)
- ✅ **Environment variables**: Verify use of environment variables for secrets
- ✅ **CORS configuration**: Confirm CORS is properly configured (not allow_origins=["*"] in production)
- ✅ **Input validation**: Verify Pydantic schemas validate all inputs
- ⚠️ **Security concerns**: Report any insecure patterns found

### 10. Error Handling
Validate proper error handling patterns:
- ✅ **HTTPException**: Check for proper use of HTTPException for errors
- ✅ **Status codes**: Verify appropriate HTTP status codes (400, 401, 404, 500)
- ✅ **Exception handlers**: Look for custom exception handlers (@app.exception_handler)
- ✅ **Error responses**: Check error responses follow consistent format
- ⚠️ **Generic errors**: Recommend specific error messages over generic ones

### 11. Health Check Endpoints
Verify monitoring and health check endpoints:
- ✅ **Basic health check**: Look for `/health` or similar endpoint
- ✅ **Database health**: Check for endpoint that verifies database connectivity
- ✅ **Readiness check**: Look for `/ready` endpoint for container orchestration
- ⚠️ **Missing health checks**: Recommend adding health checks for production deployment

### 12. Testing Implementation
Validate test coverage and quality:
- ✅ **Test directory**: Check for `tests/` directory
- ✅ **pytest**: Verify pytest is in dev dependencies
- ✅ **TestClient**: Look for FastAPI TestClient usage in tests
- ✅ **Test coverage**: Run pytest with coverage if available
- ⚠️ **Low coverage**: Recommend increasing coverage if below 80%
- ⚠️ **No tests**: Strongly recommend adding tests if none found

### 13. Development Server Validation (Optional)
Test that the API can start successfully:
- **IMPORTANT**: Only run if explicitly needed and for minimal time
- **Port**: Use port 8001 to avoid conflicts (default 8000 may be in use)
- **Command**: Use `uv run uvicorn app.main:app --host 127.0.0.1 --port 8001` for short test
- **Timeout**: Maximum 10 seconds for startup validation
- **Cleanup**: ALWAYS kill the uvicorn process after validation
- **Process Management**: Use PID tracking for reliable cleanup
- ✅ **Starts successfully**: API server starts without errors
- ❌ **Startup errors**: Report configuration or dependency issues
- ⚠️ **Skip if risky**: Don't start server if database connection required but unavailable

### 14. Configuration and Environment
Validate environment and configuration management:
- ✅ **Environment variables**: Check for .env.example or similar documentation
- ✅ **Settings management**: Look for Pydantic Settings or similar config management
- ✅ **Database URL**: Verify DATABASE_URL environment variable usage
- ⚠️ **Hardcoded secrets**: Alert if any secrets appear hardcoded

## Reporting Format
Use emoji indicators for clear status:
- ✅ for validated best practices and correct implementations
- ⚠️ for recommendations and suggestions
- ❌ for errors and required fixes

## Quick Fix Recommendations
When issues are found, provide actionable suggestions:
- Install missing dependencies: `uv add fastapi uvicorn sqlalchemy alembic pydantic`
- Initialize Alembic: `uv run alembic init alembic`
- Create health check endpoint in dedicated router
- Add async database session dependency injection
- Implement proper error handlers with HTTPException
- Add pytest and TestClient for testing: `uv add --dev pytest httpx`
- Create Pydantic schemas for request/response validation
- Add CORS middleware for cross-origin requests
- Use environment variables with pydantic-settings

## Implementation Examples
For detailed implementation patterns, reference:
- **FastAPI Patterns**: .claude/context/python/api-framework-standards.md
  - Complete project structure examples
  - Async endpoint implementations
  - Pydantic schema patterns
  - Authentication and authorization
  - Error handling strategies
  - Testing approaches
- **Database Integration**: .claude/context/orchestration/docker-standards.md
  - SQLAlchemy configuration
  - Alembic migration setup
  - PostgreSQL container configuration
- **Agent Guidance**: .claude/agents/backend-developer.md
  - When to use FastAPI
  - Why FastAPI is our standard
  - Integration with our stack

## Success Criteria
FastAPI implementation is production-ready when:
- [x] FastAPI and all required dependencies installed
- [x] Proper async patterns used throughout
- [x] Pydantic schemas validate all inputs
- [x] Database integration with SQLAlchemy + Alembic working
- [x] OpenAPI documentation accessible
- [x] Security patterns implemented (auth, CORS, input validation)
- [x] Error handling follows FastAPI best practices
- [x] Health check endpoints present
- [x] Test coverage exceeds 80%
- [x] No hardcoded secrets or insecure patterns

---

**Note**: This validation ensures FastAPI implementations follow modern best practices and integrate properly with our standardized PostgreSQL + SQLAlchemy + Alembic stack.
