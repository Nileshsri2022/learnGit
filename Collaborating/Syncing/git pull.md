Here is a clear breakdown of the key points about **`git pull`** from the text:

> Source: https://www.atlassian.com/git/tutorials/syncing/git-pull

---

### 1. What Is `git pull`?
* **Fetches** content from a remote **and immediately updates** the local repository to match.
* **`git pull` = `git fetch` + `git merge`**:
  1. `git fetch`, scoped to the branch `HEAD` points at.
  2. `git merge`, which creates a **new merge commit** and moves `HEAD` to it.
* Think of it as Git's version of **`svn update`**.

---

### 2. How It Works

#### Default (merge) pull
```
Before:            A───B───C  origin/main
                  /
          D───E───F───G  main (local)

git pull

After:             A───B───C  origin/main
                  /         \
          D───E───F───G──────H  main   ← H = new merge commit
```
* Git downloads everything since the **divergence point** (`E`), i.e. the remote commits `A-B-C`, and creates a **merge commit `H`** with a combined log message.

#### Rebase pull — `git pull --rebase`
```
After:    D───E───A───B───C───F'───G'  main
```
* **No merge commit.** Your local commits (`F`, `G`) are **rewritten on top of** the remote commits.

---

### 3. Common Options

| Command | Effect |
|---|---|
| `git pull <remote>` | Fetch the remote's copy of the current branch and merge it. Same as `git fetch <remote>` + `git merge origin/<current-branch>` |
| `git pull --no-commit <remote>` | Fetch and merge, but **don't create the merge commit** (you can review first) |
| `git pull --rebase <remote>` | Integrate using **`git rebase`** instead of `git merge` |
| `git pull --verbose` | Show what's being downloaded and the merge details |

---

### 4. `git pull` vs. `git fetch`
| `git fetch` | `git pull` |
|---|---|
| **"Safe"** | **"Unsafe"** |
| Downloads only; local state **unchanged** | Downloads **and immediately changes** local state |
| — | May leave you in a **conflicted** state unexpectedly |

* Related syncing commands: `git remote` (define endpoints), `git push` (upload).

---

### 5. Pulling via Rebase
* `--rebase` keeps a **linear history** with no unnecessary merge commits: *"put my changes on top of what everybody else has done."* It's even more like `svn update` than a plain pull.
* It's so common there's a config for it:
```bash
git config --global branch.autosetuprebase always
```
> 📝 *Extra note (not in the original article):* `branch.autosetuprebase` only affects **newly created** tracking branches. The simpler modern setting that applies to every pull is **`git config --global pull.rebase true`**. Recent Git versions also ask you to choose a pull strategy (`pull.rebase` or `pull.ff`) if none is set.

---

### 6. Examples
```bash
# Default: fetch + merge the current branch's upstream
git pull

# Pull a branch from a specific remote
git checkout new_feature
git pull <remote_repo>        # pulls new_feature from <remote_repo>, then merges

# Sync main via rebase
git checkout main
git pull --rebase origin      # move local changes on top of everyone else's
```

---

### Summary Takeaway
`git pull` is a shortcut for **`git fetch` + `git merge`**: it downloads the upstream changes for your current branch and **integrates them immediately**, creating a merge commit by default. Use **`--rebase`** (or set `pull.rebase`) for a **linear history**, `--no-commit` to review before committing, and plain **`git fetch`** when you just want to look without touching your work.
