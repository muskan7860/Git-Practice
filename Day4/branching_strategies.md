# Day 4 — Branching Strategies

> Senior DevOps Engineer Training | 4 Years Experience Level

---

## Why Does This Topic Exist?

Every company using Git must answer one question:

```
"How do our 50 engineers work on the same codebase
 without stepping on each other's toes, and how do
 we get code safely into production?"
```

The answer to that question is called a branching
strategy — the company's rulebook for how branches are
created, named, merged, and deployed.

There is no single correct strategy. Companies choose
based on release frequency, team size, whether they
support multiple production versions, and CI/CD maturity.

---

# Strategy 1 — GitFlow

## The Book and Photocopy Analogy

Imagine your code is a published book sitting in
bookstores. Customers are reading it right now.

```
main = the published book customers are reading
       (production code)
```

You cannot scribble on the published book directly —
that would confuse readers. If you want to change
something, you take a photocopy, work on the photocopy,
and only the finished, polished result becomes the new
published edition.

"Branching off X" simply means — make a copy starting
from X, and work on the copy.
"Merging into X" simply means — copy the finished work
back into X.

---

## main branch

```
main = the published book customers are reading right now.
       This is your live, working product.
```

---

## develop branch

Your team wants to start working on the next edition —
Edition 2.0. It will take months. So you make a photocopy
of the published book and call it develop. This is your
team's internal working draft for the next edition.
Customers never see develop.

```
main:     [Published Book - Edition 1.0]
              |
              | (photocopy made)
              ↓
develop:  [Working Copy - becoming Edition 2.0]
```

```bash
# Create develop branch off main
# This is "making a photocopy of main called develop"
git checkout main
git checkout -b develop
```

---

## feature branches

3 writers want to add 3 different chapters to Edition 2.0
— Login, Payment, Search. If all 3 write directly on the
develop photocopy at the same time, it gets messy. So
each writer makes their OWN photocopy of develop, writes
their chapter, and merges back into develop when done.

```
develop:              [Working Copy - Edition 2.0]
    |                        |                    |
    | (photocopy)            | (photocopy)        | (photocopy)
    ↓                        ↓                    ↓
feature/login        feature/payment      feature/search
[+ Chapter: Login]   [+ Chapter: Payment]  [+ Chapter: Search]
```

```bash
# Writer 1 — branch off develop for their chapter
git checkout develop
git checkout -b feature/login

# Writer 1 writes their chapter
echo "Login feature code" > login.js
git add login.js
git commit -m "feat: add login chapter"

# Writer 1 finishes — merge back into develop
git checkout develop
git merge --no-ff feature/login
# --no-ff = always create a merge commit
# This shows in history that feature/login existed

git branch -d feature/login
```

After all 3 writers finish and merge, develop has all
3 chapters — Login, Payment, Search.

```
develop:  [Working Copy - Edition 2.0
           + Chapter: Login
           + Chapter: Payment
           + Chapter: Search]
```

---

## release branch

develop now has all 3 chapters. Edition 2.0 is almost
ready. Before printing thousands of copies, you need
final proofreading, typo fixes, cover update with
"Edition 2.0" label, one last quality check.

You do not want to do this directly on develop because
the team might already be adding NEW chapters for
Edition 3.0 there. So you make ANOTHER photocopy — this
time of develop — and call it release/2.0.

```
develop:    [Working Copy + Login + Payment + Search]
                |
                | (photocopy made for final polish)
                ↓
release/2.0: [Final Polish Copy - Edition 2.0]
              (fix typos, update cover, final checks)
```

```bash
# Branch off develop for final polish
git checkout develop
git checkout -b release/2.0

# Final polish work — version bump
echo "App Version 2.0" > app.js
git add app.js
git commit -m "chore: bump version to 2.0.0"

# Fix a bug found during testing
echo "Search feature code - bug fixed" > search.js
git add search.js
git commit -m "fix: resolve search pagination bug"
```

Once release/2.0 is fully polished:
- It gets PRINTED and sent to bookstores
  → merge release/2.0 INTO main
- The polish fixes should also go back into develop so
  future editions have them too
  → merge release/2.0 INTO develop too

