Here is a clear breakdown of the key points about **moving a full Git repository (with history)** from the text:

> Source: https://www.atlassian.com/git/tutorials/git-move-repository

---

### 1. The Goal
* Move a **complete Git repository**, with **all history**, from one remote server to another.
* The steps also let you **choose which branches and tags** to bring along.
* Naming: **ORI** = the original repo, **NEW** = the new (empty) repo.

```
ORI (old server) ──clone──▶ temp-dir (local) ──push──▶ NEW (new server)
```

---

### 2. Step by Step

| # | Step | Command |
|---|---|---|
| 1 | **Clone ORI** into a temporary directory | `git clone <url to ORI repo> temp-dir` |
| 2 | **Go into** it | `cd temp-dir` |
| 3 | **List all branches** in ORI (local + remote) | `git branch -a` |
| 4 | **Check out every branch** you want to copy (this creates local branches) | `git checkout branch-name` |
| 5 | **Fetch all tags** from ORI | `git fetch --tags` |
| 6 | **Double-check** your local tags and branches | `git tag` · `git branch -a` |
| 7 | **Remove the link to ORI** | `git remote rm origin` |
| 8 | **Link to NEW** | `git remote add origin <url to NEW repo>` |
| 9 | **Push all branches and tags** | `git push origin --all` · `git push --tags` |
| 10 | ✅ Done: NEW is a full copy of ORI | — |

#### All together
```bash
git clone <url to ORI repo> temp-dir
cd temp-dir
git branch -a                  # see what's there
git checkout feature-x         # repeat for each branch you want
git checkout release-1.0
git fetch --tags
git tag && git branch -a       # verify
git remote rm origin
git remote add origin <url to NEW repo>
git push origin --all
git push --tags
```

* 🔑 **Why step 4 matters:** `git push --all` pushes only **local** branches. A fresh clone has just **one** local branch (the default). The others exist only as `remotes/origin/*` until you check them out.

---

### 3. Shortcut: Copy Everything with `--mirror`
To copy the **entire** repo (all branches, tags, and refs), replace **steps 1–5** with:
```bash
git clone --mirror <url to ORI repo> temp-dir
```
> 📝 *Extra note (not in the original article):* With a mirror clone, the simplest way to finish is to push everything at once:
> ```bash
> cd temp-dir
> git push --mirror <url to NEW repo>
> ```
> `--mirror` pushes **all refs** and makes NEW an exact copy. It also **deletes** refs on NEW that don't exist in ORI, so only use it on an **empty** or disposable target.

| Approach | Use when |
|---|---|
| **Step-by-step** (checkout + `--all` + `--tags`) | You want to **pick** which branches/tags to move |
| **`--mirror`** | You want an **exact, complete copy** |

---

### Summary Takeaway
To move a repo **with its full history**: **clone ORI**, **check out** every branch you want, **fetch tags**, **swap `origin`** to the NEW URL, then **`git push origin --all`** and **`git push --tags`**. To copy **everything**, use **`git clone --mirror`** (and `git push --mirror`) instead.
