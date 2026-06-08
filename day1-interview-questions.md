# Day 1 — Git Core Commands Interview Questions

> Senior DevOps Engineer Training | 4 Years Experience Level

---

### Q1. Explain the 4 stages of Git. Walk me through editing a file, staging it, committing it, and pushing it.

Git has 4 stages.

First is the Working Directory — this is where I edit files
on my local machine. Any file I create or modify lives
here first.

Second is the Staging Area, also called the Index. When I
run `git add`, I move files from the Working Directory into
the Staging Area. This is a preparation zone — I can choose
exactly which changes I want to include in my next commit.
I can stage one file, multiple files, or even specific lines
inside a file using `git add -p`.

Third is the Local Repository. When I run
`git commit -m "message"`, Git takes everything in the
Staging Area and saves a permanent snapshot into my local
repo. Each commit gets a unique ID called a SHA hash —
like `4a7f2e1`. This is stored inside the `.git` folder
on my machine.

Fourth is the Remote Repository — this is GitHub or GitLab.
When I run `git push origin main`, I send my local commits
to the remote so my team can see them. `origin` is just the
default name Git gives to the remote URL.

The flow is always:

```
edit → git add → git commit → git push
```

---
