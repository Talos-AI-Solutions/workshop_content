---
description: Validates that comprehensive design review was conducted following 7-phase methodology with live environment testing, accessibility compliance, and visual evidence
allowed-tools: Bash(git:*), Read, Grep, mcp__context7__resolve-library-id, mcp__context7__get-library-docs
---

# Design Review Validation

Verify that comprehensive design review was executed following the 7-phase "Live Environment First" methodology as defined in `.claude/agents/design-review-agent.md`.

## Validation Steps

### 1. Git Context Verification

Check that PR context was properly analyzed:

```bash
# Verify current branch status
git status

# List modified files
git diff --name-only HEAD

# Display commit history
git log --oneline -10

# Extract complete diff content
git diff HEAD
```

**Validation Criteria**:
- ✅ **PR Context Analyzed**: Modified files identified and documented
- ✅ **Diff Extracted**: Changes properly analyzed
- ❌ **Missing Context**: Unable to determine PR scope

### 2. Review Phase Completeness

Verify all 7 phases from `.claude/agents/design-review-agent.md` were executed:

```bash
# Check for review documentation
ls -la review-report.md design-review.md 2>/dev/null || echo "⚠️ Review report not found"

# Check for any markdown files created recently
find . -name "*.md" -type f -mtime -1 2>/dev/null | grep -E "(review|design)" || echo "⚠️ No recent review files"
```

**Required Phases**:
- ✅ **Phase 1 - Preparation**: PR context analyzed, live preview setup documented
- ✅ **Phase 2 - Interaction Testing**: User flows executed, interactive states validated with evidence
- ✅ **Phase 3 - Responsiveness**: Multi-viewport testing completed (1440px, 768px, 375px)
- ✅ **Phase 4 - Visual Polish**: Alignment, typography, hierarchy assessed with specifics
- ✅ **Phase 5 - Accessibility**: WCAG 2.1 AA compliance validated with criterion references
- ✅ **Phase 6 - Robustness**: Edge cases, validation, and error states tested
- ✅ **Phase 7 - Code Health**: Component patterns and design tokens reviewed

### 3. Accessibility Compliance

Validate WCAG 2.1 AA requirements were checked:

```bash
# Check for accessibility-related files or documentation
grep -r "WCAG\|accessibility\|a11y\|aria-\|alt=\|tabindex\|role=" --include="*.tsx" --include="*.jsx" --include="*.html" . 2>/dev/null | head -20 || echo "⚠️ No accessibility patterns found in code"
```

**Required Checks**:
- ✅ **Keyboard Navigation**: All interactive elements accessible via keyboard (Tab, Escape, Arrow keys)
- ✅ **Focus Indicators**: Visible focus states on all focusable elements (min 2px outline)
- ✅ **Contrast Ratios**: Minimum 4.5:1 for text, 3:1 for UI components
- ✅ **Screen Reader**: ARIA labels, semantic HTML, alt text present
- ✅ **Form Labels**: All inputs have associated labels
- ⚠️ **Partial Compliance**: Some accessibility issues found but documented
- ❌ **Non-Compliant**: Critical WCAG 2.1 AA violations found

### 4. Visual Evidence Documentation

Check for screenshot evidence as required by methodology:

```bash
# Count screenshot files created recently
find . -type f \( -name "*.png" -o -name "*.jpg" -o -name "*.jpeg" -o -name "*.gif" \) -mtime -1 2>/dev/null | wc -l

# Check for images in typical screenshot locations
ls -la screenshots/ evidence/ images/ 2>/dev/null || echo "ℹ️ No dedicated screenshot directory"
```

**Evidence Criteria**:
- ✅ **Evidence Present**: Screenshots captured for all visual findings
- ✅ **Multi-Viewport**: Evidence from desktop, tablet, mobile viewports
- ✅ **Interactive States**: Screenshots of hover, focus, active states
- ⚠️ **Limited Evidence**: Some findings lack visual proof
- ❌ **No Evidence**: No screenshots documented for visual issues

### 5. Design Standards Compliance

Validate against design principles from `.claude/context/review/design-review-standards.md`:

```bash
# Check if context file exists
ls -la .claude/context/review/design-review-standards.md 2>/dev/null || echo "❌ Design standards context file not found"

# Check for design system token usage
grep -r "var(--\|theme\.\|colors\.\|spacing\." --include="*.css" --include="*.tsx" --include="*.jsx" . 2>/dev/null | head -10 || echo "⚠️ Limited design token usage found"
```

**Standards Validation**:
- ✅ **Standards Referenced**: Design principles explicitly applied in review
- ✅ **Design System Used**: Component patterns from standards followed
- ✅ **Token Usage**: CSS variables/design tokens used consistently
- ⚠️ **Inconsistent Application**: Some standards violations documented
- ❌ **Standards Ignored**: No reference to design guidelines

### 6. Issue Triage Completeness

Verify findings are properly categorized per agent methodology:

```bash
# Check for properly triaged issues in review documentation
grep -E "\[Blocker\]|\[High-Priority\]|\[Medium-Priority\]|\[Nitpick\]" review-report.md design-review.md 2>/dev/null || echo "⚠️ No triaged issues found in standard locations"
```

