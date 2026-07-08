# Day 43 – Jobs, Steps, Env Vars & Conditionals

## Task
Today we will learn how to **control the flow** of our pipeline — multi-job workflows, passing data between jobs, environment variables, and running steps only when certain conditions are met.

---

## Multi-Job Workflow

Create `.github/workflows/multi-job.yml` with 3 jobs:
- `build` — prints "Building the app"
- `test` — prints "Running tests"
- `deploy` — prints "Deploying"

Make `test` run only **after** `build` succeeds.
Make `deploy` run only **after** `test` succeeds.

**Verify:** Check the workflow graph in the Actions tab — does it show the dependency chain?


![Dependency Chain](Dependency%20Chain.png)

---

## Environment Variables

In a new workflow, use environment variables at 3 levels:
1. **Workflow level** — `APP_NAME: myapp`
2. **Job level** — `ENVIRONMENT: staging`
3. **Step level** — `VERSION: 1.0.0`

Print all three in a single step and verify each is accessible.

Then use a **GitHub context variable** — print the commit SHA and the actor (who triggered the run).

![Use of all levels/types of environment variables](Env%20Vars.png)
![Github Context Variables](Context%20Variables.png)
![Commit Id for verification](Commit%20Id.png)

---

## Job Outputs

1. Create a job that **sets an output** — e.g., today's date as a string
2. Create a second job that **reads that output** and prints it
3. Pass the value using `outputs:` and `needs.<job>.outputs.<name>`


### Why would you pass outputs between jobs?

- Jobs run on separate, isolated runners — no shared filesystem or memory, so you need an explicit mechanism to share data.
- Avoid duplicate work — e.g. compute a version number, build tag, or changed-files list once in Job1, reuse it in Job2/Job3 instead of recalculating.
- Coordinate decisions across jobs — e.g. Job1 decides whether to deploy (based on some check), and Job2 only runs if that output says "yes."
- Keep jobs modular — each job does one thing (build, test, scan) but downstream jobs still need results (image tag, artifact URL, test status) from earlier ones.
- Enable dynamic pipelines — e.g. Job1 generates a matrix or list of values, Job2 consumes it via needs.Job1.outputs.x to run parameterized steps.


![Successfully passed output from Job1 to Job2](Output%20Sharing.png)


**Note :**
- `$GITHUB_OUTPUT` file is the Special file that runner reads after each step to populate `steps.<id>.outputs`

- `key=value` format is required so the runner knows the output's name and value

- `job.outputs.<name>` : Exposes a step output at the job level, so `needs.<job>.outputs.<name>` works in downstream jobs


---

## Conditionals

In a workflow, add:
1. A step that only runs when the branch is `main`
2. A step that only runs when the previous step **failed**
3. A job that only runs on **push** events, not on pull requests
4. A step with `continue-on-error: true` — what does this do?


![All Conditions ran successfully](Conditionals%20Jobs.png)

---

## Putting It Together

Create `.github/workflows/smart-pipeline.yml` that:
1. Triggers on push to any branch
2. Has a `lint` job and a `test` job running in parallel
3. Has a `summary` job that runs after both, prints whether it's a `main` branch push or a feature branch push, and prints the commit message


![Lint & Test Job ran in parallel , Summary ran after it](Jobs%20running%20in%20pararllel.png)
![Summary Job](Summary%20Report.png)