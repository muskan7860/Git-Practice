# Day 1 — Git Core Commands Interview Questions

> Senior DevOps Engineer Training | 4 Years Experience Level

---

### Q1. Explain the 4 stages of Git. Walk me through editing a file, staging it, committing it, and pushing it.

Git has 4 stages.

First is the Working Directory — this is where I edit
files on my local machine. Any file I create or modify
lives here first.

Second is the Staging Area, also called the Index. When
I run `git add`, I move files from the Working Directory
into the Staging Area. This is a preparation zone — I
can choose exactly which changes I want to include in my
next commit. I can stage one file, multiple files, or
even specific lines inside a file using `git add -p`.

This is what makes Git powerful — I can control exactly
what goes into each commit. I do not have to commit
everything at once.

Third is the Local Repository. When I run
`git commit -m "message"`, Git takes everything in the
Staging Area and saves a permanent snapshot into my local
repo. Each commit gets a unique ID called a SHA hash
like `4a7f2e1`. This is stored inside the `.git` folder
on my machine.

Fourth is the Remote Repository — this is GitHub or
GitLab. When I run `git push origin main`, I send my
local commits to the remote so my team can see them.
`origin` is just the default name Git gives to the
remote URL.

The flow is always:

```
edit → git add → git commit → git push
```

---

### Q2. You made a commit but forgot to include one file. You have NOT pushed yet. What exactly do you do and why?

Since I have not pushed yet, I can fix this cleanly
using `--amend`.

First I stage the forgotten file:

```bash
git add forgot-file.js
```

Then I amend the last commit without changing the message:

```bash
git commit --amend --no-edit
```

`--amend` does not add a new commit. It replaces the
last commit entirely. The old commit SHA is destroyed
and a brand new commit with a new SHA is created — now
containing both the original files and the forgotten file.

This is safe here because I have not pushed yet. Nobody
on GitHub has seen the old commit. Rewriting it affects
only me.

If I had already pushed, I would NOT use `--amend`
because it would create a history conflict on GitHub.
In that case I would simply make a new commit with the
forgotten file.

```bash
git add forgot-file.js
git commit -m "fix: add missing file to previous commit"
```

---

### Q3. Your teammate pushed 3 commits to main overnight. What is the first thing you do in the morning and why? What is the difference between git pull and git fetch?

First thing every morning I do not blindly run
`git pull`. I run `git fetch` first.

```bash
git fetch origin
```

This downloads whatever was pushed overnight but does
NOT touch my files or my current branch. My working
directory stays exactly as I left it.

Then I review what changed:

```bash
git log origin/main --oneline
```

Now I can see the new commits with my own eyes before
deciding to merge.

Then I merge:

```bash
git pull --rebase origin main
```

I use `--rebase` instead of plain `git pull` because
rebase puts my commits on top in a clean straight line.
Plain `git pull` creates an extra merge commit which
makes history messy over time.

Difference between `git pull` and `git fetch`:

`git fetch` only downloads changes from remote. It never
touches your working directory or current branch. It just
updates your local memory of what GitHub looks like.

`git pull` is `git fetch` + `git merge` in one command.
It downloads AND immediately applies changes to your
current branch.

```
git pull = git fetch + git merge
```

Senior engineers prefer `git fetch` first because it
gives a chance to review before merging. Blindly running
`git pull` can cause unexpected conflicts.

---

### Q4. You are working on a feature branch with half done work you cannot commit. Production goes down. What do you do step by step?

This is exactly where `git stash` saves me.

I have half done work that I cannot commit because it is
incomplete. So I stash it first with a clear name:

```bash
git stash save "WIP: half done login feature"
```

Stash saves everything in my working directory and
staging area separately. My working directory is now
completely clean.

Now I switch to main and fix the production bug:

```bash
git checkout main
git add .
git commit -m "hotfix: fix production null pointer"
git push origin main
```

Production is fixed. I come back to my feature branch:

```bash
git checkout feature/login
```

I check my saved stashes:

```bash
git stash list
# stash@{0}: WIP: half done login feature
```

I restore my work:

```bash
git stash pop
```

`pop` applies the stash AND deletes it from the list.

If I want to apply a specific stash and keep it in list:

```bash
git stash apply stash@{0}
```

pop vs apply:

| Command | Applies Stash | Deletes from List |
|---------|--------------|-------------------|
| git stash pop | ✅ | ✅ |
| git stash apply | ✅ | ❌ |

Always name your stashes. Unnamed stashes are confusing
when you have multiple saved.

---

### Q5. You pushed a commit to your feature branch. Commit message is wrong AND you forgot a file. Ravi has NOT pulled yet. What do you do?

Since I already pushed, I need to amend AND force push.

Step 1 — Stage the forgotten file:

```bash
git add forgot-file.js
```

Step 2 — Amend with new correct message:

```bash
git commit --amend -m "fix: correct null pointer in login"
```

I use `-m` with new message here, NOT `--no-edit`,
because I want to change the message.

Step 3 — Force push safely:

After `--amend`, my local commit has a brand new SHA.
GitHub still has the old SHA. They have diverged.

Normal push will fail:

```bash
git push origin feature/login
# ERROR: rejected — history has diverged
```

So I use --force-with-lease NOT --force:

```bash
git push --force-with-lease origin feature/login
```

`--force-with-lease` checks first — did anyone push to
this branch after my last fetch? Ravi has not pulled yet
so the check passes and force push succeeds.

--force-with-lease vs --force:

| Command | Safety Check | Risk |
|---------|-------------|------|
| --force-with-lease | ✅ Checks first | Low |
| --force | ❌ No check | Destroys teammates work |

Why this is safe here:
- It is my own feature branch
- Ravi has not pulled yet
- Nobody else's work is at risk

If Ravi had already pulled:
Do NOT amend. Make a new commit instead.
Never rewrite history others have already seen.

```bash
git add forgot-file.js
git commit -m "fix: add missing file from previous commit"
```

---
