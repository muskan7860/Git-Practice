# Day 2 — Branching & Merging Commands

> Senior DevOps Engineer Training | 4 Years Experience Level

---

## What is a Branch and Why Does it Exist?

Imagine your codebase is a tree. The trunk is your main
branch — this is production code, what your customers are
using right now. If you code directly on main and something
breaks, production is down and customers are affected.

So instead you grow a new branch off the trunk. You work
there. You break things, fix things, experiment freely.
The trunk is untouched. Customers are safe. When your
feature is ready and tested, you merge your branch back
into main.

```
main ──────────────────────────────────────▶
      \                              ↗
       \──── feature/login ─────────
```

In a real company branches are named like this:

```
main       → production. Sacred. Never commit directly.
develop    → integration. All features merge here first.
feature/x  → one feature, one developer.
bugfix/x   → fixing a specific bug.
hotfix/x   → emergency fix directly off main.
release/x  → preparing a release, final testing.
```

---

## git branch

git branch is used to create, list, and manage branches.

Real scenario: Your manager says build a new payment
feature. First thing you do is create a branch. You never
work directly on main. Your branch is your own safe
workspace where nothing you do affects production.

```bash
# See all local branches
# The * shows which branch you are currently on
git branch
# Output:
# * main
#   feature/login

# See all branches including remote branches on GitHub
git branch -a
# Output:
# * main
#   feature/login
#   remotes/origin/main
#   remotes/origin/feature/login

# Create a new branch
# Important — you stay on current branch, you do NOT switch
git branch feature/payment

# Create AND switch to new branch in one command
# This is what you use 99% of the time in real work
git checkout -b feature/payment

# Modern way using git switch command
git switch -c feature/payment

# Rename a branch
git branch -m old-name new-name
```

Branch naming rules in companies — always lowercase,
always hyphens, never spaces or capital letters:

```
feature/user-login
bugfix/null-pointer-auth
hotfix/payment-crash
release/v2.5.0
```

---

## git checkout / git switch

git checkout and git switch are used to move between
branches. Think of it like switching floors in a building.
Each floor is a branch. You take the elevator to move
between them. Your work stays on each floor separately.

Real scenario: You are on main. Your manager asks you to
continue the payment feature. You switch to your feature
branch. Or your teammate asks you to review their branch.
You switch to their branch to look at the code.

```bash
# Switch to an existing branch
git checkout feature/payment
# Output: Switched to branch 'feature/payment'

# Modern cleaner way using git switch
git switch feature/payment

# Create a new branch and switch to it in one step
git checkout -b feature/payment

# Go back to the branch you were on before
# The dash means go back to last branch
# Same concept as cd - in Linux terminal
git checkout -

# Checkout a remote branch that exists on GitHub
# but not yet on your local machine
git checkout -b feature/payment origin/feature/payment
# This creates a local branch that tracks the remote one
```

Always commit or stash your work before switching branches.
If you have uncommitted changes, Git will either block you
from switching or carry those changes to the new branch
accidentally. Both situations cause problems.

git switch is the modern replacement for git checkout for
switching branches. git checkout does too many things —
it switches branches, restores files, and checks out
commits. git switch only switches branches so it is
cleaner and less confusing.

---

## git merge

git merge is used to bring work from one branch into
another. When your feature is done and tested, you merge
it into main so the whole team and production gets it.

There are exactly 2 types of merge. Interviewers always
ask about both. Know them completely.

---

### Type 1 — Fast Forward Merge

Fast forward happens when main has NOT moved since you
created your feature branch. Nobody committed to main
while you were working on your feature.

```
You created feature branch from commit C.
Nobody touched main after that.

main    A──B──C
              \
feature        D──E
```

Main is sitting at C. Your feature has D and E on top of C.

Git says — main is at C and feature has D and E sitting
cleanly on top of C. I can simply move the main pointer
forward to E. There is no need to create any extra commit.

```
BEFORE:
main    A──B──C
              \
feature        D──E

AFTER fast forward merge:
main    A──B──C──D──E
```

Main pointer slides forward. Clean straight line.
No extra commit is created anywhere.

```bash
# Always switch to main first
# You always merge INTO the branch you are on
git checkout main

# Merge your feature branch into main
git merge feature/payment
# Output:
# Updating 3f2a1b4..9c8d7e6
# Fast-forward
#  payment.js | 25 +++++++++
# The word Fast-forward tells you which type happened
```

---

### Type 2 — 3-Way Merge

3-way merge happens when main HAS moved since you created
your feature branch. While you were building your feature,
someone else also pushed commits to main.

```
You created feature branch from C.
Ravi pushed R1 and R2 to main while you worked.

main    A──B──C──R1──R2
              \
feature        D──E
```

