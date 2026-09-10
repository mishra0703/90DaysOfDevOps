# Day 62 -- Providers, Resources and Dependencies

## Task
Yesterday we created standalone resources. But real infrastructure is connected -- a server lives inside a subnet, a subnet lives inside a VPC, a security group controls what traffic gets in. Today we will build a complete networking stack on AWS and learn how Terraform figures out what to create first.

Understanding dependencies is what separates a Terraform beginner from someone who can build production infrastructure.

---

## Explore the AWS Provider

1. Create a new project directory: `terraform-aws-infra`
2. Write a `providers.tf` file:
   - Define the `terraform` block with `required_providers` pinning the AWS provider to version `~> 5.0`
   - Define the `provider "aws"` block with your region
3. Run `terraform init` and check the output -- what version was installed?
4. Read the provider lock file `.terraform.lock.hcl` -- what does it do?
    
    
### What does `.terraform.lock.hcl` do ?    
- `.terraform.lock.hcl` locks the exact provider versions used in our project.
- When we run terraform init, Terraform records the exact provider versions (and their checksums) it downloaded into this file. Next time when we run `terraform init`, Terraform uses these locked versions instead of grabbing the latest ones.
- Ensures everyone on the team (and CI/CD) uses the same provider version — avoids "works on my machine" issues from version mismatches


### What does `~> 5.0` mean? How is it different from `>= 5.0` and `= 5.0.0`?
- `~> 5.0` (pessimistic/tilde constraint)  : Allows any version within the same major.minor set — so 5.0.x, 5.1.x, ... up to (but not including) 6.0
- `>= 5.0` Allows version `5.0` and anything above
- `= 5.0.0` Exact version only — always uses `5.0.0`

*Note :* `~> 5.0` means `>= 5.0`, `< 6.0` . If you write `~> 5.0.0` , it's stricter — allows only `5.0.x` , patch updates only.


---

## Build a VPC from Scratch

1. `aws_vpc` -- CIDR block `10.0.0.0/16`, tag it `"TerraWeek-VPC"`
2. `aws_subnet` -- CIDR block `10.0.1.0/24`, reference the VPC ID from step 1, enable public IP on launch, tag it `"TerraWeek-Public-Subnet"`
3. `aws_internet_gateway` -- attach it to the VPC
4. `aws_route_table` -- create it in the VPC, add a route for `0.0.0.0/0` pointing to the internet gateway
5. `aws_route_table_association` -- associate the route table with the subnet


---
![](VPC%20Resource%20Map.png)
---
![](Subnet%20Created.png)
---
![](IGW%20Created.png)
---
![](Route%20Table%20created%20pointing%20to%20the%20IGW.png)
---
![](Route%20Table%20associated%20with%20subnet.png)
---

## Understand Implicit Dependencies

1. The subnet references `aws_vpc.main.id` -- this is an implicit dependency
2. The internet gateway references the VPC ID -- another implicit dependency
3. The route table association references both the route table and the subnet

### How does Terraform know to create the VPC before the subnet?
- Terraform builds a dependency graph by scanning your config for references like `aws_vpc.my_vpc.id`. 
- Whenever one resource's argument references another resource's attribute, Terraform automatically knows the referenced resource must be created first. 
- No manual ordering needed — it's all inferred from these references.


### What would happen if you tried to create the subnet before the VPC existed?
- It would fail because the subnet needs a real vpc_id value, which doesn't exist until the VPC is actually created (AWS assigns the VPC ID only after creation). 
- Terraform wouldn't even let this happen normally since it auto-orders based on dependencies, but if forced, AWS would reject the API call with an error.


### Find all implicit dependencies in your config and list them

- `aws_subnet.public` → depends on `aws_vpc.my_vpc` (via vpc_id = `aws_vpc.my_vpc.id`)
- `aws_internet_gateway.my_igw` → depends on `aws_vpc.my_vpc` (via vpc_id = `aws_vpc.my_vpc.id`)
- `aws_route_table.rtable` → depends on `aws_vpc.my_vpc` (via vpc_id = `aws_vpc.my_vpc.id`)
- `aws_route_table.rtable` → depends on `aws_internet_gateway.my_igw` (via gateway_id = `aws_internet_gateway.my_igw.id`)
- `aws_route_table_association.rtable_with_subnet` → depends on `aws_subnet.public` (via subnet_id = `aws_subnet.public.id`)
- `aws_route_table_association.rtable_with_subnet` → depends on `aws_route_table.rtable` (via route_table_id = `aws_route_table.rtable.id`)

