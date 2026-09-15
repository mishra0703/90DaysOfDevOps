# Day 63 -- Variables, Outputs, Data Sources and Expressions

## Task
Our Day 62 config works, but it is full of hardcoded values -- region, CIDR blocks, AMI IDs, instance types, tags. Change the region and everything breaks. Today you make your Terraform configs dynamic, reusable, and environment-aware.

This is the difference between a config that works once and a config you can use across projects.

---

## Expected Output
- A fully parameterized Terraform config with no hardcoded values
- Separate `.tfvars` files for different environments
- Outputs printed after every apply
- A markdown file: `day-63-variables-outputs.md`

## Extract Variables

1. Create a `variables.tf` file with input variables for:
   - `region` (string, default: your preferred region)
   - `vpc_cidr` (string, default: `"10.0.0.0/16"`)
   - `subnet_cidr` (string, default: `"10.0.1.0/24"`)
   - `instance_type` (string, default: `"t2.micro"`)
   - `project_name` (string, no default -- force the user to provide it)
   - `environment` (string, default: `"dev"`)
   - `allowed_ports` (list of numbers, default: `[22, 80, 443]`)
   - `extra_tags` (map of strings, default: `{}`)

2. Replace every hardcoded value in `main.tf` with `var.<name>` references
3. Run `terraform plan` -- it should prompt you for `project_name` since it has no default


### What are the five variable types in Terraform? (`string`, `number`, `bool`, `list`, `map`)

| Type     | Example              | Use case                        | Usage in main.tf                          |
|----------|-----------------------|----------------------------------|--------------------------------------------|
| `string` | `"hello"`             | Names, IDs, single text values  | `var.project_name`                         |
| `number` | `5`, `3.14`           | Counts, sizes                   | `var.instance_count`                       |
| `bool`   | `true`/`false`        | Feature toggles                 | `var.enable_monitoring`                    |
| `list`   | `[1, 2, 3]`           | Ordered items (ports, AZs)      | `var.allowed_ports[0]` (single item) or `var.allowed_ports` (whole list, e.g. in `for_each`) |
| `map`    | `{key = "value"}`     | Named/labeled values (tags)     | `var.extra_tags["Environment"]` (single value) or `var.extra_tags` (whole map, e.g. merged into `tags = merge(...)`) |


---

## Variable Files and Precedence

1. Create `terraform.tfvars`:
```hcl
project_name = "terraweek"
environment  = "dev"
instance_type = "t2.micro"
```

2. Create `prod.tfvars`:
```hcl
project_name = "terraweek"
environment  = "prod"
instance_type = "t3.small"
vpc_cidr     = "10.1.0.0/16"
subnet_cidr  = "10.1.1.0/24"
```

3. Apply with the default file:
```bash
terraform plan                              # Uses terraform.tfvars automatically
```

4. Apply with the prod file:
```bash
terraform plan -var-file="prod.tfvars"      # Uses prod.tfvars
```

5. Override with CLI:
```bash
terraform plan -var="instance_type=t2.nano"  # CLI overrides everything
```

6. Set an environment variable:
```bash
export TF_VAR_environment="staging"
terraform plan                              # env var overrides default but not tfvars
```


### Write the variable precedence order from lowest to highest priority.

Terraform variable precedence (lowest → highest) :
1. Default value (in `variable` block)
2. Environment variables (`TF_VAR_name`)
3. `terraform.tfvars` file
4. `*.auto.tfvars` file(s) (alphabetical order)
   - `*.auto.tfvars` files ---> convenient way to split variable values into multiple files, without needing to pass `-var-file` for each one — Terraform finds and loads them automatically in alphabetical order of file names.
5. `-var-file` flag (command line)
6. `-var` flag (command line)

---

## Add Outputs

Create an `outputs.tf` file with outputs for:

1. `vpc_id` -- the VPC ID
2. `subnet_id` -- the public subnet ID
3. `instance_id` -- the EC2 instance ID
4. `instance_public_ip` -- the public IP of the EC2 instance
5. `instance_public_dns` -- the public DNS name
6. `security_group_id` -- the security group ID

Apply your config and verify the outputs are printed at the end:
```bash
terraform apply

# After apply, you can also run:
terraform output                          # Show all outputs
terraform output instance_public_ip       # Show a specific output
terraform output -json                    # JSON format for scripting
```

---
---

