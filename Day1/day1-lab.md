# Day 1 — Git Core Commands Lab

> Senior DevOps Engineer Training | 4 Years Experience Level

---

## Lab Goal

Practice every Day 1 command hands on in your terminal.
Read each step, type the command, see the output,
then move to the next step.

---

## Setup

```bash
# Create a fresh folder for this lab
mkdir day1-lab
cd day1-lab
```

---

## Part 1 — git init

```bash
# Step 1 — initialize a new Git repo
git init

# Step 2 — confirm .git folder was created
ls -la
# You should see .git/ listed

# Step 3 — look inside .git folder
ls .git
# You see: HEAD, config, objects, refs
# This hidden folder IS your entire Git database
```

---

## Part 2 — git status and first file

```bash
# Step 4 — create your first file
echo "Hello World" > app.js

# Step 5 — check status
git status
# Output:
# Untracked files:
#   app.js   ← Git sees it but does not track it yet
```

---

## Part 3 — git add

```bash
# Step 6 — stage the file
git add app.js

# Step 7 — check status again
git status
# Output:
# Changes to be committed:
#   new file: app.js   ← now staged (green)

# Step 8 — create another file
echo "login code" > login.js

# Step 9 — stage everything at once
git add .

# Step 10 — check status
git status
# Both files should be staged (green)
```

---

## Part 4 — git commit

```bash
# Step 11 — make your first commit
git commit -m "feat: initial app setup"
# Output:
# [main abc1234] feat: initial app setup
# 2 files changed

# Step 12 — add more code and commit again
echo "console.log('login')" >> app.js
git add app.js
git commit -m "feat: add login log"

# Step 13 — add one more commit
echo "payment code" > payment.js
git add payment.js
git commit -m "feat: add payment module"
```

---

## Part 5 — git log

```bash
# Step 14 — see full history
git log

# Step 15 — see clean one line history
git log --oneline --graph --all
# You should see your 3 commits listed

# Step 16 — see last 2 commits only
git log -n 2
```

---

## Part 6 — git diff

```bash
# Step 17 — make a change without staging
echo "new line" >> app.js

# Step 18 — see what changed
git diff
# Red lines = removed, Green lines = added

# Step 19 — stage it and check staged diff
git add app.js
git diff --staged
# Shows staged changes vs last commit
```

---

## Part 7 — git commit --amend

```bash
# Step 20 — commit but forget a file
echo "utils code" > utils.js
git add app.js
git commit -m "feat: update app and utils"
# Oops — forgot to stage utils.js

# Step 21 — add forgotten file and amend
git add utils.js
git commit --amend --no-edit
# Last commit now includes utils.js

# Step 22 — verify
git log --oneline
# Same commit count but SHA changed
```

---

## Part 8 — git stash

```bash
# Step 23 — make some changes without committing
echo "work in progress" >> app.js

# Step 24 — stash with a name
git stash save "WIP: adding new feature"

# Step 25 — verify working directory is clean
git status
# Output: nothing to commit, working tree clean

# Step 26 — see your stashes
git stash list
# Output: stash@{0}: WIP: adding new feature

# Step 27 — restore your stash
git stash pop

# Step 28 — verify your changes are back
git status
# app.js appears as modified again
```

---

## Part 9 — git tag

```bash
# Step 29 — create an annotated tag
git tag -a v1.0.0 -m "First release"

# Step 30 — list all tags
git tag
# Output: v1.0.0

# Step 31 — see tag details
git show v1.0.0

# Step 32 — delete the tag
git tag -d v1.0.0
```

---

## Part 10 — git clone (separate exercise)

```bash
# Step 33 — go to parent directory
cd ..

# Step 34 — clone a public repo
git clone https://github.com/github/gitignore.git

# Step 35 — go inside cloned repo
cd gitignore

# Step 36 — check history
git log --oneline -5
# You see the last 5 commits from GitHub

# Step 37 — come back to your lab
cd ../day1-lab
```

---

## Final Check

```bash
# Step 38 — see your complete history
git log --oneline --graph --all

# Step 39 — check working tree is clean
git status
# nothing to commit, working tree clean
```

---

## What You Practiced

```
✅ git init        → created repo from scratch
✅ git status      → checked file states
✅ git add         → staged files
✅ git commit      → saved snapshots
✅ git log         → viewed history
✅ git diff        → saw changes
✅ git amend       → fixed last commit
✅ git stash       → shelved WIP work
✅ git tag         → marked a release
✅ git clone       → downloaded remote repo
```

---
