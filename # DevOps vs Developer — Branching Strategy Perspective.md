# DevOps vs Developer — Branching Strategy Perspective

> Senior DevOps Engineer Training | 4 Years Experience Level

---

## Who Actually Uses Branching Strategy

```
Developers  → CREATE the branches, write code, open PRs
DevOps      → BUILD THE SYSTEM that makes the strategy
              work automatically and enforces its rules
```

Both use the SAME strategy. Their RELATIONSHIP to it is
completely different.

---

## The Restaurant Analogy

```
Branching strategy = the restaurant's recipe book and
                      kitchen rules

Developers = the CHEFS
              They follow the recipes, cook the dishes,
              create new menu items (features)

DevOps = the KITCHEN MANAGER
              They design the kitchen layout, set up
              equipment, create rules for how dishes
              move from prep station to pass to customer,
              and make sure nothing leaves the kitchen
              without passing quality checks
```

A chef does not need to know how the oven was wired. The
kitchen manager built that oven, decided where it goes,
and made sure it shuts off automatically if something burns.

---

## What Developers Do With Branching Strategy

```bash
git checkout main
git pull origin main
git checkout -b feature/payment-fix

# write code
git add .
git commit -m "feat: fix payment bug"
git push -u origin feature/payment-fix

# open Pull Request
# wait for review
# merge when approved
```

Developers EXECUTE the strategy — create feature branches,
open PRs, get reviews, merge.

---

## What DevOps Actually Does With Branching Strategy

DevOps does NOT typically write the feature code. DevOps
BUILDS AND MAINTAINS the SYSTEM AROUND the strategy.

**1. Decides the strategy (often with team leads)**

```
"We are a startup, 8 engineers, deploy daily —
 let's use GitHub Flow"
```

**2. Configures branch protection rules on GitHub**

```
main requires:
- Pull Request (no direct push)
- 1 approval minimum
- All CI checks passing before merge
```

**3. Builds the CI/CD pipeline that runs on every PR**

```
Pipeline triggers on every PR:
→ run tests
→ run linter
→ build the application
→ report status back to GitHub PR
```

**4. Sets up automatic deployment triggers**

```
When code merges to main      → auto-deploy to production
When code merges to release/* → deploy to staging
```

**5. Manages environments per branch strategy**

```
main        → production environment
develop     → staging environment (GitFlow)
release/*   → pre-production environment
```

**6. Sets up feature flag infrastructure (Trunk-Based)**

```
Tools like LaunchDarkly, or custom flag systems
DevOps builds and maintains this system
```

**7. Monitors and troubleshoots the pipeline itself**

```
"Why did the deployment from main fail?"
"Why is the CI pipeline stuck on this PR?"
This is DevOps debugging the SYSTEM, not the feature code
```

---

## Real Example — Side by Side

```
SCENARIO: New feature "Add Search" needs to go live

DEVELOPER'S JOB:
git checkout -b feature/add-search
... writes search.js ...
git push -u origin feature/add-search
Opens PR, responds to review comments, merges


DEVOPS' JOB (done BEFORE this developer even started):
- Configured that main is protected (no direct push)
- Built the GitHub Actions pipeline that automatically
  runs when ANY PR is opened:
    - npm test
    - npm run lint
    - docker build
- Configured that merging to main triggers:
    - docker build → push to registry → deploy to k8s
- Set up monitoring/alerts if deployment fails
- If this developer's PR pipeline FAILS, DevOps
  might get paged to check WHY the pipeline broke
  (not why the search feature has a bug — that is
  the developer's problem)
```

---

## Interview Answer Framework

```
1. State the strategy your company uses
2. Describe it from the DEVOPS angle —
   what YOU built/maintain around it
3. Give a specific example of something
   YOU configured or troubleshot
```

---

## Sample Answer — With Real Experience Framing

*"In my current company we use GitHub Flow. We have main
as the protected branch and feature branches for all
changes.*

*From a DevOps perspective, I set up the branch protection
rules on GitHub — main requires at least one approval and
all CI checks must pass before merging. I also built the
GitHub Actions pipeline that runs on every Pull Request —
it runs our test suite, linting, and a Docker build. If
any of these fail, the merge button is automatically
blocked.*

*When code merges to main, a separate pipeline triggers
automatically — it builds the Docker image, pushes it to
our container registry, and deploys to our Kubernetes
staging environment. Production deployment requires a
manual approval step in the pipeline, which I also
configured.*

