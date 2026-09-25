Here is a clear breakdown of the key points about the **Synchronize** phase of migrating from SVN to Git, from the text:

> Source: https://www.atlassian.com/git/tutorials/migrating-synchronize
> *(Step 3 of 5: Prepare → Convert → **Synchronize** → Share → Migrate)*

---

### 1. Goal of This Phase
* Keep the new Git repo **in sync with new commits** in the original SVN repo.
* This allows a **comfortable transition period**: the team keeps its **SVN workflow** while starting to **experiment with Git**.

---

### 2. One-Way Sync: SVN → Git
* Syncing in **both** directions is possible, but Atlassian recommends **one-way (SVN → Git)**.

| During the transition | |
|---|---|
| ✅ Commit to | **SVN** only |
| ❌ Don't commit to | **Git** (it's updated **only** by the sync process) |
| Sync | Whenever necessary. It's like Convert but **incremental**, so it's much faster |

```
Developers ──commit──▶ SVN repo ──(git svn fetch + sync-rebase)──▶ Git repo (read-only)
```

---

### 3. Update the Authors File
* `authors.txt` (SVN username → `Name <email>`) is **essential** for syncing.
* If you moved it from `~/GitMigration/authors.txt`, tell git-svn where it is:
```bash
git config svn.authorsfile <path/to/authors.txt>
```
* **New SVN committers since the last sync?** Update the mapping by:

| Method | Best for |
|---|---|
| Manually **appending** them to `authors.txt` | One-off syncs |
| **`--authors-prog`** script | **Unsupervised/scheduled** syncs |

---

### 4. Automatically Generating Authors — `--authors-prog`
* Points `git svn` to a **script** that:
  * takes the **SVN username** as its **only argument**, and
  * prints **one line**: `Name <email>` (like the right-hand side of `authors.txt`).
* It's used for users **not found** in `authors.txt`.
* Create `~/GitMigration/authors.sh`:
```bash
echo "$1 <$1@example.com>"
```
* This only makes a **dummy** name/email. Improve it if you have a better mapping source (e.g. a company directory).
> 📝 *Extra note (not in the original article):* Make the script executable (`chmod +x authors.sh`) and pass a path Git can resolve (e.g. `--authors-prog=../authors.sh` when running inside the repo, or an absolute path).

---

### 5. Fetch the New SVN Commits
* Git separates **downloading** commits (**fetch**) from **integrating** them (**merge/rebase**).
```bash
git svn fetch
# or, with the authors script:
git svn fetch --authors-prog=authors.sh
```
* Like `git svn clone`, this updates only the **remote branches** (which will exactly match SVN history). **Local branches aren't updated yet.**
> 📝 *Extra note (not in the original article):* The article says to run this in `~/GitMigration`. In practice, the `git svn` and `svn-migration-scripts.jar` commands need to run **inside the converted repo**, i.e. `~/GitMigration/<git-repo-name>`.

---

### 6. Apply the Fetched Commits
```bash
java -Dfile.encoding=utf-8 -jar ~/svn-migration-scripts.jar sync-rebase
```
* **Rebases** the fetched commits onto your **local branches** so they match their remote counterparts. The new commits now show up in `git log`.

---

### 7. Clean Up Again
Remove tags/branches that were **deleted in SVN** since the last sync:
```bash
java -Dfile.encoding=utf-8 -jar ~/svn-migration-scripts.jar clean-git --force
```
✅ The local Git repo is now **synchronized** with SVN.

---

### Sync Routine (repeat as needed)
| # | Command |
|---|---|
| 1 | *(if needed)* update `authors.txt` or use `--authors-prog` |
| 2 | `git svn fetch [--authors-prog=authors.sh]` |
| 3 | `java -Dfile.encoding=utf-8 -jar ~/svn-migration-scripts.jar sync-rebase` |
| 4 | `java -Dfile.encoding=utf-8 -jar ~/svn-migration-scripts.jar clean-git --force` |

---

### Summary Takeaway
**Synchronize** = keep the Git repo updated with a **one-way SVN → Git** sync. **Developers commit only to SVN** during the transition. Keep **authors** mapped (edit `authors.txt` or use `--authors-prog`), **`git svn fetch`** new commits, **`sync-rebase`** them onto local branches, and **`clean-git --force`** to drop deleted branches/tags. It's much simpler than two-way sync, and you can already start **integrating Git into your build process**.
