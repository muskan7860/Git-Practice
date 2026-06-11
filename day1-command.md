# Day 1 — Git Core Commands

> Senior DevOps Engineer Training | 4 Years Experience Level

---

## The 4 Zones — Memorize This First

Every Git command moves files between these 4 zones.
If you understand this, every command makes sense.

```
Working Directory → Staging Area → Local Repo → Remote Repo
(your files)        (git add)       (git commit)  (git push)
```

Working Directory — files you see and edit on your computer.
Staging Area — a review zone before committing. Also called Index.
Local Repo — your saved history stored inside .git/ folder.
Remote Repo — GitHub or GitLab. The shared server everyone pushes to.

Think of it like cooking:
- Working Dir = raw ingredients on your counter
- Staging = plated and ready to serve
- Local Repo = saved in your recipe book
- Remote = shared the recipe with the world

---

## git init

Creates a new Git repository from scratch.
Creates a hidden .git/ folder — this IS the entire Git database.

Real use: You start a brand new microservice from scratch.
In companies you rarely use git init — most repos already
exist on GitHub and you use git clone instead.
But every repo started with git init at some point.

```bash
# Create your project folder
mkdir payment-service
cd payment-service

# Initialize Git
git init
# Output: Initialized empty Git repository in /payment-service/.git/

# Confirm .git folder exists
ls -la
# You will see .git/ listed

# Look inside .git folder
ls .git
# You see: HEAD, config, objects, refs
```

> If you delete .git/ you lose all Git history.
> Your files remain but all tracking is gone forever.

---

## git clone

Downloads a remote repo to your machine with full history.

Real use: You join a new company. First thing you do is
git clone the repo link your manager sends you.

```bash
# HTTPS clone
git clone https://github.com/org/repo.git

# SSH clone — company standard, no password needed
git clone git@github.com:org/repo.git

# Clone into a custom folder name
git clone https://github.com/org/repo.git my-folder

# Shallow clone — used in CI/CD pipelines
# Only downloads latest snapshot, no full history
# A 2GB repo becomes 50MB — huge saving in pipelines
git clone --depth=1 https://github.com/org/repo.git
```

> HTTPS asks for username and password every time.
> SSH is set up once and never asks for password again.
> Companies use SSH as standard.

> --depth=1 is critical DevOps knowledge. In GitHub Actions
> or Jenkins, every pipeline run clones fresh. Without
> shallow clone a 5GB repo = 5 minutes wasted per build.

---

## git add

Moves files from Working Directory to Staging Area.
You choose exactly what goes into your next commit.

Real use: You changed 3 files but only want to commit 2.
Use git add selectively to control what goes in each commit.

```bash
# Stage a single file
git add login.js

# Stage multiple files
git add login.js payment.js

# Stage everything in current directory
git add .

# Stage a whole folder
git add src/

# Stage specific lines inside a file — patch mode
# Senior engineers use this daily
git add -p login.js
# y = stage this chunk
# n = skip this chunk
# q = quit
```

> Always run git status before git add .
> Avoid staging debug logs, temp files, or passwords.
> Use .gitignore to exclude files permanently.

> git add -p is patch mode. It lets you stage specific
> chunks of a file rather than the whole file.
> Use when you made multiple unrelated changes in one
> file and want separate clean commits.

---

## git commit

Saves a permanent snapshot of staged files into local repo.
Each commit gets a unique ID called a SHA hash like 4a7f2e1.

Real use: Every feature, bug fix, or change gets its own
commit with a clear message so teammates understand what
changed and why.

```bash
# Basic commit with message
git commit -m "fix: resolve null pointer in login"

# Commit message format used in companies
# feat     = new feature
# fix      = bug fix
# chore    = cleanup, dependency update
# refactor = code restructure, no behavior change
# docs     = documentation only

# Forgot a file? Add it to the last commit
git add forgot-file.js
git commit --amend --no-edit
# --amend    = modify the LAST commit
# --no-edit  = keep the same message

# Fix a typo in last commit message
git commit --amend -m "fix: correct null pointer in login"
```

> --amend rewrites history — creates new commit with new SHA.
> Only amend commits you have NOT pushed yet.
> If already pushed, use a new commit instead.

---

## git push

Sends your local commits to GitHub so your team can see them.

Real use: You finished a feature locally. You push to GitHub
so teammates can review and CI/CD pipeline can run tests.

```bash
# Basic push
git push origin main

# First time pushing a new branch
# -u sets upstream tracking
# After this, just git push is enough forever
git push -u origin feature/login-fix

# Safe force push — use after amending a commit
git push --force-with-lease origin feature/login-fix
# Checks if anyone pushed after your last fetch
# If yes — aborts to protect their work
# If no  — proceeds with force push

# NEVER use this on shared branches
git push --force origin main
```

> --force overwrites GitHub unconditionally.
> --force-with-lease checks first — always use this.
> Never force push on main or develop.

---

## git pull

Downloads and merges remote changes into your branch.

Real use: Every morning before starting work you pull to
get all changes your teammates pushed overnight.

