---
description: Validates Flyway or Liquibase migration configuration for Java/Spring Boot projects, ensuring schema changes are properly managed
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
  - mcp__context7__resolve-library-id
  - mcp__context7__get-library-docs
---

# Validate Database Migrations (Java)

Validate Flyway or Liquibase configuration, migration structure, and consistency with JPA entity definitions.

## Validation Steps

### 1. Detect Migration Tool
Determine which migration tool is used:
- Check pom.xml/build.gradle for Flyway or Liquibase dependency
- Verify only one migration tool is configured (not both)

### 2. Validate Flyway Configuration (if Flyway)
Check application.properties/application.yml:
- `spring.flyway.enabled=true`
- `spring.flyway.locations=classpath:db/migration`
- `spring.jpa.hibernate.ddl-auto=validate` or `none` (NOT update/create)
- Migration files in correct location (src/main/resources/db/migration)

### 3. Validate Flyway Migration Files (if Flyway)
Check migration file structure:
- Naming: `V1__description.sql`, `V2__description.sql`
- Sequential version numbers
- Descriptive names
- No gaps in version sequence
- Repeatable migrations: `R__description.sql`

### 4. Validate Liquibase Configuration (if Liquibase)
Check application.properties/application.yml:
- `spring.liquibase.enabled=true`
- `spring.liquibase.change-log=classpath:db/changelog/db.changelog-master.xml`
- `spring.jpa.hibernate.ddl-auto=validate` or `none`
- Changelog files in correct location

### 5. Validate Liquibase Changesets (if Liquibase)
Check changeset structure:
- Master changelog exists
- Changesets have unique IDs
- Changesets have author specified
- Rollback instructions provided
- Proper preconditions where needed

### 6. Check Entity-Migration Sync
Verify entities and migrations are in sync:
- Compare entity definitions with current schema
- Identify drift between entities and database
- Check for pending entity changes not migrated

### 7. Validate Migration Safety
Check migration safety practices:
- No destructive operations without backups
- Rollback strategies defined
- Foreign key constraints handled correctly
- Index creation optimized (CONCURRENTLY where applicable)

### 8. Check Against Latest Migration Standards
Use Context7 to validate:
- Flyway documentation: (if applicable)
- Liquibase documentation: (if applicable)
- Spring Boot migration best practices

## Validation Output

Generate comprehensive migration health report with configuration assessment, entity sync status, and specific recommendations.

## Supporting Documentation

For detailed implementation patterns and code examples:
- **Java/Hibernate Examples**: `.claude/context/database/java-hibernate-standards.mdc`
- **Agent Guidance**: `.claude/agents/database/java-database-administrator.md`

## Related Commands
- `/validate-database-schema` - Check schema quality
- `/validate-hibernate-entities` - Verify entity definitions
- `/validate-database-performance-java` - Check migration performance impact
- `/validate-database-security-java` - Verify migration security
