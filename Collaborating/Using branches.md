Here is a clear breakdown of the key points about **Using branches** (`git branch`) from the text:

> Source: https://www.atlassian.com/git/tutorials/using-branches

---

### 1. What Is a Git Branch?
* A branch is effectively a **pointer to a snapshot** of your changes.
* For every new feature or bug fix — big or small — you **spawn a new branch** to encapsulate the changes.
* **Benefits:**
  * Harder for **unstable code** to get merged into the main code base.
  * Lets you **clean up** a feature's history before merging.
  * Work on **multiple features in parallel** while keeping `main` free of questionable code.

---

### 2. Why Git Branches Are Lightweight
| | Other VCS (e.g. SVN) | Git |
|---|---|---|
| Implementation | Copies files from directory to directory | Stores a branch as a **reference to a commit** |
| Cost | Expensive in time & disk space | Nearly free |
| Usage | Occasional, large-scale efforts | **Everyday** workflow |

* A branch is the **tip of a series of commits**, not a *container* for commits. A branch's history is derived from **commit relationships** (parent pointers).

---

### 3. How It Works
* A branch = an **independent line of development**.
* Think of it as requesting a **brand-new working directory, staging area, and project history**.
* New commits are recorded on the **current branch**, creating a **fork** in project history.
* `git branch` can **create, list, rename, and delete** branches — but **not** switch branches or merge them. For that, use:
  * **`git checkout`** (or `git switch`) — switch branches.
  * **`git merge`** — join forked histories.

---

### 4. Common Options

| Command | What it does |
|---|---|
| `git branch` / `git branch --list` | List all local branches |
| `git branch <branch>` | Create a new branch (does **not** check it out) |
| `git branch -d <branch>` | **Safe** delete — refuses if the branch has unmerged changes |
| `git branch -D <branch>` | **Force** delete — even with unmerged changes (throws those commits away) |
| `git branch -m <branch>` | Rename the **current** branch |
| `git branch -a` | List **all** branches, including remote ones |

---

### 5. Creating Branches
* Creating a branch only creates a **new pointer** — the repository history is **unchanged**.

```bash
git branch crazy-experiment   # new pointer at the current commit
git checkout crazy-experiment # select it before committing
# ...then the usual git add / git commit
```

---

### 6. Creating Remote Branches
* First configure a remote, then push the branch to it:

```bash
git remote add new-remote-repo https://bitbucket.com/user/repo.git  # add remote to local config
git push new-remote-repo crazy-experiment                           # push a copy of the branch
```

---

### 7. Deleting Branches

#### Local — safe delete (after merging)
```bash
git branch -d crazy-experiment
```
If not merged yet, Git protects you:
```
error: The branch 'crazy-experiment' is not fully merged.
If you are sure you want to delete it, run 'git branch -D crazy-experiment'.
```

#### Local — force delete (e.g. a failed experiment)
```bash
git branch -D crazy-experiment   # no warnings — use judiciously!
```

#### Remote branch
Local deletes don't touch remote copies. To delete on the remote:
```bash
git push origin --delete crazy-experiment
# or the older syntax
git push origin :crazy-experiment
```

---

### Summary Takeaway
Git branches are **cheap pointers to commits**, so you should use them **constantly** — one per feature or fix. `git branch` **creates, lists, renames, and deletes** branches (`-d` safe, `-D` force, `-m` rename, `-a` all). Pair it with **`git checkout`/`git switch`** to move between branches and **`git merge`** to combine them. This cheap branching is what makes workflows like **Feature Branch, Gitflow, and Forking** possible.