Main is at R2. Your feature is at E. Both grew
independently from the same starting point C.

Git cannot slide the pointer forward because the two
histories have diverged. Git looks at 3 things — the
common ancestor C, the tip of main R2, and the tip of
your feature E. That is why it is called 3-way. Git
combines all 3 and creates a brand new merge commit M.
This merge commit M has two parents — R2 and E.

```
BEFORE:
main    A──B──C──R1──R2
              \
feature        D──E

AFTER 3-way merge:
main    A──B──C──R1──R2──M
              \          ↗
feature        D──E─────
```

M is the merge commit. It joins the two histories together.

```bash
git checkout main
git merge feature/payment
# Output:
# Merge made by the 'recursive' strategy
# payment.js | 25 +++++++++
# Git automatically creates merge commit M
```

---

### --no-ff Flag

--no-ff means no fast forward. Even when fast forward is
possible, Git creates a merge commit anyway. Many companies
require this for traceability so you can see exactly when
and by whom a feature branch was merged into main.

```bash
git merge --no-ff feature/payment

# WITHOUT --no-ff:
# main    A──B──C──D──E
# History is clean but no record that the branch existed

# WITH --no-ff:
# main    A──B──C────────M
#               \       ↗
# feature        D──E──
# History shows exactly when feature was merged
```

---

### Complete Merge Workflow Used in Companies

```bash
# Step 1 — bring your feature branch up to date first
git checkout feature/payment
git pull --rebase origin main

# Step 2 — switch to main
git checkout main

# Step 3 — get latest main from GitHub
git pull origin main

# Step 4 — merge feature into main
git merge --no-ff feature/payment

# Step 5 — push to GitHub
git push origin main

# Step 6 — delete the feature branch
git branch -d feature/payment
git push origin --delete feature/payment
```

---

## git rebase

git rebase takes your commits off your branch, moves your
branch pointer to the tip of another branch, then replays
your commits one by one on top. The result is a clean
linear history with no merge commits.

Real scenario: You have been working on feature/payment
for 3 days. Main got 10 new commits from teammates.
Before raising a Pull Request your tech lead says — update
your branch with latest main before I review it.

---

### What Rebase Does Internally — Step by Step

Before rebase — both branches grew from C independently:

```
main    A──B──C──R1──R2
              \
feature        D──E
```

Step 1 — Git temporarily lifts your commits D and E off
your branch and saves them aside:

```
main    A──B──C──R1──R2
feature A──B──C
               ↑ your branch looks like this temporarily
               D and E saved aside
```

Step 2 — Git moves your branch pointer to tip of main:

```
main    A──B──C──R1──R2
feature A──B──C──R1──R2
                       ↑ your branch now starts from here
```

Step 3 — Git replays your commits D and E on top:

```
main    A──B──C──R1──R2
feature A──B──C──R1──R2──D²──E²
```

D² and E² are your original commits replayed with new
SHA IDs because their parent commit changed. Same content,
same messages, brand new identities.

```bash
# Step 1 — update main locally
git checkout main
git pull origin main

# Step 2 — go to your feature branch
git checkout feature/payment

# Step 3 — rebase onto main
git rebase main
# Output:
# Successfully rebased and updated refs/heads/feature/payment

# Step 4 — force push because SHAs changed
git push --force-with-lease origin feature/payment

# If a conflict happens during rebase
# Git stops and asks you to fix it
# Fix the conflict in the file then:
git add payment.js
git rebase --continue
# Git replays the next commit

# If you want to cancel the entire rebase
git rebase --abort
# Everything goes back to before rebase started
```

---

### Interactive Rebase — Clean Up Messy Commits

Real scenario: You worked on a feature for 2 days and your
commits look embarrassing — fix typo, fix typo again, WIP,
another fix. Before raising a Pull Request you squash all
of them into one clean professional commit.

```bash
# Open last 3 commits in editor
git rebase -i HEAD~3
# HEAD~3 means go back 3 commits from current position

# Editor opens showing:
# pick a1b2c3 feat: add payment form
# pick d4e5f6 fix: typo
# pick g7h8i9 fix: another typo

# Change pick to squash on commits you want to combine
# pick   = keep this commit as is
# squash = combine this commit into the one above it
# reword = keep commit but change its message
# drop   = delete this commit completely

# pick a1b2c3 feat: add payment form
# squash d4e5f6 fix: typo
# squash g7h8i9 fix: another typo

# Save and close editor
# Git combines all 3 into 1 clean commit
# Result:
# feat: add payment form
```

---

### Golden Rules of Rebase

