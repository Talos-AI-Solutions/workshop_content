---
description: Validates Docker security practices including root users, secrets in layers, base image security, and cloud secrets integration
allowed-tools: Read(Dockerfile), Read(docker-compose.yml), Read(.env), Bash(docker history:*)
---

# Docker Security Validation

Perform comprehensive security validation of Docker configurations against modern security best practices and cloud-native patterns.

## Validation Steps

### 1. Root User Detection
Check if containers run as root (critical security violation):
- ✅ **Non-root user**: Look for `USER` directive with non-root user in Dockerfile
- ❌ **Root user**: Alert if no USER directive or USER root is specified
- ⚠️ **UID check**: Verify UID is not 0 (root)
- Check all stages in multi-stage builds for USER directives

### 2. Secrets in Docker Layers
Scan for secrets exposed in image layers:
- ❌ **ENV secrets**: Check for API keys, passwords, tokens in ENV directives
- ❌ **ARG secrets**: Check for secrets passed as build arguments (persist in image metadata)
- ❌ **Hardcoded secrets**: Scan for patterns like password=, api_key=, secret=
- ✅ **BuildKit secrets**: Verify --mount=type=secret is used for build-time secrets
- ✅ **Runtime secrets**: Check docker-compose uses secrets: directive (not environment:)

### 3. Base Image Security
Validate base image choices and sources:
- ✅ **Official images**: Verify images are from official Docker Hub or trusted registries
- ✅ **Specific tags**: Check for version tags (not :latest)
- ⚠️ **Slim/Distroless**: Recommend minimal base images to reduce attack surface
- ❌ **Outdated versions**: Alert if base image version is significantly outdated
- Query Context7 for latest security advisories and recommended versions

### 4. Cloud Secrets Integration
For production deployments, verify cloud secrets management:
- **AWS**: Check for references to AWS Secrets Manager or Parameter Store
- **Azure**: Check for Azure Key Vault integration
- **GCP**: Check for GCP Secret Manager usage
- ⚠️ **Development .env**: Acceptable for dev, but warn if no cloud secrets for production
- Verify .env files are in .gitignore and .dockerignore

### 5. Environment Variable Security
Check environment variable handling:
- ❌ **Secrets in .env committed**: Check if .env is in git history
- ✅ **.env.example**: Verify template exists without actual secrets
- ⚠️ **Database URLs**: Alert if full connection strings in docker-compose (use secrets instead)
- Check for sensitive variable names (PASSWORD, SECRET, KEY, TOKEN, etc.)

### 6. Network Isolation
Validate network security configuration:
- ✅ **Internal networks**: Check for networks with internal: true for databases
- ⚠️ **Network segmentation**: Verify multi-tier architecture (frontend, backend, database)
- ❌ **Host network mode**: Alert if network_mode: host is used (breaks isolation)
- ✅ **Port binding**: Check ports bind to 127.0.0.1 (not 0.0.0.0) for internal services

### 7. Resource Limits
Check for resource constraints to prevent DoS:
- ⚠️ **Missing limits**: Alert if no deploy.resources.limits defined
- ✅ **Memory limits**: Verify memory limits are set (prevents OOM issues)
- ✅ **CPU limits**: Check CPU limits are reasonable
- ⚠️ **Unlimited resources**: Warn about security implications

### 8. File Permissions and Ownership
Validate file ownership in Dockerfile:
- ✅ **COPY --chown**: Check for proper ownership in COPY directives
- ⚠️ **chmod in layers**: Alert if chmod 777 or overly permissive permissions
- Verify files are owned by non-root user in final stage

### 9. Exposed Ports and Services
Review port exposure configuration:
- ⚠️ **Unnecessary ports**: Check for exposed ports not used by application
- ❌ **Debug ports**: Alert if debug ports (9229, 5678) exposed in production
- ✅ **Minimal exposure**: Verify only required ports are published
- Check for expose vs ports (expose is internal only)

### 10. Image Layer Analysis
If Docker is available, analyze image layers for security:
- Run `docker history <image>` to inspect layers
- Check layer sizes for suspicious large additions
- Verify secrets were not added in intermediate layers
- Alert if sensitive files visible in history

### 11. Security Headers and Configuration
For web services, check security configurations:
- Health check endpoints should not expose sensitive information
- Verify application runs with minimal privileges
- Check for security.txt or documented security practices

### 12. Dependency Security
Validate dependency management security:
- ✅ **Lock files**: Check for package-lock.json, uv.lock, go.sum
- ⚠️ **npm audit**: Suggest running security audits
- Check for npm ci (reproducible builds) vs npm install

## Reporting Format

Use emoji indicators for clear security status:
- ✅ for validated security practices
- ⚠️ for security recommendations
- ❌ for critical security violations

## Critical Violations (Must Fix)

Report these as blocking security issues:
1. Container running as root in production
2. Secrets in ENV or ARG directives
3. Hardcoded passwords or API keys
4. .env file committed to git
5. network_mode: host in production
6. Debug ports exposed in production builds

## Recommendations

Provide actionable security improvements:
- Use BuildKit --mount=type=secret for build-time secrets
- Integrate cloud secrets manager (AWS Secrets Manager, Azure Key Vault, GCP Secret Manager)
- Create non-root user with specific UID/GID
- Use internal: true for database networks
- Set resource limits on all services
- Use specific version tags for base images
- Add .env to .gitignore and .dockerignore
- Bind internal ports to 127.0.0.1 only

## Reference Documentation

For comprehensive security patterns and code examples:
- Security patterns: `.claude/context/orchestration/docker-security-standards.md`
- General Docker: `.claude/context/orchestration/docker-standards.md`
- Query Context7 for latest security advisories and cloud secrets integration

## Post-Validation Actions

After security validation:
1. Address all critical violations (❌) immediately
2. Implement cloud secrets for production deployments
3. Review and implement security recommendations (⚠️)
4. Run vulnerability scanning with tools like Snyk or Trivy
5. Document security architecture and secrets management strategy
6. Schedule regular security reviews and dependency updates