```
release/2.0 [Final Polish Copy - Edition 2.0]
        |                              |
        | merge into main              | merge into develop
        ↓                              ↓
main: [Published Book          develop: [Working Copy
       Edition 2.0]                      + all polish fixes]
```

```bash
# Merge release into main — PUBLISH the book
git checkout main
git merge --no-ff release/2.0
git tag -a v2.0.0 -m "Release 2.0.0"

# Also merge release into develop — polish not lost
git checkout develop
git merge --no-ff release/2.0

# Delete the release branch — done with it
git branch -d release/2.0
```

---

## hotfix branch

Edition 2.0 is published and in bookstores. Customers
are reading it. Someone finds a HUGE error on page 50 —
needs fixing IMMEDIATELY. You cannot wait for Edition 3.0.

You cannot use develop — it already has NEW unfinished
chapters for Edition 3.0 mixed in. You need a photocopy
of the EXACT published book (main) so you fix ONLY page
50 and nothing else.

```
main: [Published Book - Edition 2.0]
        |
        | (photocopy made for emergency fix)
        ↓
hotfix/page50-error: [Emergency Fix Copy]
                      (fix page 50 ONLY here)
```

Once page 50 is fixed:
- The corrected book needs to go back to bookstores
  → merge hotfix INTO main (publish Edition 2.0.1)
- develop also needs this fix so Edition 3.0 does not
  have the same error
  → merge hotfix INTO develop too

```
hotfix/page50-error [Emergency Fix Copy]
        |                              |
        | merge into main              | merge into develop
        ↓                              ↓
main: [Published Book          develop: [Working Copy
       Edition 2.0.1 - FIXED]            + page 50 fix]
```

```bash
# Emergency — branch off main directly
git checkout main
git checkout -b hotfix/critical-bug

# Fix the critical bug
echo "App Version 2.0 - CRITICAL BUG FIXED" > app.js
git add app.js
git commit -m "hotfix: fix critical production bug"

# Merge hotfix into main — PUBLISH the fix immediately
git checkout main
git merge --no-ff hotfix/critical-bug
git tag -a v2.0.1 -m "Hotfix 2.0.1"

# Also merge into develop — v3.0 should not repeat this
git checkout develop
git merge --no-ff hotfix/critical-bug

git branch -d hotfix/critical-bug
```

---

## GitFlow Summary

```
main      = what customers see RIGHT NOW
develop   = what the team is building for NEXT TIME
feature   = one person's individual work, joins develop
release   = final polish before publishing, becomes main
hotfix    = emergency fix for what customers see RIGHT NOW
```

---

## GitFlow Pros and Cons

```
PROS:
Very structured — clear rules for everything
Supports multiple versions in production
Safe for slow, scheduled releases
Hotfix process is clearly defined
Good for teams with formal QA/release cycles

CONS:
Too heavy for fast-moving teams
Too many branches — complex to manage
Slows down CI/CD — not ideal for daily deployments
develop and main can drift out of sync if not careful
Overkill for small teams or simple apps
```

```
Used by: banking software, enterprise software,
desktop applications, mobile apps with app store
review cycles, embedded systems — anything with
scheduled releases and multiple supported versions
in production at once.
```

---

## GitFlow — Full Hands-On Lab

```bash
# Setup
mkdir gitflow-lab
cd gitflow-lab
git init

# Create the "published book v1.0"
echo "App Version 1.0" > app.js
git add app.js
git commit -m "feat: initial release v1.0"
git tag -a v1.0.0 -m "Release 1.0.0"
```

```bash
# Part 1 — create develop (working draft for v2.0)
git checkout -b develop
git branch
#   main
# * develop
```

```bash
# Part 2 — Writer 1 creates feature/login
git checkout develop
git checkout -b feature/login

echo "Login feature code" > login.js
git add login.js
git commit -m "feat: add login chapter"

git checkout develop
git merge --no-ff feature/login
git branch -d feature/login

ls
git log --oneline --graph --all
```

```bash
# Part 3 — Writer 2 creates feature/payment
git checkout develop
git checkout -b feature/payment

echo "Payment feature code" > payment.js
git add payment.js
git commit -m "feat: add payment chapter"

git checkout develop
git merge --no-ff feature/payment
git branch -d feature/payment

ls
git log --oneline --graph --all
```

