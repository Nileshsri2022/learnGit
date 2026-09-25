Here is a clear breakdown of the key points about **`git stash`** from the text:

> Source: https://www.atlassian.com/git/tutorials/saving-changes/git-stash

---

### 1. What Is `git stash`?
* **Temporarily shelves** your uncommitted changes so you can work on something else, then **re-apply** them later.
* Perfect when you must **switch context quickly** but you're mid-change and **not ready to commit**.
* Stashes are **local only**. They're **not** transferred to the server when you push.

---

### 2. Stashing Your Work
Takes **staged and unstaged** changes, saves them away, and **reverts** them from the working copy:
```bash
$ git status
Changes to be committed:     new file: style.css
Changes not staged for commit: modified: index.html

$ git stash
Saved working directory and index state WIP on main: 5002d47 our new homepage

$ git status
nothing to commit, working tree clean
```
* Now you can commit, switch branches, and so on, then come back.

---

### 3. Re-applying Stashed Changes

| Command | Re-applies changes | Keeps the stash? | Use case |
|---|---|---|---|
| `git stash pop` | ✅ | ❌ Removes it | Normal "restore my work" |
| `git stash apply` | ✅ | ✅ Keeps it | Apply the same changes to **multiple branches** |

---

### 4. Untracked & Ignored Files

| By default `git stash`… | |
|---|---|
| ✅ Stashes | Staged changes (index) · Changes to **tracked** files |
| ❌ Does NOT stash | **New untracked** files · **Ignored** files |

| Option | Also stashes |
|---|---|
| `-u` / `--include-untracked` | Untracked files |
| `-a` / `--all` | Untracked **and ignored** files |

---

### 5. Managing Multiple Stashes
```bash
$ git stash list
stash@{0}: WIP on main: 5002d47 our new homepage
stash@{1}: WIP on main: 5002d47 our new homepage
```
* Default labels ("WIP on …") are hard to tell apart, so **annotate your stashes**:
```bash
git stash save "add style to our site"
# stash@{0}: On main: add style to our site
```
> 📝 *Extra note (not in the original article):* `git stash save` is deprecated in modern Git. Use **`git stash push -m "message"`** instead (`push` also accepts pathspecs to stash specific files).

* `git stash pop` applies the most recent stash (`stash@{0}`) by default. To pick another one: `git stash pop stash@{2}`.

---

### 6. Viewing Stash Diffs
```bash
git stash show      # summary: files changed, insertions/deletions
git stash show -p   # full patch (--patch)
```

---

### 7. Partial Stashes — `git stash -p`
Iterates over each changed **hunk** and asks whether to stash it:

| Key | Action |
|---|---|
| `y` | Stash this hunk |
| `n` | Don't stash this hunk |
| `s` | Split into smaller hunks |
| `/` | Search for a hunk by regex |
| `q` | Quit (hunks already selected **will** be stashed) |
| `?` | Help |

* There's no explicit "abort": press **`Ctrl-C`** (SIGINT) to cancel.

---

### 8. Creating a Branch from a Stash
If your branch has **diverged** and popping would **conflict**:
```bash
git stash branch add-stylesheet stash@{1}
```
* Checks out a **new branch** from the commit the stash was **created from**, then **pops** the stash onto it.

---

### 9. Cleaning Up
```bash
git stash drop stash@{1}   # delete one stash
git stash clear            # delete ALL stashes
```

---

### 10. How `git stash` Works Under the Hood
* Stashes are **commit objects**. `.git/refs/stash` points to the latest one, and older stashes live in the **reflog** of the `stash` ref. That's why they're named `stash@{n}` (the nth reflog entry).
* One `git stash` creates **2 or 3 new commits**:

```
                 ┌── stash@{0}  (working-tree changes of tracked files)
                 │      │ parent 1 → HEAD commit at stash time
                 │      │ parent 2 → new commit: the INDEX state
                 │      │ parent 3 → new commit: UNTRACKED (+ ignored with --all) files
                 │                   (only with -u / --all and if such files exist)
```

| Invocation | Commits created |
|---|---|
| `git stash` | Working-tree commit + index commit |
| `git stash -u` | + a commit for **untracked** files |
| `git stash --all` | + a commit for **untracked + ignored** files (same commit) |

* On `git stash pop`, the working copy and index are updated from these commits and the reflog is shuffled. The popped commits aren't deleted immediately, but they become candidates for **garbage collection**.

---

### Summary Takeaway
`git stash` is your **"pause button"**. Stash (`git stash`, `-u`, `-a`, `-p`), inspect (`list`, `show -p`), restore (`pop` / `apply`), branch off (`stash branch`), and clean up (`drop` / `clear`). Label your stashes, remember they're **local only**, and know that under the hood they're just **commits** tracked by the `refs/stash` reflog.
