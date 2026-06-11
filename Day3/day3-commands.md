# Day 3 — Undoing Changes

> Senior DevOps Engineer Training | 4 Years Experience Level

---

## The Most Important Concept Before Any Command

In Git, undoing means one of 3 things:

```
1. Undo a commit but KEEP your file changes
   → you want to recommit differently

2. Undo a commit AND delete your file changes
   → you want to start completely fresh

3. Create a NEW commit that reverses a previous commit
   → safe for shared branches, history preserved
```

Every command in this topic falls into one of these 3
categories. Once you know which category a command belongs
to, you will never confuse them again.

---

## The 3 Zones Reminder

```
Working Directory → Staging Area → Local Repo → Remote
(your files)        (git add)       (git commit)  (git push)
```

Every undo command affects one or more of these zones.
This is the key to understanding which command to use.

```
reset --soft  → undoes commit only
                Staging Area keeps changes  ✅
                Working Dir keeps changes   ✅

reset --mixed → undoes commit
                Staging Area cleared        ↩
                Working Dir keeps changes   ✅

reset --hard  → undoes commit
                Staging Area cleared        ❌
                Working Dir DELETED         ❌

git revert    → does NOT touch any zone
                creates NEW commit that reverses changes
```

---

## git reset --soft

git reset --soft undoes your last commit but keeps all
your file changes staged and ready. Think of it as
peeling off the commit label while keeping everything
inside safe and prepared for recommitting.

Real scenario: You committed too early. The code works
but you forgot to add 2 more files to the same commit.
You want to undo just the commit, keep changes staged,
add missing files, and recommit properly.

```
BEFORE reset --soft:
Commit C → "feat: add payment"   ← HEAD is here
Commit B → "feat: add login"
Commit A → "initial commit"

Working Dir:  payment.js  (your code)
Staging Area: empty

AFTER git reset --soft HEAD~1:
Commit B → "feat: add login"     ← HEAD moved back here
Commit A → "initial commit"
(Commit C is gone)

Working Dir:  payment.js  (still here ✅)
Staging Area: payment.js  (back to staged ✅)
```

```bash
# Undo last 1 commit, keep changes staged
git reset --soft HEAD~1
# HEAD~1 = go back 1 commit from current position
# HEAD~2 = go back 2 commits
# HEAD~3 = go back 3 commits

# Verify — changes are still staged
git status
# Output:
# Changes to be committed:
#   modified: payment.js   ← still staged ✅

# Add missing files and recommit cleanly
git add missing-file.js
git commit -m "feat: add complete payment module"
```

> --soft is the safest reset. Nothing is deleted.
> Your work is fully preserved and staged.
> Use this when you committed too early or want to
> change what is included in the commit.

---

## git reset --mixed

git reset --mixed undoes your last commit and moves your
files back to the working directory unstaged. Your files
are safe but you need to run git add again before
committing. This is the DEFAULT reset — if you run
git reset HEAD~1 without any flag, mixed is applied
automatically.

Real scenario: You committed 5 files together but they
should have been 3 separate commits. You undo the commit,
all 5 files come back unstaged. Now you stage and commit
them separately and cleanly.

```
BEFORE reset --mixed:
Commit C → "feat: add payment"   ← HEAD is here
Staging Area: empty
Working Dir:  payment.js

AFTER git reset --mixed HEAD~1:
Commit B → "feat: add login"     ← HEAD moved back
Staging Area: empty ↩ cleared
Working Dir:  payment.js (still here ✅ but unstaged)
```

```bash
# Undo last commit, unstage changes, keep files
git reset --mixed HEAD~1

# OR just — --mixed is the default
git reset HEAD~1

# Verify — files exist but are unstaged (red)
git status
# Output:
# Changes not staged for commit:
#   modified: payment.js   ← exists but not staged

# Now stage and commit separately and cleanly
git add payment-form.js
git commit -m "feat: add payment form"

git add payment-validation.js
git commit -m "feat: add payment validation"
```

> --mixed is the default reset.
> git reset HEAD~1 without any flag = --mixed automatically.
> Files are safe but staging is cleared.
> You need git add again before committing.

---

## git reset --hard

