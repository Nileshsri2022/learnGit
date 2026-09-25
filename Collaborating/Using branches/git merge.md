Here is a clear breakdown of the key points about **`git merge`** from the text:

> Source: https://www.atlassian.com/git/tutorials/using-branches/git-merge

---

### 1. What Is `git merge`?
* **Merging is Git's way of putting a forked history back together.**
* It takes independent lines of development (created with `git branch`) and **integrates them into one branch**.
* 🔑 It always merges **into the current branch**. The current branch is updated, and the **merged branch is unaffected**.
  * So it's used with **`git checkout`** (to select the receiving branch) and **`git branch -d`** (to delete the obsolete branch).

---

### 2. How It Works
* Takes **two commit pointers** (usually branch tips), finds their **common base commit**, and creates a **merge commit** combining both sequences.
* **Merge commits have two parents.**
* Git merges automatically. If **the same data changed in both histories**, that's a **conflict**, and Git needs you to step in.

```
Before:        A───B  feature
              /
     C───D───E───F───G  main

git checkout main && git merge feature

After:         A───B
              /     \
     C───D───E───F───G───M  main   ← M has 2 parents (G and B)
```

---

### 3. Preparing to Merge
| Step | Command |
|---|---|
| 1. **Confirm the receiving branch** | `git status` → if needed `git checkout main` |
| 2. **Fetch latest remote commits** | `git fetch`, then `git pull` so `main` is up to date |
| 3. **Merge** | `git merge <branch name>` |

---

### 4. Fast-Forward Merge
* Possible when there's a **linear path** from the current tip to the target branch.
* Git doesn't really "merge". It just **moves the current branch tip forward** to the target tip.

```
Before:   C───D  main
               \
                A───B  some-feature

After:    C───D───A───B  main, some-feature   (no merge commit)
```
```bash
git checkout -b new-feature main
git add <file> && git commit -m "Start a feature"
git add <file> && git commit -m "Finish a feature"
git checkout main
git merge new-feature        # fast-forward
git branch -d new-feature    # safe: new-feature is reachable from main
```
* Common for **short-lived topic branches**.
* Want a merge commit anyway (for record-keeping)?
```bash
git merge --no-ff <branch>   # always create a merge commit
```

---

### 5. 3-Way Merge
* Needed when branches have **diverged** (e.g. `main` moved on while the feature was in progress).
* "3-way" because Git uses **three commits**: the **two branch tips** and their **common ancestor**.
```bash
git checkout -b new-feature main
git add <file> && git commit -m "Start a feature"
git add <file> && git commit -m "Finish a feature"
git checkout main
git add <file> && git commit -m "Make some super-stable changes to main"
git merge new-feature        # 3-way merge → merge commit
git branch -d new-feature
```

| Merge type | Typical use |
|---|---|
| **Fast-forward** (often after **rebasing**) | Small features, bug fixes → avoids superfluous merge commits |
| **3-way** | **Long-running** features. The merge commit symbolically joins the branches |

---

### 6. Resolving Conflicts
* If both branches changed **the same part of the same file**, Git **stops right before the merge commit**.
```bash
$ git status
On branch main
Unmerged paths:
  (use "git add/rm ..." as appropriate to mark resolution)
        both modified: hello.py
```

#### How conflicts look
```
here is some content not affected by the conflict
<<<<<<< main
this is conflicted text from main
=======
this is conflicted text from feature branch
>>>>>>> feature branch
```
| Marker | Meaning |
|---|---|
| `<<<<<<<` → `=======` | The **receiving** branch's version (e.g. `main`) |
| `=======` → `>>>>>>>` | The **merging** branch's version |

* 💡 Search the project for these markers to find every conflict.

#### Finish the merge (the normal edit → stage → commit flow)
```bash
# edit files to resolve
git add hello.py     # mark as resolved
git commit           # creates the merge commit
```
* ⚠️ Conflicts can **only** happen in a **3-way merge**, never in a fast-forward.

---

### Summary Takeaway
`git merge` **combines sequences of commits into one unified history**, always **into the current branch**. Git either **fast-forwards** (linear path, no new commit) or does a **3-way merge** (diverged branches, new two-parent merge commit; force one with `--no-ff`). Merges are automatic unless both sides changed the same lines. Then you resolve the `<<<<<<<` / `=======` / `>>>>>>>` markers, `git add`, and `git commit`.