*Recently I had to troubleshoot why some PRs were stuck
in pending status — it turned out our CI runner pool was
exhausted because too many pipelines were queued at once.
I adjusted the runner configuration to scale automatically
based on queue length."*

---

## Sample Answer — Honest Framing for Zero Experience

*"In my experience, most teams I have worked with or
studied use GitHub Flow because it balances simplicity
with safety for small to medium teams. The setup typically
includes branch protection on main — requiring Pull
Request reviews and passing CI checks before merge.*

*From a DevOps standpoint, my role would be to build and
maintain the CI/CD pipeline that runs automatically on
every PR — running tests, linting, and builds — and the
deployment pipeline that triggers when code merges to
main, handling build, push to registry, and deployment to
the target environment.*

*If the team needed to support multiple production
versions, I would recommend introducing GitFlow-style
release branches alongside the existing feature branch
workflow, while keeping the CI/CD automation consistent
across all branches."*

---

## The Critical Distinction to Remember

```
WRONG framing (sounds like a developer):
"We use feature branches and merge them into main
 after they are done."

RIGHT framing (sounds like DevOps):
"We use GitHub Flow. I configured branch protection
 on main, built the CI pipeline that gates every PR,
 and set up the deployment pipeline that triggers
 on merge to main."
```

```
Developer talks about: WRITING CODE in branches
DevOps talks about:    THE SYSTEM that makes branches
                        safe, automated, and deployable
```

---

## Quick Reference — DevOps Touchpoints Per Strategy

```
GitFlow:
- Configure environments per branch type
  (develop → staging, main → production,
   release/* → pre-prod)
- Automate tagging and changelog generation
  on release branch merges
- Set up hotfix deployment pipelines separately
  from regular release pipelines

Trunk-Based:
- Build and maintain feature flag infrastructure
- Set up VERY fast CI pipelines (must run in minutes,
  since merges happen many times a day)
- Configure automatic rollback if a deploy fails
  (since deploys happen so frequently)

GitHub Flow:
- Configure branch protection rules on main
- Build PR-triggered CI pipeline (test/lint/build)
- Build merge-triggered CD pipeline (deploy)
- Set up staging vs production deployment gates
```

---

## How DevOps Sets This Up — Step by Step (GitHub Flow Example)

This is what an interviewer means by "how did you set
this up." Walk through this exact sequence.

**Step 1 — Set branch protection on main**

```
GitHub repo → Settings → Branches → Add rule

Branch name pattern: main

Enable:
☑ Require a pull request before merging
☑ Require approvals (minimum 1)
☑ Require status checks to pass before merging
☑ Require branches to be up to date before merging
☑ Do not allow bypassing the above settings
```

**Step 2 — Create the CI pipeline file**

```yaml
# .github/workflows/ci.yml
name: CI

on:
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install dependencies
        run: npm install
      - name: Run linter
        run: npm run lint
      - name: Run tests
        run: npm test
      - name: Build
        run: npm run build
```

