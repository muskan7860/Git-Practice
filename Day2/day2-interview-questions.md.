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
