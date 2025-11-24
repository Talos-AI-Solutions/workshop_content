---
description: Validates PostgreSQL database schema against quality standards and best practices
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
  - mcp__context7__resolve-library-id
  - mcp__context7__get-library-docs
---

# Validate Database Schema

Validate PostgreSQL database schema for production-quality standards.

## Validation Steps

### 1. Discover Database Models
Locate all database model definitions:
- Use Glob to find Python files with SQLAlchemy models (look for `models/`, `database/` directories)
- Use Grep to find files containing `DeclarativeBase`, `Mapped`, or `mapped_column`
- For Java: Find files with `@Entity` annotations

### 2. Validate Primary Keys
Check that every table has a primary key:
- Every model MUST have at least one column with `primary_key=True`
- Prefer `bigserial` (BIGINT) over `serial` (INTEGER)
- Document reasoning for composite primary keys

### 3. Validate Foreign Key Constraints
Check foreign keys are properly defined and indexed:
- All foreign key columns MUST have explicit `ForeignKey()` constraint
- All foreign key columns MUST have explicit `Index()` (PostgreSQL doesn't auto-index)
- Relationships MUST use `back_populates` for bidirectional references

### 4. Validate Column Data Types
Check columns use appropriate PostgreSQL data types:
- DateTime columns MUST use `DateTime(timezone=True)`
- Email addresses should use `String` with reasonable length (e.g., 255)
- Text content: `Text` for unlimited, `String` for limited
- Boolean columns should use `Boolean` not `Integer`
- Monetary values should use `Numeric` with precision, not `Float`

### 5. Validate NOT NULL Constraints
Check nullable/non-nullable columns are appropriately defined:
- Primary keys are automatically NOT NULL
- Foreign keys should typically be NOT NULL unless optional relationships
- Critical fields (email, username) should be NOT NULL
- Optional fields should explicitly use `Optional[]` type hint

### 6. Validate Timestamp Columns
Check tables have appropriate timestamp tracking:
- Tables SHOULD have `created_at` with `server_default=func.now()`
- Mutable tables SHOULD have `updated_at` with `onupdate=func.now()`
- Both MUST use `DateTime(timezone=True)`

### 7. Validate Table Constraints
Check for additional database-level constraints:
- UNIQUE constraints for fields that should be unique (email, username)
- CHECK constraints for business logic validation
- Composite unique constraints where appropriate

### 8. Check Against Latest PostgreSQL Standards
Use Context7 to validate against current PostgreSQL best practices:
- PostgreSQL schema design: `/websites/postgresql` (topic: "schema design and constraints")
- PostgreSQL indexing: `/websites/postgresql` (topic: "indexing strategies")

## Validation Output

Generate a comprehensive validation report with:

### Schema Quality Score
- Number of tables validated
- Number of issues found (by severity)
- Overall schema quality score (percentage)

### Issues by Severity

**CRITICAL (Must Fix):**
- Tables without primary keys
- Foreign keys without indexes
- DateTime columns without timezone awareness
- Incorrect data types for common fields

**WARNING (Should Fix):**
- Missing timestamp tracking (created_at/updated_at)
- Missing UNIQUE constraints on likely-unique fields
- Optional foreign keys without explicit nullable=True
- Tables without any indexes beyond primary key

**INFO (Consider):**
- Composite index opportunities
- Denormalization opportunities
- Performance optimization suggestions

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
- **PostgreSQL Best Practices**: `.claude/context/database/postgresql-standards.mdc`
- **Java/Hibernate Examples**: `.claude/context/database/java-hibernate-standards.mdc`
- **Agent Guidance**: `.claude/agents/database/python-database-administrator.md`

## Related Commands
- `/validate-sqlalchemy-models` - Verify async patterns and relationships
- `/validate-alembic-migrations` - Check migration configuration
- `/validate-database-performance` - Identify performance issues
- `/validate-database-security` - Check security best practices
