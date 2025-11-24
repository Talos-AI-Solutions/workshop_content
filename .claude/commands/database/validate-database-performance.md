---
description: Identifies database performance issues including N+1 queries, missing indexes, and connection pool problems
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
  - mcp__context7__resolve-library-id
  - mcp__context7__get-library-docs
---

# Validate Database Performance

Identify database performance issues including N+1 queries, missing indexes, inefficient query patterns, and connection pool configuration.

## Validation Steps

### 1. Discover Database Query Patterns
Locate all database query code:
- Use Grep to find `select()`, `execute()`, `scalars()` calls
- Find relationship access patterns (potential N+1)
- Identify bulk operations
- Locate transaction boundaries

### 2. Identify N+1 Query Problems
Check for N+1 query anti-patterns:
- Relationship access inside loops without eager loading
- Missing `selectinload()` or `joinedload()` for accessed relationships
- Lazy loading in list comprehensions or iterations
- Sequential queries that could be combined

### 3. Validate Eager Loading Strategies
Check relationship loading patterns:
- One-to-many relationships use `selectinload()`
- Many-to-one relationships use `joinedload()`
- Nested relationships properly loaded
- No unnecessary eager loading (loading unused relationships)

### 4. Check Index Usage
Verify indexes exist for query patterns:
- All foreign keys have indexes
- WHERE clause columns are indexed
- JOIN columns are indexed
- ORDER BY columns are indexed
- Composite indexes match query patterns

### 5. Validate Query Patterns
Check for inefficient query patterns:
- Using ORM when raw SQL would be better (bulk operations)
- Loading full objects when projections would suffice
- Unnecessary COUNT queries
- Inefficient pagination (offset-based vs cursor-based)
- Multiple queries that could use JOIN

### 6. Check Connection Pool Configuration
Validate database connection pooling:
- Pool size appropriate for application load
- `pool_pre_ping=True` configured (connection health check)
- `pool_recycle=3600` configured (refresh connections hourly)
- Connection timeout settings reasonable
- No connection leaks (sessions properly closed)

### 7. Validate Bulk Operations
Check bulk data handling:
- Large inserts use `bulk_insert_mappings()`
- Large updates use `bulk_update_mappings()`
- Batch operations properly sized (not too large)
- Bulk operations use transactions appropriately

### 8. Check Transaction Boundaries
Validate transaction usage:
- Transactions kept short and focused
- No long-running transactions
- Read-only queries don't unnecessarily acquire locks
- Proper isolation levels for different operations

### 9. Analyze Query Complexity
Check for complex queries needing optimization:
- Subqueries that could be JOINs
- Cartesian products from improper JOINs
- Unused columns being selected
- Opportunities for database-side aggregation

### 10. Check Against Latest Performance Best Practices
Use Context7 to validate:
- SQLAlchemy performance: `/websites/sqlalchemy_en_20` (topic: "query optimization")
- PostgreSQL performance: `/websites/postgresql` (topic: "query performance")

## Validation Output

Generate a comprehensive performance validation report with:

### Performance Health Score
- Number of queries analyzed
- N+1 queries detected
- Missing indexes identified
- Overall performance health score

### Issues by Severity

**CRITICAL (Must Fix):**
- N+1 query problems
- Missing indexes on foreign keys
- Missing indexes on frequently queried columns
- Connection pool exhaustion risk
- Uncontrolled bulk operations

**WARNING (Should Fix):**
- Suboptimal eager loading strategies
- Inefficient pagination patterns
- Long-running transactions
- Missing composite indexes
- Connection pool not configured

**INFO (Consider):**
- Query optimization opportunities
- Projection usage instead of full objects
- Cursor-based pagination
- Query result caching opportunities
- Database-side aggregation

### Recommendations Format
For each issue provide:
- File and line number location
- Current implementation
- Recommended optimization
- Expected performance impact
- Reference to context documentation

## Supporting Documentation

For detailed implementation patterns and code examples:
- **Technical Examples**: `.claude/context/database/python-sqlalchemy-standards.mdc`
- **PostgreSQL Performance**: `.claude/context/database/postgresql-standards.mdc`
- **Agent Guidance**: `.claude/agents/database/python-database-administrator.md`
- **Latest Specs**: Validated against Context7 documentation

## Related Commands
- `/validate-database-schema` - Check schema and indexes
- `/validate-sqlalchemy-models` - Verify eager loading configuration
- `/validate-alembic-migrations` - Ensure indexes are migrated
- `/validate-database-security` - Check security aspects of queries
