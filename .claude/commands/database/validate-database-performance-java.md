---
description: Validates database performance for Java/Spring Boot applications, identifying N+1 queries, missing indexes, inefficient fetch strategies, and connection pool issues
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
  - mcp__context7__resolve-library-id
  - mcp__context7__get-library-docs
---

# Validate Database Performance (Java)

Identify database performance issues for Java/Spring Boot applications including N+1 queries, missing indexes, inefficient fetch strategies, and HikariCP configuration.

## Validation Steps

### 1. Discover Database Query Patterns
Locate all database query code:
- Use Grep to find repository methods, @Query annotations
- Find entity relationship access patterns
- Identify @Transactional boundaries
- Locate bulk operations

### 2. Identify N+1 Query Problems
Check for N+1 query anti-patterns:
- Entity relationship access inside loops without @EntityGraph
- Missing JOIN FETCH in JPQL queries
- Lazy loading in iterations
- Sequential repository calls that could use JOIN

### 3. Validate Eager Loading Strategies
Check fetch strategies:
- `@EntityGraph` used for entities with accessed relationships
- `@NamedEntityGraph` defined for reusable fetch strategies
- JOIN FETCH in custom JPQL queries
- No unnecessary EAGER fetching

### 4. Check Index Usage
Verify indexes exist for query patterns:
- All foreign keys have indexes (@Index)
- WHERE clause columns indexed
- JOIN columns indexed
- ORDER BY columns indexed
- Composite indexes match query patterns

### 5. Validate Repository Patterns
Check for inefficient repository usage:
- Using findAll() when pagination needed
- Loading full entities when projections sufficient
- Multiple repository calls that could use specifications
- Inefficient custom queries

### 6. Check HikariCP Configuration
Validate connection pool (application.properties/yml):
- `spring.datasource.hikari.maximum-pool-size` appropriate (default: 10)
- `spring.datasource.hikari.minimum-idle` configured (default: 5)
- `spring.datasource.hikari.connection-timeout` reasonable (default: 30000ms)
- `spring.datasource.hikari.keepalive-time` configured (300000ms)
- `spring.datasource.hikari.leak-detection-threshold` for debugging (60000ms)

### 7. Validate Transaction Boundaries
Check @Transactional usage:
- @Transactional on SERVICE layer only (not repositories/controllers)
- readOnly = true for query-only methods
- Transactions kept short and focused
- Proper propagation and isolation levels

### 8. Check Bulk Operations
Validate bulk data handling:
- Large inserts use batch processing
- @Modifying for bulk UPDATE/DELETE
- Proper batch size configuration
- Bulk operations within transactions

### 9. Check Against Latest Performance Best Practices
Use Context7 to validate:
- Hibernate performance: `/hibernate/hibernate-orm` (topic: "query optimization")
- Spring Data JPA: `/spring-projects/spring-data-jpa` (topic: "performance")
- PostgreSQL performance: `/websites/postgresql` (topic: "query performance")

## Validation Output

Generate comprehensive performance validation report with health score, N+1 queries detected, missing indexes, and specific optimization recommendations.

## Supporting Documentation

For detailed implementation patterns and code examples:
- **Java/Hibernate Examples**: `.claude/context/database/java-hibernate-standards.mdc`
- **PostgreSQL Performance**: `.claude/context/database/postgresql-standards.mdc`
- **Agent Guidance**: `.claude/agents/database/java-database-administrator.md`

## Related Commands
- `/validate-database-schema` - Check schema and indexes
- `/validate-hibernate-entities` - Verify entity fetch strategies
- `/validate-database-migrations-java` - Ensure indexes migrated
- `/validate-database-security-java` - Check security aspects
