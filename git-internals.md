# Git Internals — How Git Stores Branches

> Senior DevOps Engineer Training | 4 Years Experience Level

---

## How Git Stores Branches Internally

Most engineers use git branch every day but never
understand what is happening inside the .git folder.
This knowledge separates junior engineers from seniors
and comes up in troubleshooting interviews regularly.

---

## The .git Folder — Git's Brain

When you run git init, Git creates a hidden .git folder.
This folder IS your entire repository. Everything Git
knows — your commits, branches, history, config — lives
here. If you delete this folder, Git history is gone.

```
.git/
├── HEAD           → points to your current branch
├── config         → repo configuration
├── objects/       → stores all commits, files, trees
└── refs/
    └── heads/     → stores all your local branches
        ├── main
        ├── checkout
        ├── emailservice
        ├── feature-old
        └── feature/
            └── payment
```

---

## How a Branch is Stored

A branch in Git is simply a plain text file inside
.git/refs/heads/ containing one thing — the SHA hash
of the latest commit on that branch.

```bash
# See what is inside the main branch file
cat .git/refs/heads/main
# Output:
# 9f3a2b1c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a
```

That 40 character string is the SHA hash of the latest
commit on main. That is it. A branch is just a pointer
to a commit. Nothing more.

When you make a new commit on main, Git updates that
file with the new commit SHA. The branch pointer moves
forward automatically.

```
BEFORE commit:
.git/refs/heads/main → 9f3a2b1c...  (old commit)

AFTER git commit:
.git/refs/heads/main → 7d4e5f6a...  (new commit)
```

---

## The Slash Problem — cannot lock ref Error

This is a real error you will hit in companies.

```
error: cannot lock ref 'refs/heads/feature/payment'
```

### Why This Happens

Git stores branches as files inside .git/refs/heads/.
File names cannot contain slashes in any operating system.

So when you create a branch named feature/payment,
Git handles the slash by creating a directory structure:

```
feature/payment stored as:
.git/refs/heads/feature/    ← directory
                    payment ← file inside it
```

The problem happens when you already have a branch
named exactly feature as a plain file:

```
.git/refs/heads/feature     ← this is a FILE (branch)
```

Now you try to create feature/payment:

```
Git needs to create .git/refs/heads/feature/ as a DIRECTORY
But feature already exists as a FILE
A path cannot be both a file and a directory
Git throws the error: cannot lock ref
```

### Real Example

```bash
# You have an existing branch called feature
git branch
# * main
#   feature        ← this exists as a file

# You try to create feature/payment
git checkout -b feature/payment
# error: cannot lock ref 'refs/heads/feature/payment'
# fatal: cannot process 'refs/heads/feature/payment'
```

### How to Fix It

Option 1 — Rename the existing feature branch:

```bash
git branch -m feature feature-old
# Now feature is renamed to feature-old
# feature/ directory can be created now
git checkout -b feature/payment
# Works successfully
```

Option 2 — Delete the existing feature branch if not needed:

```bash
git branch -d feature
git checkout -b feature/payment
```

Option 3 — Use a different naming convention:

```bash
# Instead of feature/payment use feat/payment
git checkout -b feat/payment
```

---

## Why ls Shows feature in Blue

When you run ls .git/refs/heads and see:

```
cartservice  checkout  emailservice  feature  feature-old  main
```

And feature appears in blue while others appear white —
this is because ls colorizes different file types:

```
White or default color → regular file (normal branch)
Blue                   → directory (hierarchical branch)
```

feature is blue because it is a directory containing
sub-branches like feature/payment inside it.

To verify exactly what type each item is:

```bash
ls -l .git/refs/heads
# Output:
# -rw-r--r-- main          ← regular file, normal branch
# -rw-r--r-- checkout      ← regular file, normal branch
# -rw-r--r-- emailservice  ← regular file, normal branch
# -rw-r--r-- feature-old   ← regular file, normal branch
# drwxr-xr-x feature/      ← directory, has sub-branches
```

- starts with - = regular file = simple branch
- starts with d = directory = hierarchical branch

To see what is inside the feature directory:

```bash
ls .git/refs/heads/feature
# Output: payment

# This means you have a branch called feature/payment
```

---

## HEAD — How Git Knows Your Current Branch

