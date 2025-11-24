---
description: Runs JaCoCo code coverage analysis and PITest mutation testing to ensure comprehensive test quality
allowed-tools:
  - Bash(./gradlew:*)
  - Bash(./mvnw:*)
  - Bash(gradle:*)
  - Bash(mvn:*)
  - Read
  - Grep
  - Glob
---

# Java Test Coverage and Mutation Testing

Perform comprehensive test coverage analysis with JaCoCo and mutation testing with PITest to verify test effectiveness.

## Coverage Analysis Steps

### 1. Project Detection
Identify Java project and build tool:
- Check for `build.gradle` or `build.gradle.kts` (Gradle)
- Check for `pom.xml` (Maven)
- Report build tool and version

### 2. Coverage Tools Detection
Verify coverage tools are configured:

#### Gradle Projects
Check for JaCoCo plugin:
- Look for `id 'jacoco'` in build file
- Check for PITest plugin: `id 'info.solidsoft.pitest'`
- Report configured coverage tools

#### Maven Projects
Check for coverage plugins:
- Look for `jacoco-maven-plugin` in pom.xml
- Look for `pitest-maven-plugin`
- Report plugin versions

### 3. Run Unit Tests with Coverage
Execute tests with coverage collection:

#### Gradle
- Run `./gradlew clean test jacocoTestReport`
- Report test execution results
- Report coverage report generation location

#### Maven
- Run `./mvnw clean test jacoco:report`
- Report test results
- Report coverage data location

### 4. Analyze JaCoCo Coverage Results
Parse and report coverage metrics:
- Read coverage report from `build/reports/jacoco/test/html/index.html` (Gradle)
- Read coverage report from `target/site/jacoco/index.html` (Maven)
- Report coverage percentages:
  - **Instruction Coverage** (bytecode level)
  - **Branch Coverage** (decision points)
  - **Line Coverage** (source lines)
  - **Method Coverage**
  - **Class Coverage**

### 5. Verify Coverage Thresholds
Check coverage meets minimum standards:
- ✅ **Excellent**: 90%+ instruction coverage, 85%+ branch coverage
- ⚠️ **Good**: 80-89% instruction, 75-84% branch
- ❌ **Insufficient**: Below 80% instruction or 75% branch
- Report areas with low coverage

### 6. Generate Coverage Enforcement Report
If coverage verification is configured:

#### Gradle
- Check `jacocoTestCoverageVerification` task configuration
- Run `./gradlew jacocoTestCoverageVerification`
- Report pass/fail against thresholds

#### Maven
- Check for coverage rules in pom.xml
- Run `./mvnw jacoco:check`
- Report threshold violations

### 7. Run PITest Mutation Testing
Execute mutation testing to verify test quality:

#### Gradle
- Run `./gradlew pitest`
- Report mutation testing execution
- Note: PITest can take 5-10x longer than regular tests

#### Maven
- Run `./mvnw pitest:mutationCoverage`
- Report mutation analysis progress

### 8. Analyze PITest Results
Parse mutation testing results:
- Read PITest report from `build/reports/pitest/` (Gradle)
- Read PITest report from `target/pit-reports/` (Maven)
- Report mutation coverage metrics:
  - **Mutation Coverage**: Percentage of mutations killed
  - **Test Strength**: Mutation score
  - **Survived Mutations**: Weaknesses in tests
  - **Line Coverage**: Traditional coverage from PITest

### 9. Verify Mutation Coverage Thresholds
Check mutation testing meets standards:
- ✅ **Excellent**: 80%+ mutation coverage
- ⚠️ **Good**: 70-79% mutation coverage
- ❌ **Weak**: Below 70% mutation coverage
- Report specific mutations that survived

### 10. Generate Comprehensive Coverage Report
Provide actionable coverage summary:
- Compare JaCoCo line coverage vs PITest mutation coverage
- Identify classes/packages with coverage gaps
- Highlight areas with high line coverage but low mutation coverage
- Suggest specific test improvements

## Coverage Quality Metrics

### JaCoCo Metrics Interpretation
- **Instruction Coverage**: Most accurate, measures bytecode execution
- **Branch Coverage**: Critical for conditional logic validation
- **Line Coverage**: Traditional metric, can be misleading
- Target: 90%+ instruction, 85%+ branch for production code

### PITest Metrics Interpretation
- **Mutation Coverage**: Measures test effectiveness, not just coverage
- **Survived Mutations**: Indicate weak or missing assertions
- **Killed Mutations**: Confirm tests verify behavior
- Target: 80%+ mutation coverage for critical code

## Reporting Format
- ✅ for passing coverage thresholds
- ⚠️ for borderline coverage needing improvement
- ❌ for insufficient coverage requiring action

## Quick Fix Guidance
When coverage is insufficient:
- **Low branch coverage**: Add tests for edge cases and error paths
- **Low mutation coverage**: Strengthen assertions in existing tests
- **Survived mutations**: Review specific mutations and add targeted tests
- **Uncovered classes**: Identify missing test files

## Configuration Examples
Point to configuration locations:

#### Gradle
- JaCoCo config: `jacoco { }` block in build.gradle
- PITest config: `pitest { }` block in build.gradle
- Coverage thresholds: `jacocoTestCoverageVerification { }`

#### Maven
- JaCoCo config: `jacoco-maven-plugin` configuration
- PITest config: `pitest-maven-plugin` configuration
- Coverage rules: `<rules>` in jacoco-maven-plugin

## Related Commands
- `/java:lint-all` - Run static analysis before coverage
- `/database:validate-hibernate-entities` - Validate entity test coverage
- `/orchestration:validate-docker` - Include test execution in Docker

## Reference Documentation
For comprehensive testing patterns and examples:
- **Testing Standards**: `.claude/context/java/testing-standards.md`
- **Java Standards**: `.claude/context/java/java-standards.md`
- **Agent Guidance**: `.claude/agents/java/java-engineer.md`
