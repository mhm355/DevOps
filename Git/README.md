# Git Version Control

Distributed version control system for tracking changes in code.

## Configuration

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
```

---

## Basic Commands

```bash
# Initialize & Clone
git init                           # Create new repository
git clone <url>                    # Clone remote repository

# Stage & Commit
git status                         # Check working tree status
git add <file>                     # Stage specific file
git add .                          # Stage all changes
git commit -m "message"            # Commit staged changes

# Push & Pull
git push origin <branch>           # Push to remote
git pull origin <branch>           # Fetch and merge from remote
git fetch                          # Fetch without merging
```

---

## Branching

```bash
git branch                         # List branches
git branch <name>                  # Create branch
git checkout <branch>              # Switch branch
git checkout -b <name>             # Create and switch
git merge <branch>                 # Merge branch into current
git branch -d <name>               # Delete branch
```

---

## Branching Strategies

### GitFlow
```
main ──────────────────────────────────→
       ↑                    ↑
develop ─────┬──────────────┴───────→
             │
feature/xxx ─┘
```

| Branch | Purpose |
|--------|---------|
| `main` | Production-ready code |
| `develop` | Integration branch |
| `feature/*` | New features |
| `release/*` | Release preparation |
| `hotfix/*` | Production fixes |

### Trunk-Based Development
- Single `main` branch
- Short-lived feature branches (< 1 day)
- Feature flags for incomplete features
- Better for CI/CD

---

## Undoing Changes

```bash
git restore <file>                 # Discard working changes
git restore --staged <file>        # Unstage file
git reset HEAD~1                   # Undo last commit (keep changes)
git reset --hard HEAD~1            # Undo last commit (discard changes)
git revert <commit>                # Create undo commit
```

---

## Stashing

```bash
git stash                          # Save uncommitted changes
git stash list                     # List stashes
git stash pop                      # Apply and remove latest stash
git stash drop                     # Remove latest stash
```

---

## Git Hooks

Located in `.git/hooks/` - scripts that run on git events.

| Hook | Trigger |
|------|---------|
| `pre-commit` | Before commit |
| `commit-msg` | Validate commit message |
| `pre-push` | Before push |
| `post-merge` | After merge |

Example `pre-commit`:
```bash
#!/bin/sh
npm run lint
```

---

## .gitignore

```gitignore
# Dependencies
node_modules/
venv/
.pip_cache/

# Build outputs
target/
dist/
build/

# IDE
.idea/
.vscode/

# Environment
.env
*.log

# OS
.DS_Store
Thumbs.db
```

---

## Useful Commands

```bash
git log --oneline -10              # Compact history
git diff                           # Show unstaged changes
git blame <file>                   # Show who changed each line
git reflog                         # Recovery history
git cherry-pick <commit>           # Apply specific commit
```