HEAD is a special file in .git/ that always points to
your current branch. It is how Git knows which branch
you are on right now.

```bash
cat .git/HEAD
# Output:
# ref: refs/heads/main
# This means you are currently on main branch
```

When you switch branches:

```bash
git checkout feature/payment
cat .git/HEAD
# Output:
# ref: refs/heads/feature/payment
```

HEAD updated automatically to point to new branch.

When you are in detached HEAD state (after git checkout
on a commit SHA or tag):

```bash
git checkout 9f3a2b1c
cat .git/HEAD
# Output:
# 9f3a2b1c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a
# HEAD points directly to a commit, not a branch
```

Detached HEAD means you are not on any branch.
Any commits you make here are not attached to a branch
and can be lost if you switch away.

---

## How Git Objects Work Internally

Every commit, file, and folder in Git is stored as an
object inside .git/objects/. Git uses SHA hashing to
identify every object uniquely.

There are 3 types of objects:

```
blob   → stores file content
tree   → stores directory structure
commit → stores commit metadata + points to a tree
```

When you run git commit this is what happens internally:

```
Step 1 — Git creates a blob object for each changed file
         blob stores the file content with a SHA hash

Step 2 — Git creates a tree object
         tree stores the directory structure
         pointing to all the blobs

Step 3 — Git creates a commit object
         commit stores:
         - your name and email
         - timestamp
         - commit message
         - pointer to the tree object
         - pointer to parent commit SHA

Step 4 — Git updates the branch file in refs/heads/
         with the new commit SHA
```

```
commit 9f3a2b1c
  author: Ravi
  date: Mon Jun 9 2026
  message: feat: add payment form
  tree: 4d7e8f2a
  parent: 2c1b9d3e

tree 4d7e8f2a
  payment.js → blob 7a4c3b2d
  login.js   → blob 1e2f3a4b

blob 7a4c3b2d
  (full content of payment.js)
```

---

## Packed Refs — When .git/refs/heads Gets Large

In large repositories with hundreds of branches, Git
automatically packs branch references into a single file
called packed-refs for performance.

```bash
cat .git/packed-refs
# Output:
# 9f3a2b1c refs/heads/main
# 4d7e8f2a refs/heads/feature/payment
# 2c1b9d3e refs/remotes/origin/main
```

If you cannot find a branch file in .git/refs/heads/,
check packed-refs — the branch reference may be there.

---

## Quick Reference — Git Internal Files

| File/Folder | What It Stores |
|-------------|----------------|
| .git/HEAD | Your current branch pointer |
| .git/refs/heads/ | All local branch pointers |
| .git/refs/remotes/ | All remote branch pointers |
| .git/refs/tags/ | All tag pointers |
| .git/objects/ | All commits, files, trees |
| .git/config | Repo level Git configuration |
| .git/packed-refs | Packed branch references |
| .git/COMMIT_EDITMSG | Last commit message |
| .git/MERGE_HEAD | Exists during active merge |
| .git/REBASE_HEAD | Exists during active rebase |

---

## Interview Questions on This Topic

**Q. What is stored inside .git/refs/heads/?**

Each branch is stored as a plain text file containing
the SHA hash of the latest commit on that branch.
A branch is simply a pointer to a commit.

**Q. What does HEAD mean in Git?**

HEAD is a file in .git/ that points to your current
branch. When you switch branches, HEAD updates
automatically. In detached HEAD state, HEAD points
directly to a commit SHA instead of a branch name.

**Q. Why does git throw cannot lock ref error?**

Because you are trying to create a hierarchical branch
name like feature/payment when a branch named feature
already exists as a plain file. Git needs to create
feature as a directory but it already exists as a file.
Fix by renaming or deleting the existing feature branch.

**Q. Why is one branch showing blue in ls output?**

Blue in ls output means it is a directory not a file.
A branch showing as blue means it contains sub-branches.
For example feature appears as a directory when branches
like feature/payment exist inside it.

**Q. What is a detached HEAD state?**

Detached HEAD happens when HEAD points directly to a
commit SHA instead of a branch name. This happens when
you checkout a specific commit or tag. Any commits made
in detached HEAD state are not on any branch and can be
lost when you switch away. Always create a branch if you
want to keep work done in detached HEAD state.

```bash
# You are in detached HEAD — create a branch to save work
git checkout -b my-new-branch
```

---