**Triage Categories** (from `.claude/agents/design-review-agent.md`):
- ✅ **Triage Applied**: Issues categorized by severity (Blocker/High/Medium/Nitpick)
- ✅ **Blockers Identified**: Critical failures clearly documented
- ✅ **Actionable Feedback**: Clear, specific issue descriptions with impact
- ✅ **Evidence Linked**: Each issue has supporting screenshot or metric
- ⚠️ **Vague Findings**: Some issues lack specificity or evidence
- ❌ **No Categorization**: Findings not prioritized by severity

### 7. Review Report Quality

Check for comprehensive markdown-formatted report:

```bash
# Verify report exists and has structure
if [ -f "review-report.md" ] || [ -f "design-review.md" ]; then
  echo "✅ Review report found"
  # Check for key sections
  for section in "Preparation" "Interaction" "Responsiveness" "Visual" "Accessibility" "Robustness" "Code"; do
    grep -i "$section" review-report.md design-review.md 2>/dev/null && echo "  ✅ $section section present" || echo "  ⚠️ $section section missing"
  done
else
  echo "❌ Review report not found (expected: review-report.md or design-review.md)"
fi
```

**Report Quality Criteria**:
- ✅ **Complete Report**: All 7 phases documented (Preparation, Interaction, Responsiveness, Visual, Accessibility, Robustness, Code)
- ✅ **Screenshot Links**: Visual evidence embedded or linked
- ✅ **Triage System**: Issues categorized by severity with clear priorities
- ✅ **WCAG References**: Accessibility issues cite specific criteria
- ✅ **Actionable**: Clear "what" and "why" for each finding
- ⚠️ **Incomplete Report**: Some sections missing or underdeveloped
- ❌ **No Report**: Review not properly documented

## Context7 Validation

Verify latest standards were consulted:

```bash
# This validation should reference Context7 for:
# - Latest Playwright documentation
# - Current WCAG 2.1 AA guidelines
# - Modern CSS framework best practices
echo "ℹ️ Reviewer should have validated against:"
echo "  - Playwright latest docs via Context7"
echo "  - WCAG 2.1 AA current guidelines via Context7"
echo "  - Modern CSS framework patterns via Context7"
```

**Context7 Usage**:
- ✅ **Standards Current**: Reviewer validated against latest Playwright/WCAG docs
- ✅ **Framework Patterns**: Modern CSS framework patterns referenced
- ⚠️ **Partial Validation**: Some standards checked, others assumed
- ❌ **No Validation**: No Context7 consultation for current best practices

## Reference Documentation

For detailed implementation guidance and patterns:
- **Agent Methodology**: `.claude/agents/design-review-agent.md` - Full 7-phase review process
- **Design Excellence**: `.claude/context/review/design-review-standards.md` - Code examples and patterns
- **Latest Specs**: Context7 documentation for Playwright, WCAG 2.1 AA, and CSS frameworks

## Reporting Format

Use emoji indicators for clear status communication:
- ✅ for passed validations and completed criteria
- ⚠️ for warnings, recommendations, and partial completion
- ❌ for errors, missing requirements, and blockers
- ℹ️ for informational notes and guidance

## Final Deployment Readiness Assessment

Comprehensive validation summary:

**Design Review Completeness**:
- ✅ **All 7 Phases Complete**: Full methodology executed with evidence
- ✅ **WCAG 2.1 AA Compliant**: No critical accessibility violations
- ✅ **Visual Evidence**: Screenshots document all findings across viewports
- ✅ **Design Standards Applied**: Principles from context file referenced
- ✅ **Actionable Report**: Clear, prioritized findings with impact explained
- ✅ **No Blockers**: No critical issues preventing merge
- ✅ **Context7 Current**: Latest standards validated

**Merge Recommendation**:
- ✅ **APPROVED**: All criteria met, no blockers, ready for merge
- ⚠️ **APPROVED WITH CONDITIONS**: Minor issues documented, can merge with follow-up
- ❌ **BLOCKED**: Critical issues must be resolved before merge

## Common Validation Failures

**Frequently Missing Elements**:
1. **Screenshot Evidence**: Visual findings without supporting images
2. **Accessibility Testing**: Missing keyboard navigation or contrast checks
3. **Multi-Viewport**: Only tested on desktop, mobile/tablet ignored
4. **Edge Cases**: Happy path only, error states not validated
5. **Context7**: Assumed standards without validating latest docs
6. **Triage**: Issues listed without severity categorization
7. **Report Structure**: Ad-hoc notes instead of structured 7-phase report

## Validation Improvement Tips

**For Comprehensive Reviews**:
- Use Playwright scripts for automated multi-viewport screenshots
- Reference WCAG criteria explicitly (e.g., "1.4.3 Contrast Minimum")
- Test with keyboard only (no mouse) for full interaction flow
- Use browser DevTools for contrast ratio checking
- Create dedicated screenshots/ directory for evidence
- Follow report template matching 7-phase structure
- Consult Context7 for any framework/standard uncertainty

**Triangle Integration**:
- Reference `.claude/agents/design-review-agent.md` for methodology guidance
- Reference `.claude/context/review/design-review-standards.md` for code patterns
- Use Context7 for latest Playwright and WCAG documentation