```bash
# Part 4 — Writer 3 creates feature/search
git checkout develop
git checkout -b feature/search

echo "Search feature code" > search.js
git add search.js
git commit -m "feat: add search chapter"

git checkout develop
git merge --no-ff feature/search
git branch -d feature/search

ls
# app.js, login.js, payment.js, search.js
git log --oneline --graph --all
```

```bash
# Part 5 — Create release/2.0 for final polish
git checkout develop
git checkout -b release/2.0

echo "App Version 2.0" > app.js
git add app.js
git commit -m "chore: bump version to 2.0.0"

echo "Search feature code - bug fixed" > search.js
git add search.js
git commit -m "fix: resolve search pagination bug"

git log --oneline --graph --all
```

```bash
# Part 6 — Ship release/2.0 to main (PUBLISH)
git checkout main
git merge --no-ff release/2.0
git tag -a v2.0.0 -m "Release 2.0.0"

ls
git log --oneline --graph --all
```

```bash
# Part 7 — Also merge release into develop
git checkout develop
git merge --no-ff release/2.0

git branch -d release/2.0
git log --oneline --graph --all
```

```bash
# Part 8 — EMERGENCY hotfix on production
git checkout main
git checkout -b hotfix/critical-bug

echo "App Version 2.0 - CRITICAL BUG FIXED" > app.js
git add app.js
git commit -m "hotfix: fix critical production bug"

git checkout main
git merge --no-ff hotfix/critical-bug
git tag -a v2.0.1 -m "Hotfix 2.0.1"

git checkout develop
git merge --no-ff hotfix/critical-bug

git branch -d hotfix/critical-bug

git log --oneline --graph --all
git tag
# v1.0.0, v2.0.0, v2.0.1
```

```bash
# Final check — see the complete GitFlow shape
git log --oneline --graph --all --decorate
```

The key thing to notice — every merge uses --no-ff.
This is intentional. It creates a merge commit even
when fast-forward is possible, so the commit graph
visually preserves the shape of GitFlow — develop
branching off main, features branching off develop,
release branching off develop and merging into both,
hotfix branching off main and merging into both.

---

# Strategy 2 — Trunk-Based Development

## The Live Document Analogy

In GitFlow, main (published book) and develop (working
draft) were TWO separate photocopies. Trunk-Based says —
forget having two separate copies. There is only ONE
book. Everyone writes directly into it, in tiny pieces.

```
main = a live shared document that customers are
       reading in real time, AND your team is
       editing at the same time
```

You cannot take it offline to edit privately for 2 weeks
like develop in GitFlow. Customers are watching it live.

---

## How Changes Are Made

You make a TINY photocopy, write ONE small piece, and
merge it back within HOURS — not weeks.

```
main (live doc) ──[p1]──[p2]──[p3]──[p4]── ... ──[p10]──▶
                    ↑     ↑     ↑     ↑
              each [pX] merged same day
              tiny branch, lives a few hours, then gone
```

---

## Feature Flags — The Key Concept

If you add a 10-paragraph chapter over 2 weeks one
paragraph at a time, customers see an incomplete, broken
chapter for 2 weeks. Feature flags solve this.

A feature flag is a simple ON/OFF switch:

```
"IF this chapter is marked READY, show it to readers.
 IF NOT, hide this chapter completely."
```

```
Day 1:  add paragraph 1, flag = OFF, customers see nothing
Day 5:  add paragraphs 2,3,4, flag = OFF, still nothing
Day 14: all 10 paragraphs done, flag = ON
        customers see the complete chapter, all at once
```

The code for ALL 10 paragraphs is inside main the whole
time — being tested and integrated daily. Customers do
not see it until the flag flips ON.

```javascript
// This code lives in main from Day 1
if (featureFlags.newLoginButton === true) {
    showNewLoginButton()
} else {
    showOldLoginButton()
}
// Day 1-13: flag = false → customers see OLD button
// Day 14:   flag = true  → customers instantly see NEW button
// No big merge needed - it was already in main!
```

---

## Trunk-Based Pros and Cons

```
PROS:
main never gets too far ahead or behind
Small daily merges = almost no conflicts
Code is tested in main every single day
Can deploy main to production multiple times a day

CONS:
Needs feature flags for everything incomplete
Needs very good automated tests
Requires discipline — tiny daily pieces is harder
than writing it all at once
```