***Build order* Terraform figures out :** VPC → (Subnet + Internet Gateway) → Route Table → Route Table Association

---

## Add a Security Group and EC2 Instance

Add in main.tf :

1. `aws_security_group` in the VPC:
   - Ingress rule: allow SSH (port 22) from `0.0.0.0/0`
   - Ingress rule: allow HTTP (port 80) from `0.0.0.0/0`
   - Egress rule: allow all outbound traffic
   - Tag: `"TerraWeek-SG"`

2. `aws_instance` in the subnet:
   - Use Amazon Linux 2 AMI for your region
   - Instance type: `t2.micro`
   - Associate the security group
   - Set `associate_public_ip_address = true`
   - Tag: `"TerraWeek-Server"`

---
![](Ec2%20Created%20with%20Public%20IP%20and%20Custom%20Private%20IP.png)
---

## Explicit Dependencies with depends_on

Sometimes Terraform cannot detect a dependency automatically.

1. Add a second `aws_s3_bucket` resource for application logs
2. Add `depends_on = [aws_instance.main]` to the S3 bucket -- even though there is no direct reference, you want the bucket created only after the instance
3. Run `terraform plan` and observe the order

Now visualize the entire dependency tree:
   ```bash
   terraform graph | dot -Tpng > graph.png
   ```

If you don't have `dot` (Graphviz) installed, use
   ```bash
   terraform graph
   ```
and paste the output into an online Graphviz viewer.

---
![](Graph%20Before%20adding%20explicit%20dependency.png)
---
---
![](Graph%20After%20adding%20explicit%20dependency.png)
---

### When would you use `depends_on` in real projects? Give two examples.
- Only when there's a hidden dependency Terraform can't detect automatically — e.g., the S3 bucket needs the instance to exist first for some external reason (like an app running on the instance that writes logs to S3, but you're not passing any instance attribute into the bucket config).
- If there's not any kind of depency, then `depends_on` will just add unnecessary ordering/slowness. 
- Use it only when there's a real logical dependency Terraform wouldn't catch.


---

## Lifecycle Rules and Destroy

1. Add a `lifecycle` block to your EC2 instance:
```bash
lifecycle {
  create_before_destroy = true
}
```
2. Change the AMI ID to a different one and run `terraform plan` -- observe that Terraform plans to create the new instance before destroying the old one

3. Destroy everything:
```bash
terraform destroy
```
4. Watch the destroy order -- Terraform destroys in reverse dependency order. Verify in the AWS console that everything is cleaned up.

---
### Terraform is planning to create resource without even deleting it 
![](Planning%20to%20create%20before%20destroying.png)
---
### What are the three lifecycle arguments (`create_before_destroy`, `prevent_destroy`, `ignore_changes`) and when would you use each?

- `create_before_destroy`  :  Creates the new resource first, then destroys the old one (instead of default: destroy old → create new).
   - Use for resources where downtime matters — e.g., an EC2 instance serving traffic, or a security group in use. Avoids gaps where nothing exists.
- `prevent_destroy`  :  Blocks Terraform from destroying the resource — terraform destroy or apply (if it would delete this resource) will error out instead.
   - Use for critical resources we never want accidentally deleted — e.g., production database, S3 bucket with important data.
- `ignore_changes`  :  Tells Terraform to ignore changes to specific attributes — even if they drift from what's in your config, Terraform won't try to "fix" them.
   - Use When something outside Terraform modifies an attribute (e.g., auto-scaling changes desired_count, or AWS auto-assigns tags) and we don't want Terraform fighting against those changes.

---
### Destruction Order
![](Destruction%20Order.png)
---

## *Points to remember*
- `aws_vpc.main.id` syntax: `<resource_type>.<resource_name>.<attribute>`
- Use `terraform fmt` to keep your HCL clean
- If you cannot SSH into the instance, check: security group rules, public IP, route table, internet gateway
- `terraform graph` outputs DOT format -- paste it into webgraphviz.com if you don't have Graphviz
- Always destroy resources when done to avoid AWS charges

---
