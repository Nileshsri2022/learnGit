Here is a clear breakdown of the key points about **`git reflog`** from the text:

> Source: https://www.atlassian.com/git/tutorials/rewriting-history/git-reflog

---

### 1. What Is the Reflog?
* **Reference logs ("reflogs")** record **every time the tip of a branch or other ref is updated** in your **local** repository.
* A **ref** is a pointer to a commit. Many commands take one (`git checkout`, `git reset`, `git merge <ref>`).
* There's also a special reflog for the **stash**.
* Stored under `.git/logs/`:

| Path | Tracks |
|---|---|
| `.git/logs/HEAD` | Movements of `HEAD` |
| `.git/logs/refs/heads/<branch>` | Each local branch tip |
| `.git/logs/refs/stash` | The stash (if used) |

---

### 2. Basic Usage
```bash
git reflog            # shortcut for: git reflog show HEAD
```
```
eff544f HEAD@{0}: commit: migrate existing content
bf871fd HEAD@{1}: commit: Add Git Reflog outline
9a4491f HEAD@{2}: checkout: moving from main to git_reflog
9a4491f HEAD@{3}: checkout: moving from Git_Config to main
39b159a HEAD@{4}: commit: expand on git context
```
* Each line = **SHA · `ref@{n}` · action: description**. `HEAD@{0}` is the newest entry.

---

### 3. Reflog References — `name@{qualifier}`
Works for `HEAD`, branches, tags, remotes, and the stash:

| Command | Shows |
|---|---|
| `git reflog show --all` | Reflog of **all** refs |
| `git reflog show otherbranch` | Reflog of one branch (`otherbranch@{0}`, `otherbranch@{1}`…) |
| `git reflog stash` | Reflog of the stash (`stash@{0}`…) |

* Reflog refs can be passed to **other commands**:
```bash
git diff stash@{0} otherbranch@{0}
```

---

### 4. Timed Reflogs
Every entry has a **timestamp**, so the qualifier can be a **time**:

| Qualifier examples |
|---|
| `1.minute.ago` · `1.hour.ago` · `1.day.ago` · `yesterday` · `1.week.ago` · `1.month.ago` · `1.year.ago` · `2011-05-17.09:00:00` |

* Combine them (`1.day.2.hours.ago`) and use plurals (`5.minutes.ago`).
```bash
git diff main@{0} main@{1.day.ago}   # what changed on main in the last day?
```

---

### 5. Subcommands

| Subcommand | Purpose | Notes |
|---|---|---|
| `show` *(default)* | Display the log for a ref | `git reflog main@{0}` = `git reflog show main@{0}`. It's an alias for `git log -g --abbrev-commit --pretty=oneline` |
| `expire` | Prune **old/unreachable** entries | ⚠️ Can lose data. Mostly used internally. Use `-n`/`--dry-run` to preview. Default expiry is **90 days**; change it with `--expire=<time>` or `gc.reflogExpire` |
| `delete` | Delete a specific entry | ⚠️ Can lose data. Rarely used by end users |

---

### 6. Recovering "Lost" Commits
**Git never really loses anything**, even after rebases or amends.

Scenario:
```bash
git commit -am "some WIP changes"
git rebase -i origin/main          # squash several commits into "some WIP changes"
git log --oneline
# 40dhsoi some WIP changes
# 35aee4a initial commit add git-init and setting-up-a-repo docs
#   → the squashed commits seem to be GONE
```
Check the reflog:
```bash
git reflog
# … HEAD@{0}: rebase -i (finish): returning to refs/heads/git_reflog
# … HEAD@{1}: rebase -i (start): checkout origin/main
# … HEAD@{2}: commit: some WIP changes        ← state BEFORE the rebase
```
Reset back to before the rebase:
```bash
git reset HEAD@{2}
```
* `HEAD` returns to the pre-rebase "some WIP changes" commit, and the **squashed commits are restored** in history.

> 📝 *Extra note (not in the original article):* A plain `git reset` is `--mixed`, so any differences end up as unstaged changes in the working directory. Use `git reset --hard HEAD@{2}` to restore that exact state (⚠️ it discards uncommitted work). Some text on the live page also seems to have been pasted in by mistake (a Bitbucket clone error snippet) and has been left out here.

---

### 7. Reflog Cheat Sheet
| Goal | Command |
|---|---|
| See where `HEAD` has been | `git reflog` |
| See a branch's history of tips | `git reflog show <branch>` |
| See everything | `git reflog show --all` |
| Undo a bad rebase/reset | `git reflog` → `git reset --hard HEAD@{n}` |
| Recover a deleted branch | `git reflog` → `git checkout -b <branch> <sha>` |
| Compare with the past | `git diff main@{0} main@{1.week.ago}` |

---

### Summary Takeaway
The reflog is Git's **local safety net**. It records every update to `HEAD`, branch tips, and the stash, and you can address entries as **`ref@{n}`** or **`ref@{time}`**. Use `git reflog` to **find and restore commits** that seem lost after a rebase, reset, or amend. Entries expire (90 days by default), and the reflog is **local only**, not shared with remotes.
