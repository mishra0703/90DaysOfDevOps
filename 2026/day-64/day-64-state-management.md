# Day 64 -- Terraform State Management and Remote Backends

## Task
The state file is the single most important thing in Terraform. It is the source of truth -- the map between your `.tf` files and what actually exists in the cloud. Lose it and Terraform forgets everything. Corrupt it and your next apply could destroy production.

Today we will learn to manage state like a professional -- remote backends, locking, importing existing resources, and handling drift.

---

## Inspect Your Current State


Use your Day 63 config (or create a small config with a VPC and EC2 instance). Apply it and then explore the state:

```bash
terraform show                                    # Full state in human-readable format
terraform state list                              # All resources tracked by Terraform
terraform state show aws_instance.<name>          # Every attribute of the instance
terraform state show aws_vpc.<name>               # Every attribute of the VPC
```

---

Just like `terraform plan` which show What will change (future)  `terraform show` shows What will change (future)

![](terraform%20show.png)
---

`terraform state list` shows a list of all resource addresses currently tracked in your state file.

![](terraform%20state%20list.png)
---

`terraform state show aws_instance.my_instance` shows all attributes of just that one specific resource from the state file — a detailed, single-resource view.

![](terraform%20show%20aws_service.png)
---



---
### How many resources does Terraform track?

8 resources : It directly equals however many resource blocks exist in your applied config. 
`data blocks` don't count here (they're not "tracked/managed," they're just for read/fetching data).


### What attributes does the state store for an EC2 instance? (hint: way more than what you defined)


Terraform stores every attribute the AWS API returns for that resource , not just what we set in our config. That's why we'll see many extra fields (like `arn`, `private_dns`) that we never explicitly wrote. AWS auto-generates them, and Terraform captures the full picture for accurate tracking.

#### Examples : 

![](terraform%20show%20aws_service.png)
---
![](terraform%20show.png)


### Open `terraform.tfstate` in an editor -- find the `serial` number. What does it represent?

`serial` is a counter that increments every time the state file is written/updated

What it tracks:
- It's basically a version number for your state file
- Increases by 1 (or more) each time Terraform successfully applies a change and updates the state 


Why it matters:
- Helps Terraform (and remote backends like S3) detect which version of state is newer — especially important when multiple people/systems might be working with the same state
- If Terraform sees a serial mismatch (e.g., someone else applied changes and pushed a newer state), it can warn about conflicts instead of silently overwriting

---

*Here , the serial is equals to 16 because , we did `terraform apply` once which created 8 resources , and then deletes them (`terraform destroy`) later. Hence the serial got 16 (8 for creating resources , 8 for deleting it)*

![](serial%20in%20.tfstate%20file.png)
---
---

## Set Up S3 Remote Backend


Storing state locally is dangerous -- one deleted file and you lose everything. Time to move it to S3.

1. First, create the backend infrastructure (do this manually or in a separate Terraform config):
```bash
# Create S3 bucket for state storage
aws s3api create-bucket \
  --bucket terraweek-state-<yourname> \
  --region ap-south-1 \
  --create-bucket-configuration LocationConstraint=ap-south-1

# Enable versioning (so you can recover previous state)
aws s3api put-bucket-versioning \
  --bucket terraweek-state-<yourname> \
  --versioning-configuration Status=Enabled

# Create DynamoDB table for state locking
aws dynamodb create-table \
  --table-name terraweek-state-lock \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST \
  --region ap-south-1
```

2. Add the backend block to your Terraform config:
```hcl
terraform {
  backend "s3" {
    bucket         = "terraweek-state-<yourname>"
    key            = "dev/terraform.tfstate"
    region         = "ap-south-1"
    dynamodb_table = "terraweek-state-lock"
    encrypt        = true
  }
}
```


---
### *The Problem:*

- `terraform init` configures the backend BEFORE anything else runs — so if your
`.tf` file has a `backend "s3" {}` block pointing to a bucket/DynamoDB table
that doesn't exist yet, `init` fails immediately.
- You can't create the backend resources *through* the same config that already declares them as its backend.


### *The Solution :*

