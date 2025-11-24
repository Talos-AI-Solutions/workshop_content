---
description: Generate optimized Dockerfile with multi-stage builds, BuildKit features, and security hardening
allowed-tools: Read(package.json), Read(pyproject.toml), Read(pom.xml), Read(build.gradle), Read(go.mod), Bash(docker build --dry-run:*)
---

# Generate Optimized Dockerfile

Generate a production-ready Dockerfile following modern best practices with multi-stage builds, BuildKit cache mounts, and security hardening.

## Generation Steps

### 1. Project Detection
Detect the project type and framework by examining project files:
- **Node.js**: Look for package.json, check for Next.js/React/Express in dependencies
- **Python**: Look for pyproject.toml (uv) or requirements.txt, check for FastAPI/Django/Flask
- **Java**: Look for pom.xml (Maven) or build.gradle (Gradle), check for Spring Boot
- **Go**: Look for go.mod, check for main package

Query Context7 for latest base image recommendations for the detected framework.

### 2. Base Image Selection
Choose optimal base image based on project type:
- **Node.js**: Use node:20-slim for Debian compatibility with native dependencies
- **Python**: Use python:3.12-slim for balance of size and compatibility
- **Java**: Use eclipse-temurin:21-jdk-jammy for builder, eclipse-temurin:21-jre-jammy for runtime
- **Go**: Use golang:1.21-alpine for builder, scratch or alpine for runtime

Reference: `.claude/context/orchestration/docker-standards.md` for complete patterns

### 3. Multi-Stage Build Structure
Generate Dockerfile with distinct stages:
- **Base stage**: Set common environment variables and working directory
- **Dependencies stage**: Install build tools and dependencies with cache mounts
- **Builder stage**: Copy source code and build application
- **Production stage**: Copy only runtime artifacts, configure non-root user

### 4. BuildKit Cache Mounts
Add cache mount directives for package managers:
- **Python**: --mount=type=cache,target=/root/.cache/pip for pip
- **Python (uv)**: --mount=type=cache,target=/root/.cache/uv for uv
- **Node.js**: --mount=type=cache,target=/root/.npm for npm
- **Java (Maven)**: --mount=type=cache,target=/root/.m2 for Maven
- **Java (Gradle)**: --mount=type=cache,target=/root/.gradle for Gradle
- **Go**: --mount=type=cache,target=/root/.cache/go-build and --mount=type=cache,target=/go/pkg/mod

### 5. Layer Optimization
Order Dockerfile instructions from least to most frequently changing:
1. Base image and system dependencies
2. Package manager configuration files (package.json, pyproject.toml, etc.)
3. Dependency installation with cache mounts
4. Source code copy
5. Build commands
6. Runtime configuration

### 6. Security Configuration
Implement security best practices:
- Create non-root user with groupadd and useradd
- Set proper file ownership with --chown flag
- Use USER directive to switch to non-root user
- Never expose secrets in ENV or ARG (use BuildKit secrets for build-time secrets)
- Minimize attack surface with slim/distroless images

### 7. Health Check
Add HEALTHCHECK instruction appropriate for the application type:
- **HTTP services**: curl or wget health endpoint
- **TCP services**: nc (netcat) port check
- **Custom**: Python/Node script for application-specific checks

Configure reasonable intervals (30s), timeouts (3s), and retries (3)

### 8. Environment Variables and Ports
Set appropriate environment variables and expose ports:
- **Python**: PYTHONUNBUFFERED=1, PYTHONDONTWRITEBYTECODE=1
- **Node.js**: NODE_ENV=production, set PORT
- **Java**: JVM options for container awareness
- **Go**: CGO_ENABLED=0 for static binaries

### 9. .dockerignore Generation
If .dockerignore doesn't exist, generate one to exclude:
- Version control (.git, .gitignore)
- Dependencies (node_modules, .venv, target/)
- IDE files (.vscode, .idea)
- Testing (coverage, test-results)
- Documentation (README.md, docs/)
- Build artifacts (dist/, build/)
- Environment files (.env, .env.local)

### 10. Validation
After generating Dockerfile:
- Run `docker build --dry-run .` to validate syntax
- Verify multi-stage build is present (multiple FROM statements)
- Confirm non-root USER directive exists
- Check that cache mounts are used for package installation
- Ensure HEALTHCHECK instruction is included

## Quick Reference Patterns

**Query Context7** for latest documentation:
- Docker official images for base image versions
- Framework-specific Dockerfile recommendations
- Security best practices and CVE information

**Reference context files** for complete code examples:
- All Dockerfile templates: `.claude/context/orchestration/docker-standards.md`
- Security patterns: `.claude/context/orchestration/docker-security-standards.md`

## Post-Generation Actions

After generating Dockerfile:
1. Run `/docker:validate-security` to verify security compliance
2. Run `/validate-docker` for general best practices check
3. Test build with `docker build -t test-image .`
4. Verify image size is optimized (multi-stage should reduce by 50-85%)
5. Check that final image runs as non-root user (docker run --rm test-image whoami)
