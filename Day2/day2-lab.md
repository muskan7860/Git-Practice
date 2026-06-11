# Day 2 — Branching & Merging Lab

> Senior DevOps Engineer Training | 4 Years Experience Level

---

## Lab Goal

Practice every Day 2 branching command hands on.
Read each step, type the command, see the output,
then move to the next step.

---

## Setup

```bash
# Create a fresh folder for this lab
mkdir day2-lab
cd day2-lab
git init

# Create first commit on main
echo "main app code" > app.js
git add app.js
git commit -m "feat: initial commit on main"
```

---

## Part 1 — git branch

```bash
# Step 1 — see current branches
git branch
# Output: * main

# Step 2 — create a new branch without switching
git branch feature/login

# Step 3 — see all branches
git branch
# Output:
# * main          ← still on main
#   feature/login

# Step 4 — see all branches including remote
git branch -a
```

---

## Part 2 — git checkout and git switch

```bash
# Step 5 — switch to feature branch
git checkout feature/login
# Output: Switched to branch 'feature/login'

# Step 6 — confirm you switched
git branch
# Output:
#   main
# * feature/login   ← now here

# Step 7 — go back to main using dash shortcut
git checkout -
# Output: Switched to branch 'main'

# Step 8 — create and switch in one command
git checkout -b feature/payment
# Output: Switched to a new branch 'feature/payment'

# Step 9 — go back to main
git checkout main

# Step 10 — try modern git switch
git switch feature/login
git switch main
```

---

## Part 3 — Fast Forward Merge

```bash
# Step 11 — go to feature/login branch
git checkout feature/login

# Step 12 — add commits on feature branch
echo "login function" > login.js
git add login.js
git commit -m "feat: add login function"

echo "login validation" >> login.js
git add login.js
git commit -m "feat: add login validation"

# Step 13 — check log
git log --oneline --graph --all
# You see feature/login is 2 commits ahead of main

# Step 14 — switch to main
git checkout main

# Step 15 — merge — should be fast forward
git merge feature/login
# Output: Fast-forward
# No merge commit created

# Step 16 — check log
git log --oneline --graph --all
# main and feature/login point to same commit
# Clean straight line
```

---

## Part 4 — 3-Way Merge

```bash
# Step 17 — create new feature branch
git checkout -b feature/payment

# Step 18 — add commit on feature branch
echo "payment code" > payment.js
git add payment.js
git commit -m "feat: add payment module"

# Step 19 — go back to main and add commit there too
git checkout main
echo "updated main code" >> app.js
git add app.js
git commit -m "fix: update main app code"

# Step 20 — now both branches have new commits
git log --oneline --graph --all
# You see two separate lines diverging

# Step 21 — merge feature/payment into main
git merge feature/payment
# Output: Merge made by recursive strategy
# A merge commit M is created automatically

# Step 22 — check log
git log --oneline --graph --all
# You see the merge commit joining two lines
```

---

## Part 5 — git rebase

```bash
# Step 23 — create new feature branch
git checkout -b feature/dashboard

# Step 24 — add 2 commits on feature branch
echo "dashboard code" > dashboard.js
git add dashboard.js
git commit -m "feat: add dashboard"

echo "dashboard charts" >> dashboard.js
git add dashboard.js
git commit -m "feat: add dashboard charts"

# Step 25 — go to main and add a commit
git checkout main
echo "hotfix on main" >> app.js
git add app.js
git commit -m "hotfix: fix critical bug"

# Step 26 — go back to feature branch
git checkout feature/dashboard

# Step 27 — rebase onto latest main
git rebase main
# Output: Successfully rebased

# Step 28 — check log
git log --oneline --graph --all
# Your 2 commits are now on top of main hotfix
# Clean straight line
```

---

## Part 6 — Merge Conflict

```bash
# Step 29 — create conflict branch
git checkout -b feature/conflict-test

# Step 30 — edit app.js on feature branch
echo "feature version of app" > app.js
git add app.js
git commit -m "feat: change app.js on feature"

# Step 31 — go to main and edit same file
git checkout main
echo "main version of app" > app.js
git add app.js
git commit -m "fix: change app.js on main"

# Step 32 — try to merge — conflict will happen
git merge feature/conflict-test
# Output: CONFLICT (content): Merge conflict in app.js

# Step 33 — open app.js and see conflict markers
cat app.js
# You see:
# <<<<<<< HEAD
# main version of app
# =======
# feature version of app
# >>>>>>> feature/conflict-test

# Step 34 — resolve conflict manually
echo "resolved app code" > app.js

# Step 35 — stage resolved file
git add app.js

# Step 36 — complete the merge
git commit -m "merge: resolve conflict in app.js"

# Step 37 — check log
git log --oneline --graph --all
```

---

## Part 7 — git cherry-pick

```bash
# Step 38 — see all commits
git log --oneline
# Copy SHA of any commit you want to cherry-pick

# Step 39 — create a new branch
git checkout -b feature/cherry-test

# Step 40 — cherry-pick a specific commit
# Replace SHA with actual SHA from your log
git cherry-pick <paste-SHA-here>
# Output: [feature/cherry-test abc1234] <commit message>

# Step 41 — check log
git log --oneline
# Cherry-picked commit is now on this branch
# With a new SHA
```

---

## Part 8 — Delete Branches

```bash
# Step 42 — go back to main
git checkout main

# Step 43 — see all branches
git branch

# Step 44 — delete a merged branch safely
git branch -d feature/login
# Output: Deleted branch feature/login

# Step 45 — try to delete unmerged branch
git branch -d feature/cherry-test
# Output: error — branch not fully merged

# Step 46 — force delete
git branch -D feature/cherry-test
# Output: Deleted branch feature/cherry-test

# Step 47 — clean up remaining branches
git branch -d feature/payment
git branch -d feature/dashboard
git branch -d feature/conflict-test
```

---

## Final Check

```bash
# Step 48 — see final history
git log --oneline --graph --all

# Step 49 — see remaining branches
git branch
# Only main should remain
```

---

## What You Practiced

```
✅ git branch          → created and listed branches
✅ git checkout        → switched branches
✅ git switch          → modern branch switching
✅ fast forward merge  → clean linear merge
✅ 3-way merge         → merge commit created
✅ git rebase          → replayed commits on top
✅ merge conflict      → resolved manually
✅ git cherry-pick     → copied one specific commit
✅ branch deletion     → safe and force delete
```

---