When we did terraform output first we didn't added `enable_dns_support` and `enable_dns_hostnames` in `aws_vpc` block , so we didn't get public dns name for that instance.

![](Output%20Printed%20without%20DNS%20Name.png)
---

But after configuring dns_hostname in vpc block , we got instance's dns name on aws console but still not in output

![](Ec2%20Instance%20Deatils%20after%20modifying%20vpc.png)


**What and How to add in vpc block ?**

![](What%20to%20add%20in%20vpc%20block%20to%20enable%20dns.png)
---
---

Then we did `terraform destroy` and then again `terraform apply`

![](DNS%20Name%20printed%20sucessfully.png)

*Match the details*

![](Deatils%20matched%20successfully.png)
---

## Use Data Sources

Stop hardcoding the AMI ID. Use a data source to fetch it dynamically.

1. Add a `data "aws_ami"` block that:
   - Filters for Amazon Linux 2 images
   - Filters for `hvm` virtualization and `gp2` root device
   - Uses `owners = ["amazon"]`
   - Sets `most_recent = true`

2. Replace the hardcoded AMI in your `aws_instance` with `data.aws_ami.amazon_linux.id`

3. Add a `data "aws_availability_zones"` block to fetch available AZs in your region

4. Use the first AZ in your subnet: `data.aws_availability_zones.available.names[0]`




### What is a data source?

Unlike a `resource` block (which creates something new), a data block just reads/fetches existing information — from AWS, or anywhere else — without creating or managing it. Terraform uses this info as read-only reference data in your config.

```bash
data "<provider_type>" "<local_name>" {
  # filters/arguments
}

# Reference it like : data.<provider_type>.<local_name>.<attribute>
```
*Note :* Data sources make your config portable and future-proof — no hardcoded values that break when AWS changes AMIs or you deploy to a new region.


### What is the difference between a `resource` and a `data` source?

|                        | `resource`              | `data`                       |
|------------------------|--------------------------|-------------------------------|
| Purpose                | Creates/manages infra   | Reads existing info          |
| Terraform tracks it?   | Yes, in state            | Read-only, refreshed each run |
| Example                | Creates a new VPC        | Looks up for latest AMI ID       |


---
After adding data source for ami and region , we got no error and our infrastructure created sucessfully

![](Infrastructure%20before%20adding%20data%20sources.png)
---
But then we change the region from default(us-east-1) to ap-south-1 , the following things will get changed

![](changes%20that%20will%20happen%20after%20re%20applying.png)
---
Our infrastructure created sucessfully without any error , as it fetch ami automatically for ap-south-1 region we didn't have to hardcode it 

![](Infrastructure%20after%20adding%20data%20sources.png)
---
We can see the ami id and ami name are matching with our infrastructure , and also the instance is in ap-south-1 (can see in availability zone column)

![](Details%20matching%20with%20output.png)
---
---

## Use Locals for Dynamic Values

1. Add a `locals` block:
```hcl
locals {
  name_prefix = "${var.project_name}-${var.environment}"
  common_tags = {
    Project     = var.project_name
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
}
```

2. Replace all Name tags with `local.name_prefix`:
   - VPC: `"${local.name_prefix}-vpc"`
   - Subnet: `"${local.name_prefix}-subnet"`
   - Instance: `"${local.name_prefix}-server"`



### What is a local?
- A local value is like a named variable you compute yourself inside your config — unlike variable (input from user) or data (fetched from AWS), a local is a value you derive/calculate from other values, so you don't repeat the same expression everywhere.
- Think of it like a shortcut/nickname for a longer expression.

**Syntax**
```bash
locals {
  name_prefix = "${var.project_name}-${var.environment}"
  common_tags = {
    Project     = var.project_name
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
}
```


### Why use locals ?

```bash
tags = { Name = "${var.project_name}-${var.environment}-vpc" }
tags = { Name = "${var.project_name}-${var.environment}-subnet" }
tags = { Name = "${var.project_name}-${var.environment}-server" }
```
Repeating `"${var.project_name}-${var.environment}"` everywhere — if you ever change the naming pattern, you'd need to edit every resource.


But with locals

```bash
locals {
  name_prefix = "${var.project_name}-${var.environment}"
}
```

Now just reuse `local.name_prefix` everywhere 

```bash
tags = { Name = "${local.name_prefix}-vpc" }
tags = { Name = "${local.name_prefix}-subnet" }
tags = { Name = "${local.name_prefix}-server" }
```

