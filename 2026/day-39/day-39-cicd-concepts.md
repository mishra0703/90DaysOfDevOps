# Day 39 – What is CI/CD?

## The Problem
Think about a team of 5 developers all pushing code to the same repo manually deploying to production.


### 1. What can go wrong?
- Two devs touch the same file, push around the same time → merge conflicts that get resolved badly under time pressure, sometimes silently dropping someone's change.
- Someone deploys their local branch instead of main — production now has code nobody else has reviewed.
- Person A deploys, person B deploys 10 minutes later without knowing A had a fix in progress → A's change gets overwritten or production ends up in a half-A-half-B state.
- Deploys happen at random times, including right before someone goes offline — if it breaks, nobody's around to fix it.


### 2. What does "it works on my machine" mean and why is it a real problem?
- It means , the code runs fine on the developer's laptop, but breaks (or behaves differently) anywhere else like on a teammate's machine, staging, or production.
- Because it highlights environment inconsistency .
- Also it erodes trust in "tested" code — if "tested" only means "tested on one specific machine," it's not really tested.


### 3. How many times a day can a team safely deploy manually?
- Max 2-3 times , as Each manual deploy needs a human to be present, alert, sober-minded, and watching for fallout — that doesn't scale past a couple of times a day before fatigue sets in and steps get skipped.
- Also because Manual deploys are slow (SSH in, pull, build, restart, verify) 


---

## CI vs CD



### 1. **Continuous Integration** — what happens, how often, what it catches
Every dev merges/pushes code to a shared branch frequently (multiple times a day), and an automated pipeline immediately builds it and runs tests (lint, unit, sometimes integration). It catches bugs and merge conflicts early, right when they're cheap to fix, instead of letting them pile up till "integration hell" at the end of a sprint.

**Example :**
A team of 5 devs all push to feature branches multiple times a day. Every push triggers a GitHub Actions workflow that runs ESLint and unit tests on both the Node/Express backend and React frontend. *If dev B's change breaks dev A's code, the pipeline fails within minutes on the PR — not three days later when both are deep in unrelated work.*


### 2. **Continuous Delivery** — how it's different from CI, what "delivery" means
After the steps in CI , every change that passes to the pipeline is automatically packaged into a deployable artifact (e.g. a Docker image) and pushed to staging, ready to go to production at any time. "Delivery" means it's always in a releasable state — but a human still clicks the button to actually release it to prod.

**Example :**
That same pipeline, after tests pass, builds a multi-stage Docker image, and pushes it to a registry or staging server automatically — fully built, and ready to deploy. But the actual "deploy to production EC2 instance" step still needs someone to click "Run workflow" or approve it. The artifact is always release-ready; a human decides when to deliver it.


### 3. **Continuous Deployment** — how it differs from Delivery, when teams use it
The difference in Continuous Delivery and Deployment is that in Continous Deployment there's no human gate at all. If the pipeline (tests, scans, staging checks) passes, it deploys straight to production automatically. Teams use this when they have strong automated test coverage and confidence in their pipeline so passing means genuinely safe.

**Example :**
Same pipeline again, except the final SSH-deploy-to-EC2 step , it runs automatically the moment the tests pass on main — no manual approval at all. This is what companies like Etsy or Amazon do at scale: hundreds of small, low-risk changes go straight to production daily because their test coverage is trusted enough that if *pipeline is green* means it is *safe to ship*.

---

## Pipeline Anatomy

- **Trigger** — what starts the pipeline
The event that kicks off the whole pipeline. In our workflows this is usually on: push to main, or workflow_run. Could also be a PR, a schedule (cron), or a manual workflow_dispatch.


- **Stage** — a logical phase (build, test, deploy)
A broad logical phase of the pipeline, like "Build", "Test", or "Deploy". Stages run in sequence (or sometimes in parallel) and represent what the pipeline is conceptually doing at that point.


- **Job** — a unit of work inside a stage
A self-contained unit of work inside a stage, made up of multiple steps. Jobs can run in parallel or depend on each other (needs: in GitHub Actions). E.g. "lint-backend" and "lint-frontend" could be two separate jobs running in parallel within a "Test" stage.


- **Step** — a single command or action inside a job
The smallest unit or we can say one command or one action inside a job, executed in order. E.g. actions/checkout@v4, then npm install, then npm run lint are three separate steps in one job.


- **Runner** — the machine that executes the job
The actual machine (VM or container) that executes a job's steps. GitHub-hosted runners spin up fresh for each job; we could also use a self-hosted runner on our own EC2 instance.


- **Artifact** — output produced by a job
The output a job produces that later jobs/stages need — a build folder, a test report, or the Docker image itself, which gets passed from the "build" job to the "deploy" job.


---

## Draw a Pipeline

CI/CD pipeline for the scenario:
> A developer pushes code to GitHub. The app is tested, built into a Docker image, and deployed to a staging server.

![CI/CD pipeline](CI-CD%20Pipeline.png)


---

## Explore in the Wild

1. Open any popular open-source repo on GitHub (Kubernetes, React, FastAPI — pick one you know)

- We choose React's open-source repo 



2. Find their `.github/workflows/` folder

- https://github.com/react/react/blob/main/.github/workflows/



3. Open one workflow YAML file

- We took https://github.com/react/react/blob/main/.github/workflows/shared_lint.yml this yaml file for our research



4. Write in your notes:

- What triggers it?
    - It will be triggered on push on main branch , and also on a PR. Means whenever someone pushes the code on main branch this workflow will be pushed automatically and also whenever someone open a PR.

- How many jobs does it have?
    - It has four jobs , those are : prettier , eslint , check_license and test_print_warnings.

- What does it do? (best guess)
    - It checks for all kind of errors , basically it is a Lint Check Workflow. 


