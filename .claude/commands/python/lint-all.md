---
description: Runs comprehensive linting, formatting, and code quality checks
allowed-tools: Bash(npx tsc:*), Bash(npm run lint:*), Bash(npx eslint:*), Bash(npx prettier:*), Bash(uv run:*), Bash(npm run build:*), Bash(docker build --dry-run:*), Bash(npm test:*), Bash(find:*)
---

# Comprehensive Code Quality Analysis

Perform thorough linting, formatting, and code quality checks across multiple languages and frameworks.

## Quality Check Steps

### 1. Project Validation
First, verify this is a valid project by checking for @package.json. If not found, report this should be run from project root.

### 2. Available Tools Detection
Scan for available quality tools and report findings:

#### JavaScript/TypeScript Tools
- ✅ **ESLint**: Check for eslint in @package.json
- ✅ **Prettier**: Check for prettier in @package.json  
- ✅ **TypeScript**: Check for typescript in @package.json AND tsconfig.json
- ⚠️ **Missing tools**: Recommend missing tools for code quality

#### Python Tools (if Python project detected)
If pyproject.toml or requirements.txt exists:
- ✅ **uv**: Check if uv command is available
- ✅ **Python linting**: Check for black, ruff, mypy in pyproject.toml
- Report Python project detection and available tools

### 3. TypeScript Type Checking
If TypeScript is available:
- Run `npx tsc --noEmit` for type checking
- ✅ **Type check passed**: Report successful type checking
- ❌ **Type errors found**: Report failures and exit

### 4. ESLint Analysis
If ESLint is available:
- Try `npm run lint` first, fallback to `npx eslint . --ext .ts,.tsx,.js,.jsx`
- ✅ **ESLint passed**: Report no linting issues
- ❌ **ESLint issues**: Report issues and suggest `npm run lint -- --fix`

### 5. Prettier Formatting Check
If Prettier is available:
- Run `npx prettier --check .` to verify formatting
- ✅ **Formatting correct**: Report proper formatting
- ⚠️ **Formatting issues**: Offer to auto-fix with `npx prettier --write .`
- Present auto-fix option and apply if user confirms

### 6. Python Code Quality (if applicable)
For Python projects, check each tool:

#### Black Formatting
- Run `uv run black --check .` or `black --check .`
- ✅ **Black formatting**: Report correct formatting
- ⚠️ **Black issues**: Suggest auto-fix commands

#### Ruff Linting
- Run `uv run ruff check .` or `ruff check .`
- ✅ **Ruff passed**: Report no linting issues
- ❌ **Ruff issues**: Suggest auto-fix with `--fix` flag

#### MyPy Type Checking
- Run `uv run mypy .` or `mypy .`
- ✅ **MyPy passed**: Report successful type checking
- ❌ **Type issues**: Report MyPy findings

#### Bandit Security Scan
- Run `uv run bandit -r .` or `bandit -r .`
- ✅ **Security scan**: Report no security issues
- ⚠️ **Security findings**: Report potential security issues

### 7. Framework-Specific Checks

#### Next.js Projects
If Next.js is detected in @package.json:
- ⚛️ Run Next.js specific checks
- Test build with `npm run build`
- ✅ **Build successful**: Report successful build
- ❌ **Build failed**: Report failure and suggest detailed error checking

#### Docker Validation
If Dockerfile exists:
- Run `docker build --dry-run .` to validate syntax
- ✅ **Docker syntax**: Report valid Dockerfile
- ❌ **Docker issues**: Report syntax problems
- ⚠️ **Docker unavailable**: Note if Docker not available

### 8. Project Statistics and Summary
Generate comprehensive project overview:
- Count file types:
  - TypeScript/TSX files (*.ts, *.tsx)
  - JavaScript/JSX files (*.js, *.jsx)  
  - Python files (*.py)
  - Test files (*.test.*, *.spec.*)
- Report file counts for project overview

### 9. Test Execution
If test files are found:
- ✅ **Tests present**: Confirm tests exist in project
- Run `npm test` if test script exists in package.json
- ✅ **Tests passed**: Report all tests successful
- ⚠️ **Test failures**: Report some tests failed
- ⚠️ **No tests**: Recommend adding tests if none found

## Quality Standards Enforcement
Apply consistent quality standards:
- **Zero tolerance**: Type errors, linting errors must be resolved
- **High standards**: Formatting should be consistent
- **Security focus**: Run security scans when available
- **Build verification**: Projects must build successfully

## Reporting Format
Use clear status indicators:
- ✅ for passed checks and good quality
- ⚠️ for recommendations and minor issues
- ❌ for errors requiring immediate attention

## Quick Fix Commands
Provide actionable commands for common issues:
- **Format code**: `npx prettier --write .`
- **Fix ESLint**: `npm run lint -- --fix`
- **Fix Python formatting**: `uv run black .`
- **Fix Python linting**: `uv run ruff check --fix .`
- **Type check**: `npx tsc --noEmit`

## Configuration References
Point users to relevant configuration files:
- ESLint config in `.eslintrc.json`
- Prettier config in `.prettierrc`
- TypeScript config in `tsconfig.json`
- Python config in `pyproject.toml`

## Multi-Language Support
Handle projects with multiple languages:
- Detect and validate each language's tools
- Apply language-specific best practices
- Provide unified quality reporting
- Support both JavaScript/TypeScript and Python ecosystems