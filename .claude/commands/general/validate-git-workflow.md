---
description: Validates git workflow against trunk-based development and Conventional Commits v1.0.0
allowed-tools:
  - Bash
  - Read
  - Grep
  - mcp__github__*
---

# Validate Git Workflow

Validates git repository state, commit messages, PR size, and branch hygiene against trunk-based development standards.

## Validation Steps

### 1. Repository State

Check git status for uncommitted changes, untracked files, staging area state, and merge conflicts.

### 2. Commit Message Format

Validate Conventional Commits v1.0.0: type validity, scope format, subject format (imperative, lowercase, no period, 100 char max), breaking change indicators (! or BREAKING CHANGE footer), body wrapping (72 char), footer format.

### 3. PR Size

Count total lines changed, validate 400 line limit, calculate files changed, assess complexity, flag oversized as CRITICAL.

### 4. Branch Naming

Verify type/description pattern, validate type prefix (feat, fix, chore, docs, refactor, test), ensure kebab-case, check 50 char max, confirm no special characters except dash.

### 5. Branch Age

Calculate days since creation, flag 3+ days as WARNING, 7+ days as CRITICAL, check commit frequency, verify sync with main.

### 6. Linear History

Check for merge commits, verify rebase on latest main, validate no octopus merges, confirm fast-forward possible, check for duplicates.

### 7. Breaking Change Documentation

Detect ! indicator, verify BREAKING CHANGE footer when ! present, check description clarity, validate migration guidance, confirm version bump implications.

### 8. CI/CD Status

Query GitHub status checks via MCP, verify required checks passing, check for failed workflows, validate test coverage, confirm build success.

### 9. Review Status

Check approval count, verify required reviewers approved, validate no requested changes, check staleness (24+ hours), confirm review matches latest commits.

### 10. Conflict Resolution

Detect merge conflicts, verify rebase completed, check for conflict markers, validate resolution quality, confirm no temporary conflict files.

## Output Format

Return validation results with severity levels:

**CRITICAL** - Blocks merge, must be resolved
**WARNING** - Should be addressed, may block
**INFO** - Informational, does not block
**PASS** - Validation successful

Include for each validation:
- Check name
- Status (PASS/INFO/WARNING/CRITICAL)
- Details of failure if applicable
- Remediation guidance reference

## Severity Guidelines

**CRITICAL failures**:
- Invalid commit message format
- PR exceeds 400 lines
- Merge conflicts present
- Required CI checks failing
- Branch older than 7 days

**WARNING conditions**:
- Branch older than 3 days
- Missing scope in multi-module change
- Review older than 24 hours
- Body text not wrapped at 72 characters
- No description in breaking change footer

**INFO notifications**:
- Branch ahead of main by many commits
- Large number of files changed
- Complex change patterns detected
- Review suggestions available
- Style guide deviations

## Context Reference

For commit format examples, remediation patterns, and validation automation scripts:
- Reference: .claude/context/general/git-workflow-standards.md
  - Commit message examples and anti-patterns
  - Bash command examples for git operations
  - GitHub MCP integration patterns
  - Validation automation scripts (complete validation script, individual functions, git hooks, CI/CD integration)
- Agent: .claude/agents/general/git-workflow-agent.md

## Exit Behavior

Report summary with:
- Total validations run
- CRITICAL count
- WARNING count
- Overall status (PASS/FAIL)
- Next recommended action
