How do you secure a CI/CD pipeline?
How do you troubleshoot a Docker container that won't start?
What is the difference between CMD and ENTRYPOINT in Docker?
What is a Kubernetes Deployment vs StatefulSet?


These are **very likely interview questions** for a DevOps/SRE role. I’ll give you answers you can actually say in an interview—not textbook answers.

### 1. How do you secure a CI/CD pipeline?

**Interview answer:**

> “I secure a CI/CD pipeline at multiple levels. First, I make sure secrets such as passwords, tokens and certificates are not hardcoded in the repository. I use secure mechanisms like Azure Key Vault or secret variables and restrict access through RBAC.
>
> Then I secure the source code by using branch protection, pull-request reviews and proper repository permissions.
>
> In the pipeline itself, I include security and quality gates such as SonarCloud for code quality and JFrog Xray or another vulnerability scanner for dependencies and container images. If a critical vulnerability is detected, the pipeline should fail.
>
> I also follow the principle of least privilege for service connections and pipeline agents, and restrict who can approve deployments to higher environments.
>
> Finally, I maintain audit logs and make sure production deployments require appropriate approvals.”

**If they ask specifically about your experience:**

> “In my current project, I have worked with Azure DevOps pipelines where we use SonarCloud and JFrog Xray scanning, secure files and variable groups. For secrets and certificates, we avoid putting sensitive values directly into YAML.”

That's a **strong answer for your actual experience**.

---

## 2. How do you troubleshoot a Docker container that won't start?

Don't panic and start randomly changing things. Give them a systematic process.

**Interview answer:**

> “First, I check the container status and logs because the logs usually tell me why the container exited.
>
> I would use `docker ps -a` to see whether the container exited, and then `docker logs <container>` to check the application error.
>
> I would also inspect the container configuration using `docker inspect`, particularly the entrypoint, command, environment variables, mounts and networking.
>
> If necessary, I would run the image interactively with a shell to troubleshoot inside the container.
>
> I would also verify whether the application is listening on the expected port, whether required environment variables or secrets are present, and whether there are permission or volume-mount issues.
>
> Finally, I would check the image itself and rebuild it if there is an issue with the Dockerfile or missing dependencies.”

### Remember this flow:

**Status → Logs → Inspect → Enter container → Config → Dependencies**

Useful commands:

```bash
docker ps -a
docker logs <container>
docker inspect <container>
docker run -it <image> /bin/sh
```

### A good follow-up if they ask:

**“What if the container exits immediately?”**

Say:

> “I would first check the logs and the container's CMD or ENTRYPOINT. A common reason is that the main process has completed or crashed. Since a Docker container lives as long as its main process is running, if that process exits, the container stops.”

That last sentence is **very interview-friendly**.

---

# 3. What is the difference between CMD and ENTRYPOINT?

This is one you should know very clearly.

### Simple explanation

**ENTRYPOINT = what the container is supposed to run.**

**CMD = default arguments or default command.**

For example:

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

Running:

```bash
docker run myimage
```

effectively runs:

```bash
python app.py
```

But if you run:

```bash
docker run myimage test.py
```

the CMD is replaced, so it effectively becomes:

```bash
python test.py
```

### Interview answer

> “ENTRYPOINT defines the main executable that the container is intended to run, whereas CMD provides default arguments or a default command.
>
> CMD can easily be overridden when running the container, while ENTRYPOINT is generally used to make the container behave like a specific executable.
>
> They can also be used together—for example, ENTRYPOINT can define the application and CMD can provide its default arguments.”

### One important distinction

There are also **shell form** and **exec form**:

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

is generally preferable to:

```dockerfile
ENTRYPOINT python
CMD python app.py
```

for predictable signal handling and process behavior.

Don't overcomplicate this unless they ask.

---

# 4. Kubernetes Deployment vs StatefulSet

This is **very important for your interviews**, especially because Kubernetes questions have come up in your previous interviews.

The easiest way to remember it:

> **Deployment = interchangeable pods.**
> **StatefulSet = pods with identity and persistence.**

### Deployment

Used primarily for **stateless applications**.

Examples:

* REST APIs
* frontend applications
* microservices
* web servers

If you have:

```text
Deployment
 ├── Pod
 ├── Pod
 └── Pod
```

the pods are essentially interchangeable.

They might have names such as:

```text
app-7d8f9c-x1
app-7d8f9c-x2
app-7d8f9c-x3
```

If one dies, Kubernetes can create another equivalent pod.

---

### StatefulSet

Used for applications that require **stable identity and/or persistent storage**.

Examples:

* databases
* Kafka
* ZooKeeper
* some clustered systems

Pods have predictable identities:

```text
database-0
database-1
database-2
```

If `database-1` dies and Kubernetes recreates it, it comes back as:

```text
database-1
```

rather than getting an arbitrary new identity.

StatefulSets can also associate pods with persistent volumes.

---

### Interview answer

> “A Deployment is generally used for stateless applications where pods are interchangeable. Kubernetes can create, destroy and replace those pods without the application caring about the individual pod identity.
>
> A StatefulSet is used for stateful applications where pods need stable identity, stable network naming or persistent storage. StatefulSet pods have predictable names such as `app-0`, `app-1`, and `app-2`, and their associated storage can persist across pod recreation.
>
> So the simple distinction is: Deployment is generally for stateless workloads, while StatefulSet is for workloads that need identity and persistent state.”

### If they ask: **“Can you use a Deployment for a database?”**

Don't simply say **“No.”**

Say:

> “Technically you can run a database using a Deployment, but it is generally not appropriate for a database that requires stable identity and persistent storage. StatefulSet is designed specifically for those requirements.”

That's a much better answer.

---

## 🔥 The 4 answers you should memorize

If you're anxious during the interview, reduce everything to these four mental models:

| Question                       | Remember                                                                   |
| ------------------------------ | -------------------------------------------------------------------------- |
| **Secure CI/CD?**              | Secrets → RBAC → scanning → approvals → audit                              |
| **Docker won't start?**        | `ps -a` → `logs` → `inspect` → shell → config/dependencies                 |
| **CMD vs ENTRYPOINT?**         | ENTRYPOINT = executable, CMD = defaults/arguments                          |
| **Deployment vs StatefulSet?** | Deployment = interchangeable/stateless, StatefulSet = identity/persistence |

And **don't try to sound overly sophisticated**. For a face-to-face interview, a clear 30–60 second answer followed by a concrete example from your TCS work is much stronger than reciting a Kubernetes textbook.
