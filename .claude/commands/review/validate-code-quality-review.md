---
description: Validates that comprehensive code quality review was conducted following 6-category methodology with automated checks, severity classification, and actionable feedback
type: project
---

# Validate Code Quality Review

Validates that a comprehensive code quality review was conducted according to enterprise-grade standards defined in `.claude/agents/code-quality-reviewer.md`.

## Validation Checklist

### 1. Scope Identification ✓
- [ ] Changed files were identified (via git diff or user specification)
- [ ] Review scope is clearly documented
- [ ] Review focused on relevant changes, not entire codebase

### 2. Automated Checks Executed ✓
- [ ] Linting was run and results documented
- [ ] Type checking was run (if applicable to language)
- [ ] Tests were run and coverage reported
- [ ] Results clearly show PASS/FAIL status

**Verification**: Check conversation history for:
- Command execution: `ruff check`, `mypy`, `eslint`, `npm run lint`, etc.
- Output showing results
- Any failures documented in report

### 3. All 6 Categories Reviewed ✓

Each category must be explicitly addressed:

- [ ] **Code Quality & Standards**: Linting, type safety, naming, DRY, single responsibility
- [ ] **Security**: No secrets, input validation, output encoding, secure dependencies, auth/authz
- [ ] **Error Handling & Reliability**: Exception handling, error messages, edge cases, null safety, resource cleanup
- [ ] **Testing**: Coverage, critical paths, meaningful tests, test names, test quality
- [ ] **Maintainability**: Documentation, constants, dead code removal, style consistency, clean code
- [ ] **Performance & Efficiency**: Algorithms, database optimization, data structures, loop efficiency, resource usage

**Verification**: Review report structure to ensure all 6 categories appear with findings or explicit "no issues found" statements.

### 4. Issues Properly Documented ✓

Each issue must include:

- [ ] **Category**: Which of the 6 categories it belongs to
- [ ] **Severity**: Critical, Important, Recommended, or Nitpick
- [ ] **Location**: File path and line number (e.g., `src/auth.py:123`)
- [ ] **Description**: Clear problem statement
- [ ] **Impact**: What could go wrong
- [ ] **Recommendation**: Specific, actionable fix

**Example of properly documented issue:**
```markdown
### Critical Issue: Hardcoded API Key
**Category**: Security
**Location**: `src/config.py:15`
**Problem**: API key hardcoded in source code
**Impact**: Key exposed in version control, security breach risk
**Recommendation**: Move to environment variable or secrets manager
```

### 5. Severity Classification Applied ✓

Issues must be prioritized:

- [ ] **Critical issues** identified (security, data loss, breaking changes)
- [ ] **Important issues** identified (missing tests, error handling, type safety)
- [ ] **Recommended improvements** identified (duplication, magic numbers, documentation)
- [ ] **Nitpicks** separated (optional style preferences)
- [ ] Severity counts provided in summary

### 6. Report Structure Complete ✓

Report must contain:

- [ ] **Summary Section**: Files reviewed, issue counts by severity
- [ ] **Automated Checks Section**: Linting, type checking, test results
- [ ] **Critical Issues Section**: High-priority blocking issues
- [ ] **Important Issues Section**: Should-fix-before-merge issues
- [ ] **Recommended Improvements Section**: Nice-to-have improvements
- [ ] **Positive Highlights Section**: Good patterns acknowledged (optional but encouraged)

### 7. Actionable Feedback Provided ✓

Feedback must be:

- [ ] **Specific**: Not vague (❌ "improve code quality" ✅ "replace magic number 3600 with SECONDS_PER_HOUR constant at line 45")
- [ ] **Constructive**: Explains why, not just what
- [ ] **Actionable**: Developer knows exactly what to do
- [ ] **Referenced**: Links to standards/examples in context file where helpful

### 8. Context Standards Referenced ✓

- [ ] Review references `.claude/context/review/code-quality-standards.md` for examples
- [ ] Language-specific standards applied (Python/JS/Go)
- [ ] Security patterns from OWASP considered
- [ ] Testing patterns followed

### 9. Integration with Related Agents ✓

- [ ] For backend code: Considered referencing @backend-developer agent
- [ ] For Next.js code: Considered referencing @nextjs-developer agent
- [ ] For UI changes: Considered referencing @design-review-agent

## Validation Execution

When this command runs, you should:

1. **Review Conversation History**
   - Look for evidence of automated check execution
   - Verify all 6 categories were addressed
   - Check issue documentation quality

2. **Analyze Report Structure**
   - Confirm all required sections present
   - Verify severity classification used
   - Check for file:line references

3. **Assess Completeness**
   - Calculate coverage: How many checks from each category were performed?
   - Identify gaps: Were any categories skipped?
   - Check depth: Was review superficial or thorough?

4. **Generate Validation Report**

```markdown
# Code Quality Review Validation Report

## Overall Status: ✅ PASS / ⚠️ PARTIAL / ❌ FAIL

## Validation Results

### Scope Identification: ✅/❌
- Files reviewed: [list]
- Scope clearly defined: Yes/No

### Automated Checks: ✅/❌
- Linting: Executed/Not executed
- Type checking: Executed/Not executed
- Tests: Executed/Not executed

### Category Coverage: ✅/❌
- Code Quality & Standards: Reviewed/Skipped
- Security: Reviewed/Skipped
- Error Handling: Reviewed/Skipped
- Testing: Reviewed/Skipped
- Maintainability: Reviewed/Skipped
- Performance: Reviewed/Skipped

### Documentation Quality: ✅/❌
- File:line references: Present/Missing
- Severity levels: Applied/Not applied
- Impact statements: Present/Missing
- Recommendations: Actionable/Vague

### Report Completeness: ✅/❌
- Summary: Present/Missing
- Automated checks: Present/Missing
- Issues by severity: Present/Missing
- Positive highlights: Present/Missing

## Issues Found in Review Process

### Critical Gaps
[List any major issues with the review itself]

### Recommendations
[How to improve the review]

## Next Steps
[What should be done to complete or improve the review]
```

## Pass Criteria

Review is considered **COMPLETE** when:
- ✅ All automated checks executed
- ✅ All 6 categories reviewed
- ✅ Issues have file:line references
- ✅ Severity levels assigned
- ✅ Report is well-structured
- ✅ Feedback is actionable

Review is considered **PARTIAL** when:
- ⚠️ Most categories reviewed but 1-2 skipped
- ⚠️ Some automated checks missing
- ⚠️ Most issues documented well but some missing details

Review is considered **INCOMPLETE** when:
- ❌ Multiple categories not reviewed
- ❌ No automated checks run
- ❌ Issues lack specificity
- ❌ No severity classification

## Common Validation Failures

### Failure: Missing Automated Checks
**Problem**: Review was manual-only without running linters/tests
**Impact**: Missed basic issues that tools would catch
**Fix**: Run linting, type checking, and tests before manual review

### Failure: Vague Issue Descriptions
**Problem**: "Code quality issues found" without specifics
**Impact**: Developer doesn't know what to fix
**Fix**: Provide file:line references and specific recommendations

### Failure: No Severity Classification
**Problem**: All issues treated equally
**Impact**: Developer doesn't know what to prioritize
**Fix**: Apply Critical/Important/Recommended/Nitpick labels

### Failure: Skipped Categories
**Problem**: Only 2-3 categories reviewed instead of all 6
**Impact**: Incomplete review, potential issues missed
**Fix**: Systematically review all 6 categories

### Failure: Missing Context References
**Problem**: Reinventing standards instead of referencing established patterns
**Impact**: Inconsistent guidance, longer review time
**Fix**: Reference `.claude/context/review/code-quality-standards.md` examples

## Integration Points

### Agent Guidance
This validation command verifies adherence to `.claude/agents/code-quality-reviewer.md` workflow.

### Context Standards
Standards are defined in `.claude/context/review/code-quality-standards.md` with concrete examples.

### Related Validations
- `/validate-api`: For API-specific standards (backend projects)
- `/validate-nextjs`: For Next.js-specific standards
- `/validate-design-review`: For UI/UX standards

## Success Metrics

A high-quality code review validation shows:
- **Thoroughness**: All categories addressed
- **Automation**: Checks run before manual review
- **Specificity**: Clear file:line references
- **Prioritization**: Severity levels applied
- **Actionability**: Developer knows next steps
- **Balance**: Critical issues + positive highlights

## Usage Notes

**When to run**: After completing code quality review with @code-quality-reviewer agent

**Expected outcome**: Confirmation that review was comprehensive, or specific gaps to address

**Follow-up actions**: If validation fails, re-run review addressing the gaps identified
