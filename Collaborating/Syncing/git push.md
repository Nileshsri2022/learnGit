Here is a clear breakdown of the key points about **`git push`** from the text:

> Source: https://www.atlassian.com/git/tutorials/syncing/git-push

---

### 1. What Is `git push`?
* **Uploads local repository content to a remote repository**, transferring your commits to a remote repo.
* The **counterpart of `git fetch`**:

| `git fetch` | `git push` |
|---|---|
| **Imports** commits into **local** branches | **Exports** commits to **remote** branches |
| "Download" | "Upload" |

* Remotes are configured with `git remote`.
* ⚠️ Pushing **can overwrite changes**, so be careful.

---

### 2. Usage

| Command | Effect |
|---|---|
| `git push <remote> <branch>` | Push `<branch>` with all necessary commits and objects. Creates/updates that branch in the destination. **Refused if it isn't a fast-forward** |
| `git push <remote> --force` | Push **even if** it's a non-fast-forward. ⚠️ Don't use it unless you're **absolutely sure** |
| `git push <remote> --all` | Push **all** local branches |
| `git push <remote> --tags` | Push all local **tags**. Tags are **not** sent automatically, not even with `--all` |

---

### 3. How Pushing Works
```
Before:   local  main:  A──B──C──D──E
          origin main:  A──B──C

git push origin main

After:    origin main:  A──B──C──D──E   (fast-forward)
```
* `git push origin main` is essentially like running **`git merge main` inside the remote repository**.
* Part of the "syncing" family: **push = upload**, **fetch/pull = download**. After moving changesets, a **merge** integrates them at the destination.

---

### 4. Push to Bare Repositories
* Modern practice: a **`--bare`** repo (often hosted by a third party like Bitbucket) acts as the central **origin**.
* Pushing changes the remote's branch structure, so it's **safest to push to bare repos**. They have **no working directory**, so no in-progress work can be disturbed.

---

### 5. Force Pushing
* Git **refuses non-fast-forward pushes** to protect the central history. If histories have diverged: **pull → merge → push again** (like `svn update` before committing).
* **`--force`** makes the remote branch **match your local one**, **deleting any upstream changes** since your last pull.
* ✅ The **only** legitimate case: you just shared commits, then fixed them with `git commit --amend` or an interactive rebase, **and you're certain no teammate has pulled them**.

> 📝 *Extra note (not in the original article):* Prefer **`git push --force-with-lease`**. It only overwrites the remote branch if it's still where you last saw it, so you won't accidentally wipe out a teammate's new commits.

---

### 6. Examples

#### Default push (with cleanup)
```bash
git checkout main
git fetch origin main
git rebase -i origin/main     # squash commits, fix messages, etc.
git push origin main          # should be a clean fast-forward
```

#### Amended force push
```bash
# make changes, git add
git commit --amend            # update the existing commit
git push --force origin main  # the amended commit diverges from the remote, so force is needed
```

#### Deleting a remote branch or tag
```bash
git branch -D branch_name        # delete locally
git push origin :branch_name     # delete on the remote (":" prefix = push "nothing" to it)
```
> 📝 *Extra note (not in the original article):* The more readable modern form is `git push origin --delete branch_name`. It also works for tags.

---

### Cheat Sheet
| Task | Command |
|---|---|
| Publish a branch | `git push origin <branch>` |
| First push + set upstream | `git push -u origin <branch>` *(extra)* |
| All branches / all tags | `git push origin --all` / `git push origin --tags` |
| Overwrite remote (careful!) | `git push --force origin <branch>` (better: `--force-with-lease`) |
| Delete remote branch | `git push origin :<branch>` or `git push origin --delete <branch>` |

---

### Summary Takeaway
`git push` **uploads your commits** to a remote branch, like a merge performed on the remote. Git **blocks non-fast-forward pushes** to protect shared history, so **fetch/rebase or merge first**. Push to **bare** repos, push **tags explicitly**, and only use **`--force`** (preferably `--force-with-lease`) for commits you just amended or rebased that nobody else has pulled.
