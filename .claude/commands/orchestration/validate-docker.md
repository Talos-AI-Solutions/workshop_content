---
description: Validates Docker configuration against modern containerization standards
allowed-tools: Bash(docker build --dry-run:*)
---

# Docker Configuration Validation

Perform comprehensive validation of Docker configuration against modern containerization standards.

## Validation Steps

### 1. Dockerfile Presence
First, check if a Dockerfile exists in the project root. If not found, report that no Dockerfile exists and exit validation.

### 2. Base Image Standards
Examine the Dockerfile's base image:
- ✅ **Debian-based slim**: Look for `FROM node:.*-slim` (recommended)
- ⚠️ **Alpine consideration**: If using `FROM node:.*-alpine`, suggest Debian-based for native dependencies
- ⚠️ **Non-standard**: Alert if base image doesn't follow recommendations

### 3. Multi-Stage Build Check
Verify build optimization:
- ✅ **Multi-stage detected**: Count multiple `FROM` statements (2 or more)
- ⚠️ **Single-stage**: Recommend multi-stage builds for production optimization

### 4. Build Dependencies
Check for required build tools:
- ✅ **Complete toolchain**: Look for `python3`, `make`, `g++` for native compilation
- ⚠️ **Missing tools**: Alert if build dependencies are missing

### 5. Dependency Copy Order
Validate proper layer caching:
- ✅ **Schema first**: Verify schema files (Prisma) copied before `npm install`
- ⚠️ **Order issue**: Alert if dependency copying order may break caching

### 6. Security Practices
Check security configurations:
- ✅ **Non-root user**: Look for `USER` directive with non-root user
- ⚠️ **Root user**: Recommend adding non-root user for security

### 7. Docker Compose Analysis
Check for compose files and validate their content:
- Look for: `docker-compose.yml`, `docker-compose.yaml`, `docker-compose.dev.yml`, `docker-compose.prod.yml`
- For each found file, verify:
  - ✅ **Services**: Confirm `services:` section exists
  - ✅ **Networks**: Check for custom `networks:` definition
  - ✅ **Volumes**: Verify `volumes:` configuration
  - ✅ **Health checks**: Look for `healthcheck:` configurations
  - ⚠️ **Missing health checks**: Recommend adding if not found

### 8. Next.js Docker Integration
For Next.js projects, validate Docker-specific requirements:
- Check next.config.ts or next.config.js for:
  - ✅ **Standalone output**: Verify `output: "standalone"` exists (required for Docker)
  - ❌ **Missing config**: Report if missing standalone configuration
- ⚠️ **No config file**: Alert if no Next.js config found

### 9. Database Integration
For projects using Prisma:
- ✅ **Schema present**: Confirm `prisma/schema.prisma` exists
- ✅ **Binary targets**: Check for Docker-compatible binary targets like `debian-openssl-1.1.x`, `debian-openssl-3.0.x`
- ⚠️ **Missing targets**: Suggest adding Docker binary targets if missing

### 10. Build Testing
Test Docker build functionality:
- Run `docker build --dry-run .` to test syntax
- ✅ **Valid syntax**: Confirm Dockerfile syntax is correct
- ❌ **Syntax errors**: Report issues and suggest running full build for details
- ⚠️ **Docker unavailable**: Note if Docker isn't available for testing

## Reporting Format
Use emoji indicators for clear status:
- ✅ for validated best practices
- ⚠️ for recommendations and suggestions
- ❌ for errors and required fixes

## Quick Fixes Recommendations
When issues are found, provide actionable suggestions:
- Use `FROM node:20-slim` for base image
- Add `output: "standalone"` to next.config.ts
- Copy schema files BEFORE npm install  
- Use multi-stage builds for production
- Add non-root USER directive
- Include health checks in docker-compose files

## Reference Documentation
For comprehensive Docker standards, reference:
- General: .claude/context/orchestration/docker-standards.md
- Security: .claude/context/orchestration/docker-security-standards.md
- Next.js specific: .claude/context/nextjs_frontend/nextjs-docker-standards.md

## Related Commands
For comprehensive validation and generation:
- `/docker:generate-dockerfile` - Generate optimized Dockerfile
- `/docker:generate-compose` - Create docker-compose.yml configuration
- `/docker:validate-security` - Security-focused validation