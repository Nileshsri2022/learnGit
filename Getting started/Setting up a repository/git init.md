Here is a clear breakdown of the key points about **`git init`** from the text:

> Source: https://www.atlassian.com/git/tutorials/setting-up-a-repository/git-init

---

### 1. What Does `git init` Do?
* **Creates a new Git repository** — either converting an existing, unversioned project or starting a brand-new empty one.
* Usually the **first command** in a new project; most Git commands don't work outside an initialized repo.
* Creates a **`.git` subdirectory** with all repository metadata: subdirectories for **objects**, **refs**, and **template files**, plus a **`HEAD`** file pointing to the currently checked-out commit.
* The rest of your project is **left unchanged**. Unlike SVN, Git needs only **one** `.git` folder at the project root, not one in every subdirectory.

---

### 2. Usage

| Command | Effect |
|---|---|
| `git init` | Turn the **current directory** into a Git repo |
| `git init <directory>` | Create an **empty** repo in a new folder `<directory>` (containing only `.git`) |

* Much simpler than SVN: no need to create a repo on a server, import files, and check out a working copy. **No server or admin rights** are required.
* Running `git init` again on an existing repo is **safe** — it won't overwrite the existing `.git` configuration.

---

### 3. `git init` vs. `git clone`
| `git init` | `git clone` |
|---|---|
| Creates a **new, empty** repository | Creates a **copy of an existing** repository |
| — | Internally runs `git init`, then **copies the data** and **checks out** working files |

---

### 4. Bare Repositories — `git init --bare`
```bash
git init --bare <directory>
```
* Creates a repo with **no working directory**, so you **can't edit or commit** in it directly. You only `git push` to it and `git pull` from it.
* **Shared/central repositories should always be bare.** Pushing branches to a non-bare repo can **overwrite changes**.
* By convention, bare repos end in **`.git`** (e.g. `my-project.git`).
* Think of `--bare` as marking a repo as a **storage facility**, not a development environment.

| Repository | Bare? |
|---|---|
| Central repository | ✅ Bare |
| Developers' local repositories | ❌ Non-bare |

```bash
# Typical: create a central repo on a server
ssh <user>@<host>
cd path/above/repo
git init --bare my-project.git
# Developers then clone my-project.git
```

---

### 5. Templates — `--template`
```bash
git init <directory> --template=<template_directory>
```
* Copies files from `<template_directory>` into the new repo's **`.git`** directory.
* Default templates usually live in `/usr/share/git-core/templates` (the path varies by machine).
* **Main use:** start every new repo with pre-configured **Git hooks**.

---

### 6. Custom `.git` Location
* By default, config goes into `./.git`. You can change this with:
  * the **`$GIT_DIR`** environment variable, or
  * **`--separate-git-dir=<git dir>`**, which puts a text file in the project that links to the real `.git` directory.
* Use cases:
  * Keep **dotfiles** (`.bashrc`, `.vimrc`) in your home folder with the `.git` folder somewhere else.
  * Move a **very large history** to a separate high-capacity drive.
  * Keep a project in a **publicly accessible directory** (e.g. a web root) without exposing `.git`.
* Running it on an existing repo **moves** the `.git` directory to the new path.

---

### 7. All Options

| Option | Effect |
|---|---|
| `-q`, `--quiet` | Print only critical messages, errors, and warnings |
| `--bare` | Create a bare repository |
| `--template=<dir>` | Use templates from `<dir>` |
| `--separate-git-dir=<git dir>` | Store `.git` elsewhere and link to it |
| `--shared[=false\|true\|umask\|group\|all\|world\|everybody\|0xxx]` | Set Unix permissions controlling which users/groups can push and pull |

* Every form accepts a `<directory>` argument. If the directory doesn't exist, it's **created**.

---

### 8. Examples
```bash
# Version an existing code base
cd /path/to/code
git init
git add .
git commit

# Create a new bare repository
git init --bare /path/to/repo.git

# Create a template and initialize a repo from it
mkdir -p /path/to/template
echo "Hello World" >> /absolute/path/to/template/README
git init /new/repo/path --template=/absolute/path/to/template
cat /new/repo/path/.git/README   # template files are copied into .git/
```

---

### Summary Takeaway
`git init` creates the **`.git` directory** that turns any folder into a repository. It's a **one-time, safe-to-repeat** command. Use **`--bare`** for central/shared repos (no working directory), **`--template`** to pre-load hooks, and **`--separate-git-dir`** to keep the Git data somewhere else.
