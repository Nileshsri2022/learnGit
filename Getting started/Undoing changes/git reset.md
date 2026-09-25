Here is a clear breakdown of the key points about **`git reset`** from the text:

> Source: https://www.atlassian.com/git/tutorials/undoing-changes/git-reset

---

### 1. What Is `git reset`?
* A **complex, versatile tool for undoing changes**, with three main modes: **`--soft`, `--mixed`, `--hard`**.
* Each mode maps to how far the reset reaches into Git's **"three trees"**.

---

### 2. The Three Trees of Git
"Trees" isn't strictly accurate. They're node/pointer data structures Git uses to track edits.

| Tree | What it holds | How to inspect it |
|---|---|---|
| **Working Directory** | Your files on disk; immediate edits | `git status` (red, "modified") |
| **Staging Index** | Changes promoted with `git add` for the next commit (an internal cache) | `git ls-files -s` (mode, **blob SHA**, stage number, path) |
| **Commit History** (`HEAD`) | Permanent snapshots made with `git commit` | `git log` |

* ⚠️ **`git status` doesn't show the trees themselves**. It shows the **differences between** them.
* When you `git add` a file, its blob SHA in `git ls-files -s` changes (e.g. `e69de29…` → `d7d77c1…`).

```bash
mkdir git_reset_test && cd git_reset_test && git init .
touch reset_lifecycle_file && git add reset_lifecycle_file
git commit -m "initial commit"
echo 'hello git reset' > reset_lifecycle_file   # Working Directory change
git add reset_lifecycle_file                    # → Staging Index
git ls-files -s                                  # see the new blob SHA
git commit -am "update content of reset_lifecycle_file"   # → Commit History
```

---

### 3. `git reset` vs. `git checkout`
Given commits `a ── b ── c ── d` with `HEAD` and `main` on `d`:

| Command | `HEAD` | `main` | Result |
|---|---|---|---|
| `git checkout b` | → `b` | stays on `d` | **Detached HEAD** |
| `git reset b` | → `b` | → `b` | The branch itself moves back. `c` and `d` are **orphaned** |

* Reset **always** updates the Commit tree (ref pointers). The mode decides what happens to the **index** and **working directory**.

---

### 4. The Three Modes
* **Default:** `git reset` = **`git reset --mixed HEAD`**. Any commit SHA can replace `HEAD`.

| Mode | Commit History (refs) | Staging Index | Working Directory | Undone changes end up… |
|---|---|---|---|---|
| `--soft` | ✅ Reset | ❌ Untouched | ❌ Untouched | **Staged** |
| `--mixed` *(default)* | ✅ Reset | ✅ Reset | ❌ Untouched | **Unstaged** in the working directory |
| `--hard` | ✅ Reset | ✅ Reset | ✅ Reset | **Gone** ⚠️ (can't be undone) |

```
            ┌──────── --soft ────────┐
            │  Commit History (HEAD) │
            ├──────── --mixed ───────┤
            │     Staging Index      │
            ├──────── --hard ────────┤
            │   Working Directory    │
            └────────────────────────┘
```

#### `--hard` — most direct, **DANGEROUS**, and most used
```bash
echo 'new file content' > new_file && git add new_file
echo 'changed content' >> reset_lifecycle_file
git reset --hard
# HEAD is now at dc67808 ...
# nothing to commit, working tree clean → new_file and the edits are DESTROYED
```

#### `--mixed` — default
```bash
echo 'new file content' > new_file && git add new_file
echo 'append content' >> reset_lifecycle_file && git add reset_lifecycle_file
git reset --mixed
# modified: reset_lifecycle_file  (unstaged)
# Untracked files: new_file
```
* The index is reset, and pending changes **move back to the working directory** (nothing is lost).

#### `--soft` — only moves the refs
* `git reset --soft` against `HEAD` does **nothing visible** (you're already at HEAD).
* Against an **older commit**:
```bash
git commit -m "prepend content to reset_lifecycle_file"   # now 3 commits
git reset --soft 780411da   # SHA of the "initial commit"
git status
# Changes to be committed: modified: reset_lifecycle_file
git log     # only "initial commit" remains
```
* History goes back, but the **index keeps its content**, so all later changes appear **staged**. ✅ Great for **squashing** several commits into one.

---

### 5. Resetting vs. Reverting
| | `git revert` | `git reset` |
|---|---|---|
| Safety | ✅ "Safe" | ⚠️ "Dangerous": real risk of losing work |
| Designed for | Undoing **public** commits | Undoing **local** changes (index/working dir/unpushed commits) |
| How | Keeps the original, adds an **undo commit** | **Removes** the changeset from the branch |

* `git reset` **never deletes commits directly**, but it **orphans** them (no ref points to them). You can usually recover them with **`git reflog`** until **garbage collection** permanently removes them.
> 📝 *Extra note (not in the original article):* The article says GC runs "every 30 days". More precisely, `git gc` runs automatically from time to time, and **unreachable reflog entries expire after 30 days by default** (`gc.reflogExpireUnreachable`). After that, orphaned commits can be pruned.
* Uncommitted index and working-directory changes are **not** protected like commits: `--hard` loses them permanently.

---

### 6. ⚠️ Don't Reset Public History
* **Never** `git reset <commit>` if anything after `<commit>` has been **pushed** to a shared repo.
* Teammates' history will seem to **suddenly lose commits**. Your next commits **diverge** from `origin/main`, and the merge needed to sync will confuse everyone.
* Use reset for **local experiments gone wrong**. Use **`git revert`** for public commits.

---

### 7. Examples Cheat Sheet

| Command | Effect |
|---|---|
| `git reset <file>` | **Unstage** a file; working directory unchanged |
| `git reset` | Unstage **everything**; working directory unchanged (rebuild the staged snapshot) |
| `git reset --hard` | Reset index **and** working directory to the last commit → **obliterates uncommitted changes** |
| `git reset <commit>` | Move the branch tip back to `<commit>`, reset the index, and keep all changes since then in the working directory → re-commit as cleaner, **atomic** snapshots |
| `git reset --hard <commit>` | Move the branch back **and** wipe the index + working directory → destroys uncommitted changes **and all later commits** |

#### Unstaging a file
```bash
git add .              # staged hello.py and main.py
git reset main.py      # unstage main.py
git commit -m "Make some changes to hello.py"
git add main.py
git commit -m "Edit main.py"
```

#### Removing local commits
```bash
git add foo.py && git commit -m "Start developing a crazy feature"
git commit -a -m "Continue my crazy feature"
git reset --hard HEAD~2    # scrap both commits (ONLY if unpublished!)
```

---

### Summary Takeaway
`git reset` undoes **local** changes by moving `HEAD` **and** the branch ref, and optionally resetting the **Staging Index** (`--mixed`, the default) and the **Working Directory** (`--hard`). `--soft` keeps everything staged, `--mixed` keeps changes unstaged, and `--hard` **throws them away**. Orphaned commits can be rescued with `git reflog` for a while, but **never reset commits that others already have**. Use `git revert` for those.
