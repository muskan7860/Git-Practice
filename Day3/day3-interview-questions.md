# Day 3 — Undoing Changes Interview Questions

> Senior DevOps Engineer Training | 4 Years Experience Level

---

### Q1. Explain the difference between git reset --soft, --mixed, and --hard. When would you use each one?

git reset is used to undo commits. There are 3 types.

`git reset --soft HEAD~1` undoes the last commit but keeps
all file changes staged and ready. When I run git status
I see green — Changes to be committed. I use this when I
committed too early and want to add more files to the same
commit or fix the commit message.

```bash
git reset --soft HEAD~1
# git status shows:
# Changes to be committed:
#   modified: payment.js   ← still staged ✅
```

`git reset --mixed HEAD~1` undoes the last commit and moves
files back to working directory unstaged. git status shows
red — Changes not staged. I need git add again before
committing. This is the DEFAULT — git reset HEAD~1 without
any flag applies mixed automatically. I use this when I
want to split one big commit into multiple smaller commits.

```bash
git reset --mixed HEAD~1
# git status shows:
# Changes not staged for commit:
#   modified: payment.js   ← unstaged, needs git add
```

`git reset --hard HEAD~1` undoes the last commit AND
permanently deletes all file changes. Working directory
becomes completely clean. I use this only on my own private
feature branches when I want to throw away broken work.

```bash
git reset --hard HEAD~1
# git status shows:
# nothing to commit, working tree clean
# All files from that commit are gone
```

If I use --hard by mistake I can recover using git reflog.
Git secretly keeps a diary of every move for 90 days.

```bash
git reflog
# abc1234 HEAD@{1}: commit: WIP: broken experiment
git reset --hard abc1234
# Lost commit and files are recovered
```

I never use reset on shared branches like main or develop
because it rewrites history and breaks teammates local
copies. On shared branches I always use git revert instead.

---
### Q2. A bad commit was pushed to main 2 days ago. 10 engineers already pulled it. Production is affected. What do you do?

I would never use git reset here because 10 teammates
already pulled the bad commit. If I reset and force push,
GitHub history changes but teammates still have the old
commit on their laptops. Their history and GitHub history
diverge. When they push or pull they get conflicts. The
whole team gets blocked — making a bad situation worse.

The correct command is git revert.

git revert does not delete the bad commit. It creates a
brand new commit that does the exact opposite — reversing
all the bad commit's changes. History is preserved.

```bash
# Step 1 — find the bad commit
git log --oneline
# 4a7f2e1 bad feature that broke production ← this one

# Step 2 — revert it
git revert 4a7f2e1 --no-edit

# Step 3 — push normally — no force push needed
git push origin main
```

All 10 teammates simply run git pull and get the revert
commit. No conflicts. No chaos. Production is fixed.

History after revert:

```
Commit E → Revert: bad feature   ← fixes production
Commit D → bad feature           ← still in history
Commit C → feat: payment
Commit B → feat: login
```

Commit D is still in history but Commit E cancels all
its changes. This is important for audit trails.

```
reset  = rewrites history = dangerous on shared branches
revert = preserves history = always safe on shared branches
```

In production incidents I always use git revert.

---
### Q3. You edited login.js and made a mess. Nothing staged or committed yet. You want to throw away all edits and go back to last commit. What do you do?

I use git restore.

```bash
git restore login.js
```

Git takes login.js exactly as it was in my last commit
and overwrites my current edited version. All my messy
edits are thrown away permanently.

Before running restore I always check what I am about
to lose:

```bash
# See exactly what will be thrown away
git diff login.js
# Red lines = what will be deleted

# Then restore if I am sure
git restore login.js
```

After restore:

```bash
git status
# login.js does not appear — file is clean
```

Important warning — git restore on working directory
is permanent. There is no reflog for this. Once restored,
edits are gone forever. Always run git diff first.

git restore also unstages files:

```bash
# Accidentally staged a file, want to unstage it
git restore --staged login.js
# File removed from staging area
# Edits still in working directory — not lost
```

git restore is the modern replacement for:

```
git restore login.js          → git checkout -- login.js
git restore --staged login.js → git reset HEAD login.js
```

---
### Q4. You ran git reset --hard HEAD~1 by mistake and lost 2 hours of work. How do you recover it?

Even after git reset --hard my work is not truly gone.
Git has a hidden safety net called git reflog.

reflog is a secret diary Git keeps of every single HEAD
movement — every commit, reset, checkout, merge. It keeps
this diary for 90 days independently of branch history.
Even though --hard removed the commit from my branch,
reflog still has a record of it.

Step 1 — open reflog:

```bash
git reflog
# Output:
# 9c8d7e6 HEAD@{0}: reset: moving to HEAD~1
# 4a7f2e1 HEAD@{1}: commit: feat: 2 hours of work ← this
# 3b2a1c4 HEAD@{2}: commit: feat: add login
```

