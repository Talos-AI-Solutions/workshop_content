# Workshop - Claude Code Sample Project

This repository contains sample agents, commands, and design artifacts for learning Claude Code over the first day of the workshop.

## Claude Code Sub Agents

The `.claude/agents/` directory contains specialized sub agents that demonstrate best practices for different technology stacks:

- **python-engineer** - Modern Python 3.11+ development with type safety and async patterns
- **nextjs-developer** - Next.js App Router architecture with performance optimization
- **python-database-administrator** - PostgreSQL and SQLAlchemy 2.0+ database design
- **java-database-administrator** - PostgreSQL, Hibernate/JPA for Java applications
- **java-engineer** - Modern Java 17-21+ development with type safety
- **java-api-engineer** - Spring Boot 3.x REST API development
- **java-testing-agent** - Comprehensive Java testing with coverage analysis
- **docker-architect-agent** - Container orchestration and Docker best practices
- **git-workflow-agent** - Trunk-based development with Conventional Commits
- **playwright-e2e-tester** - E2E testing for Next.js applications
- **commit-message-validator** - Git commit message validation (workshop example)

Each agent is configured with specialized tools, validation commands, and standards to demonstrate how to build production-ready code with Claude Code.

## Slash Commands

The `.claude/commands/` directory contains reusable validation and automation commands organized by technology stack (python, java, nextjs_frontend, database, orchestration, review, workshop).

## Design Content

The `design/` directory contains sample artifacts for a simple CRUD tournament tracking application:
- Database schema
- OpenAPI specification
- Design system

These were generated using prompts found in `/prompts/004_requirements_extract` and serve as examples for workshop exercises.

## Workshop Exercises

This repository provides practical examples and sample prompts for hands-on exercises throughout the first day of the workshop. Use the configured agents and commands to explore Claude Code's capabilities.
