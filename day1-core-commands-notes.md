# Day 1 — Git Core Commands Notes

> Senior DevOps Engineer Training | 4 Years Experience Level

---

## The 4 Zones

```
Working Directory → Staging Area → Local Repo → Remote Repo
(your files)        (git add)       (git commit)  (git push)
```

---

## git init

```bash
mkdir payment-service
cd payment-service
git init
ls -la
```

> Creates hidden .git/ folder — this IS the repo.
> Delete .git/ and you lose all history.

---

## git clone

```bash
# HTTPS
git clone https://github.com/org/repo.git

# SSH — company standard
git clone git@github.com:org/repo.git

# Custom folder name
git clone https://github.com/org/repo.git my-folder

# Shallow clone — for CI/CD pipelines
git clone --depth=1 https://github.com/org/repo.git
```

> SSH = set up once, never type password again.
> --depth=1 = only latest snapshot, saves time in pipelines.

---

## git add

```bash
# Single file
git add login.js

# Multiple files
git add login.js payment.js

# Everything
git add .

# Specific folder
git add src/

# Specific lines inside a file
git add -p login.js
# y = stage this chunk
# n = skip
# q = quit
```

> Always run git status before git add .
> Use .gitignore to exclude files permanently.

---

## git commit

```bash
# Basic commit
git commit -m "fix: resolve null pointer in login"

# Message format
# feat     = new feature
# fix      = bug fix
# chore    = cleanup
# refactor = restructure, no behavior change
# docs     = documentation only

# Add forgotten file to last commit
git add forgot-file.js
git commit --amend --no-edit

# Fix last commit message
git commit --amend -m "fix: correct message"
```

> --amend rewrites history — new SHA created.
> Only amend commits NOT yet pushed.

---

## git push

```bash
# Basic push
git push origin main

# First push of a new branch
git push -u origin feature/login-fix

# Safe force push — after amending
git push --force-with-lease origin feature/login-fix

# NEVER use on shared branches
git push --force origin main
```

> --force-with-lease checks if anyone pushed after you.
> If yes — aborts. If no — proceeds.
> Never --force on main or develop.

---

## git pull

```bash
# Basic — fetch + merge
git pull origin main

# Rebase — cleaner linear history
git pull --rebase origin main
```

> git pull --rebase puts YOUR commits on top of theirs.
> No merge commit. Clean straight line history.

---

## git fetch

```bash
# Fetch only — does NOT touch your files
git fetch origin

# Fetch all remotes
git fetch --all

# Review what changed
git log origin/main --oneline

# Merge when ready
git merge origin/main
```

> fetch = download only. pull = download + merge.
> Senior engineers fetch first, review, then merge.

---

## git pull vs git fetch

| Command | Downloads | Merges | Touches Files |
|---------|-----------|--------|---------------|
| git fetch | ✅ | ❌ | ❌ |
| git pull | ✅ | ✅ | ✅ |

---

## git status

```bash
# Full status
git status

# Short format
git status -s
```

> Green = staged. Red = not staged. Untracked = new file.
> Run before every git add and git commit.

---

## git log

```bash
# Clean visual log — use this daily
git log --oneline --graph --all

# Last 5 commits
git log -n 5

# By author
git log --author="Ravi"

# By date
git log --since="2 weeks ago"

# Specific file history
git log -- path/to/file.js
```

---

## git diff

```bash
# Working dir vs staging
git diff

# Staging vs last commit
git diff --staged

# Two branches
git diff main feature/login

# Two commits
git diff HEAD~2 HEAD
```

> Red lines = removed. Green lines = added.

---

## git stash

```bash
# Stash with name — always name it
git stash save "WIP: half-done login feature"

# List all stashes
git stash list

# Apply latest and delete it
git stash pop

# Apply specific stash, keep it
git stash apply stash@{1}

# Delete one stash
git stash drop stash@{1}

# Delete all stashes
git stash clear
```

> Real use: Production is down. Stash WIP, fix prod,
> push hotfix, then git stash pop to continue.

---

## git tag

```bash
# Annotated tag — use for releases
git tag -a v1.0.0 -m "Release 1.0.0"

# List all tags
git tag

# Push one tag
git push origin v1.0.0

# Push all tags
git push origin --tags

# Checkout a tag
git checkout v1.0.0

# Delete tag locally and remotely
git tag -d v1.0.0
git push origin --delete v1.0.0
```

> Semantic versioning — vMAJOR.MINOR.PATCH
> Patch = bug fix. Minor = new feature. Major = breaking change.
> CI/CD pipelines trigger deployments on tag push.

---
