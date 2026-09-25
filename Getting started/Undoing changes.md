Here is a clear breakdown of the key points about **Undoing commits & changes in Git** from the text:

> Source: https://www.atlassian.com/git/tutorials/undoing-changes

---

### 1. Git Has No Traditional "Undo"
* Don't map Git to a word processor's `Ctrl+Z`. Git has its own vocabulary: **reset, revert, checkout, clean**, and more.
* **Metaphor:** Git is a **timeline management utility**.
  * **Commits** = snapshots / points of interest on the project timeline.
  * **Branches** = multiple timelines.
  * "Undoing" usually means **moving back in time**, or **to another timeline** where the mistake never happened.

---

### 2. Finding What Is Lost: Reviewing Old Commits
* `git log --oneline` lists recent commits, each with a unique **SHA-1 hash** used to travel through the timeline.

| Command | Purpose |
|---|---|
| `git log` | Commits of the **current branch** only (default) |
| `git log --branches=*` | Commits across **all branches** |
| `git branch -a` | List **all known branch names** |
| `git log <branch_name>` | Log a specific branch |
| `git checkout <commit>` | "Load" a saved snapshot into your working directory |

#### Detached `HEAD`
* Normally `HEAD` points to a **branch** (e.g. `main`).
* Checking out a **commit** makes `HEAD` point **directly to that commit** → **"detached HEAD"** state.
* Checking out an **old version of a single file** (`git checkout <commit> <file>`) does **not** move `HEAD` — you stay on your branch and can commit the old version as a new snapshot (a way to revert one file).

---

### 3. Viewing an Old Revision (Example)

```bash
git log --oneline
# b7119f2 Continue doing crazy things
# 872fa7e Try something crazy
# a1e8fb5 Make some important changes to hello.txt
# 435b61d Create hello.txt
# 9773e52 Initial import

git checkout a1e8fb5   # working dir now matches that commit exactly
# ...look around, compile, run tests, even edit — nothing is saved to the repo...
git checkout main      # return to the "current" state
```
* Once back on `main`, use **`git revert`** or **`git reset`** to undo unwanted changes.

---

### 4. Three Strategies to Undo a Committed Snapshot
Goal: undo `872fa7e Try something crazy`.

#### a) `git checkout` → new branch
```bash
git checkout a1e8fb5
git checkout -b new_branch_without_crazy_commit
```
* In detached `HEAD`, new commits become **orphaned** when you switch branches, and Git's **garbage collector** eventually deletes them permanently. Creating a branch prevents that.
* The new branch is a timeline where `872fa7e` never existed.
* ❌ Not suitable if you need to keep using the **original branch** (e.g. `main`).

#### b) `git revert` — safe for shared history
```bash
git revert HEAD
# e2f9a78 Revert "Try something crazy"   ← new inverse commit
# 872fa7e Try something crazy            ← still in history
# a1e8fb5 ...
```
* Creates a **new commit that inverts** the target commit; you keep working on the **same branch**.
* ✅ **Ideal for public/shared repositories.**
* ❌ Not ideal if you want a **curated, minimal** history.

#### c) `git reset` — cleanest history, local only
```bash
git reset --hard a1e8fb5
# a1e8fb5 Make some important changes to hello.txt
# 435b61d Create hello.txt
# 9773e52 Initial import
```
* The unwanted commits **disappear** from history — as if they never happened.
* ✅ Great for **local, unpushed** changes.
* ❌ If the commits were already pushed, `git push` will be **rejected** (Git thinks your branch is out of date because it's missing commits). Use `git revert` instead.

| Strategy | Keeps branch? | Rewrites history? | Safe on shared branches? |
|---|---|---|---|
| `checkout` + new branch | No (new timeline) | No | Yes |
| `revert` | Yes | No (adds a commit) | ✅ **Yes — preferred** |
| `reset --hard` | Yes | **Yes** (removes commits) | ❌ No |

---

### 5. Undoing the Last Commit — `git commit --amend`
* If the last commit was simply **premature**, don't remove it — **amend** it:
```bash
git add <more changes>
git commit --amend   # opens editor to edit the message; staged changes are added to the commit
```

---

### 6. Undoing Uncommitted Changes
Before committing, changes live in two places:

| Area | How to undo |
|---|---|
| **Working directory** (in sync with your file system) | Edit files normally; `git clean` removes untracked files; `git reset --mixed` / `--hard` also affect it |
| **Staging index** (populated by `git add`) | `git reset` (a `--mixed` reset moves staged changes back to the working directory) |

---

### 7. Undoing Public Changes
* **`git reset` = local undo.** Use it on **private** branches only.
* Resetting a **shared** branch and pushing it → Git **blocks** the push (branch is missing commits).
* **`git revert` = shared undo.** It never removes commits; teammates simply **pull** the new revert commit.

---

### Summary Takeaway
* Once committed, changes are **generally permanent** (and recoverable).
* Use **`git checkout`** to move around and review history.
* Use **`git revert`** to undo **shared/public** changes.
* Use **`git reset`** to undo **local/private** changes.
* Helpers: **`git log`** to find lost commits, **`git clean`** to remove uncommitted/untracked files, **`git add`** to modify the staging index, and **`git commit --amend`** to fix the last commit.