- Write the resources (`aws_s3_bucket`, `aws_dynamodb_table`) but leave the `backend "s3" {}` block commented out / absent.
- `terraform init` → uses local state (no backend configured yet)
- `terraform apply` → creates the S3 bucket + DynamoDB table, tracked in local `terraform.tfstate`
- Now uncomment / add the `backend "s3" {}` block, pointing to the bucket & table you just created. 
- `terraform init` again → Terraform detects the backend changed and asks :
   
   > "Do you want to copy existing state to the new backend?"
   
   Answer **yes**. This migrates your local state into S3.

- From here on, state lives remotely — proceed as normal.


![](create%20s3%20and%20dynamodb%20before%20writing%20backend%20block.png)
---

3. Run:
```bash
terraform init
```

---
![](state-lock%20creation.png)
---

---

Terraform will ask: "Do you want to copy existing state to the new backend?" -- say yes.

![](state-lock%20created.png)
---

4. Verify:
   - Check the S3 bucket -- you should see `dev/terraform.tfstate`
   - Your local `terraform.tfstate` should now be empty or gone
   - Run `terraform plan` -- it should show no changes (state migrated correctly)

---

## Test State Locking


State locking prevents two people from running `terraform apply` at the same time and corrupting the state.

1. Open **two terminals** in the same project directory
2. In Terminal 1, run:
```bash
terraform apply
```
3. While Terminal 1 is waiting for confirmation, in Terminal 2 run:
```bash
terraform plan
```
4. Terminal 2 should show a **lock error** with a Lock ID



---

1st terminal ran first , and acquired state lock and prevent anyone else from making any kind of change to the infrastructure

![](terminal%201st%20ran%20and%20acquired%20state%20lock.png)
---

2nd terminal got failed and returned a lock-info with whoever is locking the state right now , we can see the name of our 1st terminal...

![](terminal%202nd%20got%20failed%20with%20a%20lock%20info.png)
---

Now , we did the revers. We acquired state-lock from terminal 2 and hence terminal 1 got failed and showed us Lock Info

![](terminal%201%20got%20failed%20with%20lock%20info.png)
---

As a proof we can see terminal 2 is acquiring the state lock and hence terminal 1 command got failed 

![](terminal%202nd%20acquired%20state%20lock.png)
---


---

Format of Lock Info we got on our dynamoDB when some-one tries to do something but ended up got failure due to state locking...

![](tflock%20file%20from%20s3%20bucket.png)
---


### What is the error message? Why is locking critical for team environments?

Error Message

```bash
Error: Error acquiring the state lock

  Error message: operation error DynamoDB: PutItem, https response error StatusCode: 400,
  ConditionalCheckFailedException: The conditional request failed
    Lock Info:
      ID:        9b9fda0d-42f7-f8d5-8e73-1aa69567b647
      Path:      remote-bucket-by-prem/terraform.tfstate
      Operation: OperationTypeApply
      Who:       DESKTOP-RS9RRCR\Prem@DESKTOP-RS9RRCR
      Version:   1.15.8
      Created:   2026-09-24 18:01:07.4852196 +0000 UTC
```


Without locking, two people (or two CI pipelines) running `terraform apply`
at the same time would both read the same state file, make changes, and then
try to write their own updated version back — whichever writes last silently
overwrites the other's changes. This is a **race condition**, and it can
lead to:
- **State corruption** — the state file no longer accurately reflects real
  infrastructure, since one person's changes got clobbered.
- **Duplicate or conflicting resources** — two applies creating the same
  resource independently, or one destroying something the other just created.
- **Drift that's hard to diagnose** — the state says one thing, AWS has
  another, and nobody knows why until something breaks in production.

Locking solves this by ensuring **only one Terraform operation can modify
state at a time** — everyone else has to wait their turn, exactly like a
database transaction lock. It's what makes shared, remote state genuinely
safe to use with multiple engineers or automated pipelines working against
the same infrastructure.


5. After the test, if you get stuck with a stale lock:
```bash
terraform force-unlock <LOCK_ID>
```

---

## Import an Existing Resource


Not everything starts with Terraform. Sometimes resources already exist in AWS and you need to bring them under Terraform management.