```
Rule 1 — Only rebase your OWN private feature branches.
         Never rebase main, develop, or any shared branch.

Rule 2 — Rebase creates brand new SHA IDs for your commits.
         If teammates already have your old SHAs,
         their history breaks completely.
         Private branch = only you have it = safe to rebase.

Rule 3 — After rebasing a branch you already pushed,
         you must force push because SHAs changed.
         git push --force-with-lease origin feature/payment

Rule 4 — When in doubt use merge.
         Merge is always safe.
         Rebase is powerful but needs discipline.
```

---

## git cherry-pick

git cherry-pick copies one specific commit from any branch
and applies it to your current branch. It does not bring
the whole branch — just that one exact commit.

Real scenario: Ravi fixed a critical bug on his feature
branch but his full feature is not ready to merge. You
need just that one bug fix on main right now. Cherry-pick
lets you take exactly that one commit without touching
anything else Ravi did.

```
main    A──B──C
feature A──B──C──D──E──F
                    ↑
               You only want this commit E
               Not D, not F. Just E.

AFTER cherry-pick:
main    A──B──C──E²
                 ↑
        E copied to main with a new SHA
```

```bash
# First find the SHA of the commit you want
git log feature/payment --oneline
# Output:
# f3a9b2c feat: add payment form
# e7d4c1a fix: fix critical null pointer  ← this one
# b2e8f3d chore: add tests

# Switch to the branch you want to add the commit to
git checkout main

# Cherry-pick that specific commit using its SHA
git cherry-pick e7d4c1a
# Output:
# [main 9b3f2e1] fix: fix critical null pointer
# That commit is now on main with a brand new SHA

# Cherry-pick multiple specific commits
git cherry-pick e7d4c1a f3a9b2c

# Cherry-pick a range of commits
git cherry-pick e7d4c1a..f3a9b2c
```

Real use case in companies — a bug is fixed on the develop
branch. The release branch needs that fix urgently but
cannot take all the other develop changes. You cherry-pick
just the fix commit onto the release branch.

---

## Merge Conflicts — When Git Cannot Auto-Merge

A conflict happens when you and a teammate both edited
the same line in the same file on different branches.
Git does not know whose version to keep so it stops and
asks you to decide manually.

Real scenario: You both edited line 10 of login.js on
separate branches. Now you are merging and Git has no
way to know which version is correct. It stops and waits
for you to decide.

```bash
# You try to merge and get a conflict
git merge feature/payment
# Output:
# CONFLICT (content): Merge conflict in login.js
# Automatic merge failed; fix conflicts and then commit.

# Open login.js — you see conflict markers inserted by Git
# <<<<<<< HEAD
# const user = getUser()        ← YOUR version
# =======
# const user = getUser(id)      ← THEIR version
# >>>>>>> feature/payment

# Step 1 — edit the file manually
# Delete ALL conflict markers: <<<<<<, =======, >>>>>>>
# Keep whichever code is correct
# Final result:
# const user = getUser(id)

# Step 2 — stage the resolved file
git add login.js

# Step 3 — complete the merge
git commit -m "merge: resolve conflict in login.js"

# If you want to cancel the merge completely
# and go back to before you started
git merge --abort
```

Always talk to the person who wrote the other version
before resolving a conflict. Never just delete their code
without understanding it. You might break their feature.

VS Code shows conflicts visually with buttons —
Accept Current Change, Accept Incoming Change, Accept Both.
Most engineers in companies resolve conflicts in VS Code
not in raw terminal.

---

## Deleting Branches — Clean Up After Merging

Real scenario: Feature is done, merged into main, Pull
Request is closed. The feature branch is now dead weight.
Clean it up both locally and on GitHub. Good teams keep
their repo clean — otherwise after 6 months you have
200 dead branches nobody knows about.

```bash
# Delete local branch safely
# Git refuses if branch is NOT merged yet — safe
git branch -d feature/payment
# Output: Deleted branch feature/payment (was 9b3f2e1)

# Force delete local branch
# Deletes even if branch has unmerged work — dangerous
git branch -D feature/payment

# Delete the branch on GitHub remote
git push origin --delete feature/payment
# Output: - [deleted] feature/payment

# Clean up stale remote tracking references
# Removes your local memory of remote branches
# that no longer exist on GitHub
git fetch --prune

# See which branches are already merged into main
# Safe to delete all of these
git branch --merged
```

GitHub has a setting to auto-delete branches after a
Pull Request is merged. Good teams always enable this.

---

## Merge vs Rebase — When to Use Which

```
Use MERGE when:
→ Merging a feature branch into main (final merge)
→ Working on a shared branch with multiple people
→ You want to preserve the exact history of what happened

Use REBASE when:
→ Updating your feature branch with latest main changes
→ Keeping your branch current before raising a Pull Request
→ Cleaning up messy commits before code review
→ You want clean linear history
```

---
