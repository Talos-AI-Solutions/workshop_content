---
description: Runs comprehensive Java code quality checks including SpotBugs, PMD, Checkstyle, Error Prone, and build verification
allowed-tools:
  - Bash(./gradlew:*)
  - Bash(./mvnw:*)
  - Bash(gradle:*)
  - Bash(mvn:*)
  - Read
  - Grep
  - Glob
---

# Comprehensive Java Code Quality Analysis

Perform thorough static analysis, code quality checks, and build verification for Java projects.

## Quality Check Steps

### 1. Project Detection
First, identify the Java project type:
- Check for `build.gradle` or `build.gradle.kts` (Gradle project)
- Check for `pom.xml` (Maven project)
- Report project type and build tool version
- If neither found, report this should be run from Java project root

### 2. Build Tool Verification
Verify build wrapper availability:
- Check for `gradlew` or `gradlew.bat` (Gradle wrapper)
- Check for `mvnw` or `mvnw.bat` (Maven wrapper)
- Report wrapper availability or suggest using system-installed version

### 3. Static Analysis Tools Detection
Scan for configured quality tools:

#### Gradle Projects
Check `build.gradle` or `build.gradle.kts` for:
- SpotBugs plugin: `id 'com.github.spotbugs'`
- PMD plugin: `id 'pmd'`
- Checkstyle plugin: `id 'checkstyle'`
- Error Prone plugin: `net.ltgt.errorprone`
- JaCoCo plugin: `id 'jacoco'`
- Report configured plugins

#### Maven Projects
Check `pom.xml` for:
- SpotBugs: `spotbugs-maven-plugin`
- PMD: `maven-pmd-plugin`
- Checkstyle: `maven-checkstyle-plugin`
- Error Prone: `error-prone`
- JaCoCo: `jacoco-maven-plugin`
- Report configured plugins

### 4. Compilation and Type Checking
Run full compilation to verify type safety:

#### Gradle
- Run `./gradlew clean compileJava compileTestJava`
- Report compilation success or failures
- Report number of warnings and errors

#### Maven
- Run `./mvnw clean compile test-compile`
- Report compilation success or failures
- Report number of warnings and errors

### 5. SpotBugs Static Analysis
If SpotBugs is configured:

#### Gradle
- Run `./gradlew spotbugsMain spotbugsTest`
- Report bug categories found (high/medium/low priority)
- Report specific bug patterns detected

#### Maven
- Run `./mvnw spotbugs:check`
- Report bug categories and patterns
- Suggest reviewing HTML report if failures found

### 6. PMD Analysis
If PMD is configured:

#### Gradle
- Run `./gradlew pmdMain pmdTest`
- Report rule violations by category
- Report violation counts and severity levels

#### Maven
- Run `./mvnw pmd:check`
- Report violations and suggest reviewing reports
- Categorize by priority

### 7. Checkstyle Validation
If Checkstyle is configured:

#### Gradle
- Run `./gradlew checkstyleMain checkstyleTest`
- Report style violations and categories
- Suggest configuration adjustments if too strict

#### Maven
- Run `./mvnw checkstyle:check`
- Report violations by severity
- Provide violation statistics

### 8. Error Prone Analysis
If Error Prone is configured:
- Already runs during compilation
- Report any Error Prone warnings from compile output
- Categorize by severity (ERROR, WARNING, SUGGESTION)

### 9. Build Verification
Run full build including tests:

#### Gradle
- Run `./gradlew build`
- Report overall build success
- Report test results summary

#### Maven
- Run `./mvnw verify`
- Report build phases completed
- Report test execution results

### 10. Project Statistics
Generate project overview:
- Count Java source files: `find src/main/java -name "*.java" | wc -l`
- Count test files: `find src/test/java -name "*.java" | wc -l`
- Report lines of code if available
- Calculate test-to-code ratio

## Quality Standards Enforcement
Apply consistent quality standards:
- **Zero tolerance**: Compilation errors must be resolved
- **High priority**: SpotBugs high-priority bugs must be fixed
- **Medium priority**: PMD critical violations should be addressed
- **Code style**: Checkstyle violations should be minimal
- **Build verification**: Full build must succeed

## Reporting Format
Use clear status indicators:
- ✅ for passed checks and successful builds
- ⚠️ for warnings and medium-priority issues
- ❌ for errors requiring immediate attention

## Quick Fix Guidance
When issues are found:
- **Compilation errors**: Review error messages and fix type issues
- **SpotBugs issues**: Check HTML report in `build/reports/spotbugs/`
- **PMD violations**: Review report in `build/reports/pmd/`
- **Checkstyle**: Check report in `build/reports/checkstyle/`
- **Test failures**: Run tests individually to isolate issues

## Configuration References
Point users to configuration files:
- Gradle config: `build.gradle` or `build.gradle.kts`
- Maven config: `pom.xml`
- Checkstyle rules: `config/checkstyle/checkstyle.xml`
- PMD ruleset: `config/pmd/ruleset.xml`
- SpotBugs exclusions: `config/spotbugs/excludeFilter.xml`

## Related Commands
For additional validation:
- `/java:test-coverage` - Run JaCoCo and PITest mutation testing
- `/orchestration:validate-docker` - Validate Java Docker configuration
- `/database:validate-hibernate-entities` - Validate JPA entities

## Reference Documentation
For comprehensive Java quality standards:
- **Java Standards**: `.claude/context/java/java-standards.md`
- **Testing Patterns**: `.claude/context/java/testing-standards.md`
- **Agent Guidance**: `.claude/agents/java/java-engineer.md`
