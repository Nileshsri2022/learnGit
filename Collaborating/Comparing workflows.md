Here is a clear breakdown of the key points about **Comparing Git workflows** from the text:

> Source: https://www.atlassian.com/git/tutorials/comparing-workflows

---

### 1. What Is a Git Workflow?
* A **recipe or recommendation** for using Git to get work done **consistently and productively**.
* Git is very flexible and has **no standard process**, so a team must **agree** on how changes flow.
* Workflows are **guidelines, not rules** — mix and match to suit your team.

#### What makes a workflow successful?
Consider your **team culture** — the workflow should boost productivity, not burden it. Ask:
* ✅ Does it **scale** with team size?
* ✅ Is it easy to **undo mistakes**?
* ✅ Does it avoid adding **unnecessary cognitive overhead**?

---

### 2. Centralized Workflow
* Best for teams **transitioning from SVN** and **small teams**.
* A **central repository** is the single point of entry for all changes; everyone commits to **`main`** (instead of SVN's `trunk`). **No other branches** are required.
* No defined pull-request or forking patterns.

#### Advantages over SVN (even with the same workflow)
1. Every developer has a **full local copy** → work independently, commit locally, sync upstream **when convenient**.
2. Access to Git's **robust branching and merging** model.

#### How it works
| Step | Command |
|---|---|
| **1. Initialize the central (bare) repo** | `ssh user@host git init --bare /path/to/repo.git` *(or let a host like Bitbucket create it)* |
| **2. Clone it** | `git clone ssh://user@host/path/to/repo.git` → creates `origin` |
| **3. Edit, stage, commit locally** | `git status` · `git add <some-file>` · `git commit` |
| **4. Push to central** | `git push origin main` |

* Central repos should **always be bare** (no working directory); the `.git` suffix on the name conventionally marks a bare repo.
* Pushing is like `svn commit`, but uploads **all local commits** not yet in central `main`.

#### Managing conflicts
* Central history is **sacred and immutable** — if your local commits **diverge**, Git **refuses** the push.
* Fix: **fetch** the new central commits and **rebase** your changes on top → a perfectly **linear** history.
* Conflicts are resolved with the same `git status` / `git add` commands you already know; `git rebase --abort` gets you out of trouble.

---

### 3. Example — John & Mary (Centralized Workflow)

1. **John** and **Mary** each develop features locally (edit → stage → commit). Local repos are **private**, so they don't care what the other is doing.
2. **John publishes first:**
   ```bash
   git push origin main   # works — central hasn't changed since he cloned
   ```
3. **Mary tries to publish:**
   ```bash
   git push origin main
   ```
   ```
   error: failed to push some refs to '/path/to/repo.git'
   hint: Updates were rejected because the tip of your current branch is behind
   hint: its remote counterpart. Merge the remote changes (e.g. 'git pull')
   hint: before pushing again.
   ```
4. **Mary rebases on top of John's commits:**
   ```bash
   git pull --rebase origin main
   ```
   * `--rebase` moves Mary's commits to the tip of the updated `main`.
   * Without it the pull still works, but creates a **superfluous merge commit** every time. For this workflow, **always rebase**.
5. **Mary resolves a conflict (if any):**
   * Rebase replays commits **one at a time**, so conflicts are handled **commit by commit** — cleaner history, easier bug hunting and rollbacks.
   ```
   CONFLICT (content): Merge conflict in <some-file>
   ```
   ```bash
   git status                 # conflicted files listed under "Unmerged paths" (both modified)
   # edit the file(s) to fix the conflict
   git add <some-file>
   git rebase --continue
   # confused? start over:
   git rebase --abort
   ```
6. **Mary publishes successfully:**
   ```bash
   git push origin main
   ```

* ⚠️ Limitation: the conflict-resolution step can become a **bottleneck as the team grows**. Next step: the **Feature Branch Workflow**.

---

### 4. Other Common Workflows

| Workflow | Core idea |
|---|---|
| **Feature Branching** | Logical extension of Centralized. **All feature work happens in dedicated branches**, never directly on `main` → `main` never contains broken code (great for CI). |
| **Gitflow** | Published by **Vincent Driessen (nvie) in 2010**. A **strict branching model built around releases**; no new commands — just specific **roles** for branches and rules for how they interact. |
| **Forking** | Every developer gets their **own server-side repo** → each contributor has **two repos**: a private local one and a public server-side one. |

* The Centralized Workflow is a **building block** — most workflows still have a central repo that developers push to and pull from.

---

### 5. Guidelines for Choosing a Workflow
* There's **no one-size-fits-all** workflow. It should fit **team culture** *and* **business culture** (e.g. branches matching tasks in a tracker like Jira).

| Guideline | Why |
|---|---|
| **Short-lived branches** | The longer a branch lives apart from production, the higher the risk of **merge conflicts** and deployment pain. |
| **Minimize & simplify reverts** | Prevent bad merges proactively (e.g. test branches before merging to `main`), and make reverts easy when accidents happen. |
| **Match the release schedule** | Releasing many times a day → keep `main` **stable**. Releasing less often → use **Git tags** to mark versions. |

---

### Summary Takeaway
* There is **no one-size-fits-all** Git workflow.
* A workflow should be **simple** and **enhance team productivity**.
* **Business requirements** should shape your workflow.
* Start with **Centralized** (great for SVN migrants and small teams), then grow into **Feature Branch**, **Gitflow**, or **Forking** as needs evolve.
