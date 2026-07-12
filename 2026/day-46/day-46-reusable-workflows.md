# Day 46 – Reusable Workflows & Composite Actions

## Task
We've been writing workflows from scratch every time. In the real world, teams **don't repeat themselves** — they create reusable workflows that any repo can call like a function. Today we will learn `workflow_call` and composite actions.

---

## Understand `workflow_call`


### 1. What is a **reusable workflow**?
- A workflow file that can be called from other workflows, instead of copy-pasting the same steps everywhere. You define common CI/CD logic once (e.g. "build and push Docker image") and multiple workflows/repos can invoke it with different inputs.



### 2. What is the `workflow_call` trigger?
- A special on: trigger that makes a workflow callable by other workflows (rather than triggered by push/PR directly).

```bash
on:
  workflow_call:
    inputs:
      image-tag:
        required: true
        type: string
    secrets:
      DOCKERHUB_TOKEN:
        required: true
```

Without workflow_call, a workflow can only be triggered by events like push or pull_request — it can't be called by another workflow.



### 3. How is calling a reusable workflow different from using a regular action (`uses:`)?

#### Reusable Workflow vs. a Regular Action (`uses:`)

| | Action (`uses: actions/checkout@v4`) | Reusable Workflow (`uses: org/repo/.github/workflows/x.yml@main`) |
|---|---|---|
| **Scope** | A single step inside a job | An entire job (or multiple jobs) |
| **Contains** | One task (checkout, setup, etc.) | Full `jobs:`, each with their own `steps:` |
| **Called from** | Inside a `steps:` list | Inside a `jobs:` list, via `uses:` at job level |
| **Runs on** | The same runner as the calling job | Its own separate runner |
| **Can have secrets/inputs** | Yes (action inputs) | Yes (`inputs:` and `secrets:` block) |

**In short:** an action is a reusable *step*, while a reusable workflow is a reusable *job* (or set of jobs).



### 4. Where must a reusable workflow file live?
- Inside .github/workflows/ (same as any workflow) — not in a special subfolder.
- Can be in the same repo as the caller, or a different repo (public or private, with proper permissions).

- It is called like : 

```bash
jobs:
  call-docker-build:
    uses: mishra0703/githubActions/.github/workflows/docker-publish.yml@main
    with:
      image-tag: latest
    secrets: inherit
```
The @main (or @v1, @<sha>) specifies which branch/tag/commit of the reusable workflow to use — same versioning model as actions.



---

## Create Your First Reusable Workflow

1. Set the trigger to `workflow_call`
2. Add an `inputs:` section with:
   - `app_name` (string, required)
   - `environment` (string, required, default: `staging`)
3. Add a `secrets:` section with:
   - `docker_token` (required)
4. Create a job that:
   - Checks out the code
   - Prints `Building <app_name> for <environment>`
   - Prints `Docker token is set: true` (never print the actual secret)

*Check reusable-build.yml in this directory*

---

## Create a Caller Workflow

Create `.github/workflows/call-build.yml`:
1. Trigger on push to `main`
2. Add a job that uses your reusable workflow:
   ```yaml
   jobs:
     build:
       uses: ./.github/workflows/reusable-build.yml
       with:
         app_name: "my-web-app"
         environment: "production"
       secrets:
         docker_token: ${{ secrets.DOCKER_TOKEN }}
   ```
3. Push to `main` and watch it run



![Successfully called reusable-build.yml from call-build.yml](Called%20Reusable%20Workflow.png)
*Check call-build.yml in this directory*


---

## Add Outputs to the Reusable Workflow

Extend `reusable-build.yml`:
1. Add an `outputs:` section that exposes a `build_version` value
2. Inside the job, generate a version string (e.g., `v1.0-<short-sha>`) and set it as output
3. In your caller workflow, add a second job that:
   - Depends on the build job (`needs:`)
   - Reads and prints the `build_version` output


![Successfully Printed Version from reusable-build.yml file](Printed%20output%20from%20reusable%20workflow.png)


---

## Create a Composite Action

Create a **custom composite action** in your repo at `.github/actions/setup-and-greet/action.yml`:
1. Define inputs: `name` and `language` (default: `en`)
2. Add steps that:
   - Print a greeting in the specified language
   - Print the current date and runner OS
   - Set an output called `greeted` with value `true`
3. Use the composite action in a new workflow with `uses: ./.github/actions/setup-and-greet`


![](Composite%20Action.png)

---

## Reusable Workflow vs Composite Action


| | Reusable Workflow | Composite Action |
|---|---|---|
| Triggered by | `workflow_call` | `uses:` in a step |
| Can contain jobs ? | Yes ,full jobs: block, can have multiple jobs | No , as it has no concept of jobs, just a flat list of steps |
| Can contain multiple steps ? | Yes — each job has its own `steps:` | Yes — `runs.steps:` can have many steps |
| Lives where ? | `.github/workflows/*.yml` in a repo | Its own dir with action.yml (e.g. `.github/actions/my-action/action.yml`), can be same repo or separate repo |
| Can accept secrets directly ? | Yes — `secrets:` block in workflow_call, passed explicitly or via secrets: inherit | No dedicated `secrets:` block — secrets must be passed in as regular `inputs:` (from the caller's `${{ secrets.X }}`) |
| Best for | Sharing entire CI/CD pipelines — build+test+deploy logic across repos/teams | Sharing small, reusable step sequences — a specific task like "setup + lint + notify" |

