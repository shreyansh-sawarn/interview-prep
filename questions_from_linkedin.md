
---

# 1. Terraform — create 5 EC2 instances with unique names and instance types

A clean answer using `for_each`:

```hcl
variable "instances" {
  default = {
    web1 = "t3.micro"
    web2 = "t3.small"
    app1 = "t3.medium"
    app2 = "t3.small"
    db1  = "t3.large"
  }
}

resource "aws_instance" "example" {
  for_each = var.instances

  ami           = "ami-xxxxxxxx"
  instance_type = each.value

  tags = {
    Name = each.key
  }
}
```

### What you say

> "I would use `for_each` because each instance has a unique key and potentially a different instance type. The key becomes the instance name and the value becomes the instance type."

### Why `for_each` is nice here

You get:

```text
web1 → t3.micro
web2 → t3.small
app1 → t3.medium
app2 → t3.small
db1  → t3.large
```

And Terraform tracks them individually.

---

# 2. Delete only ONE specific EC2 instance using Terraform

This is a **very likely practical question**.

Suppose:

```hcl
resource "aws_instance" "example" {
  for_each = var.instances
  ...
}
```

You can target a specific instance:

```bash
terraform destroy -target='aws_instance.example["web1"]'
```

Terraform will target only:

```text
aws_instance.example["web1"]
```

### Interview answer

> "If I need to remove only one Terraform-managed resource, I can use the `-target` option with the exact resource address. For a `for_each` resource, I specify the key, for example `terraform destroy -target='aws_instance.example["web1"]'`."

### Important caveat

If they ask whether you should routinely use `-target`:

> "No. Targeting is mainly intended for exceptional situations such as recovery or specific resource operations. Normally I prefer changing the configuration and allowing Terraform to calculate the complete plan."

That's a **very good senior-level detail**.

---

# 3. Write a Dockerfile for an application

They probably won't expect some huge Dockerfile.

A simple Node.js application:

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

### Explain it:

> "`FROM` selects the base image. `WORKDIR` sets the working directory. I copy the package files first so Docker can cache the dependency installation layer. Then I copy the application code, expose the application port and use CMD to start the application."

### ⭐ One thing to remember

This:

```dockerfile
COPY package*.json ./
RUN npm ci
COPY . .
```

is better than:

```dockerfile
COPY . .
RUN npm ci
```

because dependency installation can remain cached when only application code changes.

---

# 4. Conditional resource creation in Terraform

This is another pattern you should **memorize**.

Suppose they ask:

> "Create an EC2 instance only if `create_instance` is true."

```hcl
variable "create_instance" {
  type    = bool
  default = true
}

resource "aws_instance" "example" {
  count = var.create_instance ? 1 : 0

  ami           = "ami-xxxxxxxx"
  instance_type = "t3.micro"
}
```

If:

```hcl
create_instance = true
```

→ resource is created.

If:

```hcl
create_instance = false
```

→ zero instances.

### Interview answer

> "For simple conditional creation, I can use a boolean variable with a conditional expression: `count = var.create_instance ? 1 : 0`."

---

# 5. Terraform Variables vs Locals

This is straightforward.

### Variable

**Input to the module/configuration.**

```hcl
variable "environment" {
  type = string
}
```

You might provide:

```text
dev
sit
prod
```

### Local

**A value calculated or reused internally.**

```hcl
locals {
  app_name = "myapp-${var.environment}"
}
```

### Interview answer

> "Variables are inputs to a Terraform module and can be supplied from outside, whereas locals are internally defined values used to simplify expressions and avoid repeating logic."

### Easy memory trick

**Variable = outside → inside**

**Local = inside → reusable**

---

# 6. `count` vs `for_each`

This is **very likely**.

### count

Use when resources are essentially identical or indexed.

```hcl
resource "aws_instance" "server" {
  count = 5

  ami           = "ami-xxxx"
  instance_type = "t3.micro"
}
```

Terraform addresses them as:

```text
aws_instance.server[0]
aws_instance.server[1]
aws_instance.server[2]
...
```

### for_each

Use when each resource needs a meaningful unique key.

```hcl
resource "aws_instance" "server" {
  for_each = {
    web = "t3.micro"
    app = "t3.small"
  }

  ami           = "ami-xxxx"
  instance_type = each.value
}
```

Addresses:

```text
aws_instance.server["web"]
aws_instance.server["app"]
```

### Interview answer

> "`count` creates resources based on a numeric number and uses indexes, whereas `for_each` creates resources from a map or set and uses keys. I prefer `for_each` when resources have distinct identities or configurations because it gives more stable resource addressing."

### ⭐ Why this matters

Suppose:

```text
count = 3
```

and you remove the second item.

Terraform's indexes can shift.

With:

```hcl
for_each = {
  web = ...
  api = ...
  db  = ...
}
```

removing `api` doesn't change the identity of `web` or `db`.

---

# 7. Using `length()` with `count`

This is basically another pattern.

Suppose:

```hcl
variable "instances" {
  default = [
    "web",
    "api",
    "db"
  ]
}
```

You can create one resource per item:

```hcl
resource "aws_instance" "server" {
  count = length(var.instances)

  ami           = "ami-xxxx"
  instance_type = "t3.micro"

  tags = {
    Name = var.instances[count.index]
  }
}
```

`length(var.instances)` returns:

```text
3
```

Therefore:

```text
count = 3
```

And:

```hcl
var.instances[count.index]
```

gives:

```text
web
api
db
```

### Interview answer

> "`length()` can dynamically determine how many resources `count` should create based on the size of a list or set."

---

# 8. Accidentally deleted Terraform state file

This is an important scenario.

First, **don't immediately run `terraform apply`**.

The state file tells Terraform the relationship between the configuration and existing infrastructure.

### Interview answer

> "First I would check whether the state exists remotely or whether we have a backup. In a team environment I would normally use remote state such as an S3 backend with state locking and versioning enabled, so I could recover a previous version of the state."

If using S3 versioning, recover the previous state version.

### Important concept

Your infrastructure may still exist:

```text
AWS infrastructure
      ↓
still running
```

but Terraform may have lost its record:

```text
terraform.tfstate
      ↓
missing
```

So **don't recreate everything blindly**.

If there is no usable backup, you may need to reconstruct the state using:

```bash
terraform import
```

for the existing resources.

### Very good answer:

> "The infrastructure and Terraform state are separate. Deleting the state file does not automatically delete the infrastructure. I would recover the state from the remote backend or backup if possible; otherwise, I would import the existing resources into a newly managed state."

---

# 9. AWS ALB returning 502 Bad Gateway

This is a **classic troubleshooting question**.

Remember:

> **ALB received the request but couldn't get a valid response from the backend.**

Your investigation should be:

```text
Client
  ↓
ALB
  ↓
Target Group
  ↓
EC2 / Application
```

Check in that order.

### Interview answer

> "For an ALB 502, I would first check the target group's health and whether the targets are healthy. Then I would verify that the application is actually listening on the expected port and that the target group's port matches the application port.
>
> I would check security groups and network connectivity, and then check the application logs for crashes, connection resets or malformed responses.
>
> I would also verify the ALB listener and target group configuration and review ALB access logs if available."

### Checklist

**1. Target health**

```text
Healthy?
```

**2. Application running?**

```bash
systemctl status <service>
```

or application/container logs.

**3. Correct port?**

Example:

```text
ALB → 80
Target → 8080
Application → 8080
```

**4. Security groups**

ALB SG should be allowed to reach the target SG.

**5. Network**

Subnet / routing / NACL issues.

**6. Application logs**

Look for:

```text
connection reset
application crash
timeout
invalid response
```

---

# 10. Kubernetes pod can't connect to database

This one needs a **systematic answer**, not "check the database."

Think:

```text
Pod
 ↓
DNS
 ↓
Network
 ↓
DB port
 ↓
Database
 ↓
Authentication
```

### Interview answer

> "I would first verify whether the pod itself is healthy and then test connectivity from inside the pod. I would check DNS resolution of the database hostname, verify that the required port is reachable, and then check Kubernetes NetworkPolicies and cloud security groups or firewall rules.
>
> If network connectivity is successful, I would check the database credentials, connection string, SSL certificates if applicable, and database-side logs."

Useful commands:

```bash
kubectl get pods
kubectl describe pod <pod>
kubectl exec -it <pod> -- sh
```

Then inside the pod:

```bash
nslookup <db-host>
```

and depending on what's available:

```bash
nc -vz <db-host> <port>
```

### If DNS fails:

Investigate:

```text
CoreDNS
service/DNS configuration
hostname
```

### If DNS works but port fails:

Investigate:

```text
NetworkPolicy
Security Group
Firewall
routing
DB listener
```

### If port works but login fails:

Investigate:

```text
username/password
connection string
TLS
database permissions
```

That's a **very strong troubleshooting framework**.

---

# 11. Only 20 of 40 pods were created

This is another one I'd expect them to push you on.

Suppose:

```yaml
replicas: 40
```

but:

```text
kubectl get pods
```

shows only 20.

### Don't say:

> "I'll restart the deployment."

Instead:

> **Why can't Kubernetes satisfy the desired state?**

Start with:

```bash
kubectl get deployment
kubectl describe deployment <deployment>
kubectl get rs
kubectl get pods
kubectl describe pod <pod>
kubectl get events
```

### Possible causes

#### 1. Insufficient cluster resources

For example:

```text
CPU insufficient
Memory insufficient
```

Pods remain:

```text
Pending
```

#### 2. Node capacity

Not enough nodes/resources to schedule 40 pods.

#### 3. Resource requests too high

Example:

```yaml
resources:
  requests:
    cpu: "4"
    memory: "8Gi"
```

The scheduler may not find enough capacity.

#### 4. Pod scheduling constraints

Such as:

* nodeSelector
* affinity
* taints/tolerations
* topology constraints

#### 5. Quotas

Namespace might have:

```text
ResourceQuota
```

preventing more pods.

#### 6. HPA / controller behavior

Check whether another controller is modifying replicas.

### Interview answer

> "I would first check the Deployment and ReplicaSet to determine whether the desired replica count is actually 40. Then I would inspect the pending pods and Kubernetes events. If pods are Pending, I would look for insufficient CPU or memory, node capacity, taints, affinity rules, resource quotas or other scheduling constraints. If pods were created but immediately failed, I would investigate image pull errors, configuration errors or application crashes."

**That answer is excellent.**

---

# 12. What is Desired State in Kubernetes?

This is fundamental.

Suppose your YAML says:

```yaml
replicas: 5
```

That is your **desired state**.

Kubernetes continuously observes the actual state:

```text
Desired: 5
Actual: 3
```

The controller tries to reconcile:

```text
3 → 4 → 5
```

### Interview answer

> "Desired state is the state that we declare Kubernetes should maintain, usually through Kubernetes manifests. For example, if a Deployment specifies 5 replicas, Kubernetes continuously works to maintain 5 healthy pods. If one pod crashes and only 4 remain, the controller creates another pod to reconcile the actual state with the desired state."

### ⭐ This concept connects to everything

```text
YAML
 ↓
Desired State
 ↓
Kubernetes Controller
 ↓
Actual State
 ↓
Reconciliation
 ↓
Desired State
```

This is the **heart of Kubernetes**.

---

# 13. Why investigate pod issues in Kubernetes rather than Jenkins?

This is an important distinction because they're testing whether you understand **CI/CD vs runtime**.

### Jenkins

Primarily handles:

```text
Build
Test
Package
Deploy
```

### Kubernetes

Handles:

```text
Run
Schedule
Restart
Scale
Maintain
```

### Interview answer

> "Jenkins is responsible for the CI/CD workflow, such as building the image and deploying the application. Once the deployment has been successfully submitted to Kubernetes, Kubernetes becomes responsible for running and maintaining the pods.
>
> So if the pipeline succeeded but pods are crashing, pending or unable to connect to another service, I would investigate Kubernetes because that's where the runtime state exists.
>
> I would only go back to Jenkins if there is evidence that the deployment itself was incorrect—for example, the wrong image tag, incorrect manifest or failed deployment step."

### ⭐ This distinction is worth memorizing

**Jenkins answers:**

> "Did we build and deploy it?"

**Kubernetes answers:**

> "Is it running correctly?"

---

# The most important thing for YOUR interview

Looking at this list, I would divide your preparation into **three buckets**.

### 🟢 Bucket 1 — You should be very comfortable

These are highly learnable:

* Terraform variables vs locals
* `count` vs `for_each`
* `length()`
* conditional resource creation
* Dockerfile
* CMD vs ENTRYPOINT
* Deployment vs StatefulSet
* Kubernetes desired state
* CI/CD security

### 🟡 Bucket 2 — Memorize the troubleshooting framework

Don't memorize 30 commands. Remember the investigation path:

**ALB 502**

> ALB → Target Group → Target health → Port → SG/network → Application logs

**Kubernetes DB connectivity**

> Pod → DNS → Network → Port → Firewall/NetworkPolicy → Authentication → DB

**20/40 pods**

> Deployment → ReplicaSet → Pending pods → Events → Resources → Scheduling constraints → Quotas

### 🔴 Bucket 3 — Terraform practical coding

You should practice writing these **without looking at the answer**:

1. `count`
2. `for_each`
3. `length() + count`
4. conditional `count`
5. destroy one resource with `-target`
6. basic EC2 resource
7. basic Dockerfile

You **do not need to become a Terraform programmer overnight**. You need to be able to produce these small patterns on a whiteboard/editor and explain what you're doing.

And one particularly important thing: **if they ask you to write code and you forget syntax, don't freeze.** Explain the approach first:

> "I would use `for_each` here because the instances have unique identities. The map will contain the instance name and instance type. Then I'll use `each.key` for the Name tag and `each.value` for the instance type."

Even if you make a small syntax mistake afterward, you've demonstrated that you understand the Terraform concept.

Given the list in this image, **I would expect the interview to be much more scenario/troubleshooting-heavy than pure coding-heavy**, which is actually better for you.