git reset --hard undoes your last commit AND permanently
deletes all your file changes. Your working directory
becomes completely clean — exactly as it was at the
previous commit. This is the most dangerous Git command.

Real scenario: You committed broken experimental code on
your private feature branch. You want to completely throw
it away and go back to the last known good state.
Everything since that commit is deleted permanently.

```
BEFORE reset --hard:
Commit C → "WIP: broken experiment"  ← HEAD here
Commit B → "feat: add login"

Working Dir:  broken-code.js (exists)
Staging Area: some-file.js (staged)

AFTER git reset --hard HEAD~1:
Commit B → "feat: add login"  ← HEAD moved back

Working Dir:  broken-code.js  ❌ GONE FOREVER
Staging Area: some-file.js    ❌ GONE FOREVER
```

```bash
# Go back 1 commit and DELETE all changes
git reset --hard HEAD~1

# Go back to a specific commit SHA
git reset --hard 4a7f2e1

# Discard ALL uncommitted changes right now
# HEAD = current commit, resets to clean state
git reset --hard HEAD

# Verify — everything is gone
git status
# Output: nothing to commit, working tree clean

ls
# Files from deleted commit do not exist
```

> NEVER use --hard on shared branches like main or develop.
> Only use on your own private feature branches.
> Always double check you are on the right branch first.
> git branch to confirm before running --hard.

---

## git reflog — Recovery After reset --hard

reflog is a secret diary Git keeps of every single HEAD
movement — every commit, reset, checkout, rebase, merge.
Git keeps this diary for 90 days completely independently
of branch history. Even after reset --hard your lost
commits are recoverable using reflog.

Why reflog works after --hard:
reset --hard removes the commit from branch history only.
reflog is a completely separate hidden log that tracks
every HEAD movement independently. These are two
different systems in Git. --hard affects one, not the
other.

```bash
# Open the secret diary
git reflog
# Output:
# 9c8d7e6 HEAD@{0}: reset: moving to HEAD~1
# 4a7f2e1 HEAD@{1}: commit: WIP: broken experiment ← lost
# 3b2a1c4 HEAD@{2}: commit: feat: add login

# Find your lost commit — look for the line that says
# "commit" and matches what you lost
# Copy the SHA next to it — 4a7f2e1

# Recover your lost commit
git reset --hard 4a7f2e1

# Verify recovery
git log --oneline
# 4a7f2e1 WIP: broken experiment   ← back!

ls
# All your files are back
```

> reflog is your safety net after reset --hard.
> Git keeps every HEAD movement for 90 days.
> Even after --hard you can recover using reflog.
> Always try reflog before panicking about lost work.

---

## git revert

git revert is the safe way to undo changes on shared
branches. It does NOT delete or rewrite any history.
Instead it creates a brand new commit that does the
exact opposite of the commit you want to undo. The
original bad commit stays in history but the new revert
commit cancels all its changes.

Real scenario: A bad commit was pushed to main 2 days
ago. 10 teammates already pulled it. Production is
affected. You cannot use reset because rewiring history
would break all 10 teammates local copies. You use
revert — it creates a new commit reversing the damage.
Teammates simply git pull and get the fix.

```
BEFORE revert:
Commit D → "bad feature"    ← broke production
Commit C → "feat: payment"
Commit B → "feat: login"

AFTER git revert HEAD:
Commit E → "Revert: bad feature"  ← NEW commit
Commit D → "bad feature"          ← still in history
Commit C → "feat: payment"
Commit B → "feat: login"

Commit E cancels everything Commit D did.
History preserved. Everyone is safe.
```

```bash
# Revert the last commit
git revert HEAD --no-edit
# --no-edit = use default message, skip editor
# Output:
# [main e8f9a2b] Revert "bad feature that broke prod"

# Revert a specific commit by SHA
git revert 4a7f2e1 --no-edit

# Revert but do not commit yet
# Useful when reverting multiple commits
git revert HEAD --no-commit
git revert HEAD~1 --no-commit
git commit -m "revert: remove bad payment feature"

# Push normally — no force push needed
# History preserved so normal push works
git push origin main

# Teammates get the fix with a simple pull
# git pull origin main
```

