# Day 2 — Branching & Merging Interview Questions

> Senior DevOps Engineer Training | 4 Years Experience Level

---

### Q1. What is a branch in Git and why do we use it? Give me a real example from your work experience.

A branch in Git is an isolated workspace where I can work
on a feature, bug fix, or experiment without touching the
main production code.

In real companies main branch is sacred — it is what
customers are using right now. If I commit broken code
directly to main, production goes down immediately.

So instead I create a branch off main, do all my work
there, test it completely, and only merge back into main
when everything is verified and approved through code
review.

In my project we follow this branch structure:

```
main      → production code, never commit directly
develop   → developers merge features here first
feature/x → one branch per feature, one developer
bugfix/x  → dedicated branch for bug fixes
hotfix/x  → emergency fix taken directly off main
release/x → final testing before production release
```

For example if I am building a payment gateway, I create
`feature/payment-gateway`, build and test there, raise a
Pull Request, get code review approval, then merge into
develop and eventually into main.

We never commit directly to main. Every change goes
through a branch, a Pull Request, and code review first.

---
### Q2. What is the difference between fast forward merge and 3-way merge? When does each one happen?

Fast forward merge happens when main has NOT moved since
I created my feature branch. Nobody committed to main
while I was working. Git simply slides the main pointer
forward to my latest commit. No extra commit is created.
History stays clean and linear.

```
main      A──B──C
                \
feature          D──E

AFTER fast forward:
main      A──B──C──D──E
```

The key condition — main did NOT move after I branched.
Git just slides the pointer forward. No merge commit.

3-way merge happens when BOTH my feature branch and main
have new commits since I branched. Git cannot slide the
pointer because two histories have diverged.

```
main      A──B──C──R1──R2
                \
feature          D──E

AFTER 3-way merge:
main      A──B──C──R1──R2──M
                \          ↗
feature          D──E─────
```

Git looks at 3 points — the common ancestor C, the tip
of main R2, and the tip of my feature E. It combines all
3 and creates merge commit M. This M has two parents —
R2 and E.

The simplest way to remember:

```
Main did NOT move after branching → Fast Forward
Main DID move after branching     → 3-Way Merge
```

---
### Q3. Your tech lead says rebase your feature branch onto latest main before raising a Pull Request. What does that mean and what do you do step by step?

When my tech lead says rebase onto latest main it means —
take my commits and put them on top of the latest main
so my branch is fully up to date with clean linear history.

Without rebase my branch looks like this:

```
main      A──B──C──R1──R2
                \
feature          D──E
```

My branch is behind main by R1 and R2. If I raise a PR
now my tech lead sees outdated code and potential conflicts.

So I rebase. Rebase does 3 things internally:

```
Step 1 — Git lifts my commits D and E off my branch
Step 2 — Git moves my branch pointer to tip of main R2
Step 3 — Git replays my commits D and E on top of R2
```

Result after rebase:

```
main      A──B──C──R1──R2
feature   A──B──C──R1──R2──D²──E²
```

Clean straight line. My branch has all of main's latest
changes plus my work on top. No merge commit anywhere.

The exact steps I follow:

```bash
# Step 1 — update main locally
git checkout main
git pull origin main

# Step 2 — go to feature branch
git checkout feature/payment

# Step 3 — rebase onto latest main
git rebase main

# Step 4 — force push because SHAs changed after rebase
git push --force-with-lease origin feature/payment
```

After rebase my commits get brand new SHA IDs because
their parent commit changed. Normal git push gets rejected.
I must use --force-
### Q4. Production is down. Ravi fixed a critical bug on his feature branch but his full feature is not ready. You need just that one bug fix on main right now. What do you do?

This is exactly the scenario where I use git cherry-pick.

Cherry-pick copies one specific commit from any branch
and applies it to my current branch. I do not need to
merge Ravi's entire feature branch — just that one commit.

Step 1 — Find the exact commit SHA from Ravi's branch:

```bash
git log feature/ravi-feature --oneline
# Output:
# f3a9b2c feat: add new dashboard
# e7d4c1a fix: fix critical null pointer  ← this one
# b2e8f3d chore: add tests
```

Step 2 — Switch to main and update it:

```bash
git checkout main
git pull origin main
```

Step 3 — Cherry-pick that exact commit:

```bash
git cherry-pick e7d4c1a
# Output:
# [main 9b3f2e1] fix: fix critical null pointer
```

Step 4 — Push to GitHub:

```bash
git push origin main
```

What cherry-pick does internally — it takes commit
e7d4c1a from Ravi's branch, copies the exact same
changes, and creates a brand new commit with a new SHA
on main. Ravi's original commit stays untouched.

```
main      A──B──C──E²
                    ↑ E copied here with new SHA

feature   A──B──C──D──E──F
                    ↑ original E still here untouched
```

If a conflict happens during cherry-pick:

```bash
# Fix the conflict then
git add fixed-file.js
git cherry-pick --continue

# Or cancel completely
git cherry-pick --abort
```

Real use case — bug fixed on develop branch. Release
branch needs that fix urgently but cannot take all other
develop changes. Cherry-pick just the fix commit onto
the release branch.

---with-lease which checks if anyone
pushed after me before overwriting.

Tech lead prefers rebase over merge because rebase gives
clean linear history with no merge commits. The Pull
Request is easy to read and review.

---
### Q5. You and Ravi both edited the same file on different branches. You merge and get a conflict. Walk me through exactly what you see and how you resolve it step by step.

A conflict happens at merge time when Git sees two
different versions of the same line from two different
branches and cannot decide which one to keep.

Important — conflict does NOT happen while editing.
It only happens when you run git merge and Git tries
to combine two different histories.

When I run git merge I see:

```bash
git merge feature/payment
# Output:
# CONFLICT (content): Merge conflict in login.js
# Automatic merge failed; fix conflicts and then commit.
```

I open login.js and see conflict markers Git inserted:

```
<<<<<<< HEAD
const user = getUser()        ← MY version on main
=======
const user = getUser(id)      ← RAVI's version
>>>>>>> feature/payment
```

These 3 markers mean:

```
<<<<<<< HEAD      = start of my version
=======           = divider between two versions
>>>>>>> feature   = end of Ravi's version
```

Before touching anything I talk to Ravi and understand
why he changed that line. Never delete someone's code
without understanding it — you might break their feature.

After discussing I edit the file, keep the correct version,
and delete ALL conflict markers completely:

```
const user = getUser(id)
```

Then I complete the merge:

```bash
# Stage the resolved file
git add login.js

# Complete the merge with clear message
git commit -m "merge: resolve conflict in login.js"

# Push to GitHub
git push origin main
```

If the conflict is too complex and I need to start over:

```bash
git merge --abort
# Everything goes back to before merge started
```

Always run git status after resolving to make sure ALL
conflicts are resolved before committing. Sometimes
multiple files are conflicted and missing one causes
broken code on main.

---
