Here is a clear breakdown of the key points about **`git revert`** from the text:

> Source: https://www.atlassian.com/git/tutorials/undoing-changes/git-revert

---

### 1. What Is `git revert`?
* An "undo" command, but **not a traditional undo**. Instead of **removing** a commit, it works out the **inverse** of that commit's changes and **appends a new commit** containing it.
* **History is never lost**, which matters for the integrity of your history and for reliable collaboration.
* Typical use: you find a bug introduced by **one specific commit**. Instead of fixing it by hand, `git revert` does the undo and commits it for you.

---

### 2. How It Works
| Command | Moves `HEAD`/branch ref to an old commit? | Result |
|---|---|---|
| `git checkout`, `git reset` | ✅ Yes | Points back at an older commit |
| `git revert` | ❌ No | Creates a **new "revert commit"**, which becomes the new branch tip |

Example:
```bash
mkdir git_revert_test && cd git_revert_test
git init .
touch demo_file && git add demo_file
git commit -am "initial commit"
echo "initial content" >> demo_file
git commit -am "add new content to demo file"
echo "prepended line content" >> demo_file
git commit -am "prepend content to demo file"

git log --oneline
# 86bb32e prepend content to demo file
# 3602d88 add new content to demo file
# 299b15f initial commit

git revert HEAD      # a commit ref is REQUIRED
# [main b9cd081] Revert "prepend content to demo file"

git log --oneline
# b9cd081 Revert "prepend content to demo file"
# 86bb32e prepend content to demo file
# 3602d88 add new content to demo file
# 299b15f initial commit
```
* Like a merge, a revert **opens your editor** for the commit message.
* The reverted commit **stays in history**. The 2nd and 4th commits now have **identical code**, and you can still go back to the 3rd if needed.

```
Before:  299b15f ── 3602d88 ── 86bb32e                  (main)
After:   299b15f ── 3602d88 ── 86bb32e ── b9cd081        (main)
                                          "Revert …" = inverse of 86bb32e
```

---

### 3. Common Options

| Option | Effect |
|---|---|
| `-e` / `--edit` | *(default)* Open the editor to edit the revert message |
| `--no-edit` | Don't open the editor; use the default message |
| `-n` / `--no-commit` | **Don't commit**. Put the inverse changes in the **staging index and working directory** instead (useful for reverting several commits into one) |

---

### 4. Resetting vs. Reverting
* `git revert` undoes **one single commit**. It does **not** take the project back to an earlier state by removing all later commits. **That's called a reset.**

| | `git revert` | `git reset` |
|---|---|---|
| History | **Preserved** (adds a new commit) | **Rewritten** (commits removed/orphaned) |
| Safe on **shared/public** commits? | ✅ Yes | ❌ Dangerous |
| Target | **Any** commit at any point in history | Only works **backward from the current commit** |
| Undoing an old commit | One command | Remove all later commits, drop the target, re-commit the rest (messy) |

```
revert:  A ── B ── C ── D ── C'     (C' undoes C; history kept)
reset:   A ── B                     (C and D gone from the branch)
```

---

### Summary Takeaway
`git revert` is a **forward-moving, safe undo**: it creates a **new commit that inverts** a chosen commit instead of deleting history. That makes it the right choice for **already-pushed/shared** commits and for undoing an **arbitrary old commit**. Use `--no-edit` to skip the editor and `-n` to stage the inverse without committing. Use **`git reset`** only for local, unshared history.
