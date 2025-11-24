---
description: Validates git commit messages against Conventional Commits v1.0.0 specification
allowed-tools:
  - Bash
  - Read
  - Grep
---

# Validate Commit Message

Validate commit message format, type, length, and breaking change documentation against Conventional Commits v1.0.0.

## Validation Steps

### 1. Format Validation

Check structure matches type(scope): subject:
- Type exists and is lowercase
- Scope is optional, lowercase, in parentheses
- Colon and space after type/scope
- Subject exists after colon-space
- No period at end of subject

### 2. Type Validation

Verify type is allowed:
- feat (new feature)
- fix (bug fix)
- docs (documentation)
- style (formatting)
- refactor (code restructuring)
- perf (performance)
- test (testing)
- build (build system)
- ci (CI configuration)
- chore (maintenance)
- revert (revert commit)

### 3. Subject Length

Validate subject line:
- Maximum 50 characters
- Count from after "type(scope): " to end
- Exclude trailing period if present
- Report violation if exceeds limit

### 4. Body Wrapping

Check body format if present:
- Blank line after subject
- Lines wrapped at 72 characters
- Report lines exceeding limit
- Verify imperative mood usage

### 5. Breaking Change Validation

Check breaking change indicators:
- Detect ! after type/scope
- Verify BREAKING CHANGE: footer exists
- Validate footer has description
- Confirm both if ! present

### 6. Footer Format

Validate footer structure:
- token: value format
- token #value for issue refs
- Multi-line footer support
- No blank lines within footer

## Output Format

Return validation results with severity:

CRITICAL - Invalid format, wrong type, missing breaking change description
WARNING - Subject too long, body not wrapped, missing scope on multi-module change
INFO - Style suggestions, best practice recommendations
PASS - All validations successful

## Context Reference

For commit format examples and correction patterns:
- Reference: .claude/context/workshop/commit-message-standards.mdc
