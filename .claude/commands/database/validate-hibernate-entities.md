---
description: Validates JPA entities follow Hibernate 6+ and JPA 3.1+ best practices with explicit annotations, proper relationships, and lazy loading
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
  - mcp__context7__resolve-library-id
  - mcp__context7__get-library-docs
---

# Validate Hibernate Entities

Validate JPA entities for Hibernate 6+ and JPA 3.1+ best practices including explicit annotations, proper relationships, lazy loading, and Spring Data JPA compatibility.

## Validation Steps

### 1. Discover JPA Entities
Locate all entity classes:
- Use Grep to find files with `@Entity` annotation
- Identify all entity class declarations
- Locate repository interfaces

### 2. Validate Entity Annotations
Check entities have explicit annotations:
- All entities MUST have `@Entity` and `@Table(name = "table_name")`
- All columns MUST have `@Column(name = "column_name", nullable = false/true)`
- Primary keys use `@Id` with `@GeneratedValue(strategy = GenerationType.IDENTITY)`
- No reliance on default naming conventions

### 3. Validate Relationship Annotations
Check relationships are properly configured:
- All relationships specify `@JoinColumn(name = "foreign_key_column")`
- Bidirectional relationships have `mappedBy` on inverse side
- All relationships explicitly specify `fetch = FetchType.LAZY`
- No use of `FetchType.EAGER` unless documented necessity

### 4. Validate Foreign Key Indexing
Check foreign key columns have indexes:
- All `@JoinColumn` fields MUST have `@Index` via `@Table(indexes = {...})`
- Critical for PostgreSQL performance (doesn't auto-index FKs)

### 5. Validate Cascade Configuration
Check cascade rules are intentional:
- Cascade types explicitly defined (PERSIST, MERGE, REMOVE)
- No blind use of `CascadeType.ALL`
- Dangerous cascades (REMOVE, delete-orphan) documented with comments

### 6. Validate Entity Design Patterns
Check entity design follows best practices:
- Audit fields use `@CreatedDate`, `@LastModifiedDate` with `@EntityListeners(AuditingEntityListener.class)`
- Temporal fields use `@Temporal(TemporalType.TIMESTAMP)` stored in UTC
- Proper equals() and hashCode() implementation
- Lombok annotations used appropriately

### 7. Check Against Latest Hibernate Standards
Use Context7 to validate:
- Hibernate ORM: `/hibernate/hibernate-orm` (topic: "entity mapping")
- Spring Data JPA: `/spring-projects/spring-data-jpa` (topic: "entity relationships")

## Validation Output

Generate comprehensive validation report with entity quality score, issues by severity (CRITICAL/WARNING/INFO), and specific recommendations.

## Supporting Documentation

For detailed implementation patterns and code examples:
- **Hibernate/JPA Examples**: `.claude/context/database/java-hibernate-standards.mdc`
- **PostgreSQL Standards**: `.claude/context/database/postgresql-standards.mdc`
- **Agent Guidance**: `.claude/agents/database/java-database-administrator.md`

## Related Commands
- `/validate-database-schema` - Check schema quality
- `/validate-database-migrations-java` - Verify Flyway/Liquibase
- `/validate-database-performance-java` - Identify N+1 queries
- `/validate-database-security-java` - Check security practices
