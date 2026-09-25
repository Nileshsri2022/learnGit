Here is a clear breakdown of the key points about **Git prune** from the text:

> Source: https://www.atlassian.com/git/tutorials/git-prune

---

### 1. What Is `git prune`?
* An **internal housekeeping utility** that deletes **unreachable ("orphaned") Git objects**.
* **Unreachable** = not accessible from **any ref**. Any commit that can't be reached through a **branch or tag** counts.
* It's a **garbage-collection command** and a **child command of `git gc`** — you **generally don't run it directly**.

---

### 2. Demo — Creating & Pruning an Unreachable Commit

#### Step 1 — Make a repo with two commits
```bash
mkdir git-prune-demo && cd git-prune-demo
git init .
echo "hello git prune" > hello.txt
git add hello.txt
git commit -am "added hello.txt"                  # 994b122

echo "this is second line txt" >> hello.txt
git commit -am "added another line to hello.txt"  # 5178bec
```

#### Step 2 — Make the second commit unreachable
```bash
git reset --hard 994b122
git log            # only "added hello.txt" is shown now
```
* The commit looks lost, but **Git is very strict about not deleting history** — it's just **detached**:
```bash
git checkout 5178bec   # "You are in 'detached HEAD' state…" — the commit is still there
git checkout main      # "Warning: you are leaving 1 commit behind, not connected to any of your branches"
```
* Git helpfully suggests keeping it: `git branch <new-branch-name> 5178bec`.

#### Step 3 — Try to prune it
```bash
git prune --dry-run --verbose
```
* 🤔 **Empty output** — nothing would be deleted! Why? The commit is **still referenced** somewhere — most likely by the **reflog** (`git reflog` shows the actions you took). Git also has **internal expiration dates** before pruning detached commits.
* This shows (1) why `git prune` shouldn't be used **standalone outside `git gc`**, and (2) how **hard it is to truly lose data** in Git.

#### Step 4 — Wipe the reflog (demo only!)
```bash
git reflog expire --expire=now --expire-unreachable=now --all
```
* ⚠️ A **brutal and dangerous** command — a casual Git user should **never** need it.

#### Step 5 — Now the prune finds the objects
```bash
git prune --dry-run --verbose --expire=now
```
```
1782293bdfac16b5408420c5cb0c9a22ddbdd985 blob
5178becc2ca965e1728554ce1cb8de2f2c2370b1 commit
a1b3b83440d2aa956ad6482535cbd121510a3280 commit
f91c3433eae245767b9cd5bdb46cd127ed38df26 tree
```

---

### 3. Options

| Option | Effect |
|---|---|
| `-n`, `--dry-run` | Don't prune — just **show** what would happen |
| `-v`, `--verbose` | Show all objects and actions taken |
| `--progress` | Show progress output |
| `--expire <time>` | Only expire objects **older than** `<time>` |
| `<head>…` | Keep objects reachable from the given head ref(s) |

---

### 4. `git prune` vs. `git fetch --prune` vs. `git remote prune`

| Command | What it deletes |
|---|---|
| `git prune` | **Local unreachable objects** (detached commits, blobs, trees) |
| `git remote prune <remote>` | Local **remote-tracking refs** for branches that **no longer exist on the remote** |
| `git fetch --prune` | Same as above, but **fetches the latest remote state first** ≈ `git fetch --all && git remote prune` |

* Deleting stale remote refs is very handy in team workflows where remote branches are **deleted after merging to `main`**.

---

### 5. FAQ
* **How do I clean outdated branches?** → **`git fetch --prune`** is the best tool: it fetches all remote refs and deletes the ones no longer used on the remote.
* **Does `git remote prune origin` delete my local branch?** → **No.** Git keeps local refs and remote-tracking refs separately (`refs/heads/*` vs. `refs/remotes/origin/*`). It only prunes the **remote-tracking** refs, leaving your **local work safe**.

---

### Summary Takeaway
`git prune` removes **unreachable objects** and is meant to run **as part of `git gc`** — you'll almost never call it yourself. Thanks to the **reflog** and expiration rules, "lost" commits are very hard to truly delete. For day-to-day cleanup of **stale remote branches**, use **`git fetch --prune`** (or `git remote prune origin`), which never touches your local branches.
