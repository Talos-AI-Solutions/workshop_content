---
description: Validates SQLAlchemy 2.0+ models for async patterns, relationships, and type safety
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
  - mcp__context7__resolve-library-id
  - mcp__context7__get-library-docs
---

# Validate SQLAlchemy Models

Validate SQLAlchemy models for 2.0+ async patterns, proper relationships, and type safety compatibility with FastAPI.

## Validation Steps

### 1. Discover SQLAlchemy Models
Locate all model files and analyze imports:
- Use Grep to find files with: `DeclarativeBase`, `Mapped`, `mapped_column`, `AsyncAttrs`, `relationship`
- Identify model base class definitions
- Identify all model class declarations

### 2. Validate Base Class Configuration
Check base classes use async-compatible configuration:
- Base class MUST inherit from `AsyncAttrs` for lazy loading support
- Base class MUST inherit from `DeclarativeBase`
- No legacy `declarative_base()` usage

### 3. Validate Type Hints
Check all columns use modern `Mapped[]` type annotations:
- All columns MUST use `Mapped[Type]` annotations
- Use `Mapped[Optional[Type]]` for nullable columns
- Import types from `typing` module
- Type hints MUST match column definitions

### 4. Validate Relationship Configuration
Check relationships are properly configured:
- All relationships MUST use `back_populates` parameter
- Relationship type hints MUST use `Mapped[]`
- One-to-many: `Mapped[List[Type]]`
- Many-to-one: `Mapped[Type]`
- Optional relationships: `Mapped[Optional[Type]]`

### 5. Validate Foreign Key Indexing
Check foreign key columns have indexes:
- All foreign key columns MUST have `index=True` or explicit `Index()`
- Critical for PostgreSQL performance (doesn't auto-index FKs)

### 6. Validate Cascade Configuration
Check cascade rules are intentional and documented:
- Cascade rules SHOULD be explicitly defined
- Dangerous cascades (delete, delete-orphan) MUST be documented with comments
- Default cascade is usually safe (save-update, merge)

### 7. Validate Async Query Patterns
Check code uses async-compatible query patterns:
- Use `select()` for all queries, not legacy Query API
- Use `AsyncSession` not synchronous `Session`
- Use `await session.execute()` or `await session.scalars()`
- Use `selectinload()` or `joinedload()` for eager loading

### 8. Validate Session Management
Check sessions are properly managed:
- Sessions MUST be created per request (not globally)
- Sessions MUST be closed after use (use context managers)
- Use `async with` for session lifecycle
- Explicit commits with `await session.commit()`

### 9. Check Against Latest SQLAlchemy Standards
Use Context7 to validate against SQLAlchemy 2.0+ best practices:
- SQLAlchemy 2.0 async: `/websites/sqlalchemy_en_20` (topic: "async session patterns")
- SQLAlchemy relationships: `/websites/sqlalchemy_en_20_orm` (topic: "relationship configuration")

## Validation Output

Generate a comprehensive validation report with:

### Model Quality Score
- Number of models validated
- Number of relationships checked
- Number of issues found (by severity)
- Overall model quality score (percentage)

### Issues by Severity

**CRITICAL (Must Fix):**
- Missing AsyncAttrs base class
- Foreign keys without indexes
- Synchronous Session usage in async code
- Missing await on async operations
- Legacy Query API usage

**WARNING (Should Fix):**
- Missing type hints (`Mapped[]`)
- Relationships without `back_populates`
- Using `backref` instead of `back_populates`
- Missing eager loading (potential N+1)
- Cascade rules without documentation

**INFO (Consider):**
- Opportunities for query optimization
- Alternative eager loading strategies
- Type hint improvements

### Recommendations Format
For each issue provide:
- File and line number location
- Current implementation
- Recommended implementation
- Impact explanation
- Reference to context documentation

## Supporting Documentation

For detailed implementation patterns and code examples:
- **Technical Examples**: `.claude/context/database/python-sqlalchemy-standards.mdc`
- **PostgreSQL Standards**: `.claude/context/database/postgresql-standards.mdc`
- **Agent Guidance**: `.claude/agents/database/python-database-administrator.md`
- **Latest Specs**: Validated against Context7 documentation

## Related Commands
- `/validate-database-schema` - Check schema quality and constraints
- `/validate-alembic-migrations` - Verify migration configuration
- `/validate-database-performance` - Identify N+1 queries and missing indexes
- `/validate-database-security` - Check security best practices
