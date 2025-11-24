---
description: Validates Playwright test structure, semantic locators, and testing standards compliance
allowed-tools: Bash(npx playwright:*), Bash(npm test:*), Read, Grep, Glob
---

# Playwright Testing Validation

Perform a comprehensive validation of Playwright test structure, locator patterns, and standards compliance.

## Validation Steps

### 1. Project Type Verification
First, verify Playwright is installed by checking @package.json for @playwright/test dependency. If not found, report that Playwright is not configured.

### 2. Playwright Configuration
Check for playwright.config.ts or playwright.config.js and validate:
- Valid baseURL configured (e.g., http://localhost:3000)
- Trace configuration (on-first-retry or retain-on-failure)
- Screenshot configuration (only-on-failure)
- Video configuration (retain-on-failure)
- Timeout settings (timeout, expect timeout)
- Worker configuration for parallelization
- Missing config: Report if no playwright.config file exists

### 3. Test File Structure
Validate test organization:
- Structure: Look for tests/ directory or e2e/ directory
- Test files: Verify .spec.ts or .test.ts files exist
- Page objects: Check for tests/pages/ or similar directory
- Helpers: Check for tests/helpers/ or tests/fixtures/
- Missing structure: Report if no test files found

### 4. Semantic Locator Analysis
Scan test files for locator patterns:
- Fragile selectors: Search for CSS class selectors (.class-name)
- Fragile selectors: Search for complex CSS selectors (div > span)
- Data attributes: Search for data-testid usage (acceptable fallback)
- Semantic locators: Count getByRole, getByLabel, getByText usage
- Recommendation: Suggest replacing fragile selectors with semantic ones

### 5. Timeout Anti-Pattern Detection
Check for arbitrary timeouts:
- Search for page.waitForTimeout usage
- Count occurrences
- Report violations
- Recommend: Use auto-waiting or specific state waits

### 6. Page Object Model Validation
Check for Page Object pattern:
- Verify tests/pages/ directory exists
- Count page object classes
- Check for proper TypeScript class structure
- Recommendation: Suggest POM for complex workflows

### 7. Authentication State Management
Check authentication setup:
- Look for global-setup.ts or auth.setup.ts
- Check for .auth/ directory in .gitignore
- Verify setup project configuration in playwright.config
- Recommendation: Suggest setup projects for authentication

### 8. Test Data Strategy
Analyze test data patterns:
- Search for unique data generation (Date.now(), faker)
- Check for hardcoded test data
- Recommendation: Suggest unique data generation

### 9. User Behavior Testing
Validate test naming and structure:
- Check test names start with "user can" or similar patterns
- Search for implementation detail testing (API response checks)
- Verify tests focus on UI interaction
- Recommendation: Suggest user-focused test names

### 10. Debugging Tools Setup
Check for debugging configuration:
- Trace viewer configured
- UI mode available (npx playwright test --ui)
- Debug mode available (npx playwright test --debug)
- Codegen available (npx playwright codegen)

### 11. Test Execution Validation
Run Playwright tests:
- Execute npx playwright test --list to verify tests are discoverable
- Optional: Run npx playwright test if quick validation needed
- Success: Report test execution status
- Failure: Report failures and suggest debugging commands

## Validation Output

Generate comprehensive validation report with:

### Semantic Locator Quality Score
- Count of semantic locators (getByRole, getByLabel, getByText)
- Count of fragile selectors (CSS classes, complex selectors)
- Semantic locator percentage
- Quality score

### Issues by Severity

**CRITICAL (Must Fix)**:
- Arbitrary timeouts found (page.waitForTimeout)
- No semantic locators found
- Tests testing implementation details
- No Playwright configuration

**WARNING (Should Fix)**:
- High percentage of CSS class selectors
- No Page Object Models
- No authentication setup project
- Hardcoded test data
- Tests not using unique data

**INFO (Consider)**:
- Add trace viewer configuration
- Implement parallel execution
- Add accessibility testing
- Configure visual regression testing

### Recommendations Format
For each issue provide:
- File and line number location
- Violation type
- Current pattern (code snippet reference)
- Recommended pattern
- Reference to standards documentation

## Debugging Commands Reference

Include helpful commands in output:
- npx playwright test --grep "test name" --debug
- npx playwright test --ui
- npx playwright codegen localhost:3000
- npx playwright test --trace on
- npx playwright show-report

## Supporting Documentation

For detailed standards and patterns:
- **Playwright Testing**: `.claude/context/nextjs_frontend/playwright-testing-standards.mdc`

## Related Commands
- Run validation after writing new tests
- Re-validate after refactoring locators
- Check before committing test changes
