---
description: Comprehensive standards for trunk-based development with Conventional Commits v1.0.0 and GitHub MCP integration
globs: [".git/**/*", "**/.gitignore", ".github/**/*"]
alwaysApply: false
---

# Git Workflow Standards

Comprehensive standards for trunk-based development with Conventional Commits v1.0.0 and GitHub MCP integration.

## Table of Contents

1. [Conventional Commits v1.0.0 Specification](#conventional-commits-v100-specification)
2. [Commit Message Examples](#commit-message-examples)
3. [Bash Command Examples](#bash-command-examples)
4. [GitHub MCP Operations](#github-mcp-operations)
5. [Quality Indicators](#quality-indicators)
6. [Anti-Patterns](#anti-patterns)

## Conventional Commits v1.0.0 Specification

### Format Structure

```
<type>[optional scope][optional !]: <description>

[optional body]

[optional footer(s)]
```

### Type Values

**Required types**:
- `feat` - New feature for the user
- `fix` - Bug fix for the user
- `docs` - Documentation only changes
- `style` - Code style changes (formatting, missing semicolons, etc)
- `refactor` - Code change that neither fixes a bug nor adds a feature
- `perf` - Performance improvement
- `test` - Adding or modifying tests
- `build` - Changes to build system or dependencies
- `ci` - Changes to CI configuration files and scripts
- `chore` - Other changes that don't modify src or test files
- `revert` - Reverts a previous commit

### Scope Rules

- Optional but recommended for multi-module projects
- Must be noun describing section of codebase
- Enclosed in parentheses after type
- Use kebab-case for multi-word scopes
- Examples: `api`, `parser`, `auth-service`, `ui-components`

### Breaking Change Indicator

Two methods (both valid):
1. Append `!` after type/scope: `feat(api)!: remove deprecated endpoint`
2. Add footer: `BREAKING CHANGE: description of the breaking change`

### Subject Line Rules

- Use imperative mood ("add" not "added" or "adds")
- No capitalization of first letter
- No period at the end
- Maximum 100 characters
- Be concise but descriptive

### Body Rules

- Optional, provides additional context
- Use imperative mood
- Wrap at 72 characters
- Separate from subject with blank line
- Explain what and why, not how

### Footer Rules

- Optional, follows body after blank line
- Use for breaking changes, issue references, metadata
- Format: `<token>: <value>` or `BREAKING CHANGE: <description>`
- Multiple footers allowed
- Common tokens: `Fixes`, `Closes`, `Refs`, `BREAKING CHANGE`

## Commit Message Examples

### Feature Addition (Simple)

```
feat: add user authentication endpoint
```

### Feature Addition (With Scope)

```
feat(auth): implement JWT token refresh mechanism
```

### Bug Fix (With Body)

```
fix(parser): handle null values in JSON response

Previously the parser would crash when encountering null values
in nested objects. This change adds null checking before accessing
nested properties.
```

### Bug Fix (With Issue Reference)

```
fix(api): prevent memory leak in websocket handler

Fixes #847
```

### Breaking Change (Method 1: Exclamation)

```
feat(api)!: remove support for API v1 endpoints
```

### Breaking Change (Method 2: Footer)

```
feat(api): migrate to REST API v2

BREAKING CHANGE: API v1 endpoints are no longer supported. Clients
must upgrade to v2 endpoints. See migration guide at docs/api-v2.md
```

### Breaking Change (Both Methods)

```
refactor(core)!: restructure configuration format

BREAKING CHANGE: configuration files must now use YAML format instead
of JSON. Use the migration script at scripts/migrate-config.sh to
convert existing configurations.

Refs #1234
```

### Documentation Update

```
docs: update installation instructions for Windows
```

### Performance Improvement

```
perf(db): optimize query performance for user lookups

Reduces average query time from 45ms to 12ms by adding composite
index on user_id and tenant_id columns.
```

### Refactoring

```
refactor(auth): extract validation logic into separate module
```

### Test Addition

```
test(api): add integration tests for error handling
```

### Build System Change

```
build: upgrade typescript to version 5.3
```

### CI Configuration

```
ci: add automated security scanning to pipeline
```

### Chore

```
chore: update dependencies to latest versions
```

### Revert

```
revert: feat(api): add experimental caching layer

This reverts commit 1a2b3c4d5e6f7g8h9i0j.
```

### Multi-Scope Change

```
feat(api,ui): add real-time notification system

Implements WebSocket-based notifications on the backend and
updates the UI to display notifications in real-time.

Closes #456
```

### Complex Example

```
feat(auth)!: implement role-based access control

This change introduces a comprehensive RBAC system replacing
the previous permission-based approach. Users are now assigned
roles which determine their access levels across the platform.

The new system provides better granularity and easier management
of user permissions at scale.

BREAKING CHANGE: the permissions API has been replaced with roles
API. Existing permission assignments must be migrated to roles
using the migration script at scripts/migrate-to-rbac.sh

Closes #789
Refs #456, #567
```

## Bash Command Examples

### Repository State Checks

**Check working tree status**:
```bash
git status --porcelain
```

**Check if branch is up to date with remote**:
```bash
git fetch origin && git status
```

**Check for uncommitted changes**:
```bash
if [[ -n $(git status -s) ]]; then echo "Uncommitted changes"; fi
```

**Verify clean working tree**:
```bash
git diff --quiet && git diff --cached --quiet || echo "Working tree not clean"
```

### Commit Message Validation

**Get last N commit messages**:
```bash
git log -n 10 --pretty=format:"%s"
```

**Get full commit with body and footer**:
```bash
git log -1 --pretty=format:"%B"
```

**Check commit message format**:
```bash
git log -1 --pretty=format:"%s" | grep -E "^(feat|fix|docs|style|refactor|perf|test|build|ci|chore|revert)(\(.+\))?!?: .{1,100}$"
```

**Extract commit type**:
```bash
git log -1 --pretty=format:"%s" | sed -E 's/^([a-z]+).*/\1/'
```

**Check for breaking change indicator**:
```bash
git log -1 --pretty=format:"%s" | grep -E "!:" || git log -1 --pretty=format:"%B" | grep "BREAKING CHANGE:"
```

**Validate subject length**:
```bash
subject=$(git log -1 --pretty=format:"%s")
if [[ ${#subject} -gt 100 ]]; then echo "Subject too long"; fi
```

### PR Size Calculation

**Count lines changed in branch**:
```bash
git diff --shortstat origin/main...HEAD | awk '{print $4 + $6}'
```

**Count files changed**:
```bash
git diff --name-only origin/main...HEAD | wc -l
```

**Detailed change stats**:
```bash
git diff --stat origin/main...HEAD
```

**Check if PR exceeds size limit**:
```bash
changes=$(git diff --shortstat origin/main...HEAD | awk '{print $4 + $6}')
if [[ $changes -gt 400 ]]; then echo "PR too large: $changes lines"; fi
```

### Branch Management

**Get current branch name**:
```bash
git branch --show-current
```

**Check if branch exists**:
```bash
git show-ref --verify --quiet refs/heads/feat/new-feature && echo "Branch exists"
```

**Get branch creation date**:
```bash
git log --reverse --pretty=format:"%ai" origin/main..HEAD | head -1
```

**Calculate branch age in days**:
```bash
created=$(git log --reverse --pretty=format:"%at" origin/main..HEAD | head -1)
now=$(date +%s)
days=$(( (now - created) / 86400 ))
echo "Branch age: $days days"
```

**List branches older than 3 days**:
```bash
for branch in $(git for-each-ref --format='%(refname:short)' refs/heads/); do
  age=$(git log -1 --format="%at" $branch)
  if [[ $(($(date +%s) - age)) -gt 259200 ]]; then
    echo "$branch"
  fi
done
```

**Delete merged branches**:
```bash
git branch --merged main | grep -v "main\|master" | xargs git branch -d
```

### Linear History Validation

**Check for merge commits**:
```bash
git log --oneline --merges origin/main..HEAD
```

**Count merge commits in branch**:
```bash
git log --oneline --merges origin/main..HEAD | wc -l
```

**Verify fast-forward possible**:
```bash
git merge-base --is-ancestor origin/main HEAD && echo "Fast-forward possible"
```

**Check if branch needs rebase**:
```bash
if [[ $(git rev-list --left-only --count origin/main...HEAD) -gt 0 ]]; then
  echo "Branch needs rebase"
fi
```

### Conflict Detection

**Check for merge conflicts**:
```bash
git merge-tree $(git merge-base origin/main HEAD) origin/main HEAD | grep -q "<<<<<<<"
```

**List conflicting files**:
```bash
git diff --name-only --diff-filter=U
```

**Check for conflict markers**:
```bash
grep -r "^<<<<<<< \|^=======$\|^>>>>>>>" --include="*.js" --include="*.ts" .
```

### Rebase Workflows

**Rebase on main**:
```bash
git fetch origin && git rebase origin/main
```

**Interactive rebase last 5 commits**:
```bash
git rebase -i HEAD~5
```

**Continue rebase after resolving conflicts**:
```bash
git add . && git rebase --continue
```

**Abort rebase**:
```bash
git rebase --abort
```

### Branch Hygiene

**Sync with main**:
```bash
git fetch origin main && git rebase origin/main
```

**Force push after rebase**:
```bash
git push --force-with-lease origin feat/my-feature
```

**Clean up local branches**:
```bash
git fetch --prune
```

**Delete remote branch**:
```bash
git push origin --delete feat/old-feature
```

## GitHub MCP Operations

### Repository Operations

**Get repository information**:
```javascript
mcp__github__get-repository({
  owner: "organization-name",
  repo: "repository-name"
})
```

**List repository branches**:
```javascript
mcp__github__list-branches({
  owner: "organization-name",
  repo: "repository-name"
})
```

**Get branch protection rules**:
```javascript
mcp__github__get-branch-protection({
  owner: "organization-name",
  repo: "repository-name",
  branch: "main"
})
```

### Pull Request Operations

**Create pull request**:
```javascript
mcp__github__create-pull-request({
  owner: "organization-name",
  repo: "repository-name",
  title: "feat(auth): implement OAuth2 integration",
  body: "## Summary\n- Add OAuth2 provider support\n- Implement token refresh\n\n## Test Plan\n- [ ] Unit tests pass\n- [ ] Integration tests pass",
  head: "feat/oauth2",
  base: "main"
})
```

**Get pull request**:
```javascript
mcp__github__get-pull-request({
  owner: "organization-name",
  repo: "repository-name",
  pull_number: 123
})
```

**List pull request files**:
```javascript
mcp__github__list-pull-request-files({
  owner: "organization-name",
  repo: "repository-name",
  pull_number: 123
})
```

**Merge pull request**:
```javascript
mcp__github__merge-pull-request({
  owner: "organization-name",
  repo: "repository-name",
  pull_number: 123,
  merge_method: "squash",
  commit_title: "feat(auth): implement OAuth2 integration (#123)",
  commit_message: "Implements OAuth2 provider support with token refresh"
})
```

**Update pull request**:
```javascript
mcp__github__update-pull-request({
  owner: "organization-name",
  repo: "repository-name",
  pull_number: 123,
  title: "feat(auth)!: implement OAuth2 integration",
  body: "Updated description with breaking changes"
})
```

### Review Operations

**Create review**:
```javascript
mcp__github__create-review({
  owner: "organization-name",
  repo: "repository-name",
  pull_number: 123,
  event: "APPROVE",
  body: "LGTM - code follows standards"
})
```

**List reviews**:
```javascript
mcp__github__list-reviews({
  owner: "organization-name",
  repo: "repository-name",
  pull_number: 123
})
```

**Request reviewers**:
```javascript
mcp__github__request-reviewers({
  owner: "organization-name",
  repo: "repository-name",
  pull_number: 123,
  reviewers: ["user1", "user2"]
})
```

### Status Check Operations

**Get commit status**:
```javascript
mcp__github__get-commit-status({
  owner: "organization-name",
  repo: "repository-name",
  ref: "feat/oauth2"
})
```

**List status checks**:
```javascript
mcp__github__list-status-checks({
  owner: "organization-name",
  repo: "repository-name",
  ref: "abc123def"
})
```

### Issue Operations

**Create issue**:
```javascript
mcp__github__create-issue({
  owner: "organization-name",
  repo: "repository-name",
  title: "Bug: authentication fails on refresh",
  body: "Description of the issue",
  labels: ["bug", "auth"]
})
```

**Link PR to issue**:
Include in PR body:
```markdown
Fixes #847
Closes #123, #456
Refs #789
```

## Quality Indicators

### Commit Quality Metrics

**Excellent commit**:
- Follows Conventional Commits v1.0.0 exactly
- Subject under 72 characters (ideal: 50)
- Body wraps at 72 characters
- Explains why, not what
- References related issues
- Single logical change

**Good commit**:
- Correct type and format
- Clear subject line
- Has body when needed
- Reasonable scope

**Poor commit**:
- Vague subject ("fix bug", "update code")
- No type prefix
- Multiple unrelated changes
- No context in body
- Too large (>200 lines)

### PR Quality Metrics

**Excellent PR**:
- Under 200 lines changed
- Single feature/fix
- All CI checks pass
- Reviewed within 8 hours
- Merged within 24 hours
- Clear description and test plan

**Good PR**:
- Under 400 lines changed
- Cohesive changes
- CI passing
- Merged within 48 hours
- Has description

**Poor PR**:
- Over 400 lines changed
- Multiple features mixed
- No description
- Stale (>3 days old)
- Failing CI

### Branch Quality Metrics

**Healthy branch**:
- Less than 2 days old
- Less than 10 commits ahead
- No merge commits
- Synced with main
- Descriptive name

**Needs attention**:
- 3-7 days old
- 10-20 commits ahead
- Few merge commits
- Behind main

**Critical**:
- Over 7 days old
- Over 20 commits ahead
- Many merge commits
- Conflicts with main

## Anti-Patterns

### Commit Message Anti-Patterns

**Vague messages**:
```
# BAD
fix: bug fix
feat: new feature
chore: stuff

# GOOD
fix(parser): handle null values in nested objects
feat(auth): implement JWT token refresh
chore(deps): upgrade typescript to version 5.3
```

**Wrong mood**:
```
# BAD
feat: added user login
fix: fixed the parser bug
docs: updated readme

# GOOD
feat: add user login
fix: resolve parser bug
docs: update readme
```

**Capitalized or punctuated**:
```
# BAD
feat: Add User Login.
Fix: Parser bug

# GOOD
feat: add user login
fix: parser bug
```

**Missing type**:
```
# BAD
add user authentication
update dependencies

# GOOD
feat: add user authentication
chore: update dependencies
```

**Wrong type usage**:
```
# BAD
feat: fix typo in documentation
fix: add new API endpoint

# GOOD
docs: fix typo in documentation
feat: add new API endpoint
```

**Multiple changes in one commit**:
```
# BAD
feat: add login, update UI, fix parser bug

# GOOD
feat(auth): add user login
feat(ui): update login page styling
fix(parser): handle null values
```

**Breaking change not marked**:
```
# BAD
refactor(api): change endpoint structure

# GOOD
refactor(api)!: change endpoint structure

BREAKING CHANGE: API endpoints now use /v2/ prefix. Update client
code to use new endpoint URLs.
```

### Branch Management Anti-Patterns

**Long-lived branches**:
```
# BAD
feature/big-refactor (30 days old, 150 commits)

# GOOD
feat/auth-oauth2 (2 days old, 5 commits)
```

**Poor branch names**:
```
# BAD
my-branch
test
new-stuff
fix

# GOOD
feat/oauth2-integration
fix/memory-leak-websocket
chore/update-dependencies
```

**Merge commits in feature branch**:
```
# BAD
* Merge branch 'main' into feat/my-feature
* feat: add feature part 2
* Merge branch 'main' into feat/my-feature
* feat: add feature part 1

# GOOD
* feat: add feature part 2
* feat: add feature part 1
(rebased on main)
```

### PR Anti-Patterns

**Massive PRs**:
```
# BAD
feat: complete refactor of entire codebase
+2500 lines, -1800 lines, 45 files changed

# GOOD
refactor(auth): extract validation logic
+85 lines, -60 lines, 4 files changed
```

**Missing description**:
```
# BAD
Title: feat: add feature
Body: (empty)

# GOOD
Title: feat(auth): implement OAuth2 integration
Body:
## Summary
- Add OAuth2 provider support
- Implement token refresh mechanism

## Test Plan
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Manual testing completed
```

**Stale PRs**:
```
# BAD
Created: 14 days ago
Last updated: 10 days ago
Conflicts with main

# GOOD
Created: 1 day ago
Last updated: 2 hours ago
Up to date with main
```

### Workflow Anti-Patterns

**Committing directly to main**:
```
# BAD
git commit -m "feat: quick fix" && git push origin main

# GOOD
git checkout -b fix/quick-issue
git commit -m "fix: resolve issue"
git push origin fix/quick-issue
(create PR, get review, merge)
```

**Force push without lease**:
```
# BAD
git push --force origin feat/my-branch

# GOOD
git push --force-with-lease origin feat/my-branch
```

**Ignoring CI failures**:
```
# BAD
CI: 3 tests failing
Action: Merge anyway

# GOOD
CI: 3 tests failing
Action: Fix tests, push update, wait for green
```

**No code review**:
```
# BAD
Create PR -> Immediately merge

# GOOD
Create PR -> Request review -> Address feedback -> Merge
```

## References

- [Conventional Commits v1.0.0](https://www.conventionalcommits.org/en/v1.0.0/)
- [Trunk-Based Development](https://trunkbaseddevelopment.com/)
- GitHub MCP Server Documentation
- Git Best Practices
