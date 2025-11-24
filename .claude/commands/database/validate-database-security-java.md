---
description: Validates database security best practices for Java/Spring Boot applications, checking credentials management, SQL injection prevention, and connection security
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
  - mcp__context7__resolve-library-id
  - mcp__context7__get-library-docs
---

# Validate Database Security (Java)

Validate database security best practices for Java/Spring Boot applications including credential management, SQL injection prevention, access control, and data protection.

## Validation Steps

### 1. Check Credential Management
Verify database credentials are handled securely:
- application.properties/yml use environment variables or external config
- No hardcoded passwords in source code
- spring-boot-starter-data-jpa configured with secure properties
- Production uses external configuration (AWS Secrets Manager, Vault)
- Credentials not committed to version control

### 2. Validate SQL Injection Prevention
Check for SQL injection vulnerabilities:
- Repositories use JPA query methods (type-safe)
- @Query uses parameter binding (`:param` or `?1` syntax)
- No string concatenation in JPQL/native queries
- nativeQuery queries use proper parameter binding
- User input validated before queries

### 3. Check Connection Security
Validate database connection security:
- SSL/TLS enabled (spring.datasource.url includes sslmode=require)
- Certificate validation configured
- No plaintext passwords in connection strings
- Connection encryption in transit

### 4. Validate Access Control
Check database access patterns:
- Principle of least privilege (app user minimal permissions)
- Read-only datasource for reporting
- No database admin/superuser accounts in application
- Row-level security where applicable

### 5. Check Data Protection
Validate sensitive data handling:
- Passwords hashed with BCryptPasswordEncoder or similar
- PII encrypted at rest (JPA @Convert with encryption)
- Credit card data follows PCI DSS
- Sensitive fields use encryption converters
- Audit logging for sensitive operations (Spring Data JPA Auditing)

### 6. Validate Input Validation
Check input validation:
- @Valid and @Validated on controller inputs
- Bean Validation annotations (@NotNull, @Size, @Pattern)
- Custom validators for complex rules
- Type safety from strong typing
- Business logic validation in service layer

### 7. Check Error Handling
Validate error message security:
- @ExceptionHandler doesn't expose database details
- Custom error responses hide internal schema
- Stack traces not exposed in production
- Generic messages for authentication failures
- Detailed errors logged server-side only

### 8. Validate Transaction Security
Check transaction handling:
- @Transactional(readOnly = true) for queries
- Proper isolation levels prevent data races
- No transaction timeout vulnerabilities
- Rollback on exceptions configured

### 9. Check Logging Security
Validate logging practices:
- No logging of sensitive data (passwords, tokens)
- SQL logging disabled in production
- Hibernate statistics disabled in production (hibernate.generate_statistics=false)
- Audit logging for security events

### 10. Check Against Latest Security Standards
Use Context7 to validate:
- Spring Security: (if applicable)
- Hibernate security: `/hibernate/hibernate-orm` (topic: "security")
- PostgreSQL security: `/websites/postgresql` (topic: "security best practices")

## Validation Output

Generate comprehensive security validation report with health score, critical vulnerabilities, and specific secure implementation recommendations.

## Supporting Documentation

For detailed implementation patterns and code examples:
- **Java/Hibernate Security**: `.claude/context/database/java-hibernate-standards.mdc`
- **PostgreSQL Security**: `.claude/context/database/postgresql-standards.mdc`
- **Agent Guidance**: `.claude/agents/database/java-database-administrator.md`

## Related Commands
- `/validate-database-schema` - Check schema design
- `/validate-hibernate-entities` - Verify entity security
- `/validate-database-migrations-java` - Ensure migrations secure
- `/validate-database-performance-java` - Check performance security issues
