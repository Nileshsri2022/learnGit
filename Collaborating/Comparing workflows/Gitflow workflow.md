Here is a clear breakdown of the key points about the **Gitflow Workflow** from the text:

> Source: https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow

---

### 1. A Legacy Workflow ⚠️
* Gitflow was once a **disruptive, novel** branching strategy, but it has **fallen in popularity** in favor of **trunk-based development**, now considered best practice for modern continuous delivery and **DevOps**.
* It can be **challenging with CI/CD**. Atlassian documents it mainly **for historical purposes**.

---

### 2. What Is Gitflow?
* A branching model using **feature branches + multiple primary branches**, first published by **Vincent Driessen (nvie)**.
* Compared to trunk-based development: **more, longer-lived branches** and **larger commits**. Features are merged only when **complete**.
  * ⚠️ Long-lived branches need more collaboration to merge, risk **drifting from trunk**, and can introduce **conflicts**.
* Suits projects with a **scheduled release cycle**.
* Adds **no new concepts or commands** beyond the Feature Branch Workflow. It assigns **specific roles** to branches and defines **how and when they interact**. You still get PRs, isolated experiments, and efficient collaboration.

---

### 3. The Branches

| Branch | Branches off | Merges into | Purpose |
|---|---|---|---|
| **`main`** | — | — | **Official release history** (tag every commit with a version number) |
| **`develop`** | `main` | — | **Integration branch** for features, holds the complete history |
| **`feature/*`** | `develop` | `develop` | One feature each. **Never touches `main`** |
| **`release/*`** | `develop` | `main` **and** `develop` | Prepare a release: **only bug fixes, docs, release tasks**, no new features |
| **`hotfix/*`** | **`main`** (the only branch that does) | `main` **and** `develop` (or the current release) | Quickly **patch production** |

```
main     ●─────────────────●───────────●──────  (v0.1.0)  (v0.1.1)
          \               / \         /
hotfix     \             /   ●──●────●  ──────────────┐
            \           /                              ▼
release      \    ●──●──●  ──────────────────┐        (merged into develop too)
              \  /                            ▼
develop        ●──●──────●──────●─────────────●──────●
                   \    /  \   /
feature             ●──●    ●─●
```

---

### 4. `develop` and `main`
```bash
git branch develop
git push -u origin develop
```
* `develop` contains the **complete history**, and `main` an **abridged** one. Other devs clone and track `develop`.
* With the **git-flow extension**:
```bash
$ git flow init
Branch name for production releases: [main]
Branch name for "next release" development: [develop]
Feature branches? [feature/]
Release branches? [release/]
Hotfix branches? [hotfix/]
Support branches? [support/]
Version tag prefix? []
```

---

### 5. Command Reference — Plain Git vs. git-flow Extension

| Action | Plain Git | git-flow |
|---|---|---|
| **Start feature** | `git checkout develop` · `git checkout -b feature_branch` | `git flow feature start feature_branch` |
| **Finish feature** | `git checkout develop` · `git merge feature_branch` | `git flow feature finish feature_branch` |
| **Start release** | `git checkout develop` · `git checkout -b release/0.1.0` | `git flow release start 0.1.0` |
| **Finish release** | `git checkout main` · `git merge release/0.1.0` (+ merge into `develop`, tag, delete branch) | `git flow release finish '0.1.0'` |
| **Start hotfix** | `git checkout main` · `git checkout -b hotfix_branch` | `git flow hotfix start hotfix_branch` |
| **Finish hotfix** | `git checkout main` · `git merge hotfix_branch` · `git checkout develop` · `git merge hotfix_branch` · `git branch -D hotfix_branch` | `git flow hotfix finish hotfix_branch` |

---

### 6. Why Release & Hotfix Branches?
* **Release branches**
  * One team **polishes the current release** while another keeps building features for the **next** one.
  * Creates **well-defined phases** ("This week we're preparing v4.0") that are visible in the repo structure.
  * Must be merged back into `develop`, because critical fixes made during release prep are needed by future features.
  * A great place for a **pull request** if your org stresses code review.
* **Hotfix branches**
  * Fix production issues **without interrupting** the workflow or waiting for the next release.
  * Think of them as **ad hoc release branches that work directly with `main`**.
  * After merging, **tag `main`** with an updated version number.

---

### 7. Complete Examples
```bash
# Feature flow
git checkout main
git checkout -b develop
git checkout -b feature_branch
# work happens on feature branch
git checkout develop
git merge feature_branch
git checkout main
git merge develop
git branch -d feature_branch

# Hotfix flow
git checkout main
git checkout -b hotfix_branch
# work is done, commits are added to the hotfix_branch
git checkout develop
git merge hotfix_branch
git checkout main
git merge hotfix_branch
```

---

### 8. The Overall Flow
1. `develop` is created from `main`.
2. `release` is created from `develop`.
3. `feature` branches are created from `develop`.
4. A finished `feature` → merged into `develop`.
5. A finished `release` → merged into `develop` **and** `main`.
6. An issue in `main` → a `hotfix` is created from `main`.
7. A finished `hotfix` → merged into `develop` **and** `main`.

---

### Summary Takeaway
Gitflow gives each branch a **strict role**: `main` (releases), `develop` (integration), `feature/*`, `release/*`, and `hotfix/*`. That makes it a good fit for **scheduled, release-based** software, with a **dedicated channel for production hotfixes**. But its **long-lived branches** clash with modern **CI/CD**, so it's now considered **legacy**, and **trunk-based development** is generally preferred.