```bash
# Basic pull — fetch + merge in one command
git pull origin main

# Pull with rebase — cleaner linear history
# Puts YOUR commits on top of theirs
# No merge commit created
git pull --rebase origin main
```

> Many teams set this globally:
> git config --global pull.rebase true
> So every git pull auto-rebases automatically.

---

## git fetch

Downloads changes from remote WITHOUT touching your files.

Real use: You want to see what teammates pushed before
deciding to merge. Fetch first, review, then merge.

```bash
# Fetch from origin
git fetch origin

# Fetch from all remotes
git fetch --all

# After fetching review what changed
git log origin/main --oneline

# Then merge when ready
git merge origin/main
```

> git fetch = download only. git pull = download + merge.
> Senior engineers fetch first, review, then merge.
> Never blindly pull without knowing what changed.

---

## git pull vs git fetch

| Command | Downloads | Merges | Touches Files |
|---------|-----------|--------|---------------|
| git fetch | ✅ | ❌ | ❌ |
| git pull | ✅ | ✅ | ✅ |

```
git pull = git fetch + git merge
```

---

## git status

Shows the state of your working directory and staging area.

Real use: Run this before every git add and git commit.
Make it a reflex — it costs nothing and prevents mistakes.

```bash
# Full status
git status

# Short format
git status -s

# Output explained:
# Changes to be committed  = staged files (green)
# Changes not staged       = modified but not staged (red)
# Untracked files          = new files Git does not know yet
```

> Run git status before every git add and git commit.
> Make it a habit — it costs nothing and saves mistakes.

---

## git log

Views commit history.

Real use: Every morning run git log to see what your
teammates committed overnight and where all branches are.

```bash
# Clean one line per commit — use this daily
git log --oneline --graph --all
# --oneline = one line per commit
# --graph   = shows branch lines visually
# --all     = shows ALL branches not just current

# Last 5 commits only
git log -n 5

# Filter by author
git log --author="Ravi"

# Filter by date
git log --since="2 weeks ago"

# History of a specific file
git log -- path/to/file.js
```

> git log --oneline --graph --all every morning.
> Shows where all branches are, who committed what,
> and what is merged into what.

---

## git diff

Shows exactly what changed in your files.

Real use: Before staging always check what you are
about to commit. Before pushing review your changes.

```bash
# Working directory vs staging area
git diff

# Staging area vs last commit
git diff --staged

# Compare two branches
git diff main feature/login

# Compare 2 commits back to now
git diff HEAD~2 HEAD

# Output explained:
# - red lines  = removed lines
# + green lines = added lines
```

---

## git stash

Temporarily shelves your work in progress.

Real scenario: Production is down. Hotfix needed urgently.
You have half done work you cannot commit yet.
Stash it, fix production, push, then stash pop to continue.

```bash
# Stash with a clear name — always name your stashes
git stash save "WIP: half done login feature"

# See all stashes
git stash list
# Output: stash@{0}: WIP: half done login feature

# Apply latest stash and delete it — most common
git stash pop

# Apply a specific stash and keep it in list
git stash apply stash@{1}

# Delete a specific stash
git stash drop stash@{1}

# Delete ALL stashes
git stash clear
```

> Always name stashes — unnamed stashes are confusing
> when you have multiple saved at the same time.

---

## git tag

Marks specific commits — used for production releases.

Real use: Every production deployment gets a tag.
It is your rollback reference point. If something breaks
you checkout v2.5.0 to go back to the last good release.

```bash
# Annotated tag — always use this for releases
git tag -a v1.0.0 -m "Release 1.0.0 — payment gateway"

# List all tags
git tag

# Tags do NOT push automatically — must push manually
git push origin v1.0.0

# Push all tags at once
git push origin --tags

# Checkout a tag — view that release code
git checkout v1.0.0
# Note: puts you in detached HEAD state

# Delete a tag locally and remotely
git tag -d v1.0.0
git push origin --delete v1.0.0
```

> Semantic versioning — vMAJOR.MINOR.PATCH
> v2.5.0 → v2.5.1 = bug fix (patch)
> v2.5.0 → v2.6.0 = new feature (minor)
> v2.5.0 → v3.0.0 = breaking change (major)

> In CI/CD pipelines pushing a tag triggers automatic
> build, test, and deployment to production.

---

## --force-with-lease vs --force Deep Dive

This is one of the most asked interview questions.
Know the difference completely.

```bash
# --force = overwrites GitHub unconditionally
# Destroys teammates work if they pushed after you
git push --force origin feature/login
# NEVER use this on shared branches

# --force-with-lease = checks first
# Did anyone push to this branch after my last fetch?
# YES → aborts — protects teammates work
# NO  → proceeds — safe to overwrite
git push --force-with-lease origin feature/login
```

The safe workflow before force pushing:

```bash
# Step 1 — fetch fresh from GitHub
git fetch origin

# Step 2 — check if anyone pushed after you
git log origin/feature/login-fix --oneline

# Step 3 — only now force push
git push --force-with-lease origin feature/login-fix
```

> --force-with-lease checks if you FETCHED.
> It does NOT check if you READ what you fetched.
> The git log step is YOU manually reviewing.
> That is the complete safe workflow.

---
