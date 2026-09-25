Here is a clear breakdown of the key points about the **Convert** phase of migrating from SVN to Git, from the text:

> Source: https://www.atlassian.com/git/tutorials/migrating-convert
> *(Step 2 of 5: Prepare → **Convert** → Synchronize → Share → Migrate)*

---

### 1. Goal of This Phase
* **Import the SVN repository into a new Git repository** using **`git svn`** (bundled with most Git distributions).
* Then **clean up** the result with `svn-migration-scripts.jar`.

---

### 2. Before You Start ⚠️
* **It can take a long time.** Benchmark: a **400 MB repo with 33,000 commits** on trunk took about **12 hours**, even from a local SVN repo.
* Run it on the **migration lead's machine** for normal repos. For **very large** repos, run `git svn clone` **on the SVN server** to avoid network overhead.
* The migration scripts were written for **Git 1.8.x**, and that version is required for them to work.

---

### 3. Clone the SVN Repository — `git svn clone`
Converts SVN **trunk, branches, and tags** into a Git repo. Run it inside `~/GitMigration`.

#### Standard layout (`/trunk`, `/branches`, `/tags`)
```bash
git svn clone --stdlayout --authors-file=authors.txt \
    <svn-repo>/<project> <git-repo-name>

# Example
git svn clone --stdlayout --authors-file=authors.txt \
    https://svn.atlassian.com/Confluence ConfluenceAsGit
```

| Placeholder | Meaning |
|---|---|
| `<svn-repo>` | URI of the SVN repository |
| `<project>` | Project to import |
| `<git-repo-name>` | Directory name for the new Git repo |

#### Non-standard layout
Specify the locations yourself (repeat `--branches` for several folders):
```bash
git svn clone --trunk=/trunk --branches=/branches \
    --branches=/bugfixes --tags=/tags --authors-file=authors.txt \
    <svn-repo>/<project> <git-repo-name>
```

* `--authors-file=authors.txt` uses the mapping from the **Prepare** step so commits get real names and emails.

---

### 4. Inspect the New Git Repository
* You'll find `~/GitMigration/<git-repo-name>`, and standard Git commands work in it.
* ⚠️ **Surprise:** SVN branches and tags **don't show up** in `git branch` or `git tag`.

| SVN item | Imported as | See with |
|---|---|---|
| Branches | **Remote** branches | `git branch -r` |
| Tags | **Remote** branches prefixed with **`tags/`** | `git branch -r` |

* This helps with **two-way sync** but is **confusing** for a **one-way migration**, so the next step converts them.

---

### 5. Clean the New Git Repository — `clean-git`
Turns SVN branches into **local Git branches** and SVN tags into **real Git tags**.

⚠️ **Destructive:** afterwards you **can't move commits from Git back to SVN**.
* Fine for this guide's **one-way SVN → Git** approach (the Git repo is **read-only** until the Migrate step).
* **Don't** run it if you plan to commit to **both** Git and SVN during the migration (an advanced, not-recommended scenario).

```bash
cd ~/GitMigration/<git-repo-name>

# Dry run: show what would change
java -Dfile.encoding=utf-8 -jar ~/svn-migration-scripts.jar clean-git

# Apply the changes
java -Dfile.encoding=utf-8 -jar ~/svn-migration-scripts.jar clean-git --force
```
* ✅ Now `git branch` lists your SVN branches and `git tag` lists your SVN tags, so the conversion is complete.

---

### Flow at a Glance
```
SVN repo ──git svn clone──▶ Git repo (branches/tags as remotes)
                               │
                   clean-git (dry run) → clean-git --force
                               ▼
                 Git repo with local branches + real tags
```

---

### Summary Takeaway
**Convert** = run **`git svn clone`** (with `--stdlayout` or explicit `--trunk/--branches/--tags`, plus `--authors-file`) to turn the SVN history into a Git repo. Then run **`clean-git`** (dry run first, then `--force`) to turn imported remote branches and `tags/` into **real local branches and tags**. Be patient, since large repos can take hours. Next you'll **Synchronize** the Git repo with new SVN commits.
