Here is a clear breakdown of the key points about **`git clone`** from the text:

> Source: https://www.atlassian.com/git/tutorials/setting-up-a-repository/git-clone

---

### 1. What Is `git clone`?
* Targets an **existing repository** and creates a **copy (clone)** of it in a new directory.
* The source can be on the **local filesystem** or a **remote machine** (over supported protocols).
* The most common way to get a **development copy** of a project from a central repository. Like `git init`, it's usually a **one-time** operation.
* Similar to `svn checkout`, **except** the result is a **full-fledged Git repository** with its own history and files, completely **isolated** from the original.

---

### 2. Repo-to-Repo Collaboration
* **SVN:** central repository ↔ working copies (different things).
* **Git:** **no distinction** — every copy is a complete repository. Collaboration means **pushing and pulling commits between repositories**.
* A "central" repo is only a **convention** (e.g. to replicate a centralized workflow), not something built into Git.

---

### 3. The Automatic `origin` Remote
* Cloning creates a remote called **`origin`** pointing back to the original repo, by:
  * creating refs to the remote branch heads under **`refs/remotes/origin`**, and
  * setting the **`remote.origin.url`** and **`remote.origin.fetch`** config variables.

```bash
git clone ssh://john@example.com/path/to/my-project.git
cd my-project
# Start working on the project
```
* The local folder is `my-project` (no `.git` suffix), because the local copy is **non-bare**.

---

### 4. Common Usages

| Command | Effect |
|---|---|
| `git clone <repo>` | Clone into a folder named after the repo |
| `git clone <repo> <directory>` | Clone into a specific folder |
| `git clone --branch <tag> <repo>` | Clone and check out a specific **tag** |
| `git clone --depth=1 <repo>` | **Shallow clone**: only the most recent commit |

* **Shallow clones** help with repos that have a **huge history**, which otherwise causes disk-space limits and long clone times.

---

### 5. Configuration Options

#### `--branch`
* Clone and check out a **specific branch** (or tag) instead of the one the remote `HEAD` points to (usually `main`).
```bash
git clone --branch <branch> <repo>
```

#### `--bare` vs. `--mirror`
| | `--bare` | `--mirror` |
|---|---|---|
| Working directory | ❌ None | ❌ None (implies `--bare`) |
| Can push/pull | ✅ | ✅ |
| Remote branches configured | ❌ No | ✅ Keeps remote tracking config |
| Refs copied | Branches & tags | **All** refs (extended refs too) |
| Use case | Create a **hosted** repo nobody edits directly | An **exact mirror**. Run `git remote update` to overwrite all refs from origin |

#### `--template`
```bash
git clone --template=<template_directory> <repo location>
```
* Applies a template directory (e.g. with hooks) to the new clone. See the `git init` notes.

* Full list of options: [official Git docs](https://git-scm.com/docs/git-clone).

---

### 6. Git URL Protocols

| Protocol | URL format | Notes |
|---|---|---|
| **SSH** | `ssh://[user@]host.xz[:port]/path/to/repo.git/` | Authenticated and enabled on most servers. You need credentials first |
| **Git** | `git://host.xz[:port]/path/to/repo.git/` | Git's own daemon on port **9418**. Like SSH but with **no authentication** |
| **HTTP(S)** | `http[s]://host.xz[:port]/path/to/repo.git/` | The web protocol. Git can use it too |

---

### Summary Takeaway
1. `git clone` creates a **copy of a target repo**, which is a complete, independent repository.
2. The target can be **local or remote**.
3. Git connects over **SSH, Git, or HTTP(S)** protocols.
4. Options like **`--branch`, `--depth`, `--bare`, `--mirror`, and `--template`** change what the clone contains. The clone automatically gets an **`origin`** remote.