This pipeline runs automatically on EVERY Pull Request.
If any step fails, GitHub shows a red X on the PR and the
merge button is blocked (because of Step 1's settings).

**Step 3 — Create the CD pipeline file**

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build Docker image
        run: docker build -t myapp:${{ github.sha }} .
      - name: Push to registry
        run: docker push myapp:${{ github.sha }}
      - name: Deploy to production
        run: kubectl set image deployment/myapp myapp=myapp:${{ github.sha }}
```

This pipeline runs automatically ONLY when code is pushed
to main — which only happens after a PR is merged.

**Step 4 — Set up monitoring and alerts**

```
Configure alerts for:
- Pipeline failure notifications to Slack
- Deployment health checks after deploy
- Rollback trigger if health check fails
```

---

## Common Errors You Will Face — And How to Troubleshoot

This is the section interviewers probe deeply. Know
these scenarios completely.

---

### Error 1 — "Merge button is disabled / greyed out"

```
What you see:
PR page shows "Merging is blocked"
Merge button is greyed out, cannot click it
```

```
Possible causes and troubleshooting:

1. CI checks have not finished yet
   → Check the "Checks" tab on the PR
   → Wait for pipeline to complete

2. CI checks FAILED
   → Click on the failed check to see logs
   → Common reasons: test failure, lint error,
     build error
   → Fix the code, push again, pipeline re-runs

3. Required approvals missing
   → Need at least 1 approval from a teammate
   → Request review from a team member

4. Branch is out of date with main
   → "Require branches to be up to date" is enabled
   → Run: git pull --rebase origin main
   → Push again
```

---

### Error 2 — "Pipeline stuck in queued/pending status"

```
What you see:
Yellow dot on PR, pipeline shows "Queued" for a long time
Never starts running
```

```
Troubleshooting steps:

1. Check GitHub Actions runner availability
   Settings → Actions → Runners
   If using self-hosted runners, check if they
   are online and not all busy

2. Check concurrency limits
   GitHub has limits on parallel jobs
   If too many PRs are open at once, jobs queue up

3. Check workflow file syntax
   A malformed .yml file can cause the workflow
   to fail to even start
   Validate YAML syntax

4. Solution applied in real scenario:
   Configured runner auto-scaling based on
   queue length — when queue exceeds threshold,
   spin up additional runners automatically
```

---

### Error 3 — "Deployment succeeded but application is broken"

```
What you see:
CD pipeline shows green checkmark — deploy succeeded
But the application is throwing errors in production
```

```
Troubleshooting steps:

1. Check application logs immediately
   kubectl logs deployment/myapp
   or check centralized logging (ELK, Datadog, etc.)

2. Check if environment variables/secrets
   are correctly configured for this environment
   A common cause — new code expects a new env
   variable that was not added to production config

3. Check database migrations
   If new code expects a new database column/table
   that the migration did not run yet — app crashes

4. ROLLBACK immediately while investigating
   kubectl rollout undo deployment/myapp
   This is why CD pipelines should always have
   a rollback mechanism ready

5. Root cause typically found in:
   - Missing environment-specific config
   - Database migration not run before deploy
   - Feature flag not configured for this environment
```

---

### Error 4 — "git push rejected — non-fast-forward" during automated pipeline

```
What you see:
CI/CD pipeline step that pushes a tag or commit
(like version bump) fails with:
! [rejected] main -> main (non-fast-forward)
```

```
Troubleshooting steps:

1. This usually happens when:
   - Two pipelines ran at the same time
   - Both tried to push a version bump commit
   - Second push is rejected because first
     already changed main

2. Solution:
   Pipeline should pull latest before pushing:
   git pull --rebase origin main
   git push origin main

3. Better solution — avoid pipelines pushing to
   main directly. Instead:
   - Pipeline creates a PR with the version bump
   - PR goes through normal review process
   - Avoids race conditions entirely
```

---

### Error 5 — "Branch protection rule blocking emergency hotfix"

```
What you see:
Production is down. You need to merge a hotfix
IMMEDIATELY. But branch protection requires
1 approval and CI checks — nobody is available
to approve right now.
```

```
Troubleshooting / Resolution:

1. DO NOT disable branch protection entirely
   (removes safety for everyone, even temporarily)

2. Better approach — configure an "emergency"
   process BEFORE this happens:
   - Designate specific people as emergency approvers
     who get paged immediately
   - Some teams allow repo admins to bypass rules
     ONLY for hotfix/* branch pattern, with audit
     logging enabled

3. If using GitHub Environments:
   - Production deployment requires manual approval
   - During incident, on-call engineer can approve
     directly from incident channel

4. Real solution applied:
   Created a separate fast-track workflow for
   hotfix/* branches — still requires 1 approval
   but skips the full test suite, runs only
   critical smoke tests, reducing pipeline time
   from 15 minutes to 2 minutes during incidents
```

---

## Interview Question Practice — This Topic

**Q. How do you ensure code quality before it reaches main?**

```
I configure branch protection on main requiring Pull
Request reviews and passing CI checks. The CI pipeline
runs automatically on every PR — running tests, linting,
and a build step. If anything fails, GitHub blocks the
merge button automatically. This means no human can
forget to run tests — it is enforced by the system.
```

**Q. Tell me about a time a deployment pipeline failed.
How did you troubleshoot it?**

```
Once a deployment succeeded according to the pipeline
status, but the application crashed in production. I
checked the application logs first and found it was
throwing an error about a missing environment variable.
The new code required a new config value that existed
in staging but was never added to production secrets.
I rolled back immediately using kubectl rollout undo,
added the missing environment variable to production
config, then re-triggered the deployment. Afterward I
added a pipeline step that validates required environment
variables exist before deployment proceeds, to catch this
earlier next time.
```

**Q. What happens if someone tries to push directly to main?**

```
With branch protection enabled, the push is rejected
immediately by GitHub. The error looks like:
remote: error: GH006: Protected branch update failed
The person must create a branch, open a PR, get it
reviewed, pass CI checks, and merge through the normal
process. There is no way around this except for repo
admins, and even then it can be configured to be
disallowed entirely.
```

---
