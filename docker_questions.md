# Difference between copy and add in dockerfile

This is a **common Docker interview question**. The easiest way to remember it:

> **COPY = just copy files.**
> **ADD = copy files + some extra functionality.**

### `COPY`

Copies files/directories from the build context into the image.

```dockerfile
COPY package.json /app/
COPY . /app/
```

It's **simple and predictable**, so it's generally preferred when you only need to copy files.

---

### `ADD`

`ADD` can do everything `COPY` does, plus:

1. **Automatically extract local tar archives**
2. Support certain **remote URLs** in its source form

Example:

```dockerfile
ADD app.tar.gz /app/
```

A local tar archive can be automatically extracted into `/app`.

---

### Interview answer

> "`COPY` is used simply to copy files and directories from the build context into the Docker image. `ADD` has additional capabilities, such as automatically extracting local tar archives and supporting remote sources. Because `COPY` is more explicit and predictable, I generally prefer `COPY` unless I specifically need functionality provided by `ADD`."

### One important interview nuance

Don't say **"ADD is used to download files from any URL"** as your main explanation. Modern Docker guidance generally favors `COPY` for normal file copying, and remote downloads are usually better handled explicitly with tools like `curl`/`wget` in a `RUN` step.

### Quick memory

| `COPY`                      | `ADD`                                  |
| --------------------------- | -------------------------------------- |
| Copies files/directories    | Copies + extra features                |
| Simple & predictable        | More functionality                     |
| Usually preferred           | Use when you need its special behavior |
| No automatic tar extraction | Can extract local tar archives         |

**One-liner to memorize:**

> **"COPY is explicit file copying; ADD is COPY plus extra features, so I prefer COPY by default."**

# docker ps and docker ps -a difference

The core difference is that docker ps only shows currently active (running) containers, while docker ps -a (short for --all) shows all containers on your system, regardless of their status. [1, 2] 
## Quick Summary

| Command | What It Displays | Common Use Case |
|---|---|---|
| docker ps | Only running containers. | Checking active services, uptime, and port mappings. |
| docker ps -a | All containers (Running, Exited, Created, Paused). | Finding stopped containers to restart them or clean up disk space. |

------------------------------
## Key Differences Broken Down

* 
* Visibility: If a container crashes, exits successfully, or is stopped manually, docker ps will immediately hide it. Adding the -a flag unhides these dead or paused processes. [1, 2, 3, 4, 5] 
* The "STATUS" Column:
* In docker ps, the status will almost always read Up X hours/minutes.
   * In docker ps -a, you will see statuses like Exited (0) 5 minutes ago (successful shutdown) or Exited (137) (often out-of-memory crashes).
* Modern Equivalent: Note that docker ps is an older shorthand. In modern workflows, you can also use docker container ls (for running) and docker container ls -a (for all). [6, 7, 8, 9, 10] 
* 

## Pro-Tip for Cleanup
If docker ps -a reveals a massive list of old, stopped containers clogging your system, you can permanently wipe them out all at once using the docker container prune command. [11] 

# and is there something like docker ls?

Yes, but docker ls does not exist as a standalone command. [1] 
Because Docker manages many different types of resources, it uses ls (list) as a sub-command under specific categories. [2, 3, 4] 
## Modern Docker ls Commands
Instead of one generic docker ls, you use the specific resource you want to see:

* docker container ls: Lists containers. This is the exact modern equivalent of docker ps.
* docker image ls: Lists all downloaded or built images. This is the exact equivalent of docker images.
* docker volume ls: Lists all persistent data volumes.
* docker network ls: Lists all isolated or bridged networks.
* docker context ls: Lists different Docker environments you can connect to. [5, 6, 7, 8, 9] 

## Shorthand vs. Management Commands
Docker originally started with shortcuts like docker ps and docker images. To organize the tool better, Docker later introduced Management Commands which use the object sub-command structure: [10, 11, 12] 

