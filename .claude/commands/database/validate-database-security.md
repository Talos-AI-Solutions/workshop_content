---
description: Validates database security best practices including credential management, SQL injection prevention, and access control
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
  - mcp__context7__resolve-library-id
  - mcp__context7__get-library-docs
---

# Validate Database Security

Validate database security best practices including credential management, SQL injection prevention, access control, and data protection.

## Validation Steps

### 1. Check Credential Management
Verify database credentials are handled securely:
- Database URLs use environment variables (no hardcoded passwords)
- Credentials not committed to version control (.env in .gitignore)
- Connection strings don't expose credentials in logs
- Secrets management system used for production (e.g., AWS Secrets Manager)

### 2. Validate SQL Injection Prevention
Check for SQL injection vulnerabilities:
- All queries use parameterized statements
- No string concatenation or f-strings in SQL queries
- User input properly sanitized
- ORM queries preferred over raw SQL
- When raw SQL needed, proper parameter binding used

### 3. Check Connection Security
Validate database connection security:
- SSL/TLS enabled for database connections (sslmode=require)
- Certificate validation configured
- Connection encryption in transit
- No plaintext passwords in connection strings

### 4. Validate Access Control
Check database access control patterns:
- Principle of least privilege (app user has minimal permissions)
- Read-only users for analytics/reporting queries
- No use of database superuser/admin accounts
- Row-level security configured where applicable

### 5. Check Data Protection
Validate sensitive data handling:
- Passwords hashed with strong algorithms (bcrypt, argon2)
- Personally Identifiable Information (PII) encrypted at rest
- Credit card data follows PCI DSS standards
- Sensitive columns use encryption functions
- Audit logging for sensitive data access

### 6. Validate Input Validation
Check input validation and sanitization:
- User input validated before database operations
- Type checking on inputs
- Length limits enforced
- Special characters handled properly
- Business logic validation before persistence

### 7. Check Error Handling
Validate error message security:
- Database errors don't expose schema details to users
- Stack traces not exposed in production
- Generic error messages for authentication failures
- Detailed errors logged server-side only

### 8. Validate Session Security
Check database session security:
- Session timeouts configured
- No session fixation vulnerabilities
- Session data not exposed in URLs
- Proper session cleanup on logout

### 9. Check Backup Security
Validate database backup practices:
- Backups encrypted at rest
- Backup credentials separate from application credentials
- Backup files access controlled
- Regular backup testing performed

### 10. Validate Audit Logging
Check audit logging implementation:
- Sensitive operations logged
- User actions traceable
- Log tampering prevented
- Logs retained per compliance requirements

### 11. Check Against Latest Security Standards
Use Context7 to validate:
- PostgreSQL security: `/websites/postgresql` (topic: "security best practices")
- SQLAlchemy security: `/websites/sqlalchemy_en_20` (topic: "SQL injection prevention")

## Validation Output

Generate a comprehensive security validation report with:

### Security Health Score
- Number of security checks performed
- Critical vulnerabilities found
- Overall security health score

### Issues by Severity

**CRITICAL (Must Fix Immediately):**
- Hardcoded database credentials
- SQL injection vulnerabilities
- Plaintext password storage
- Superuser account used by application
- No SSL/TLS for database connections
- PII stored unencrypted

**WARNING (Should Fix):**
- Environment variables not used
- Missing input validation
- Weak password hashing algorithm
- Database errors exposing schema
- No audit logging for sensitive operations
- Missing backup encryption

**INFO (Consider):**
- Row-level security implementation
- Enhanced audit logging
- Additional encryption for sensitive columns
- Security scanning automation
- Penetration testing recommendations

### Recommendations Format
For each issue provide:
- File and line number location
- Current implementation
- Security risk explanation
- Recommended secure implementation
- Reference to context documentation

## Supporting Documentation

For detailed implementation patterns and code examples:
- **Security Examples**: `.claude/context/database/python-sqlalchemy-standards.mdc`
- **PostgreSQL Security**: `.claude/context/database/postgresql-standards.mdc`
- **Agent Guidance**: `.claude/agents/database/python-database-administrator.md`
- **Latest Specs**: Validated against Context7 documentation

## Related Commands
- `/validate-database-schema` - Check schema design
- `/validate-sqlalchemy-models` - Verify model security
- `/validate-alembic-migrations` - Ensure migrations are secure
- `/validate-database-performance` - Check for performance-related security issues