```
Used by: Google, Facebook, Netflix, most modern SaaS
companies — anyone deploying multiple times per day
with strong CI/CD and feature flag systems.
```

---

## GitFlow vs Trunk-Based — One Sentence Each

```
GitFlow:
"Work privately for weeks on a big feature branch,
 then merge it all at once when fully done."

Trunk-Based:
"Work directly on main every day in tiny pieces,
 hide unfinished work behind feature flags until ready."
```

---

## Trunk-Based — Full Hands-On Lab

```bash
# Setup
mkdir trunk-lab
cd trunk-lab
git init

echo "App Version 1.0" > app.js
git add app.js
git commit -m "feat: initial app v1.0"

echo "newLoginButton: false" > feature-flags.yml
git add feature-flags.yml
git commit -m "chore: add feature flags config"
```

Notice — there is NO develop branch here. Only main.

```bash
# Day 1 — tiny branch, merge same day
git checkout main
git checkout -b feature/login-button-part1

echo "function renderLoginButton() { /* part 1 - HTML */ }" > login-button.js
git add login-button.js
git commit -m "feat: add login button HTML (behind flag)"

git checkout main
git merge --no-ff feature/login-button-part1
git branch -d feature/login-button-part1

git log --oneline --graph --all
cat feature-flags.yml
# newLoginButton: false  ← still OFF
```

```bash
# Day 2 — another tiny piece
git checkout main
git checkout -b feature/login-button-part2

echo "function handleLoginClick() { /* part 2 - click logic */ }" >> login-button.js
git add login-button.js
git commit -m "feat: add login button click logic (behind flag)"

git checkout main
git merge --no-ff feature/login-button-part2
git branch -d feature/login-button-part2

git log --oneline --graph --all
```

```bash
# Day 3 — one more tiny piece
git checkout main
git checkout -b feature/login-button-part3

echo "function validateLoginInput() { /* part 3 - validation */ }" >> login-button.js
git add login-button.js
git commit -m "feat: add login button validation (behind flag)"

git checkout main
git merge --no-ff feature/login-button-part3
git branch -d feature/login-button-part3

cat login-button.js
# all 3 functions present
git log --oneline --graph --all
```

```bash
# Day 14 — feature complete, flip the flag
echo "newLoginButton: true" > feature-flags.yml
git add feature-flags.yml
git commit -m "chore: enable new login button feature flag"

git log --oneline --graph --all
cat feature-flags.yml
# newLoginButton: true ← NOW customers see it
```

```bash
# Comparison — what if this was GitFlow instead?
cd ..
mkdir gitflow-comparison
cd gitflow-comparison
git init

echo "App Version 1.0" > app.js
git add app.js
git commit -m "feat: initial app v1.0"

git checkout -b develop
git checkout -b feature/login-button

# ALL 3 parts on ONE branch over "14 days"
echo "function renderLoginButton() { /* part 1 */ }" > login-button.js
git add login-button.js
git commit -m "feat: add login button HTML"

echo "function handleLoginClick() { /* part 2 */ }" >> login-button.js
git add login-button.js
git commit -m "feat: add login button click logic"

echo "function validateLoginInput() { /* part 3 */ }" >> login-button.js
git add login-button.js
git commit -m "feat: add login button validation"

# ONE big merge after "14 days"
git checkout develop
git merge --no-ff feature/login-button
git branch -d feature/login-button

git log --oneline --graph --all
```

```bash
# Compare both
cd ../trunk-lab
git log --oneline --graph --all
# main updated 4 times over 14 days
# each update small, easy to review and debug

cd ../gitflow-comparison
git log --oneline --graph --all
# develop updated 1 time after 14 days
# update = 3 commits worth of changes at once
```

This is why trunk-based reduces risk — small frequent
changes are easier to debug than one big change.

---

# Strategy 3 — GitHub Flow

## The Simple Middle Ground

GitFlow has 5 branch types. Trunk-Based has only main
plus tiny branches with feature flags. GitHub Flow sits
in the middle — simpler than GitFlow, slightly more
structured than pure trunk-based.

```
"There is one main branch — always deployable.
 Every change happens in a feature branch.
 Open a Pull Request. Get it reviewed. Merge. Deploy."
```