| Old Shorthand | Modern Management Command |
|---|---|
| docker ps | docker container ls |
| docker ps -a | docker container ls -a |
| docker images | docker image ls |
| docker rm [id] | docker container rm [id] |
| docker rmi [id] | docker image rm [id] |

# docker exec complete command

The docker exec command runs a new command inside an already running container. [1] 
The complete command structure and the most common ways to use it are detailed below. [2] 
## Standard Command Syntax

docker exec [OPTIONS] CONTAINER COMMAND [ARG...]

## The Most Common "Complete" Command
If you want to open an interactive terminal session inside a running container, use this exact syntax:

docker exec -it <container_name_or_id> /bin/bash

(If the container is minimal and doesn't have bash, use /bin/sh or just sh instead). [3, 4, 5] 
------------------------------
## Key Options (-it) Explained

* -i (interactive): Keeps standard input (STDIN) open, allowing you to type.
* -t (tty): Allocates a pseudo-TTY, giving you a terminal prompt (like root@container:/#).
* -u (user): Runs the command as a specific user. (Example: -u root).
* -w (workdir): Changes the directory inside the container before running the command. [6, 7, 8, 9, 10] 

------------------------------
## Practical Examples## 1. Open an Interactive Shell (Most Popular) [11] 

docker exec -it my_web_server bash

## 2. Run a Single Command Without Entering (Non-Interactive) [12] 
You do not need -it if you just want to see a quick result: [13] 

docker exec my_postgres_db psql -U postgres -c "SELECT 1;"

## 3. Run a Command as Root (Bypass Default User) [14] 

docker exec -it -u root my_app_container sh

## 4. Check Internal Container Environment Variables [15] 

docker exec my_container env

# how are env variables used in docker?

Environment variables in Docker are used to configure applications without changing the underlying image or code. They are highly flexible and can be injected during the build phase, the runtime phase, or hardcoded inside a Dockerfile. [1, 2, 3, 4, 5] 
Here is exactly how to use them across different scenarios.
------------------------------
## 1. Passing Variables at Runtime (Most Common)
If you want to inject variables dynamically when starting a container, use the runtime flags with docker run. [6] 

* The -e Flag (Single Variable): Pass key-value pairs directly.

docker run -d -e DB_USER=admin -e DB_PASS=secret123 mysql

* The --env-file Flag (Multiple Variables): Point to a local file containing variables. This keeps secrets out of your terminal history.

# 1. Create a .env file
DB_USER=admin
DB_PASS=secret123
# 2. Run the container
docker run -d --env-file .env mysql

[7, 8, 9] 

------------------------------
## 2. Hardcoding Variables in a Dockerfile
If a variable is constant and safe to share (not a secret), you can bake it directly into the image using the ENV instruction. [10] 

FROM node:20# Set the variableENV PORT=3000ENV NODE_ENV=production
EXPOSE $PORTCMD ["node", "server.js"]

Note: Any variable set via ENV in a Dockerfile can still be overridden at runtime using the -e flag. [11, 12] 
------------------------------
## 3. Using Variables in Docker Compose
Docker Compose makes managing environment variables much cleaner for multi-container setups. You can define them directly in your docker-compose.yml file. [13, 14] 

version: '3.8'services:
  web:
    image: node:20
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - API_KEY=${EXTERNAL_API_KEY} # Pulls from your host machine's .env file

------------------------------
## 4. Viewing Active Variables Inside a Container
If you need to verify that your variables were injected correctly, use the docker exec command you learned earlier to list them. [15] 

docker exec <container_id_or_name> env

------------------------------
## ⚠️ Crucial Distinction: ENV vs. ARG
It is common to confuse ENV and ARG. They serve completely different purposes: [16, 17] 

| Feature | ENV (Environment Variable) | ARG (Build Argument) |
|---|---|---|
| When is it used? | Available during runtime while the container runs. | Available only during build time (docker build). |
| Is it in the final image? | Yes, it persists in the image metadata. | No, it disappears after the image is built. |
| Use Case | Database passwords, API URLs, app modes. | Setting software versions or download paths. |
