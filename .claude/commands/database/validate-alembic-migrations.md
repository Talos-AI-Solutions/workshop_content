---
description: Validates Alembic migration configuration and ensures migrations are properly structured
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
  - mcp__context7__resolve-library-id
  - mcp__context7__get-library-docs
---

# Validate Alembic Migrations

Validate Alembic configuration, migration structure, and consistency with model definitions.

## Validation Steps

### 1. Check Alembic Initialization
Verify Alembic is properly set up:
- `alembic.ini` configuration file exists
- `alembic/` directory structure exists with `env.py`, `script.py.mako`
- `alembic/versions/` directory contains migration files
- `env.py` is configured for async operations

### 2. Validate Alembic Configuration
Check `alembic.ini` and `env.py` settings:
- Database URL configuration (should use environment variables)
- Async engine configuration with `asyncpg` driver
- Correct import paths for models
- Logging configuration

### 3. Validate Migration Structure
Check each migration file:
- All migrations have both `upgrade()` and `downgrade()` functions
- Revision identifiers are unique
- Migration chain is linear (no branches)
- Down revisions are correctly linked

### 4. Validate Migration Naming
Check migration filenames and descriptions:
- Descriptive revision messages
- Consistent naming conventions
- Timestamps in revision IDs

### 5. Check Model-Migration Sync
Verify models and migrations are in sync:
- Run `alembic check` to detect drift
- Compare current HEAD with model definitions
- Identify any pending model changes
- Check for manual schema changes outside Alembic

### 6. Validate Async Configuration
Check `env.py` for async patterns:
- Uses `create_async_engine()`
- Uses `async with engine.begin()` context manager
- Imports `asyncio` and uses `asyncio.run()`
- Connection configuration compatible with async operations

### 7. Test Migration Safety
Validate migration safety practices:
- No destructive operations without backups
- Downgrade functions properly reverse upgrades
- Foreign key constraints handled correctly
- Index creation doesn't block table operations

### 8. Check Against Latest Alembic Standards
Use Context7 to validate against Alembic best practices:
- Alembic documentation: `/sqlalchemy/alembic` (topic: "async migrations")
- SQLAlchemy async: `/websites/sqlalchemy_en_20` (topic: "async engine configuration")

## Validation Output

Generate a comprehensive validation report with:

### Migration Health Score
- Number of migrations validated
- Configuration quality assessment
- Model sync status
- Overall migration health score

### Issues by Severity

**CRITICAL (Must Fix):**
- Missing Alembic initialization
- Migrations without downgrade functions
- Model-migration drift detected
- Synchronous engine with async code
- Broken migration chain

**WARNING (Should Fix):**
- Missing descriptive revision messages
- Environment variables hardcoded
- No migration safety checks
- Inconsistent naming conventions

**INFO (Consider):**
- Migration optimization opportunities
- Better downgrade strategies
- Additional safety checks

### Recommendations Format
For each issue provide:
- File and line number location
- Current configuration
- Recommended configuration
- Impact explanation
- Reference to context documentation

## Supporting Documentation

For detailed implementation patterns and code examples:
- **Technical Examples**: `.claude/context/database/python-sqlalchemy-standards.mdc`
- **Agent Guidance**: `.claude/agents/database/python-database-administrator.md`
- **Latest Specs**: Validated against Context7 documentation

## Related Commands
- `/validate-database-schema` - Check schema quality
- `/validate-sqlalchemy-models` - Verify model definitions
- `/validate-database-performance` - Check migration performance impact
- `/validate-database-security` - Verify migration security
