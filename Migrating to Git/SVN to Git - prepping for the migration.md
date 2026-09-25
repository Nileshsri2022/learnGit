Here is a clear breakdown of the key points about **Preparing for an SVN to Git migration** from the text:

> Source: https://www.atlassian.com/git/tutorials/svn-to-git-prepping-your-team-migration

---

### 1. The Golden Rule: Git Is Not SVN
* Once you've decided to switch, the next step is migrating your **development workflow**.
* To realize Git's full potential, be **open to new ways of thinking** about version control.
* Adoption can take **a few days to several months**, depending on team size.

---

## PART A — For Administrators

### 2. Training: Basic Git Commands
Git once had a reputation for a steep learning curve, but **sensible defaults** and **contextual help messages** have made onboarding much easier. Core commands to get a team started:

| Task | Command |
|---|---|
| Tell Git who you are | `git config --global user.name "Sam Smith"` · `git config --global user.email sam@example.com` |
| Create a local repo | `git init` |
| Copy a repo (local / remote) | `git clone /path/to/repository` · `git clone username@host:/path/to/repository` |
| Stage files | `git add *` |
| Commit (locally, not to remote) | `git commit -m "Commit message"` |
| Commit all tracked changes | `git commit -a` |
| Push to remote `main` | `git push origin main` |
| Status | `git status` |
| Connect / list remotes | `git remote add origin <server>` · `git remote -v` |
| Create + switch branch | `git checkout -b <branchname>` |
| Switch branch | `git checkout <branchname>` |
| List branches | `git branch` |
| Delete local branch | `git branch -d <branchname>` |
| Push a branch / all branches | `git push origin <branchname>` · `git push --all origin` |
| Delete remote branch | `git push origin :<branchname>` |
| Fetch + merge from remote | `git pull` |
| Merge another branch into active one | `git merge <branchname>` |
| View conflicts / vs. base / preview | `git diff` · `git diff --base <filename>` · `git diff <source> <target>` |
| Mark conflict resolved | `git add <filename>` |
| Tag a release | `git tag 1.0.0 <commitID>` (ID from `git log`) |
| Push all tags | `git push --tags origin` |
| Discard working-tree changes to a file | `git checkout -- <filename>` |
| Drop **all** local changes & commits | `git fetch origin` · `git reset --hard origin/main` |
| Search working directory | `git grep "foo()"` |

* Note: Git strips some characters (e.g. trailing periods) from `user.name`.

---

### 3. Choosing a Migration Strategy
| Option | Notes |
|---|---|
| **Migrate everything** and drop SVN | ✅ **Preferred** — least workflow complexity |
| Keep old projects in SVN, **new projects in Git** | Hybrid |
| Migrate **some** projects, keep others in SVN | Hybrid |
| Use **SVN and Git simultaneously** on the same projects | Hybrid — often needed by large companies |

* Large companies with dozens of teams/hundreds of projects often find a **hybrid approach safer**.

### 4. Migration Tools
| Tool | Best for | Key points |
|---|---|---|
| **Atlassian's migration scripts** | **Abrupt**, full transition | Convert SVN repos into **native Git history** — no interoperability issues afterward. Walkthrough covers extracting authors and reorganizing non-standard SVN layouts. |
| **SVN Mirror for Stash (Bitbucket) plugin** | **Hybrid** codebase, large companies | Use SVN **and** Git on the same project for as long as you like; teams migrate at their own pace. |
| **git-svn** | Exploration & **training phase** | Commit locally with Git, push to a central SVN repo (`svn commit`-style). ⚠️ **Temporary only** — SVN backend means no real Git branching/advanced workflows. |

---

### 5. Rollout Strategies (Moving the *People*)

| Strategy | ✅ Pros | ❌ Cons |
|---|---|---|
| **External Git consultants** (e.g. Atlassian Partners) | Workflow tailored for you quickly; expert training available | Team may be **left in the dark** when the consultant leaves |
| **Internal Git champion** (an enthusiastic in-house developer) | Expertise stays **in-house**; ideal for strong developer cultures | Big time investment; risk of choosing/implementing the **wrong workflow** |
| **Pilot team** (small team, isolated project) | **Whole-team buy-in**; missing pieces caught early | More initial training; temporary productivity dip — *"short-term pain is worth the long-term gain"* |

* 💡 Best combo: a **pilot team** that includes **consultants + an internal champion**.

---

### 6. Security & Permissions — A Fundamental Rethink
* **SVN:** one big central repo; access restricted **per folder**.
* **Git:** developers must retrieve the **entire repository**; you can't check out a subset. Permissions apply to **whole repositories only**.
* ➡️ You must **split a monolithic SVN repo into several smaller Git repos**.
  * Example: Atlassian's **Jira** team kept all plugins in one SVN repo; after migrating, **each plugin got its own Git repo**.
* Git was designed to securely integrate contributions from **thousands of Linux developers**, so any access-control model is achievable — it may just require rethinking your **build cycle**.
* For dependencies between many repos, consider a **dependency management layer** (also helps build times via caching).

---

## PART B — For Developers

### 7. A Repository for Every Developer
* Biggest adjustment: Git is **distributed** — everyone has a **full copy** of the repository.
* `svn checkout` (working copy) → **`git clone`** (whole repo).
* Collaboration = **moving branches between repositories** with `git push`, `git fetch`, `git pull`.
* A Git commit represents the **state of the whole project**, not just file modifications.
* You can **commit locally without sharing** → experiment freely, work offline, and almost every command is faster.
* A **remote** is just a **bookmark** for a URL, not a live link. Until you push/pull, you work in isolation.
* **Local repo** = where you develop. **Remote repo** = where code is shared; no active development happens there.

### 8. Don't Be Scared of Branching or Merging
| SVN | Git |
|---|---|
| Edit → `svn commit` → others `svn update` | Life revolves around **branching and merging** |
| Branches reserved for big, long-running efforts because **merging is dangerous** | Start **everything** on a new branch: `git checkout -b <branch>` |
| Bound to one line of development (`trunk/`) | Broke it beyond repair? `git branch -d <branch>` and throw it away. Built something useful? **Open a pull request** to merge into `main`. |

### 9. Potential Git Workflows
| Workflow | When to use |
|---|---|
| **Centralized** | Closest to SVN — good place to **start** |
| **Feature branch** | Isolate work-in-progress, protect shared branches; basis for **pull requests** |
| **Gitflow** | Formal extension of feature branching for **larger teams with defined release cycles** |
| **Forking** | **Maximum isolation/control**, many contributors to one repo |

* Workflows can be **combined** by role (e.g. core developers use feature branches, contractors work from forks).
* ⭐ Recommended for professional teams: the **feature branch workflow** — truly distributed, secure, scalable, and agile.

---

### Summary Takeaway
Migrating from SVN to Git is as much about **people and process** as code. Admins should pick a **migration strategy** (full vs. hybrid), the right **tools** (Atlassian scripts, SVN Mirror, git-svn), a **rollout strategy** (consultants, champions, pilot teams), and rethink **permissions** by splitting monolithic repos. Developers must embrace a **repository per developer**, **local commits**, and **fearless branching and merging** — ideally through a **feature branch workflow**.