Only 2 types of branches — main and feature/x. No
develop. No release branches. No hotfix branches —
hotfixes are just feature branches too.

```
main (always deployable) ──A────────M───────────M──▶
                              \      │  \        │
                       feature/x ────  feature/y─
                       (branches off main)
                       (merges back via PR)
```

---

## The 6 Steps of GitHub Flow

```
1. Create a branch off main
2. Add commits — work on your feature
3. Open a Pull Request — even if not finished
4. Discuss and review — push more commits as needed
5. Merge — once approved, merge into main
6. Deploy — main is always deployable, CI/CD deploys it
```

---

## GitHub Flow Pros and Cons

```
PROS:
Simple — easy for new team members to understand
main is always deployable — confidence in production
Works great with CI/CD — automatic deploy on merge
Pull Request is the central collaboration point
Good balance of speed and safety

CONS:
No dedicated develop branch — less staging area
Not ideal for supporting multiple production versions
Requires good test coverage — main deploys fast
Less structure for very large teams with complex releases
```

```
Used by: most web applications, SaaS products, startups,
small to medium teams using GitHub with CI/CD pipelines —
the MOST COMMONLY used strategy in companies you will
interview at.
```

---

## GitHub Flow — Full Hands-On Lab

```bash
# Setup
mkdir githubflow-lab
cd githubflow-lab
git init

echo "App Version 1.0" > app.js
git add app.js
git commit -m "feat: initial commit"
```

```bash
# Step 1 — create a branch off main
git checkout main
git checkout -b feature/add-search

# Step 2 — work and commit
echo "search bar code" > search.js
git add search.js
git commit -m "feat: add search bar UI"

# Step 3 — push early, open PR even if not done
git push -u origin feature/add-search
# Open PR on GitHub: feature/add-search → main
# Mark as Draft PR if not ready for review yet
```

```bash
# Step 4 — continue working, push more commits
echo "search logic" >> search.js
git add search.js
git commit -m "feat: add search filtering logic"
git push origin feature/add-search
# PR automatically updates with new commits
```

```bash
# Step 5 — teammate reviews, requests changes
echo "fix review comments" >> search.js
git add search.js
git commit -m "fix: address review comments"
git push origin feature/add-search
```

```bash
# Step 6 — approved! Merge via GitHub UI
# Usually "Squash and Merge" to keep main clean

# main is deployed automatically by CI/CD
# Clean up locally
git checkout main
git pull origin main
git branch -d feature/add-search
```

---

## Hotfixes in GitHub Flow

There is no separate hotfix branch type. A hotfix is
just a feature branch off main, reviewed quickly,
merged quickly.

```bash
git checkout main
git pull origin main
git checkout -b fix/critical-payment-bug

echo "fixed payment code" > payment.js
git add payment.js
git commit -m "fix: critical payment null pointer"
git push -u origin fix/critical-payment-bug

# Fast-track PR review — merge immediately
# main deploys automatically — bug fixed in production
```

---

# Side by Side Comparison

```
                  GitFlow         Trunk-Based      GitHub Flow
Branches          5 types         1 + short-lived  2 types
Release cycle     Slow/scheduled  Continuous       Continuous
Complexity        High            Low              Medium
Team size         Large           Any (needs CI)   Small-Medium
Multiple versions Yes              No               No
Feature flags     Not required    Required         Optional
Best for          Banking,         Google,          Most startups,
                  Enterprise       Netflix          SaaS, web apps
```

---

## The One Question Interviewers Always Ask

```
"If you were setting up branching strategy for a new
 startup building a SaaS product with a small team of
 5 engineers, which strategy would you choose and why?"
```

ANSWER FRAMEWORK:

```
1. State your choice
2. Give the reason based on team size and release frequency
3. Mention what you would add as the team grows
```

Example answer:

```
"I would choose GitHub Flow. With 5 engineers and a SaaS
product, we need simplicity and speed. main stays always
deployable, feature branches are short-lived, and Pull
Requests give us code review without heavy process.

As the team grows past 20-30 engineers or if we need to
support multiple product versions for enterprise clients,
I would consider introducing a develop branch — moving
toward GitFlow elements. But starting with GitHub Flow
keeps us fast without unnecessary overhead."
```

---
