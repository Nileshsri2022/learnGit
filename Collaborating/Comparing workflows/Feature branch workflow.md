Here is a clear breakdown of the key points about the **Git Feature Branch Workflow** from the text:

> Source: https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow

---

### 1. The Core Idea
* **All feature development happens in a dedicated branch**, never directly on `main`.
* Benefits:
  * Many developers can work on a feature **without disturbing the main codebase**.
  * **`main` never contains broken code**, which is a huge plus for **continuous integration**.
  * Enables **pull requests**, which are discussions around a branch: teammates can **sign off** on a feature, or you can ask for help when stuck.

---

### 2. A Composable, Branch-Focused Workflow
| Feature Branch Workflow | Other workflows (Gitflow, Forking) |
|---|---|
| **Branching-model focused**: a framework for creating and managing branches | More **repository focused** |
| Can be **incorporated into** other workflows | Traditionally **use** the Feature Branch Workflow for their branching model |

---

### 3. How It Works
* Assumes a **central repository**, where `main` = the **official project history**.
* Create a new branch **for every feature**, with **descriptive names** like `animated-menu-items` or `issue-#1061`, so each branch has a clear, focused purpose.
* Git makes **no technical distinction** between `main` and feature branches: edit, stage, and commit as usual.
* **Push feature branches** to the central repo. That lets you share work without touching official code, and it's a handy **backup**. Only `main` is "special", so many feature branches on the server cause no problems.

---

### 4. Lifecycle of a Feature Branch

| Step | Commands |
|---|---|
| **1. Start from the latest `main`** | `git checkout main` · `git fetch origin` · `git reset --hard origin/main` |
| **2. Create a branch** | `git checkout -b new-feature` |
| **3. Update, add, commit** | `git status` · `git add <some-file>` · `git commit` (as many commits as needed) |
| **4. Push to the remote** | `git push -u origin new-feature` (`-u` sets up tracking, so later a plain `git push` works) |
| **5. Open a pull request** | In Bitbucket/GitHub: add reviewers |
| **6. Resolve feedback** | Fix locally → commit → push. Updates appear in the PR |
| **7. Merge the PR** | Resolve any conflicts, then merge into `main` once approved |

---

### 5. Pull Requests
* When a feature is done, you **don't merge immediately**. You push the branch and **file a pull request** to merge into `main`, so others can **review** first.
* Code review is the big benefit, but PRs are really a **generic way to talk about code**: a **discussion dedicated to a branch**. Use them **early** too, e.g. to ask for help. Interested people are notified and see your question **right next to the commits**.
* After approval, publishing works like the **Centralized Workflow**: sync local `main` → merge the feature → push `main`.

---

### 6. Example: Mary, Bill & John

| Scene | What happens | Commands |
|---|---|---|
| **Mary begins a feature** | Creates an isolated branch, commits as usual | `git checkout -b marys-feature main` · `git add` · `git commit` |
| **Mary goes to lunch** | Pushes for backup/visibility | `git push -u origin marys-feature` |
| **Mary finishes** | Pushes latest commits, files a PR (`marys-feature` → `main`) | `git push` |
| **Bill receives the PR** | Reviews and requests changes; back-and-forth **in the PR** | — |
| **Mary makes changes** | Same edit → stage → commit → push cycle. Everything shows up in the PR. Bill could also pull the branch and add commits himself | `git commit` · `git push` |
| **Mary publishes** | Bill or Mary merges into `main` | `git checkout main` · `git pull` · `git pull origin marys-feature` · `git push` |
| **Meanwhile, John…** | Does the same thing on **his own** feature branch, fully independently | — |

* The merge usually creates a **merge commit** (a "symbolic joining" of the feature). Prefer a linear history? **Rebase the feature onto `main`** first so the merge is a **fast-forward**.
* Many GUIs do all of this with an **"Accept"/"Merge"** button, or at least **auto-close** the PR once the branch is merged.

```
main:            ●───────●───────────────●  (merge)
                  \                     /
marys-feature:     ●───●───●───(review)●
                  \
johns-feature:     ●───●───●  … (independent)
```

---

### Summary Takeaway
The Feature Branch Workflow keeps **every feature in its own branch**, so **`main` stays stable** and **pull requests** drive code review and discussion. It's **branching-focused**, and repo-focused workflows like **Gitflow** and **Forking** build on it. Push branches often (for backup and visibility), collect feedback in the PR, and merge (optionally after a **rebase** for a linear history) once approved.
