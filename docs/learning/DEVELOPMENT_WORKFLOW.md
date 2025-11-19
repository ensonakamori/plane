# Development Workflow - Professional Git & CI/CD

**Work like a pro!** This guide covers the professional development workflow used in Plane: Git branching, commits, pull requests, code review, and deployment.

**📅 Documentation Date:** November 19, 2025
**⏱️ Estimated Reading Time:** 45-60 minutes
**📊 Difficulty Level:** Intermediate
**🎯 Target Audience:** Developers contributing to Plane

---

## Table of Contents

1. [Git Workflow](#git-workflow)
2. [Branch Naming](#branch-naming)
3. [Commit Messages](#commit-messages)
4. [Pull Requests](#pull-requests)
5. [Code Review](#code-review)
6. [Testing Before Merging](#testing-before-merging)
7. [CI/CD Pipeline](#cicd-pipeline)
8. [Deployment Process](#deployment-process)

---

## Git Workflow

### Branch Strategy ✅ CURRENT

**Plane uses trunk-based development** with feature branches.

```
main (production)
  ↓
  ├── feature/add-comments
  ├── fix/login-bug
  └── refactor/issue-card
```

### Step-by-Step Workflow

**1. Create a new branch**

```bash
# Make sure you're on main and up-to-date
git checkout main
git pull origin main

# Create feature branch
git checkout -b feature/add-issue-comments

# Branch naming: type/description
# Types: feature, fix, refactor, docs, test, chore
```

**2. Make changes**

```bash
# Edit files
# Run tests
# Ensure code works locally
```

**3. Commit changes**

```bash
# Stage files
git add apps/web/components/comments/
git add apps/api/plane/app/views/comment.py

# Commit with descriptive message
git commit -m "feat: add comment functionality to issues

- Add Comment model with author and content fields
- Create CommentViewSet with CRUD endpoints
- Add CommentList and CommentForm components
- Integrate with issue detail page"
```

**4. Push to remote**

```bash
# First push
git push -u origin feature/add-issue-comments

# Subsequent pushes
git push
```

**5. Create pull request**

```bash
# Using GitHub CLI
gh pr create --title "Add comment functionality to issues" --body "Implements #123"

# Or visit GitHub.com and create PR manually
```

**6. Address review feedback**

```bash
# Make requested changes
# Commit and push
git add .
git commit -m "refactor: extract comment form to separate component"
git push

# PR automatically updates
```

**7. Merge when approved**

```bash
# Using GitHub CLI
gh pr merge --squash

# Or click "Squash and merge" on GitHub
```

---

## Branch Naming

### Naming Convention ✅ CURRENT

```
<type>/<description>

# Examples:
feature/add-comments
fix/login-redirect-bug
refactor/issue-card-component
docs/update-readme
test/add-comment-tests
chore/update-dependencies
```

### Types

| Type | When to Use |
|------|-------------|
| `feature/` | New functionality |
| `fix/` | Bug fixes |
| `refactor/` | Code restructuring (no behavior change) |
| `docs/` | Documentation updates |
| `test/` | Adding or updating tests |
| `chore/` | Maintenance (deps, config) |
| `hotfix/` | Critical production bug |

### Good Branch Names

```bash
✅ feature/issue-comments
✅ fix/authentication-token-expiry
✅ refactor/issue-store-methods
✅ docs/add-api-documentation

❌ my-branch
❌ test
❌ fixes
❌ john-dev
```

---

## Commit Messages

### Conventional Commits ✅ CURRENT

**Format:**

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types

| Type | Description | Example |
|------|-------------|---------|
| `feat` | New feature | `feat: add comment system` |
| `fix` | Bug fix | `fix: resolve login redirect issue` |
| `refactor` | Code restructuring | `refactor: simplify issue store` |
| `docs` | Documentation | `docs: update API guide` |
| `test` | Tests | `test: add comment component tests` |
| `chore` | Maintenance | `chore: update dependencies` |
| `style` | Formatting | `style: fix linting errors` |
| `perf` | Performance | `perf: optimize issue query` |

### Good Commit Messages

```bash
✅ feat: add comment functionality to issues

- Add Comment model with author and content fields
- Create CommentViewSet with CRUD endpoints
- Add CommentList and CommentForm components
- Integrate with issue detail page

Closes #123

✅ fix: resolve authentication token expiry bug

The access token was not being refreshed when expired,
causing 401 errors. Added automatic token refresh logic
to APIService.

Fixes #456

✅ refactor: extract issue priority badge to component

Moved priority badge rendering logic into reusable
IssuePriorityBadge component for better maintainability.
```

### Bad Commit Messages

```bash
❌ updated files
❌ fix bug
❌ changes
❌ WIP
❌ asdfasdf
```

### Tips for Good Commits

**1. Use imperative mood** (like giving a command)

```bash
✅ "Add comment feature"
❌ "Added comment feature"
❌ "Adding comment feature"
```

**2. Keep subject line under 50 characters**

```bash
✅ "feat: add comment system"
❌ "feat: add a comprehensive commenting system with replies, reactions, and notifications"
```

**3. Use body for detailed explanation**

```bash
feat: add comment system

This commit adds a full commenting system for issues:
- Users can add, edit, and delete comments
- Comments support markdown formatting
- Real-time updates via WebSockets
- Email notifications for new comments

Technical details:
- Added Comment model with soft delete
- Created CommentViewSet with permissions
- Implemented CommentList and CommentForm components
- Integrated with existing issue detail page

Closes #123
```

**4. Reference issues in footer**

```bash
Closes #123
Fixes #456
Relates to #789
```

---

## Pull Requests

### PR Title

**Format:** Same as commit messages

```
feat: add comment functionality to issues
fix: resolve authentication token expiry
refactor: simplify issue store logic
```

### PR Description Template

```markdown
## Summary
Brief description of what this PR does.

## Changes
- List of specific changes
- Added Comment model and API endpoints
- Created frontend components
- Updated issue detail page

## Testing
How to test these changes:
1. Create a new issue
2. Click "Add Comment"
3. Enter text and submit
4. Verify comment appears immediately
5. Refresh page and verify comment persists

## Screenshots
[Add screenshots if UI changes]

## Checklist
- [ ] Code follows project conventions
- [ ] Tests added/updated
- [ ] Documentation updated
- [ ] No console errors/warnings
- [ ] Tested locally
- [ ] Backend migrations created (if applicable)

## Related Issues
Closes #123
Relates to #456
```

### Creating a Good PR

**1. Keep PRs small and focused**

```bash
✅ One feature per PR
✅ 50-300 lines changed
✅ Single responsibility

❌ Multiple unrelated features
❌ 2000+ lines changed
❌ "Misc fixes and features"
```

**2. Write clear description**

- What problem does this solve?
- How does it solve it?
- How to test?
- Any caveats or limitations?

**3. Add screenshots/videos for UI changes**

**4. Link related issues**

```markdown
Closes #123
Fixes #456
Relates to #789
```

**5. Request specific reviewers**

```bash
# Using GitHub CLI
gh pr create --reviewer alice,bob
```

---

## Code Review

### For Author (Submitting PR)

**Before requesting review:**

- [ ] Code is complete and working
- [ ] Tests added and passing
- [ ] No console errors
- [ ] Follows project conventions
- [ ] Documentation updated
- [ ] Self-reviewed (read your own code first)
- [ ] Commits are clean and descriptive

**During review:**

- Respond to all comments
- Ask for clarification if needed
- Be open to feedback
- Explain decisions when appropriate
- Make requested changes promptly

### For Reviewer

**What to look for:**

**1. Functionality**
- Does it solve the problem?
- Are there edge cases not handled?
- Could this break existing features?

**2. Code Quality**
- Is it readable?
- Are names clear and descriptive?
- Is it maintainable?
- Are there better patterns?

**3. Performance**
- Are queries optimized?
- Any N+1 query issues?
- Large files downloaded unnecessarily?

**4. Security**
- Input validation?
- Authentication/authorization?
- XSS/SQL injection risks?

**5. Tests**
- Are tests included?
- Do tests cover edge cases?
- Are tests meaningful?

### Review Comments

**Be constructive:**

```markdown
✅ "Consider extracting this logic into a custom hook for reusability:
   `const { comments, isLoading } = useComments(issueId)`"

❌ "This is bad code"

✅ "This query could cause N+1 issues. Suggested fix:
   `Issue.objects.select_related('project', 'state')`"

❌ "Performance issue"

✅ "Nice solution! One small suggestion: we could simplify this by..."

❌ "Wrong"
```

**Use GitHub review features:**

- **Comment** - Ask questions, suggest improvements
- **Approve** - Code is good to merge
- **Request changes** - Must fix before merge

---

## Testing Before Merging

### Pre-Merge Checklist

**1. Run tests locally**

```bash
# Frontend tests
cd apps/web
pnpm test

# Backend tests
cd apps/api
python manage.py test

# Run specific test
python manage.py test plane.app.tests.test_comments
```

**2. Lint code**

```bash
# Frontend
pnpm lint

# Backend
black apps/api/plane
flake8 apps/api/plane
```

**3. Type check**

```bash
# Frontend
pnpm typecheck
```

**4. Manual testing**

- Test happy path (everything works)
- Test error cases (network failures, validation errors)
- Test edge cases (empty states, long text, special characters)
- Test on different browsers (Chrome, Firefox, Safari)
- Test responsiveness (mobile, tablet, desktop)

**5. Check CI/CD**

- All GitHub Actions passing ✅
- No merge conflicts
- Branch up-to-date with main

---

## CI/CD Pipeline

### GitHub Actions Workflow ✅ CURRENT

**Plane uses GitHub Actions for CI/CD**

**On every push:**

```yaml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  frontend-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '22'
      - run: pnpm install
      - run: pnpm test
      - run: pnpm build

  backend-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: '3.12'
      - run: pip install -r requirements.txt
      - run: python manage.py test

  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: pnpm lint
      - run: black --check apps/api/plane
      - run: flake8 apps/api/plane
```

### What CI/CD Checks

- ✅ Tests pass
- ✅ Linting passes
- ✅ Type checking passes
- ✅ Build succeeds
- ✅ No security vulnerabilities

### If CI/CD Fails

**1. Check logs**

```bash
# On GitHub: Click "Details" next to failed check
```

**2. Fix issue locally**

```bash
# Run same commands locally
pnpm test
pnpm lint
```

**3. Commit and push fix**

```bash
git add .
git commit -m "fix: resolve linting errors"
git push
```

**4. CI/CD re-runs automatically**

---

## Deployment Process

### Environments

```
Development (local)
  ↓
Staging (staging.plane.so)
  ↓
Production (plane.so)
```

### Deployment Flow

**1. Merge to main**

```bash
# After PR approval
gh pr merge --squash
```

**2. Automatic deployment to staging**

- GitHub Actions triggers
- Builds Docker images
- Deploys to staging environment
- Runs smoke tests

**3. Test on staging**

```bash
# Visit staging environment
https://staging.plane.so

# Test new feature
# Verify no regressions
```

**4. Deploy to production**

```bash
# Manually trigger production deployment
gh workflow run deploy-production

# Or create release tag
git tag v1.2.3
git push origin v1.2.3
```

### Rollback Procedure

**If deployment fails:**

```bash
# Rollback to previous version
gh workflow run rollback --ref previous-version

# Or revert the merge commit
git revert HEAD
git push origin main
```

---

## Best Practices Summary

### Git
- ✅ Create feature branches from main
- ✅ Keep branches short-lived (< 3 days)
- ✅ Pull main frequently to avoid conflicts
- ✅ Write descriptive commit messages
- ✅ Squash commits when merging

### Pull Requests
- ✅ Keep PRs small (< 300 lines)
- ✅ Write clear descriptions
- ✅ Add screenshots for UI changes
- ✅ Link related issues
- ✅ Respond to reviews promptly

### Code Review
- ✅ Review within 24 hours
- ✅ Be constructive and specific
- ✅ Ask questions, don't demand
- ✅ Approve when ready, request changes when needed

### Testing
- ✅ Write tests for new features
- ✅ Run tests locally before pushing
- ✅ Test manually in browser
- ✅ Verify CI/CD passes

### Deployment
- ✅ Test on staging before production
- ✅ Monitor for errors after deployment
- ✅ Have rollback plan ready

---

## Quick Reference

### Common Commands

```bash
# Start new feature
git checkout main
git pull origin main
git checkout -b feature/my-feature

# Commit changes
git add .
git commit -m "feat: add new feature"

# Push and create PR
git push -u origin feature/my-feature
gh pr create

# Update from main
git checkout main
git pull origin main
git checkout feature/my-feature
git merge main

# Squash commits
git rebase -i HEAD~3

# Fix last commit message
git commit --amend

# Delete branch after merge
git branch -d feature/my-feature
git push origin --delete feature/my-feature
```

---

## Next Steps

### You've Learned Professional Workflow! 🎉

You now understand:
- ✅ Git branching strategy
- ✅ Commit message conventions
- ✅ Pull request process
- ✅ Code review best practices
- ✅ CI/CD pipeline
- ✅ Deployment process

### Continue Learning

1. **[TESTING_GUIDE.md](./TESTING_GUIDE.md)** - Write tests for your features
2. **[DEBUGGING_GUIDE.md](./DEBUGGING_GUIDE.md)** - Debug issues efficiently

---

**You're now ready to contribute professionally to Plane! 🚀**