Change the pattern once, and it updates everywhere automatically



---
Instance name using locals  : 

![](Instance%20name%20using%20locals.png)
---
VPC name using locals  : 

![](VPC%20name%20using%20locals.png.png)
---
Subnet name using locals  : 

![](Subnet%20name%20using%20locals.png.png)
---
---

## Built-in Functions and Conditional Expressions

Practice these in `terraform console`:
```bash
terraform console
```

1. **String functions:**
   - `upper("terraweek")` -> `"TERRAWEEK"`
   - `join("-", ["terra", "week", "2026"])` -> `"terra-week-2026"`
   - `format("arn:aws:s3:::%s", "my-bucket")`

   ### Usage example
   ```bash
   tags = {
     Name = upper(var.project_name)   # forces consistent casing
   }

   bucket = join("-", [var.project_name, var.environment, "logs"])  # "terraweek-dev-logs"
   ```

2. **Collection functions:**
   - `length(["a", "b", "c"])` -> `3`
   - `lookup({dev = "t2.micro", prod = "t3.small"}, "dev")` -> `"t2.micro"`
   - `toset(["a", "b", "a"])` -> removes duplicates

   ### Usage example
   ```bash
   instance_type = lookup({ dev = "t2.micro", prod = "t3.small" }, var.environment, "t2.micro"  # default if key not found )

   # toset() - required when a resource argument needs a set instead of list (e.g., some for_each blocks require sets)
   for_each = toset(var.allowed_ports)
   ```


3. **Networking function:**
   - `cidrsubnet("10.0.0.0/16", 8, 1)` -> `"10.0.1.0/24"`

   ### Usage example
   ```bash
   resource "aws_subnet" "public" {
   cidr_block = cidrsubnet(var.vpc_cidr, 8, 1)
   # vpc_cidr = "10.0.0.0/16" -> becomes "10.0.1.0/24"

   # cidrsubnet(prefix, newbits, netnum)
   
   # prefix — your VPC's CIDR block (e.g., "10.0.0.0/16")
   # newbits — how many extra bits to add to the prefix (this determines subnet size)
   # netnum — which subnet number you want (0, 1, 2, 3...)
   
      # netnum	Resulting CIDR
      # 0   	10.0.0.0/24
      # 1   	10.0.1.0/24
      # 2   	10.0.2.0/24
      # 3   	10.0.3.0/24
   }
   ```

---
Tried all commands

![](terraform%20console%20practice.png)
---


4. **Conditional expression** -- add this to your config:
```hcl
instance_type = var.environment == "prod" ? "t3.small" : "t2.micro"
```

Apply with `environment = "prod"` and verify the instance type changes.

---
Set environment to prod in terraform.tfvars file , and we got an t3.small instance 

![](t3-small%20instance%20created%20using%20conditional%20func.png)
---


### Five most useful functions :

- lookup()

   Fetches a value from a map by key, with a fallback default if the key doesn't exist. Great for environment-based configs.

   ```bash
   lookup({ dev = "t2.micro", prod = "t3.small" }, var.environment, "t2.micro")
   ```

- cidrsubnet()

   Calculates a subnet CIDR block from a larger VPC CIDR — auto-generates non-overlapping subnet ranges instead of manual math.

   ```bash
   cidrsubnet("10.0.0.0/16", 8, 1)  # -> "10.0.1.0/24"
   ```

- join()

   Combines list elements into a single string, separated by a delimiter — useful for  building resource names.

   ```bash
   join("-", ["terraweek", "dev", "server"])  # -> "terraweek-dev-server"
   ```


- length()

   Returns the number of items in a list/map/string. Commonly used with count to conditionally create resources.

   ```bash
   count = length(var.allowed_ports) > 0 ? 1 : 0
   ```


- format()

   Builds a formatted string using placeholders — useful for constructing ARNs, names, or any pattern-based string.

   ```bash
   format("arn:aws:s3:::%s/*", aws_s3_bucket.my_bucket.bucket)   
   ```


---

## *Points to remember*

- `terraform.tfvars` is loaded automatically. Any other `.tfvars` file needs `-var-file`
- `terraform console` is an interactive REPL for testing expressions and functions
- Data sources are read-only -- they fetch information, they don't create resources
- `merge()` combines two maps -- great for tags
- `terraform output -json` is useful when piping output into other scripts

