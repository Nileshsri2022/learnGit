Here is a clear breakdown of the key points about **Resetting, checking out & reverting** from the text:

> Source: https://www.atlassian.com/git/tutorials/resetting-checking-out-and-reverting

---

### 1. Why Compare These Three?
* `git reset`, `git checkout`, and `git revert` all **undo some kind of change**. `reset` and `checkout` can work on **commits or individual files**.
* Because they're so similar, they're **easy to mix up**. The trick: think about their effect on Git's **three trees**:

| Tree | Also called |
|---|---|
| **Working directory** | Your files on disk |
| **Staged snapshot** | The index / staging area |
| **Commit history** | The committed snapshots (`HEAD`, branches) |

---

### 2. One-Line Definitions
| Operation | Definition | Scope |
|---|---|---|
| **Checkout** | Moves the **`HEAD` ref** to a specified commit (updates the commit-history tree). A file-level checkout changes a file's contents to those of a specific commit. | Commit **or** file |
| **Revert** | Creates a **new commit that inverses** a specified commit | Commit **only** |
| **Reset** | Resets the **three trees** to match a specified commit; has 3 modes matching the 3 trees | Commit **or** file |

* **Checkout & reset** → **local/private** undos (they modify history → conflicts when pushing).
* **Revert** → **safe public** undo (creates new history, doesn't overwrite what teammates depend on).

---

### 3. ⭐ Quick Reference Table

| Command | Scope | Common use cases |
|---|---|---|
| `git reset` | Commit-level | Discard commits in a **private** branch or throw away uncommitted changes |
| `git reset` | File-level | **Unstage** a file |
| `git checkout` | Commit-level | **Switch branches** or **inspect old snapshots** |
| `git checkout` | File-level | **Discard changes** in the working directory |
| `git revert` | Commit-level | **Undo commits in a public branch** |
| `git revert` | File-level | *(N/A)* |

* **Scope rule:** no file path → operates on **whole commits**; with a file path → operates on **that file only**.

---

### 4. Commit-Level Operations

#### a) `git reset` — move a branch tip
```bash
git checkout hotfix
git reset HEAD~2      # move hotfix back two commits
```
* The two removed commits become **dangling/orphaned** → deleted at the next **garbage collection**.
* Your go-to for undoing **unshared** work: *"Oh crap, what am I doing? I should just start over."*

| Flag | Staged snapshot | Working directory |
|---|---|---|
| `--soft` | Not changed | Not changed |
| `--mixed` *(default)* | **Updated** to match commit | Not changed |
| `--hard` | **Updated** | **Updated** (changes lost!) |

* Think of the modes as defining the **scope** of the reset.

#### b) `git checkout` — move `HEAD`
```bash
git checkout hotfix   # switch branches: move HEAD + update working directory
git checkout HEAD~2   # inspect the grandparent of the current commit
```
* Unlike `reset`, checkout **doesn't move any branches**.
* Git forces you to **commit or stash** changes that would be overwritten.
* Checking out a commit → **detached `HEAD`**. New commits there are **unreachable** once you switch away → always **create a new branch** before committing in detached state.

#### c) `git revert` — undo public commits
```bash
git checkout hotfix
git revert HEAD~2     # new commit that undoes the changes of the 2nd-to-last commit
```
* **No chance of rewriting history** → safe on **public** branches. (`reset` alters history → **private** branches only.)
* Mental model: **`git revert`** undoes **committed** changes; **`git reset HEAD`** undoes **uncommitted** changes.
* Like checkout, it asks you to commit or stash changes that would be overwritten.

---

### 5. File-Level Operations
Passing a **file path** dramatically changes the behavior of `reset` and `checkout`.

#### a) `git reset <commit> <file>` → updates the **staged snapshot**
```bash
git reset HEAD~2 foo.py   # stage the version of foo.py from 2 commits ago
git reset HEAD foo.py     # most common: UNSTAGE foo.py (changes stay in working dir)
```
* `--soft` / `--mixed` / `--hard` have **no effect** here: stage is **always** updated; working dir **never**.

#### b) `git checkout <commit> <file>` → updates the **working directory**
```bash
git checkout HEAD~2 foo.py  # make foo.py match the version from 2 commits ago
git checkout HEAD foo.py    # most common: DISCARD unstaged changes to foo.py
```
* Does **not** move `HEAD` (you stay on your branch).
* Committing the checked-out file "reverts" that file — but note it removes **all subsequent changes** to the file, while `git revert` undoes **only** the changes of one specific commit.
* `git checkout HEAD foo.py` ≈ `git reset --hard HEAD`, but for **one file**.

---

### 6. Which Trees Does Each Command Touch?

| Command | Commit history | Staged snapshot | Working directory |
|---|---|---|---|
| `git reset --soft <commit>` | ✅ moves branch | — | — |
| `git reset --mixed <commit>` | ✅ | ✅ | — |
| `git reset --hard <commit>` | ✅ | ✅ | ✅ |
| `git checkout <branch/commit>` | ✅ moves `HEAD` only | ✅ | ✅ |
| `git revert <commit>` | ✅ adds new commit | ✅ | ✅ |
| `git reset <commit> <file>` | — | ✅ | — |
| `git checkout <commit> <file>` | — | — (and ✅ index in practice) | ✅ |

---

### Summary Takeaway
All three commands **undo** things, but differ in **what they touch**:
* **`reset`** → moves a branch tip (and optionally the stage/working dir). Use for **private** undos; with a file path, it **unstages**.
* **`checkout`** → moves `HEAD` to switch branches or inspect old commits; with a file path, it **discards working-directory changes**.
* **`revert`** → adds an **inverse commit**. The **safe choice for public branches**.

Think in terms of the **three trees** — working directory, staged snapshot, commit history — and choosing the right command becomes easy.