Step 2 — identify the lost commit. Look for the line
that says commit and matches what was lost.
Copy that SHA — 4a7f2e1.

Step 3 — recover it:

```bash
git reset --hard 4a7f2e1
```

Step 4 — verify recovery:

```bash
git log --oneline
# 4a7f2e1 feat: 2 hours of work   ← back!

ls
# All files are back
```

2 hours of work recovered in under 30 seconds.

Why reflog works after --hard:

```
--hard removes commit from branch history only.
reflog is a completely separate hidden log that
tracks every HEAD movement independently.
--hard affects branch history. Not reflog.
That is why recovery is always possible within 90 days.
```

---
### Q5. What is the difference between git revert and git reset? Why should you never use reset on shared branches?

git reset and git revert both undo changes but in
completely different ways.

git reset moves the branch pointer backwards and rewrites
history. The commit is removed from history completely.

git revert creates a brand new commit that does the exact
opposite of the bad commit. History is preserved. The bad
commit stays but a new commit on top cancels its changes.

Real disaster scenario with reset on main:

```
GitHub main:
Commit D → bad feature    ← broke production
Commit C → feat: payment
Commit B → feat: login

10 teammates already pulled — they all have commit D.

I run:
git reset --hard HEAD~1
git push --force origin main

GitHub main now:
Commit C → feat: payment
Commit B → feat: login
Commit D is GONE from GitHub

But all 10 teammates still have commit D locally.
Their history and GitHub history are now different.

Teammate tries to push → REJECTED
Teammate tries to pull → CONFLICTS
All 10 engineers are blocked.
Production still broken AND team is blocked.
One reset command created a company wide crisis.
```

The correct solution:

```bash
git revert HEAD --no-edit
git push origin main
```

All 10 teammates simply run git pull and get the revert
commit. No conflicts. No blocked engineers. Everyone
continues working normally.

```
reset  = rewrites history = force push needed
         breaks every teammate's local copy
         NEVER use on main or develop

revert = preserves history = normal push works
         teammates just git pull
         ALWAYS safe on shared branches
```

The rule I follow:
If commit is only on my private feature branch that
nobody else has pulled → reset is fine.
If commit is on a shared branch others have pulled
→ revert is the only safe option. Always.

---
### Q6. You edited login.js and made a mess. Nothing staged or committed yet. You want to throw away all edits and go back to last commit. What do you do?

I use git restore.

```bash
git restore login.js
```

Git takes login.js exactly as it was in my last commit
and overwrites my current edited version. All my messy
edits are thrown away permanently.

Before running restore I always check what I am about
to lose:

```bash
# See exactly what will be thrown away
git diff login.js
# Red lines = what will be deleted

# Then restore if I am sure
git restore login.js
```

After restore:

```bash
git status
# login.js does not appear — file is clean
```

Important warning — git restore on working directory
is permanent. There is no reflog for this. Once restored,
edits are gone forever. Always run git diff first.

git restore also unstages files:

```bash
# Accidentally staged a file, want to unstage it
git restore --staged login.js
# File removed from staging area
# Edits still in working directory — not lost
```

git restore is the modern replacement for:

```
git restore login.js           → git checkout -- login.js
git restore --staged login.js  → git reset HEAD login.js
```

---

### Q7. Your build process created hundreds of temp files and log files. Git does not know about them. You want to delete all of them and get back to a clean state. What do you do?

I use git clean.

git clean permanently deletes untracked files — files
that were never added to Git with git add. Git does not
know about them at all.

The most important rule — always run dry run first:

```bash
# Step 1 — dry run, preview what will be deleted
# Nothing is deleted yet — just showing you
git clean -n
# Output:
# Would remove temp.log
# Would remove debug.txt
# Would remove build/output.js
```

After reviewing the list and confirming these are safe
to delete:

```bash
# Step 2 — actually delete untracked files
# -f = force, Git requires this as a safety measure
git clean -f
# Output:
# Removing temp.log
# Removing debug.txt
# Removing build/output.js
```

If there are also untracked folders to delete:

```bash
# -f = force, -d = include untracked directories
git clean -fd
```

If build artifacts ignored by .gitignore also need
to be deleted:

```bash
# -x = also delete files ignored by .gitignore
git clean -fdx
```

Verify everything is clean:

```bash
git status
# nothing to commit, working tree clean
# No untracked files anywhere
```

Critical warning — git clean permanently deletes files
that were never committed. There is absolutely no
recovery. No reflog. No trash bin. No undo.

```
Always run git clean -n first.
Never skip the dry run step.
The 5 seconds it takes to preview
can save hours of lost work.
```

---
