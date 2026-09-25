Here is a clear breakdown of the key points about **`git rm`** from the text:

> Source: https://www.atlassian.com/git/tutorials/undoing-changes/git-rm

---

### 1. What Is `git rm`?
* Answers the common question: *"How do I tell Git to stop tracking a file?"*
* Removes files from a Git repository. It's the **inverse of `git add`**.
* Removes tracked files from the **staging index**, and by default from the **working directory** too.
* There's **no option** to remove a file **only** from the working directory (that's just a normal `rm`).
* **Safety check:** the file must match the version in `HEAD`. If the index or working-tree version differs, Git **blocks** the removal to protect in-progress changes.
* `git rm` does **not** remove branches.

---

### 2. Usage & Options

| Option | Effect |
|---|---|
| `<file>…` | Targets: one file, a space-separated list (`file1 file2`), or a glob (`./directory/*`) |
| `-f`, `--force` | Override the "matches `HEAD`" safety check |
| `-n`, `--dry-run` | Show what **would** be removed without deleting |
| `-r` | **Recursive**: remove a directory and everything in it |
| `--` | Separates options from file names (for names that look like options) |
| `--cached` | Remove from the **staging index only**. The file stays on disk (becomes untracked) |
| `--ignore-unmatch` | Exit with status **0** even if nothing matched (good for scripts that must fail gracefully) |
| `-q`, `--quiet` | Hide the one-line-per-file output |

---

### 3. Undoing `git rm`
`git rm` is **not permanent** until you commit. It only changes the **index** and **working directory**.

| Situation | How to undo |
|---|---|
| Not committed yet | `git reset HEAD` (restores the index) and then restore the file in the working directory, e.g. `git checkout HEAD -- <file>` |
| Already committed | Use **`git reflog`** to find a ref from before the removal |

> 📝 *Extra note (not in the original article):* The article says `git reset HEAD` or `git checkout .` alone will undo a `git rm`. In practice, `git reset HEAD` only restores the **index**, so the file is still missing from disk. `git checkout .` restores from the index, so it won't bring the file back either unless the index was reset first. The most reliable one-liners are `git checkout HEAD -- <file>` or, in modern Git, **`git restore --staged --worktree <file>`**.

---

### 4. Globs — Be Careful ⚠️
* `<file>` can be an exact path, a glob, or a directory name. Only **committed** paths are removed.
* Globs **match across directories**:

| Pattern | Removes |
|---|---|
| `directory/*` | Everything **inside** `directory/` |
| `directory*` | **Sibling** paths like `directory1`, `directory2`, `directory_whatever` (probably not what you meant) |

---

### 5. Scope
* Operates on the **current branch** only.
* Affects the **working directory** and **staging index**. The removal is recorded in history only when you **commit**.

---

### 6. Why `git rm` Instead of `rm`?
| Command | Working directory | Staging index |
|---|---|---|
| Shell `rm file` | ✅ File deleted | ❌ Not updated (you still need `git add file`) |
| `git rm file` | ✅ File deleted | ✅ Removal staged |

* **`git rm` = `rm` + `git add`** in one step.

---

### 7. Examples
```bash
git rm Documentation/\*.txt   # all .txt files under Documentation/ (and subdirs)
                              # the \* stops the SHELL from expanding it; Git expands it instead
git rm -f git-*.sh            # force-remove matching files from index + working dir
git rm --cached debug.log     # stop tracking but keep the file (pair with .gitignore)
```

---

### 8. Files Already Deleted with Plain `rm`
If you deleted several files with shell `rm`:
* To stage the deletions as part of your next commit: **`git commit -a`** picks them up.
* To explicitly stage all of them:
```bash
git diff --name-only --diff-filter=D -z | xargs -0 git rm --cached
```
  (lists deleted files → pipes them to `git rm --cached` → updates the index)

---

### Summary Takeaway
`git rm` removes files from the **staging index and working directory** in one step (**`rm` + `git add`**). Use **`--cached`** to stop tracking a file but keep it on disk, `-r` for directories, `-n` to preview, and `-f` to bypass the safety check. It's **undoable until you commit** (and afterwards through `git reflog`). Watch your **globs**.
