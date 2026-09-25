Here is a clear breakdown of the key points about **Merging vs. rebasing** from the text:

> Source: https://www.atlassian.com/git/tutorials/merging-vs-rebasing

---

### 1. The Big Idea
* `git rebase` has a reputation as "magical hocus pocus" beginners should avoid — but **used with care** it makes a team's life much easier.
* **`git merge` and `git rebase` solve the same problem:** integrating changes from one branch into another. They just do it **very differently**.

#### The scenario
You're working on a `feature` branch; meanwhile a teammate adds new commits to `main` → **forked history**. The new `main` commits are relevant to your feature. Two options:

```
          A---B---C  feature
         /
    D---E---F---G    main
```

---

### 2. Option 1 — Merge

```bash
git checkout feature
git merge main
```
* Creates a new **merge commit** in `feature` that ties both histories together.

```
          A---B---C---M  feature
         /           /
    D---E---F-------G    main
```

| ✅ Pros | ❌ Cons |
|---|---|
| **Non-destructive** — existing branches are not changed | An **extra merge commit** every time you pull in upstream changes |
| Avoids all rebase pitfalls | An active `main` can **pollute** feature history and make it hard for others to understand |

---

### 3. Option 2 — Rebase

```bash
git checkout feature
git rebase main
```
* Moves the **entire feature branch** to start at the **tip of `main`**.
* Instead of a merge commit, it **rewrites history** by creating **brand-new commits** for each original commit.

```
                  A'--B'--C'  feature
                 /
    D---E---F---G             main
```

| ✅ Pros | ❌ Cons |
|---|---|
| **Cleaner history** — no unnecessary merge commits | **Safety** — breaking the Golden Rule can be catastrophic for collaboration |
| **Perfectly linear** history — easier to navigate with `git log`, `git bisect`, `gitk` | **Traceability** — you lose the context of *when* upstream changes were incorporated |

---

### 4. Interactive Rebasing (`git rebase -i`)
* Alter commits **as they are moved** — **complete control** over the branch's history.
* Typically used to **clean up a messy history before merging** a feature into `main`.

```bash
git checkout feature
git rebase -i main
```
Your editor shows the commits to be moved:
```
pick 33d5b7a Message for commit #1
pick 9480b3d Message for commit #2
pick 5c67e61 Message for commit #3
```
Change commands and/or reorder lines. E.g. commit #2 fixes a small issue in #1 → fold it in with `fixup`:
```
pick 33d5b7a Message for commit #1
fixup 9480b3d Message for commit #2
pick 5c67e61 Message for commit #3
```
* Eliminating insignificant commits makes history **much easier to understand** — something **`git merge` simply cannot do**.

---

### 5. ⚠️ The Golden Rule of Rebasing
> **Never use `git rebase` on *public* branches.**

* Example: rebasing **`main` onto `feature`** moves all of `main`'s commits in **your** repo only. Everyone else still has the original `main`.
* Rebasing creates **new commits**, so Git thinks your `main` has **diverged**. Syncing requires a merge → an extra merge commit **plus two sets of commits with the same changes**. Very confusing!
* Before rebasing, always ask: **"Is anyone else looking at this branch?"**
  * **Yes** → hands off the keyboard; use a **non-destructive** approach (e.g. `git revert`).
  * **No** → rewrite history as much as you like.

#### Force-pushing
```bash
# Be very careful with this command!
git push --force
```
* Git rejects pushing a rebased branch that conflicts with the remote; `--force` **overwrites** the remote branch.
* One of the only valid uses: you pushed a **private** feature branch (e.g. for backup), then cleaned it up locally — *"take the current version instead."* Make sure **nobody** is working from the old commits.

---

### 6. Workflow Walkthrough — Where Rebasing Helps
**Step 0:** always develop each feature in a **dedicated branch**.

#### a) Local cleanup (the best way to start using rebase)
* Periodically run an **interactive rebase** so every commit is **focused and meaningful** — write code freely and tidy commits afterward.
* Two choices for the new base:
  1. The parent branch (e.g. `main`), or
  2. An **earlier commit in your feature** — to fix up just the last few commits:
```bash
git checkout feature
git rebase -i HEAD~3      # rewrite only the last 3 commits (doesn't pull in upstream changes)
```
* To rewrite the whole feature, find its original base:
```bash
git merge-base feature main   # returns the commit ID to pass to git rebase -i
```
* Only for **private** branches. There's **no `git merge` alternative** for this kind of cleanup.

#### b) Incorporating upstream changes into a feature
* Merge = safe, preserves history. Rebase = linear history by moving your feature onto the tip of `main`.
* You can also rebase onto a **remote branch** — e.g. you and John both commit to `feature`:
  * Merge your `feature` with `john/feature`, **or**
  * Rebase your `feature` onto `john/feature` → *"add my changes to what John has already done."*
* This **doesn't break the Golden Rule** — only **your local** commits move.
* `git pull` merges by default; use **`git pull --rebase`** to rebase instead.

#### c) Reviewing a feature with a pull request
* Once a PR is open, the branch is **public** → **don't rebase** it (Git and teammates can't track follow-up commits).
* Incorporate others' changes with **`git merge`**.
* 💡 Clean up with an interactive rebase **before** submitting the PR.

#### d) Integrating an approved feature
* Optionally **rebase the feature onto `main`**, then `git merge` it → guarantees a **fast-forward** merge and a **perfectly linear** history (and a chance to squash PR follow-up commits).
* Not confident? Rebase in a **temporary branch** so you can always go back:
```bash
git checkout feature
git checkout -b temporary-branch
git rebase -i main
# [Clean up the history]
git checkout main
git merge temporary-branch
```

---

### 7. Quick Comparison

| | `git merge` | `git rebase` |
|---|---|---|
| Rewrites history? | ❌ No | ✅ Yes (new commits) |
| Extra merge commits | Yes | No |
| History shape | Preserves forks | **Linear** |
| Safe on public branches | ✅ Yes | ❌ **Never** |
| Can clean up commits | ❌ No | ✅ With `-i` |
| Shows when upstream was integrated | ✅ Yes | ❌ No |

---

### Summary Takeaway
Want a **clean, linear history** free of unnecessary merge commits? Reach for **`git rebase`** (especially **`-i`** for local cleanup). Want to **preserve complete history** and avoid rewriting public commits? Stick with **`git merge`**. Both are valid — just obey the **Golden Rule: never rebase public branches**.
