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
