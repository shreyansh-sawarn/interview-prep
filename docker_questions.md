Difference between copy and add in dockerfile

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
