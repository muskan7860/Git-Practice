# Day 3 — Undoing Changes Lab

> Senior DevOps Engineer Training | 4 Years Experience Level

---

## Lab Goal

Practice every Day 3 undo command hands on in terminal.
Read each step, type the command, see the output,
then move to the next step.

---

## Setup

```bash
# Create a fresh folder for this lab
mkdir day3-lab
cd day3-lab
git init
```

---

## Part 1 — Create Commits to Work With

```bash
# Step 1 — create first commit
echo "login code" > login.js
git add login.js
git commit -m "feat: add login"

# Step 2 — create second commit
echo "payment code" > payment.js
git add payment.js
git commit -m "feat: add payment"

# Step 3 — create third commit
echo "broken experiment" > broken.js
git add broken.js
git commit -m "WIP: broken experiment"

# Step 4 — check your history
git log --oneline
# You should see 3 commits:
# abc1234 WIP: broken experiment   ← HEAD is here
# def5678 feat: add payment
# ghi9012 feat: add login
```

---

## Part 2 — git reset --soft

```bash
# Step 5 — undo last commit but keep files staged
git reset --soft HEAD~1

# Step 6 — check status
git status
# Output:
# Changes to be committed:
#   new file: broken.js   ← still staged ✅

# Step 7 — check log
git log --oneline
# WIP commit is gone
# broken.js is still staged and ready

# Step 8 — recommit it properly
git commit -m "feat: add experiment module"

# Step 9 — check log again
git log --oneline
# New clean commit is there
```

---

## Part 3 — git reset --mixed

```bash
# Step 10 — undo last commit, unstage files
git reset --mixed HEAD~1
# OR just: git reset HEAD~1

# Step 11 — check status
git status
# Output:
# Changes not staged for commit:
#   new file: experiment   ← exists but NOT staged (red)

# Step 12 — check log
git log --oneline
# Last commit is gone

# Step 13 — restage and recommit
git add .
git commit -m "feat: recommit after mixed reset"
```

---

## Part 4 — git reset --hard

```bash
# Step 14 — create a bad commit first
echo "terrible broken code" > terrible.js
git add terrible.js
git commit -m "WIP: terrible mistake"

# Step 15 — check log
git log --oneline
# You see 3 commits

# Step 16 — hard reset — DELETE everything
git reset --hard HEAD~1

# Step 17 — check status
git status
# Output: nothing to commit, working tree clean

# Step 18 — check log
git log --oneline
# WIP commit is gone

# Step 19 — confirm file is deleted
ls
# terrible.js does NOT exist
```

---

## Part 5 — git reflog Recovery

```bash
# Step 20 — open the secret diary
git reflog
# Output:
# abc1234 HEAD@{0}: reset: moving to HEAD~1
# def5678 HEAD@{1}: commit: WIP: terrible mistake ← lost
# ghi9012 HEAD@{2}: commit: feat: add login

# Step 21 — copy SHA of lost commit from your output
# It is the SHA next to "commit: WIP: terrible mistake"

# Step 22 — recover your lost commit
git reset --hard <paste-your-SHA-here>

# Step 23 — verify recovery
git log --oneline
# WIP commit is back!

ls
# terrible.js is back!

# Step 24 — clean up by hard resetting again
git reset --hard HEAD~1
```

---

## Part 6 — git revert

```bash
# Step 25 — check current log
git log --oneline
# You should see 2 commits

# Step 26 — revert the last commit
git revert HEAD --no-edit
# Output:
# [main abc1234] Revert "feat: recommit after mixed reset"

# Step 27 — check log
git log --oneline
# You see 3 commits now
# NEW revert commit added on top
# OLD commit still in history ← preserved

# Step 28 — check status
git status
# Output: nothing to commit, working tree clean
```

---

## Part 7 — git restore

```bash
# Step 29 — edit login.js without staging
echo "messy broken edit" >> login.js

# Step 30 — check what changed
git diff login.js
# You see your messy edit

# Step 31 — restore file to last commit state
git restore login.js

# Step 32 — check diff again
git diff login.js
# Nothing. File is clean. Edit is gone.

# Step 33 — practice restore --staged
echo "some change" >> login.js
git add login.js

# Step 34 — check status — file is staged
git status
# Changes to be committed: login.js

# Step 35 — unstage without losing edits
git restore --staged login.js

# Step 36 — check status
git status
# Changes not staged: login.js
# File is unstaged but your edit is still there
```

---

## Part 8 — git clean

```bash
# Step 37 — create some untracked files
touch temp.log debug.txt
echo "build output" > build-output.js

# Step 38 — check status
git status
# Untracked files:
#   temp.log
#   debug.txt
#   build-output.js

# Step 39 — dry run ALWAYS first
git clean -n
# Output:
# Would remove build-output.js
# Would remove debug.txt
# Would remove temp.log
# Nothing deleted yet — just preview

# Step 40 — now actually delete them
git clean -f

# Step 41 — verify
ls
# temp.log, debug.txt, build-output.js are gone

git status
# No untracked files
```

---

## Final Check

```bash
# Step 42 — see complete history
git log --oneline --graph

# Step 43 — confirm clean state
git status
# nothing to commit, working tree clean
```

---

## What You Practiced

```
✅ git reset --soft    → undo commit, files stay staged
✅ git reset --mixed   → undo commit, files unstaged
✅ git reset --hard    → undo commit, files deleted
✅ git reflog          → recover after hard reset
✅ git revert          → safe undo, new commit added
✅ git restore         → discard file edits
✅ git restore --staged → unstage a file
✅ git clean -n        → preview untracked deletions
✅ git clean -f        → delete untracked files
```

---

## Quick Reference — Safe vs Dangerous

```
SAFE on shared branches:
✅ git revert          → always safe, preserves history

SAFE on private branches only:
⚠  git reset --soft   → keep files staged
⚠  git reset --mixed  → keep files unstaged
❌ git reset --hard   → delete everything

FILE LEVEL only:
git restore file       → discard working dir edits
git restore --staged   → unstage a file
git clean -f           → delete untracked files
```

---