1. Manually create an S3 bucket in the AWS console -- name it `terraweek-import-test-<yourname>`
2. Write a `resource "aws_s3_bucket"` block in your config for this bucket (just the bucket name, nothing else)
3. Import it:
```bash
terraform import aws_s3_bucket.imported terraweek-import-test-<yourname>
```
4. Run `terraform plan`:
   - If you see "No changes" -- the import was perfect
   - If you see changes -- your config does not match reality. Update your config to match, then plan again until you get "No changes"

5. Run `terraform state list` -- the imported bucket should now appear alongside your other resources


---

Terraform Plan before importing s3 bucket from aws console

![](terraform%20plan%20before%20importing%20bucket.png)
---

Terraform Plan after importing s3 bucket

![](terraform%20import%20and%20plan%20.png)
---

Terraform State List after importing bucket

!{[](terraform%20state%20list%20after%20importing%20bucket.png)
---


### What is the difference between `terraform import` and creating a resource from scratch?

**Creating from scratch (`terraform apply` on a new resource block):**
- We write a resource block, Terraform calls the cloud provider's "create"
  API, and a brand-new resource is provisioned.
- Terraform manages the resource from birth — it knows every attribute
  because it set them all itself, so state and reality start out perfectly
  in sync.


**`terraform import`:**
- The resource **already exists** in AWS (or wherever) — created manually
  via the console/CLI, by someone else's Terraform run.
- `import` doesn't create anything new. It only **links an existing real
  resource to a resource block in your `.tf` code**, by writing that
  resource's current attributes into your state file.
- After import, Terraform treats it as "managed" — but it's now on you to
  make sure your `.tf` resource block's arguments actually match the real
  resource's current configuration. If they don't match, the very next
  `terraform plan` will show a diff and try to *change* the real resource
  to match your code (which may not be what you want).


**Summary :** Creating from scratch is Terraform *building* something and
then remembering what it built. Importing is Terraform means *adopting* something
someone else built, and trusting you to describe it accurately in code
afterward.


---

## State Surgery -- mv and rm


Sometimes you need to rename a resource or remove it from state without destroying it in AWS.

1. **Rename a resource in state:**
```bash
terraform state list                              # Note the current resource names
terraform state mv aws_s3_bucket.imported aws_s3_bucket.logs_bucket
```
Update your `.tf` file to match the new name. Run `terraform plan` -- it should show no changes.

2. **Remove a resource from state (without destroying it):**
```bash
terraform state rm aws_s3_bucket.logs_bucket
```
Run `terraform plan` -- Terraform no longer knows about the bucket, but it still exists in AWS.

3. **Re-import it** to bring it back:
```bash
terraform import aws_s3_bucket.logs_bucket terraweek-import-test-<yourname>
```


---

**`terraform state mv`**
- Renames or moves a resource **within the state file**, without touching
  the real infrastructure.
- Use case: you rename a resource in your `.tf` code, or move it into/out
  of a module — without this, Terraform would think the old resource was
  deleted and a new one needs to be created.

![](terraform%20state%20mv%20(renaming).png)
---

**`terraform state rm`**
- Removes a resource **from the state file only** — Terraform "forgets"
  about it, but the real resource in AWS is untouched and keeps running.
- Use case: you want Terraform to stop managing something (e.g. hand it off
  manually, or re-import it elsewhere) without destroying it.

![](terraform%20state%20rm%20(resource%20deleting%20from%20state%20list).png)
---


### When would you use `state mv` in a real project? When would you use `state rm`?

When to use `state mv` :
- Renaming a resource in your code
  - If you just rename it in code and apply, Terraform thinks the old one was deleted and a new one needs creating (destroy + recreate!). Instead we use `state mv` this tells Terraform "same resource, just a new name" — no destroy/recreate, no downtime. Also after renaming in CLI don't forget to rename it in code too...
- Moving a resource into a module
  - `terraform state mv aws_instance.server module.ec2.aws_instance.server`
  - Useful when refactoring your flat config into reusable modules — keeps existing infra intact instead of recreating it.
- Reorganizing state across workspaces/state files (advanced team setups)


When to use `state rm`:
- Resource is now managed elsewhere (e.g., migrating a resource to be managed by a different Terraform project/team)
  - The resource still exists in AWS(or elsewhere) — Terraform just "forgets" about it and stops managing it.
- Fixing state drift/mistakes 
  - E.g. you accidentally imported the wrong resource, or a resource was manually deleted outside Terraform but still shows in state causing errors
- Splitting one Terraform project into multiple — remove resources from the old state so they can be terraform imported into a new, separate state file


**Important** : Neither command touches actual AWS resources — they only edit Terraform's internal bookkeeping (the state file). That's the core idea: sometimes you need to fix Terraform's "memory" without affecting what's actually running in AWS.

---

## Simulate and Fix State Drift


State drift happens when someone changes infrastructure outside of Terraform -- through the AWS console, CLI, or another tool.

1. Apply your full config so everything is in sync
2. Go to the **AWS console** and manually:
   - Change the Name tag of your EC2 instance to `"ManuallyChanged"`
   - Change the instance type if it's stopped (or add a new tag)
3. Run:
```bash
terraform plan
```
You should see a **diff** -- Terraform detects that reality no longer matches the desired state.

4. You have two choices:
   - **Option A:** Run `terraform apply` to force reality back to match your config (reconcile)
   - **Option B:** Update your `.tf` files to match the manual change (accept the drift)

5. Choose Option A -- apply and verify the tags are restored.

6. Run `terraform plan` again -- it should show "No changes." Drift resolved.




---

We changed name , type and added a tag to our Ec2 and this is how it looks before fixing state drifting

![](ec2%20before%20fixing%20state%20drifting.png)
---

After doing changes to Ec2 on console and checking terraform plan we can see some changes 

![](State%20Drifting.png)
---

After doing terraform apply to fix state drifting , every changes we made to the Ec2 on console got revert back to original desired state we defined in our code (.tf file) 

![](Ec2%20back%20to%20it's%20original%20settings%20after%20fixing%20state%20drifting.png)
---



### How do teams prevent state drift in production? (hint: restrict console access, use CI/CD for all changes)

What is state drift?
- When real infrastructure changes outside of Terraform (e.g., someone manually edits a resource in AWS Console) — Terraform's state no longer matches reality. Next plan/apply either tries to "fix" it back or gets confused.


Key prevention strategies
1. Restrict AWS Console/CLI access
    - Give engineers read-only access to production AWS accounts
    - Removes the temptation/ability to manually click-fix something in the console
2. All changes go through CI/CD pipelines
    - No one runs terraform apply from their local laptop against prod
    - Changes go through: Pull Request → Review → Automated plan → Approval → Automated apply
    - Ensures every change is reviewed, logged, and consistent
3. Use remote state with locking
    - Store state in S3 + DynamoDB (or Terraform Cloud) — not local files
    - Prevents concurrent applies corrupting state (as we tested earlier with state locking)
4. Regular drift detection
    - Run `terraform plan` periodically (E.g., nightly via CI/CD) — even without applying — just to detect if plan shows unexpected changes (meaning something drifted). Some teams use `terraform plan -detailed-exitcode` in automated jobs to alert if drift is found.
5. Tag/audit everything
    - Use AWS CloudTrail to log all API calls/changes — helps detect who made a manual change if drift is found
    - Tag resources with ManagedBy = "Terraform" (like you did with common_tags) — makes it clear which resources should never be touched manually
6. Immutable infrastructure practices
    - Prefer replacing resources over manually patching them (E.g., new AMI + new instance instead of SSH-ing in and changing config)
    - Reduces the chance of untracked manual changes accumulating over time

    

---

## *Points to remember*

- DynamoDB table must have a `LockID` string key -- this is what Terraform uses for locking
- `terraform init -migrate-state` explicitly triggers state migration
- `terraform refresh` (or `terraform apply -refresh-only`) updates state to match real infrastructure without making changes
- State locking only works with backends that support it (S3+DynamoDB, Consul, Terraform Cloud)
- `terraform force-unlock` should only be used when you are sure no other operation is running
- Always version your S3 bucket so you can recover a previous state file if something goes wrong

