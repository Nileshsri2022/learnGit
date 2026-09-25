Here is a clear breakdown of the key points about **Making a Pull Request** from the text:

> Source: https://www.atlassian.com/git/tutorials/making-a-pull-request

---

### 1. What Is a Pull Request (PR)?
* A feature of hosting platforms (Bitbucket, GitHub, GitLab…) that provides a **web interface for discussing proposed changes** before they're integrated into the official project.
* **Simplest form:** a notification — *"my feature branch is ready, please review and merge it into `main`."*
* **More than a notification:** a **dedicated discussion forum** for the feature.
  * Teammates post **feedback** and can even push **follow-up commits**.
  * All activity is **tracked inside the PR**.
* **Why it's better than email:** SVN/Git can send notification emails, but discussing changes over email threads gets messy — especially with follow-up commits.

---

### 2. Anatomy of a Pull Request
Filing a PR = **requesting** that another developer (e.g. the maintainer) **pulls** a branch from your repo into theirs. You need **4 pieces of information**:

| Field | Example |
|---|---|
| **Source repository** | Your repo / fork |
| **Source branch** | `some-feature` |
| **Destination repository** | The official project repo |
| **Destination branch** | `main` |

* Hosting platforms set **sensible defaults**; your workflow may require different values.

---

### 3. How It Works — General Process
1. Developer creates the feature in a **dedicated branch** in their local repo.
2. Developer **pushes** the branch to a public (hosted) repository.
3. Developer **files a pull request**.
4. The team **reviews, discusses, and alters** the code.
5. The maintainer **merges** the feature into the official repo and **closes** the PR.

* ✅ Works with **Feature Branch**, **Gitflow**, and **Forking** workflows.
* ❌ Doesn't work with the **Centralized Workflow** — PRs need **two distinct branches or two distinct repositories**.

---

### 4. Pull Requests in Different Workflows

| Workflow | Source → Destination | Notes |
|---|---|---|
| **Feature Branch** | `feature` branch → `main` (**same** repo) | Open a PR instead of merging straight into `main`. You can even open a PR for **work-in-progress** to ask for help. |
| **Gitflow** | feature → `develop`; release/hotfix → `develop` **and** `main` | PRs give a place to discuss release and maintenance branches and formally manage all merges. |
| **Forking** | Your **public fork** → official repo `main` (**different** repos) | Notification is crucial — the maintainer can't otherwise know you pushed commits to your fork. |

* **Forking bonus:** you can PR into a **teammate's fork** to co-develop a feature, then open another PR to the official project when done.

---

### 5. Full Example — Forking Workflow (Mary & John)
*Mary* = developer, *John* = project maintainer (owns the official repo).

#### Step 1 — Mary forks the official project
* On Bitbucket, she opens John's repo and clicks **Fork** → gets a **server-side copy**.

#### Step 2 — Mary clones her fork
```bash
git clone https://user@bitbucket.org/user/repo.git
```
* `origin` automatically points to **Mary's fork**.

#### Step 3 — Mary develops a feature on a new branch
```bash
git checkout -b some-feature
# Edit some code
git commit -a -m "Add first draft of some feature"
```
* Use as many commits as needed; clean up messy history with an **interactive rebase** (squash/remove commits) so the maintainer can follow the PR easily.

#### Step 4 — Mary pushes to **her own** repo (not the official one)
```bash
git push origin some-branch
```

#### Step 5 — Mary creates the pull request
* Clicks **Pull request** in her fork. Source repo is pre-filled; she sets:
  * Source branch → her **feature branch**
  * Destination repo → **John's** repo
  * Destination branch → **`main`**
* Adds a **title**, **description**, and optional **Reviewers**.
* John is notified (feed and optionally email).

#### Step 6 — John reviews
* Sees the **description**, **commit history**, and **diff** of all changes.
* Can **Merge** right away, or **comment** on the whole PR or on specific commits (e.g. he found a small bug).

#### Step 7 — Mary adds a follow-up commit
* Replies in the PR, fixes the bug, commits, and **pushes again** → the commit **automatically appears in the same PR** next to John's comment.

#### Step 8 — John accepts
* Merges the feature into `main` and closes the PR. Others get it with a normal **`git pull`**.

---

### Summary Takeaway
Pull requests are a **formal, web-based way to propose, discuss, and review changes** before merging. They require a **source repo/branch** and a **destination repo/branch**, work with Feature Branch, Gitflow, and Forking workflows, and keep all feedback and follow-up commits in **one place**. PRs don't replace Git workflows — they are a **convenient addition** that makes collaboration accessible to the whole team.
