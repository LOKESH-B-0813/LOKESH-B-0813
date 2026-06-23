# Advanced Git Commands Guide

A comprehensive guide to mastering advanced Git commands with practical examples and best practices.

---

## Table of Contents
1. [git stash](#git-stash)
2. [git cherry-pick](#git-cherry-pick)
3. [git revert](#git-revert)
4. [git reset](#git-reset)
5. [Comparison & Decision Tree](#comparison--decision-tree)
6. [Practical Workflow Examples](#practical-workflow-examples)

---

## git stash

### Description
`git stash` saves your uncommitted changes temporarily, allowing you to switch branches or pull updates without committing incomplete work.

### Basic Usage

```bash
# Stash current changes
git stash

# Stash with a descriptive message
git stash save "work in progress on feature X"

# View all stashes
git stash list

# Apply the most recent stash
git stash apply

# Apply a specific stash
git stash apply stash@{2}

# Apply and remove the stash
git stash pop

# Delete a specific stash
git stash drop stash@{1}

# Delete all stashes
git stash clear
```

### Example Scenario

```bash
# You're working on a feature branch
$ git status
modified: src/auth.js

# Your manager asks you to fix a critical bug on main
$ git stash
Saved working directory clean

# Switch to main and fix the bug
$ git checkout main
$ git pull

# After fixing, return to your feature
$ git checkout feature-branch
$ git stash pop
```

### Pro Tips
- Use descriptive messages with `git stash save "message"` to easily identify stashes later
- `git stash list` shows all saved stashes with their indices
- `git stash apply` keeps the stash, while `git stash pop` removes it after applying
- Use `git stash show -p stash@{0}` to preview changes before applying

---

## git cherry-pick

### Description
`git cherry-pick` applies changes from specific commits to your current branch. This is useful when you need to apply only certain commits without merging entire branches.

### Basic Usage

```bash
# Apply a single commit
git cherry-pick <commit-hash>

# Apply multiple commits
git cherry-pick <commit1> <commit2> <commit3>

# Apply a range of commits
git cherry-pick <start-commit>..<end-commit>

# Apply commits and edit them
git cherry-pick -e <commit-hash>

# Continue after resolving conflicts
git cherry-pick --continue

# Abort the cherry-pick
git cherry-pick --abort
```

### Example Scenario

```bash
# You have bugfix on branch 'hotfix' (abc123) that you need on 'main'
$ git log hotfix
commit abc123 - Fix critical authentication bug
commit def456 - Add new feature

# Switch to main and cherry-pick only the bugfix
$ git checkout main
$ git cherry-pick abc123

# If there are conflicts, resolve them
$ git status
# Edit conflicted files
$ git add .
$ git cherry-pick --continue
```

### Advanced Examples

```bash
# Cherry-pick commits from one branch to another
$ git checkout develop
$ git cherry-pick feature-branch~2..feature-branch

# Cherry-pick with automatic commit message editing
$ git cherry-pick -e abc123

# Cherry-pick multiple non-consecutive commits
$ git cherry-pick abc123 def456 ghi789
```

### Pro Tips
- Useful for backporting bug fixes from develop to main without merging all changes
- Great for pulling specific commits across branches
- Always resolve conflicts and test after cherry-picking
- Use `git cherry-pick --abort` if things go wrong

---

## git revert

### Description
`git revert` creates a new commit that undoes previous changes. It's safer than `reset` for shared branches because it preserves history and doesn't rewrite the past.

### Basic Usage

```bash
# Revert a specific commit
git revert <commit-hash>

# Revert without auto-committing (for batch reverts)
git revert -n <commit-hash>

# Revert a range of commits
git revert <oldest>..<newest>

# Continue after resolving conflicts
git revert --continue

# Abort the revert
git revert --abort
```

### Example Scenario

```bash
# You committed a bad deployment (commit xyz789) to main
$ git log --oneline main
xyz789 - Deploy to production with bugs
abc123 - Working feature

# Safely revert the bad commit
$ git revert xyz789
# This creates a new commit that undoes xyz789

# The history is preserved:
$ git log --oneline main
new123 - Revert "Deploy to production with bugs"
xyz789 - Deploy to production with bugs
abc123 - Working feature
```

### Revert vs Reset Comparison

```bash
# REVERT (safe for shared branches) - creates new commit
git revert abc123
# Result: History preserved, safe to push
# Timeline: c1 -> c2 -> c3 -> c3' (undo commit)

# RESET (destructive, use only on personal branches) - rewrites history
git reset --hard abc123
# Result: Commits are removed, history changed
# Timeline: c1 -> c2 (c3 deleted)
```

### Pro Tips
- Always use `revert` on shared/public branches
- It's safe because it doesn't change history
- Creates a clear audit trail of what was undone and when
- Better for production rollbacks

---

## git reset

### Description
`git reset` changes which commit your branch points to. It's powerful but destructive and should only be used on personal branches where you haven't pushed to shared repositories.

### Basic Usage

```bash
# Soft reset - keeps changes staged
git reset --soft <commit>

# Mixed reset (default) - keeps changes unstaged
git reset --mixed <commit>
# or simply: git reset <commit>

# Hard reset - discards all changes
git reset --hard <commit>

# Reset to remote tracking branch
git reset --hard origin/main
```

### Understanding the Three Reset Types

```bash
# Initial state: 3 commits, last one adds "hello world"
$ git log --oneline
c3 - Add hello world
c2 - Add goodbye
c1 - Initial commit

# SOFT RESET (keeps changes, staged)
$ git reset --soft c1
$ git status
Changes to be committed:
  new file: hello.txt
  new file: goodbye.txt

# MIXED RESET (keeps changes, unstaged) - DEFAULT
$ git reset --mixed c1
# or: git reset c1
$ git status
Untracked files:
  hello.txt
  goodbye.txt

# HARD RESET (discards everything)
$ git reset --hard c1
$ git status
On branch main
nothing to commit
# Files are deleted!
```

### Example Scenarios

```bash
# Undo last commit but keep changes staged
$ git reset --soft HEAD~1

# Undo last 3 commits but keep all changes unstaged
$ git reset HEAD~3

# Completely remove last commit (dangerous!)
$ git reset --hard HEAD~1

# Discard all uncommitted changes
$ git reset --hard HEAD

# Undo a git rebase that went wrong
$ git reset --hard ORIG_HEAD

# Go back to a specific commit completely
$ git reset --hard abc123
```

### HEAD Notation

```bash
HEAD      # Current commit
HEAD~1    # Previous commit
HEAD~3    # 3 commits ago
HEAD^     # Parent commit
HEAD~2..HEAD  # Last 3 commits
```

### Pro Tips
- ⚠️ **Never use `reset --hard` on shared branches** - it rewrites history
- Always create a backup branch before doing a hard reset: `git branch backup`
- Use `git reflog` to recover commits after an accidental reset
- On personal branches only, use for cleanup before pushing

---

## Comparison & Decision Tree

### Command Comparison Table

| Command | Use Case | Preserves History | Safe for Shared | Creates Commits |
|---------|----------|-------------------|-----------------|-----------------|
| **stash** | Temporarily save work | Yes | Yes | No |
| **cherry-pick** | Apply specific commits | Yes | Yes | Yes |
| **revert** | Undo published changes | Yes | Yes ✓ | Yes |
| **reset** | Rewrite local history | No | No ✗ | No |

### Decision Tree

```
Need to undo changes?
│
├─ On a shared/public branch?
│  └─ Use: git revert ✓ (SAFE)
│
├─ On a personal/local branch only?
│  └─ Use: git reset ✓ (POWERFUL)
│
├─ Need temporary storage?
│  └─ Use: git stash ✓ (TEMPORARY)
│
└─ Need specific commits from another branch?
   └─ Use: git cherry-pick ✓ (SELECTIVE)
```

### When to Use Each Command

**git stash** - Use when:
- You need to switch branches temporarily
- You want to save work-in-progress
- You're not ready to commit yet

**git cherry-pick** - Use when:
- You need specific commits from another branch
- Backporting critical fixes
- You don't want to merge entire branches

**git revert** - Use when:
- You need to undo commits on shared branches
- You're working on main/production branches
- You want to preserve history

**git reset** - Use when:
- You only work on personal branches
- You haven't pushed yet
- You want to clean up local history

---

## Practical Workflow Examples

### Example 1: Fix Urgent Bug While Working on Feature

```bash
# You're on feature branch, main has a critical bug
$ git status
On branch feature-branch
modified: src/feature.js

# Save your work temporarily
$ git stash
Saved working directory clean

# Switch to main
$ git checkout main
$ git pull origin main

# Create and fix the bug
$ git checkout -b hotfix/urgent-bug
$ # Fix the bug
$ git add src/buggy-file.js
$ git commit -m "Fix critical bug in production"
$ git push origin hotfix/urgent-bug

# Create PR, merge to main, then return to feature
$ git checkout feature-branch
$ git stash pop

# Optional: Get the bugfix in your feature branch
$ git cherry-pick hotfix/urgent-bug

# Continue working
$ # Continue development
```

### Example 2: Sync Feature Branch with Latest Main

```bash
# Your feature branch is behind main by several commits
$ git checkout feature-branch
$ git log --oneline | head -5
f1 - Feature: Add new API
f2 - Feature: Setup base
m1 - Main: Update dependencies
m2 - Main: Security fix

# Option 1: Cherry-pick specific main commits
$ git cherry-pick main~2..main

# Option 2: Rebase feature on main (cleaner)
$ git rebase main

# Option 3: Merge main into your feature
$ git merge main
```

### Example 3: Clean Up Messy Commit History (Local Only)

```bash
# You have 5 messy commits with "WIP" messages
$ git log --oneline | head -5
c5 - WIP: refactoring
c4 - WIP: more changes
c3 - WIP: testing
c2 - Feature: initial
c1 - Initial commit

# Undo 5 commits but keep all changes
$ git reset --soft HEAD~5

# Your changes are all staged
$ git status
Changes to be committed:
  modified: src/file1.js
  modified: src/file2.js
  modified: src/file3.js

# Make one clean commit
$ git commit -m "Implement feature X with comprehensive refactoring"

# Verify history looks clean
$ git log --oneline | head -5
new - Implement feature X with comprehensive refactoring
c1 - Initial commit

# Push with force-with-lease (safer than force)
$ git push --force-with-lease origin feature-branch
```

### Example 4: Rollback a Bad Deployment

```bash
# Bad commit was pushed to main
$ git log --oneline main
abc789 - Deploy version 2.0 (BROKEN!)
abc123 - Deploy version 1.9
...

# Safely revert on a shared branch
$ git checkout main
$ git revert abc789

# A new commit is created that undoes the changes
$ git log --oneline main
new456 - Revert "Deploy version 2.0 (BROKEN!)"
abc789 - Deploy version 2.0 (BROKEN!)
abc123 - Deploy version 1.9

# The revert commit is safe to push
$ git push origin main
```

### Example 5: Recover from Accidental Reset

```bash
# Oh no! You did a hard reset on the wrong commit
$ git reset --hard abc123

# Don't panic! Use git reflog
$ git reflog
xyz789 HEAD@{0}: reset: moving to abc123
def456 HEAD@{1}: commit: Important feature
abc789 HEAD@{2}: pull

# Recover the commit you lost
$ git reset --hard def456

# You're back to normal!
```

---

## Common Scenarios & Solutions

### Scenario: "I committed to the wrong branch"

```bash
# You committed to main instead of feature-branch
$ git reset --soft HEAD~1      # Undo the commit, keep changes
$ git checkout feature-branch
$ git commit -m "Feature message"
$ git push origin feature-branch
$ git checkout main
$ git pull
```

### Scenario: "I need to undo a published commit"

```bash
# On a shared branch, use revert (NOT reset)
$ git revert <commit-hash>
$ git push origin main
# Never push after reset on shared branches!
```

### Scenario: "I have too many stashes, which one is mine?"

```bash
$ git stash list
stash@{0}: WIP on feature-branch: abc123 working on X
stash@{1}: WIP on main: def456 debugging Y
stash@{2}: On feature-branch: message I gave it

# View what's in a specific stash
$ git stash show -p stash@{2}

# Apply the one you want
$ git stash apply stash@{2}
```

### Scenario: "I want to cherry-pick multiple commits at once"

```bash
# Get all commits from feature-branch since it branched from main
$ git checkout main
$ git cherry-pick main..feature-branch

# Or specific range
$ git cherry-pick abc123..def456
```

---

## Safety Tips & Best Practices

### ✅ DO
- ✓ Use `revert` on shared/public branches
- ✓ Use `stash` to temporarily save work
- ✓ Test after cherry-picking
- ✓ Create backup branches: `git branch backup-name`
- ✓ Use `--force-with-lease` instead of `--force`
- ✓ Review changes before hard reset: `git diff HEAD~1`

### ❌ DON'T
- ✗ Use `reset --hard` on shared branches
- ✗ Force push without good reason
- ✗ Use `reset` instead of `revert` on main/production
- ✗ Assume cherry-picked code works without testing
- ✗ Delete stashes without reviewing their content
- ✗ Use `--force` instead of `--force-with-lease`

---

## Git Reflog - Your Safety Net

```bash
# See all recent actions on your repo
$ git reflog

# Recover from any mistake
$ git reset --hard <reflog-entry>

# Example:
$ git reflog
abc123 HEAD@{0}: reset: moving to HEAD~2
def456 HEAD@{1}: commit: important work
xyz789 HEAD@{2}: pull origin main

# Oh no! I reset too far, recover my commit
$ git reset --hard def456
```

---

## Quick Reference

### Common Commands

```bash
# Temporary save
git stash / git stash pop

# Apply specific commit
git cherry-pick <hash>

# Undo safely
git revert <hash>

# Undo locally
git reset --soft/--mixed/--hard <hash>

# View recent actions
git reflog

# See what changed
git diff HEAD~1
```

### Useful Aliases (add to ~/.gitconfig)

```bash
[alias]
    undo = reset --soft HEAD~1
    amend = commit --amend --no-edit
    recent = log --oneline -10
    branches = branch -a
    cleanup = reset --soft HEAD~
```

---

## Conclusion

Mastering these advanced Git commands gives you powerful control over your version control workflow. Remember:

- **Shared branches**: Use `revert`, `cherry-pick`, and `stash`
- **Personal branches**: Use `reset`, `cherry-pick`, and `stash`
- **Always test** after rewriting history
- **Use reflog** to recover from mistakes
- **Ask for help** if unsure

Happy Git-ing! 🚀
