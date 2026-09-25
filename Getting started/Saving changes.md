Here is a clear breakdown of the key points about **Saving changes in Git** (`git add`) from the text:

> Source: https://www.atlassian.com/git/tutorials/saving-changes

---

### 1. What Does "Saving" Mean in Git?
* In a word processor, "saving" is a **file system operation** that overwrites or writes one file.
* In Git, the equivalent of "save" is a **commit** — an operation that acts on a **collection of files and directories** at once.

#### Git vs. SVN
| | SVN | Git |
|---|---|---|
| Model | **Centralized** | **Distributed** |
| Commit ("check-in") | Pushes directly to a central server → **needs internet** | Recorded **locally**, pushed later (e.g. `git push -u origin main`) |
| Robustness | Single point of failure (the server) | No single point of failure |

* The trio **`git add`**, **`git status`**, and **`git commit`** is used together to save a snapshot of the project's current state.

---

### 2. Other Saving-Related Mechanisms
* **The stash (`git stash`):** a temporary (ephemeral) storage area for changes that are **not ready to be committed**. It operates on the working directory.
* **`.gitignore`:** lets you tell Git to **ignore** specific files/directories so their changes are never saved.

---

### 3. What `git add` Does
* Moves a change from the **working directory** to the **staging area**.
* It tells Git: *"include this update in the next commit."*
* It does **not** change the repository history — nothing is recorded until you run **`git commit`**.
* Use **`git status`** alongside it to see the state of the working directory and staging area.

---

### 4. The Basic Workflow: Edit → Stage → Commit (→ Push)

```
Working Directory ──git add──▶ Staging Area ──git commit──▶ Commit History ──git push──▶ Remote
```

1. **Edit** files in the working directory.
2. **Stage** the changes you want with `git add`.
3. **Commit** the staged snapshot with `git commit`.
4. **Push** commits to a remote with `git push` so teammates can access them.

* `git reset` is used to **undo** a commit or a staged snapshot (including undoing a `git add`).

#### `git add` ≠ `svn add`
* `svn add` is called **once per file** to add it to the repository.
* `git add` works on **changes** — you call it **every time you modify** a file you want in the next commit. It seems redundant, but it keeps projects much more organized.

---

### 5. The Staging Area
* Think of it as a **buffer between the working directory and project history**.
* It is one of Git's **"three trees"**: working directory, staging area (index), and commit history.
* **Why it matters:** instead of committing *everything* since the last commit, you can group **related changes** into focused snapshots.
  * Make edits across many unrelated files, then split them into **logical, atomic commits**.
  * Atomic commits make it easier to **track down bugs** and **revert** changes with minimal side effects.

---

### 6. Common Options

| Command | What it does |
|---|---|
| `git add <file>` | Stage all changes in `<file>` |
| `git add <directory>` | Stage all changes in `<directory>` |
| `git add -p` | **Interactive** staging — choose individual chunks (hunks) of a file |

#### `git add -p` prompt keys
| Key | Action |
|---|---|
| `y` | Stage this chunk |
| `n` | Skip this chunk |
| `s` | Split into smaller chunks |
| `e` | Manually edit the chunk |
| `q` | Quit |

---

### 7. Examples

```bash
# Initial commit of a new project (like `svn import`)
git add .
git commit

# Add a new file (or record changes to an existing one)
git add hello.py
git commit
```

* Git makes **no distinction** between staging a new file and staging changes to an already-tracked file.

---

### Summary Takeaway
`git add` is the **first step** in saving a snapshot: it promotes changes from the **working directory** to the **staging area**. Confirm with **`git status`**, undo with **`git reset`**, and finally record the snapshot in history with **`git commit`**. The staging area is what lets you craft clean, atomic commits.
