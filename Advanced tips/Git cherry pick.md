Here is a clear breakdown of the key points about **Git cherry-pick** from the text:

> Source: https://www.atlassian.com/git/tutorials/cherry-pick

---

### 1. What Is `git cherry-pick`?
* A powerful command that lets you **pick any commit by reference** and **append it to your current `HEAD`**.
* **Cherry picking** = taking a commit from **one branch** and applying it to **another**.
* Also useful for **undoing mistakes** — e.g. a commit made on the **wrong branch**: switch to the right branch and cherry-pick it there.

---

### 2. When to Use It (and When Not To)
* ⚠️ Useful, but **not always best practice**:
  * It can create **duplicate commits** (same change, different IDs).
  * In many cases where cherry-pick would work, a **traditional merge is preferred**.
* Handy in a few specific scenarios:

| Scenario | Example |
|---|---|
| **Team collaboration** | A backend dev creates a data structure the frontend also needs. The frontend dev cherry-picks **just that commit** to keep making progress. |
| **Bug hotfixes** | While building a feature, a dev finds and fixes an existing bug in a dedicated commit → cherry-pick that fix **straight to `main`** so users get it fast. |
| **Undoing changes / restoring lost commits** | A stale feature branch never merged, or a PR closed without merging — Git never loses those commits; find them with **`git log`** or **`git reflog`** and cherry-pick them back to life. |

---

### 3. How to Use It

```bash
git cherry-pick <commitSha>   # find the SHA with git log
```

#### Example
Starting state:
```
a - b - c - d          main
         \
          e - f - g    feature
```
Bring commit `f` into `main`:
```bash
git checkout main      # 1. be on the RECEIVING branch
git cherry-pick f      # 2. apply commit f
```
Result:
```
a - b - c - d - f'     main      (f' = a NEW commit with f's changes)
         \
          e - f - g    feature
```

---

### 4. Useful Options

| Option | Effect |
|---|---|
| `--edit` (`-e`) | Prompt for a new **commit message** before applying |
| `--no-commit` (`-n`) | Apply the changes to your **working directory/index** without creating a commit |
| `--signoff` (`-s`) | Add a **"Signed-off-by"** line to the commit message |
| Merge strategy options | Same strategy options as `git merge` |

#### Handling conflicts
If a cherry-pick hits a merge conflict:
```bash
# fix the conflicted files, then
git add <file>
git cherry-pick --continue   # finish the cherry-pick
git cherry-pick --abort      # or cancel and return to the pre-cherry-pick state
git cherry-pick --quit       # or stop, keeping the progress made so far
```

---

### 5. Quick Workflow
1. `git log` (or `git reflog`) → find the commit SHA you want.
2. `git checkout <target-branch>` → switch to the branch that should receive it.
3. `git cherry-pick <sha>` → apply it.
4. Resolve conflicts if needed → `--continue`.

---

### Summary Takeaway
`git cherry-pick` copies **individual commits** from one branch onto another — great for **sharing a specific change with a teammate**, shipping **urgent hotfixes** to `main`, and **rescuing lost or unmerged commits**. Use **`git log`/`git reflog`** to find commits, and remember: cherry-picking creates **duplicate commits**, so it **shouldn't replace `git merge` or `git rebase`** for normal branch integration.
