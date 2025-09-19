# Git Branching Strategy for Method Development

This guide covers basic Git branching practices for Method development work.

## Overview

Method development follows standard Git practices with feature branches and pull requests. This guide covers the essential branching workflow for Method developers.

## Prerequisites

- Git installed and configured
- SSH keys set up for GitHub
- Repository access configured
- Understanding of basic Git commands

## Basic Branching Workflow

### Main Branch

- **Main development branch:** Usually `main` or `master`
- **Always keep main stable:** Only merge tested, working code
- **Pull latest before starting:** Always start from the latest main branch

### Feature Development

1. **Start from main:**
   ```bash
   git checkout main
   git pull origin main
   ```

2. **Create feature branch:**
   ```bash
   git checkout -b feature/your-feature-name
   ```

3. **Work on your feature:**
   - Make commits as you work
   - Push to remote regularly
   ```bash
   git add .
   git commit -m "Descriptive commit message"
   git push origin feature/your-feature-name
   ```

4. **Create pull request:**
   - Go to GitHub
   - Create PR from your feature branch to main
   - Request reviews from team members

5. **Merge after approval:**
   - Merge via GitHub after approval
   - Delete feature branch after merge

### Bug Fixes

Follow the same process as features but use descriptive branch names:

```bash
git checkout -b bugfix/fix-login-issue
# or
git checkout -b fix/authentication-error
```

## Branch Naming Conventions

Use descriptive, lowercase branch names with hyphens:

- `feature/user-authentication`
- `feature/dashboard-improvements`
- `bugfix/login-validation`
- `fix/memory-leak`
- `hotfix/critical-security-fix`

## Method-Specific Practices

### DeveloperTools Repository

When working with the DeveloperTools repository:
- Follow the same branching strategy
- Test scripts thoroughly before submitting PRs
- Update documentation for script changes

### Multiple Repository Work

When working across multiple Method repositories:
- Keep changes focused to single repositories when possible
- Coordinate with team for cross-repository changes
- Use consistent branch names across related repositories

## Pull Request Guidelines

### Creating Pull Requests

1. **Clear title:** Describe what the PR does
2. **Good description:** Explain the changes and why
3. **Link issues:** Reference any related tickets or issues
4. **Request reviewers:** Add relevant team members

### Code Review Process

- **All changes require review:** No direct pushes to main
- **Address feedback promptly:** Respond to review comments
- **Test before requesting review:** Make sure your code works
- **Keep PRs focused:** Avoid large, multi-purpose PRs

## Common Git Commands

### Daily Development

```bash
# Get latest changes
git pull origin main

# Check status
git status

# See changes
git diff

# Stage and commit
git add .
git commit -m "Clear commit message"

# Push changes
git push origin your-branch-name
```

### Branch Management

```bash
# List branches
git branch -a

# Switch branches
git checkout branch-name

# Delete local branch
git branch -d branch-name

# Delete remote branch
git push origin --delete branch-name
```

## Troubleshooting

### Common Issues

**Merge Conflicts:**
- Resolve conflicts in your editor
- Stage resolved files: `git add filename`
- Complete merge: `git commit`

**Behind Main Branch:**
```bash
git checkout main
git pull origin main
git checkout your-feature-branch
git merge main  # or git rebase main
```

**Accidentally Committed to Main:**
- Create a branch from current state
- Reset main to previous commit
- Push your changes in the new branch

### Getting Help

- **Ask team members** for Git help
- **Use GitHub desktop** if command line is difficult
- **Reference Git documentation** for complex operations

## Best Practices

1. **Commit often:** Small, focused commits are better
2. **Clear messages:** Write descriptive commit messages  
3. **Test before pushing:** Make sure your code works
4. **Keep branches short-lived:** Don't let branches get stale
5. **Update regularly:** Pull latest changes frequently
6. **Delete merged branches:** Clean up after merging

## Integration with Method Setup

### Automated Scripts

The Method DeveloperTools includes scripts that:
- Clone repositories to correct locations
- Switch to master/main branches
- Pull latest changes
- Build all projects

### Repository Organization

Keep your local Method repositories organized:
```
C:\MethodDev\
├── DeveloperTools\
├── method-platform-ui\
├── runtime-core\
└── [other repositories]
```

## Next Steps

After understanding branching strategy:

1. **Learn development workflow:** [Development Workflow](./development-workflow.md)
2. **Start working with repositories:** Clone and set up projects
3. **Practice with small changes:** Make test commits to understand the flow
4. **Collaborate with team:** Join code reviews and collaborate on PRs

**Back to:** [GitHub Setup](./README.md)