> revert is always safe on shared branches.
> It adds a new commit on top — never rewrites history.
> Teammates can git pull normally and get the revert.
> No conflicts, no broken history, no blocked engineers.
> Use revert for production incidents always.

---

## git restore

git restore is used to discard file changes without
touching commits at all. It has two main uses —
discarding working directory edits and unstaging files.

Real scenario 1: You edited login.js and made a mess.
Not staged or committed yet. You want to throw away
your edits and restore the file to last commit state.

Real scenario 2: You staged a file with git add by
mistake and want to unstage it without losing your edits.

```bash
# Use 1 — discard changes in working directory
# Restore file to last committed state
git restore login.js
# Your edits are GONE permanently — no recovery

# Restore ALL files in working directory
git restore .

# Always check what you are about to lose first
git diff login.js
# Red lines = what will be deleted
# Then restore if you are sure
git restore login.js

# Verify
git status
# login.js does not appear — file is clean

# Use 2 — unstage a file
# Remove from staging area, keep edits in working dir
git restore --staged login.js
# File is unstaged but edits still exist
# Just needs git add again

# Restore a file from a specific commit
git restore --source 4a7f2e1 login.js
# Gets login.js exactly as it was in that commit
```

> git restore on working directory is permanent.
> No reflog for this. Edits are gone forever.
> Always run git diff first to see what you are losing.

> git restore is the modern replacement for:
> git restore login.js         = git checkout -- login.js
> git restore --staged login.js = git reset HEAD login.js
> Both old commands still work but restore is cleaner.

---

## git clean

git clean permanently deletes untracked files — files
that were never added to Git with git add. These are
files Git does not know about at all.

Real scenario: You ran a build process that created
hundreds of temp files and build artifacts. These are
untracked. You want to wipe them all and get back to
a clean state.

```bash
# ALWAYS run dry run first
# Shows what WOULD be deleted without deleting anything
git clean -n
# Output:
# Would remove temp.log
# Would remove build/output.js
# Would remove debug.txt

# Delete untracked files
# -f = force, required by Git as safety measure
git clean -f
# Output:
# Removing temp.log
# Removing debug.txt

# Delete untracked files AND untracked directories
git clean -fd
# -f = force
# -d = include untracked directories

# Delete ignored files too (build artifacts, logs)
git clean -fdx
# -x = also delete files ignored by .gitignore

# Interactive mode — choose what to delete one by one
git clean -i
```

> git clean permanently deletes untracked files.
> There is no recovery. No reflog. No trash bin.
> Always run git clean -n first to preview.
> Never skip the dry run step.

---

## Safe vs Dangerous — The Decision Tree

This is the most asked interview topic for Day 3.
Know this completely.

```
Question 1:
Is the bad commit on a shared branch?
(main, develop, did teammates already pull it?)

YES → git revert only. Never use reset.
NO  → go to Question 2.

Question 2:
Do I want to keep my file changes?

YES keep staged   → git reset --soft HEAD~1
YES keep unstaged → git reset --mixed HEAD~1
NO delete all     → git reset --hard HEAD~1

Question 3:
I just want to fix one file, not undo commits?

→ git restore filename

Question 4:
I want to delete files Git does not know about?

→ git clean -n first then git clean -f
```

---

## Quick Reference Table

| Command | Commit Undone | Staging Area | Working Dir |
|---------|--------------|--------------|-------------|
| reset --soft | ✅ Yes | Files kept staged | Files kept |
| reset --mixed | ✅ Yes | Cleared | Files kept |
| reset --hard | ✅ Yes | Cleared | Files DELETED |
| revert | ❌ New commit added | Unchanged | Unchanged |
| restore | ❌ No | Can unstage | Can discard |
| clean | ❌ No | No effect | Untracked deleted |

---

## The Golden Rule — Safe vs Dangerous

```
reset  = rewrites history
         NEVER use on shared branches
         Only use on your own private feature branch
         Requires force push if already pushed
         Can recover via reflog within 90 days

revert = preserves history
         ALWAYS safe on any branch including main
         Normal push works, no force push needed
         Use this in production incidents always
```

---
