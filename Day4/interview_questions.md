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
### Q2. Explain Trunk-Based Development. How is it different from GitFlow? What is a feature flag and why is it essential?

Trunk-Based Development has only ONE long-lived branch —
main. There is no develop branch like in GitFlow.

The difference from GitFlow is significant. GitFlow has
5 branch types and a slow structured release cycle —
weeks or months, with one big merge at the end. Trunk-Based
has just main, and changes go into it continuously —
multiple times a day, in small pieces.

In GitFlow, a 10-day feature lives on ONE feature branch
for all 10 days, then merges once at the end.

In Trunk-Based, that same 10-day feature is broken into
small daily pieces. Each day a tiny branch is created,
a small piece is written, and merged into main THE SAME DAY.

```bash
# Day 1
git checkout main
git checkout -b feature/payment-part1
# ... small piece of work ...
git checkout main
git merge --no-ff feature/payment-part1
git branch -d feature/payment-part1

# Repeat daily for 10 days
```

Feature flags are ESSENTIAL — not just useful. If
incomplete pieces merge into main every day, and main is
what customers see, customers would see a half-built,
broken feature for 10 days. Unacceptable.

A feature flag is an ON/OFF switch in code:

```javascript
if (featureFlags.newPayment === true) {
    showNewPaymentFlow()
} else {
    showOldPaymentFlow()
}
```

For all 10 days, the flag stays OFF. Customers see the OLD
flow normally. New code is already inside main, tested and
integrated daily. On day 10, flip the flag ON.

```bash
echo "newPayment: true" > feature-flags.yml
git add feature-flags.yml
git commit -m "chore: enable new payment feature flag"
git push origin main
```

Without feature flags, Trunk-Based Development would be
impossible — there would be no way to merge incomplete
work into main without breaking production.

Benefit over GitFlow — if something breaks after merging
part 5, only part 5's small change needs checking, not 10
days worth of changes at once. Small daily merges make
debugging easier, not harder.

```
GitFlow:     5 branches, slow release, one big merge
Trunk-Based: 1 branch, continuous release, small daily merges
```

Companies like Google, Netflix, and Facebook use this
because they deploy multiple times a day and have strong
automated testing and feature flag infrastructure.

---
### Q3. Explain GitHub Flow. How is it different from GitFlow and Trunk-Based Development? Walk me through the 6 steps.

GitHub Flow has only 2 branch types — main and feature
branches. No develop, no release, no hotfix.

The 6 steps:

```
1. Create a branch off main
2. Add commits — work on the feature
3. Open a Pull Request early — even if unfinished
4. Discuss and review — push more commits as needed
5. Merge into main once approved
6. Deploy — main is always deployable, CI/CD deploys it
```

The key difference is Step 3 — opening the Pull Request
EARLY, while still working. Code review happens DURING
development, not after it is fully done.

Comparing with GitFlow and Trunk-Based — all 3 use main,
but differ in HOW OFTEN and HOW main gets updated.

GitFlow has 5 branch types. main only gets updated through
release or hotfix branches — not directly from feature
branches. Releases happen on a slow schedule, monthly or
quarterly. main is touched rarely but in big batches.

Trunk-Based has main plus very short-lived branches —
living hours, not days. Feature flags are REQUIRED because
incomplete code merges into main multiple times a day.
main is touched constantly, in tiny pieces.

GitHub Flow sits in between. Branches live for days, not
hours and not weeks. Pull Requests provide review without
needing feature flags for most cases — by the time a PR
is merged, the feature is genuinely complete and approved,
just smaller in scope than a GitFlow feature branch.

Simplest way to remember:

```
GitFlow:     main updated rarely, big batches,
             through release/hotfix branches only

Trunk-Based: main updated constantly, tiny pieces,
             feature flags required

GitHub Flow: main updated when each feature/fix is
             complete and reviewed, branches live days
```

GitHub Flow is the most commonly used strategy at most
companies — balances simplicity with safety.

---
### Q4. Startup with 5 engineers wants to deploy whenever a feature is ready. Which strategy and why? What if the team grows to 50 engineers needing 3 supported versions?

For a 5 engineer startup that wants to deploy whenever a
feature is ready — not on a fixed schedule — I would
choose GitHub Flow.

GitHub Flow has only main and feature branches. main is
always deployable. The moment a feature is reviewed and
merged, CI/CD deploys it. No waiting for a scheduled
release window. This directly matches the requirement.

```bash
git checkout main
git checkout -b feature/new-dashboard
# ... work, PR, review, merge ...
git checkout main
git merge feature/new-dashboard
git push origin main
# CI/CD deploys immediately
```

GitFlow would be overkill here — develop branch, release
branches, and scheduled cycles add complexity a small
team does not need.

For Part 2 — if the team grows to 50 engineers and needs
to support 3 different versions for enterprise clients,
this changes everything. A single main branch cannot
represent 3 live versions at once. Tags alone are not
enough — a tag marks a point in history but you cannot
make new commits on a tag if a bug is found later.

This is exactly the scenario GitFlow was designed for.
I would introduce long-lived release branches — one per
supported version:

```bash
release/v1.0   ← enterprise clients still on v1.0
release/v2.0   ← enterprise clients on v2.0
release/v3.0   ← latest version, most clients
main           ← latest development, becomes v4.0 eventually
```

If a critical bug affects all 3 versions, fix it on the
relevant branch and cherry-pick into the others:

```bash
git checkout release/v1.0
git checkout -b hotfix/critical-bug
# ... fix ...
git checkout release/v1.0
git merge --no-ff hotfix/critical-bug
git tag -a v1.0.5 -m "Hotfix 1.0.5"

git checkout release/v2.0
git cherry-pick <fix-commit-SHA>

git checkout release/v3.0
git cherry-pick <fix-commit-SHA>
```

My answer evolves with the company — start simple with
GitHub Flow for speed with one live version. As the team
grows and multiple versions need support, introduce
GitFlow-style release branches per version, while keeping
day-to-day feature workflow similar to GitHub Flow.

---
### Q5. New engineer asks why there is no develop branch in GitHub Flow and if merging directly into main is risky. How do you explain the safety mechanisms?

That is a fair question — many engineers from GitFlow
backgrounds ask this. The role develop used to play —
being a safe testing ground before code reaches
production — is replaced by faster, more rigorous checks
that happen on EVERY change, not in batches.

**1. Pull Request reviews**

Every change to main goes through a Pull Request. A
teammate reviews the code before merge — within hours,
not after weeks of mixing with other features in develop.

```bash
git checkout -b feature/payment-fix
# ... work ...
git push -u origin feature/payment-fix
# PR opened — teammate reviews BEFORE merge
```

**2. Automated CI/CD tests**

Tests run automatically on every Pull Request — unit
tests, integration tests, linting, build checks — all
BEFORE merge is possible. If any test fails, merge is
blocked.

**3. Branch protection rules**

main is set as a protected branch on GitHub:
- No one can push directly to main
- Pull Request required for every change
- At least 1 approval required
- All CI/CD checks must pass before merge

This is enforced by GitHub itself — not by hoping
engineers follow a process.

**4. Feature flags for risky changes**

For especially risky changes, teams can borrow the
feature flag concept from Trunk-Based Development — merge
code into main but hide it from users until confident.

**The key mindset shift:**

```
GitFlow:     main is safe because code is pre-tested
             in develop over time, arrives in batches

GitHub Flow: main is safe because every single change
             is reviewed and automatically tested
             before it is allowed to arrive — continuously
```

Removing develop did not remove safety — it moved safety
checks earlier, made them automatic, and applied them to
every change individually instead of in groups.

---


