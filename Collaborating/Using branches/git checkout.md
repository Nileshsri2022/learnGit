Here is a clear breakdown of the key points about **`git checkout`** from the text:

> Source: https://www.atlassian.com/git/tutorials/using-branches/git-checkout

---

### 1. What Is a "Checkout"?
* In Git, a checkout means **switching between different versions** of a target entity.
* `git checkout` works on **three entities**: **files**, **commits**, and **branches**.
* This page focuses on **branches**. Like checking out old commits/files, it updates the working directory, but **new commits are recorded on the branch**, so it's **not read-only**.

| `git clone` | `git checkout` |
|---|---|
| Fetches code **from a remote** repository | Switches between versions **already on your machine** |

---

### 2. Checking Out Branches
* Navigates between branches created with `git branch`:
  * Updates the working directory to match the branch.
  * Tells Git to **record new commits on that branch**.
* Think of it as **selecting which line of development** you're working on.
* A branch per feature (a big shift from SVN) makes experiments **safe** and lets you work on **many unrelated features** at once.

---

### 3. Usage

#### Existing branches
```bash
git branch
# main
# another_branch
# feature_inprogress_branch
git checkout feature_inprogress_branch
```

#### New branches
| Command | Effect |
|---|---|
| `git branch new_branch` + `git checkout new_branch` | Create, then switch |
| `git checkout -b <new-branch>` | **Create and switch** in one step (runs `git branch` first) based on current `HEAD` |
| `git checkout -b <new-branch> <existing-branch>` | Create based on `<existing-branch>` instead of `HEAD` |

#### Switching branches
```bash
git checkout <branchname>   # points HEAD at the tip of <branchname>
```
* Every checkout is recorded in the **reflog** (`git reflog`).

---

### 4. Checking Out a Remote Branch
First fetch the branch:
```bash
git fetch --all
```

| Git version | Command |
|---|---|
| **Modern** Git | `git checkout <remotebranch>`: automatically creates a local tracking branch |
| **Older** Git | `git checkout -b <remotebranch> origin/<remotebranch>` |
| Alternative | `git checkout -b <branchname>` then `git reset --hard origin/<branchname>` |

---

### 5. Detached `HEAD`
* **`HEAD`** = Git's pointer to the **current snapshot**. `git checkout` just moves `HEAD` to a **branch** or a **commit**.

| `HEAD` points to… | State |
|---|---|
| A **branch** | ✅ Attached (normal) |
| A **commit** (or tag, or remote branch) | ⚠️ **Detached HEAD** |

```
Attached:   A──B──C  ← main ← HEAD

Detached:   A──B──C  ← main
               ↑
              HEAD ──▶ new commits D──E have NO branch pointing to them
```
* Commits made while detached have **no branch reference**. After you check out something else, you can't easily get back to them (except through the reflog).
* ✅ **Always develop on a branch.** A detached HEAD is fine for just **looking** at old commits.
* If you did commit while detached: `git checkout -b <new-branch>` saves the work.

> 📝 *Extra note (not in the original article):* Modern Git splits `git checkout` into two clearer commands: **`git switch`** for branches (`git switch <branch>`, `git switch -c <new-branch>`) and **`git restore`** for files. `git checkout` still works.

---

### Summary Takeaway
On branches, `git checkout` **moves `HEAD`**. Use it to **switch** (`git checkout <branch>`), **create and switch** (`-b`), and **check out remote branches** after `git fetch`. Checking out a commit gives a **detached HEAD**, so always create a branch before committing. Together with **`git merge`**, it's the backbone of every Git workflow.
