Here is a clear breakdown of the key points about **Git Bash** from the text:

> Source: https://www.atlassian.com/git/tutorials/git-bash

---

### 1. Why Git Bash Exists
* At its core, Git is a set of **command-line utilities** designed for a **Unix-style** command-line environment.

| OS | Terminal | Fit with Git |
|---|---|---|
| **Linux / macOS** | Built-in **Unix** terminals | ✅ Natural fit |
| **Windows** | **Command Prompt** — a *non-Unix* environment | ❌ Needs help |

* On Windows, Git is often bundled inside **GUI applications** that hide Git's underlying primitives.
  * 👍 Great for **beginners** to contribute quickly.
  * 👎 As team collaboration grows, you need to understand how **raw Git** works → time to drop the GUI for the **command line**.
* **Git Bash** provides that terminal Git experience on Windows.

---

### 2. What Is Git Bash?
* A Windows application that provides an **emulation layer** for a Git command-line experience.
* **Bash** = **B**ourne **A**gain **Sh**ell.
  * A **shell** is a terminal application for interacting with the OS through written commands.
  * Bash is the popular default shell on Linux and macOS.
* **Git Bash package =** Bash + common Bash utilities + Git, all for Windows.

---

### 3. Installing Git Bash
1. Download **[Git for Windows](https://gitforwindows.org/)** (Git Bash is included).
2. Install it like any other Windows application.
3. Run the included **`.exe`** to open Git Bash.

---

### 4. Using Git Bash — Navigating Folders
Git Bash works like standard Bash. Key navigation commands vs. their Windows equivalents:

| Bash command | Meaning | Windows (cmd) equivalent |
|---|---|---|
| `pwd` | **P**rint **w**orking **d**irectory — where your session is | `cd` (with no arguments) |
| `ls` | **List** contents of the current directory | `dir` |
| `cd <dir>` | **C**hange **d**irectory | `cd <dir>` |

```bash
pwd                 # /c/Users/you
ls                  # list files
cd projects/my-repo # move into a folder
```

---

### 5. Extra Commands Included
* Extra commands live in **`/usr/bin`** of the emulation — Git Bash offers a fairly **robust shell** on Windows.
* Includes tools like:

| Command | Purpose |
|---|---|
| `ssh` | Secure shell connections (also used for Git SSH auth) |
| `scp` | Secure file copy |
| `cat` | Print file contents |
| `find` | Search for files |

* Plus the **full set of Git core commands**: `git clone`, `git commit`, `git checkout`, `git push`, and more.

---

### Summary Takeaway
Git was built for **Unix-style terminals**, which Linux and macOS have built in but Windows lacks. **Git Bash** (installed with **Git for Windows**) fills that gap by bundling **Bash, common Unix utilities (`ssh`, `scp`, `cat`, `find`…), and Git** — letting Windows users learn and use **raw Git commands** exactly like everyone else, with familiar navigation via `pwd`, `ls`, and `cd`.
