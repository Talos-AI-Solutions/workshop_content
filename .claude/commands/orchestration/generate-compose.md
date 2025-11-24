---
description: Generate docker-compose.yml for multi-service orchestration with network isolation, health checks, and volume management
allowed-tools: Read(package.json), Read(pyproject.toml), Read(Dockerfile), Bash(docker compose config:*)
---

# Generate Docker Compose Configuration

Generate a production-ready docker-compose.yml file with multi-service orchestration, network isolation, health checks, and volume management following modern best practices.

## Generation Steps

### 1. Service Discovery
Analyze project structure to identify required services:
- **Application service**: Detected from Dockerfile or project files
- **Database**: Check for PostgreSQL, MySQL, MongoDB connections in config
- **Cache**: Check for Redis usage in application code
- **Message queue**: Check for RabbitMQ, Kafka references
- **Background workers**: Detect worker processes or job queues

Query Context7 for latest official images and recommended versions.

### 2. Network Topology Design
Create network isolation strategy with multiple networks:
- **frontend network**: Public-facing services (nginx, frontend apps)
- **backend network**: Internal API services
- **database network**: Database access (most restricted, internal only)

Configure network connectivity:
- Frontend services connect to frontend and backend networks
- Backend services connect to backend and database networks
- Database services only on database network (internal: true)

### 3. Service Configuration
For each service, configure:
- **build** context and Dockerfile location (for custom images)
- **image** name and tag (for official images)
- **ports** mapping (host:container), bind to 127.0.0.1 for security
- **environment** variables from .env file
- **depends_on** with health check conditions
- **networks** assignment based on topology
- **volumes** for data persistence or development
- **healthcheck** with appropriate test command
- **deploy.resources** with CPU and memory limits

### 4. Health Check Configuration
Add health checks for each service with appropriate commands:
- **HTTP services**: CMD curl -f http://localhost:port/health or CMD wget --spider
- **PostgreSQL**: CMD-SHELL pg_isready -U username
- **MySQL**: CMD mysqladmin ping -h localhost
- **Redis**: CMD redis-cli ping
- **MongoDB**: CMD mongo --eval "db.adminCommand('ping')"

Configure timing:
- interval: 10-30s depending on service criticality
- timeout: 3-5s
- retries: 3-5 attempts before marking unhealthy
- start_period: 30-60s for slow-starting services

### 5. Dependency Management
Use depends_on with conditions for proper startup ordering:
- condition: service_started (basic dependency)
- condition: service_healthy (wait for health check)
- condition: service_completed_successfully (for init containers)

Example: API depends on database being healthy, frontend depends on API being healthy

### 6. Volume Strategy
Configure volumes based on service type:
- **Named volumes** for data persistence (postgres-data, redis-data)
- **Bind mounts** for development (./src:/app/src for hot reloading)
- **Anonymous volumes** to prevent overwriting (node_modules)
- **Tmpfs mounts** for temporary data

Set volume drivers and options:
- driver: local for persistent storage
- driver_opts for custom mount points
- Mark volumes as external if pre-existing

### 7. Environment Variable Management
Configure environment variables using .env file pattern:
- Reference with ${VAR_NAME} syntax
- Provide defaults with ${VAR_NAME:-default}
- Use env_file directive to load from .env
- Document required variables in .env.example

For production, integrate cloud secrets:
- AWS Secrets Manager references
- Azure Key Vault integration
- GCP Secret Manager

### 8. Resource Constraints
Add resource limits to prevent resource exhaustion:
- deploy.resources.limits for maximum allocation
- deploy.resources.reservations for minimum guarantee
- Typical limits: cpus: '1-2', memory: 512M-2G
- Critical services get higher limits

### 9. Validation
After generating docker-compose.yml:
- Run `docker compose config` to validate syntax
- Verify all services have health checks
- Confirm network isolation is properly configured
- Check that named volumes are defined for all persistent data
- Ensure depends_on conditions are appropriate
- Validate env_file references exist

## Network Isolation Best Practices

**Three-tier architecture:**
- Public tier (frontend network): External access allowed
- Application tier (backend network): Internal only
- Data tier (database network): Most restricted, internal: true

## Secrets Management

For development:
- Use .env files with environment variables
- Add .env to .gitignore
- Provide .env.example template

For production:
- Reference cloud provider secrets services
- Never commit secrets to version control
- Use Docker secrets with external: true for Swarm
- Document cloud secrets setup in README

## Quick Reference Patterns

**Query Context7** for latest versions:
- Official Docker images (postgres, redis, nginx, etc.)
- Framework-specific compose patterns
- Cloud secrets integration documentation

**Reference context files** for complete code examples:
- Complete compose examples: `.claude/context/orchestration/docker-standards.md`
- Security patterns: `.claude/context/orchestration/docker-security-standards.md`

## Post-Generation Actions

After generating docker-compose.yml:
1. Create .env.example with all required variables
2. Run `docker compose config` to validate syntax
3. Run `/docker:validate-security` for security compliance
4. Run `/validate-docker` for general configuration check
5. Test startup with `docker compose up` and verify health checks pass
6. Document service URLs and access patterns in README
