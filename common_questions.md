Question 1
Explain CI/CD.

Answer:

CI stands for Continuous Integration.

Developers frequently merge their code into a shared repository.

Every commit automatically triggers

build
unit tests
code quality
security scans

CD stands for Continuous Delivery or Continuous Deployment.

Once the build succeeds, the application is deployed automatically through environments like DEV, SIT, UAT and Production with approvals where required.

The objective is to reduce manual work and increase deployment reliability.

Mention:

"In our project we implemented Azure DevOps multi-stage YAML pipelines."

Question 2
Explain the pipeline you built.

This is almost guaranteed.

Answer using STAR.

Situation:
Developers needed automated deployments.

Task:
Create reusable pipelines.

Action:
I created YAML templates that handled build, testing, SonarCloud analysis, artifact publishing to JFrog, and deployments across environments using Azure DevOps Environments with approvals.

Result:
Reduced manual deployment effort, standardized deployments, and improved release consistency.

Question 3
What is Terraform?

Answer

Terraform is an Infrastructure as Code tool.

Instead of manually creating cloud resources, we define infrastructure in configuration files.

Terraform compares the desired state with the existing infrastructure and generates an execution plan before applying changes.

Benefits include

version control
repeatability
consistency
easy rollback
Question 4
What is Terraform state?

Answer

Terraform state keeps track of all resources managed by Terraform.

It maps configuration to actual cloud resources.

Without state Terraform wouldn't know what already exists.

For teams, remote state is preferred to avoid conflicts.

Question 5
What are Terraform modules?

Answer

Modules allow reusable infrastructure code.

Instead of writing the same VM configuration multiple times, we create a module and reuse it with different variables.

Benefits

consistency
easier maintenance
less duplication
Question 6
Why Docker?

Answer

Docker packages the application along with its dependencies.

This ensures it behaves the same across development, testing and production.

Advantages include

portability
faster deployments
isolation
better resource utilization
Question 7
Difference between Docker and Virtual Machine?

Docker

shares host OS kernel
lightweight
starts in seconds

VM

full operating system
larger
slower startup
Question 8
What is Kubernetes?

Answer

Kubernetes is a container orchestration platform.

It automates

deployment
scaling
self-healing
rolling updates
service discovery

It manages containers running across multiple nodes.

Question 9
Difference between Pod and Container?

Answer

A container runs an application.

A Pod is the smallest deployable unit in Kubernetes.

A Pod can contain one or more containers that share networking and storage.

Question 10
Rolling update?

Answer

Rolling update replaces application instances gradually.

Old Pods are terminated one by one while new Pods are created.

This minimizes downtime.

Question 11
Blue Green deployment

Answer

Two identical environments exist.

Blue is live.

Green contains the new version.

Once testing completes, traffic switches from Blue to Green.

Rollback is fast because Blue still exists.

Question 12
Canary deployment

Answer

The new version is released to a small percentage of users first.

If monitoring looks healthy, traffic is gradually increased.

Question 13
What is SonarQube?

Answer

SonarQube performs static code analysis.

It checks

bugs
vulnerabilities
code smells
maintainability
test coverage

In our project we integrated SonarCloud into Azure DevOps pipelines.

Question 14
What is JFrog Artifactory?

Answer

Artifactory stores versioned build artifacts.

Instead of rebuilding every time, deployments use artifacts already stored in Artifactory.

Question 15
Explain Git branching.

Answer

Each developer works on a feature branch.

Changes are merged through Pull Requests after review.

After validation, code is merged into the main branch for deployment.

Question 16
Difference between merge and rebase?

Merge preserves branch history.

Rebase creates a cleaner linear history by replaying commits.

Merge is safer for shared branches.

Question 17
Production deployment failed. What do you do?

Answer

Check pipeline logs.
Identify failing stage.
Verify infrastructure connectivity.
Review application logs.
Check configuration changes.
Roll back if necessary.
Perform root cause analysis.
Document findings.
Question 18
Tell me about a difficult production issue.

Use your Kong Gateway work.

Explain

issue
investigation
root cause
solution
business impact

Interviewers love real stories.

Question 19
Why are you leaving TCS?

Never complain.

Say

"I've had a great opportunity to build strong DevOps experience at TCS. I'm now looking for broader technical challenges, greater ownership, and exposure to more cloud-native technologies while continuing to grow my career."

Question 20
Why should we hire you?

Answer

"I already have hands-on experience working in enterprise production environments. I've built CI/CD pipelines, automated deployments, worked with Terraform, Docker, Azure, SonarCloud, JFrog, production troubleshooting and cross-functional teams. I learn quickly and enjoy solving automation problems, so I believe I can start contributing with minimal ramp-up while continuing to expand my Kubernetes and cloud-native expertise."