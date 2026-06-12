# Day 4 — Branching Strategies Interview Questions

> Senior DevOps Engineer Training | 4 Years Experience Level

---

### Q1. Explain GitFlow and its branch types. When would a company choose GitFlow over a simpler strategy?

GitFlow has 5 branch types — main, develop, feature,
release, and hotfix.

main is the production code customers are using right now.

develop is the team's working draft for the next edition.
Multiple writers cannot work directly on develop at the
same time — that gets messy and causes conflicts. So each
writer takes a copy of develop, called a feature branch,
works independently, and merges back into develop when done.

```bash
git checkout develop
git checkout -b feature/login
# ... work, commit ...
git checkout develop
git merge --no-ff feature/login
git branch -d feature/login
```

Once develop has all the features for the next edition,
a release branch is created off develop for final polish —
fixing typos, bumping version numbers, last quality checks.
This is separate from develop because the team may already
be working on the NEXT edition (edition 3) on develop, and
we cannot disturb that.

```bash
git checkout develop
git checkout -b release/2.0
# ... final polish, version bump ...
```

Once release is ready, it merges into BOTH main (this
becomes the new published edition) AND develop (so future
editions also have these polish fixes).

```bash
git checkout main
git merge --no-ff release/2.0
git tag -a v2.0.0 -m "Release 2.0.0"

git checkout develop
git merge --no-ff release/2.0

git branch -d release/2.0
```

hotfix is for emergencies — if customers find a critical
bug in the currently published edition, we cannot use
develop because edition 3 work is happening there and
would create conflicts. Instead hotfix branches directly
off main, fixes the issue, and merges into BOTH main
(immediate fix to production) AND develop (so edition 3
does not repeat the same bug).

```bash
git checkout main
git checkout -b hotfix/critical-bug
# ... fix ...
git checkout main
git merge --no-ff hotfix/critical-bug
git tag -a v2.0.1 -m "Hotfix 2.0.1"

git checkout develop
git merge --no-ff hotfix/critical-bug
```

I always use --no-ff for every merge in GitFlow. This
creates a merge commit even when fast-forward is possible,
so the commit graph visually preserves the shape of the
strategy.

A company would choose GitFlow over a simpler strategy
when they release on a slow, scheduled cycle — monthly
or quarterly — and need to support multiple versions in
production at the same time. Banking software, enterprise
applications, and mobile apps with app store review delays
are good examples. The structure is worth the extra
complexity because mistakes are costly and rollbacks need
to be precise.

For a fast-moving startup deploying multiple times a day,
GitFlow would be overkill — Trunk-Based or GitHub Flow
fits better there.

---
