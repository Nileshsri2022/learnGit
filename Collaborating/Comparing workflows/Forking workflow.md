Here is a clear breakdown of the key points about the **Forking Workflow** from the text:

> Source: https://www.atlassian.com/git/tutorials/comparing-workflows/forking-workflow

---

### 1. What Is the Forking Workflow?
* **Fundamentally different** from other workflows: instead of one central server-side repo, **every developer gets their own server-side repository**.
* Each contributor has **two** repos:

| Repo | Visibility | Who pushes |
|---|---|---|
| **Local** | Private | The developer |
| **Server-side fork** | Public | **Only** that developer (others can pull from it) |

* Most often seen in **public open-source projects**.

---

### 2. Main Advantage
* Contributions are integrated **without everyone pushing to one central repo**.
* Developers push to **their own** forks. **Only the maintainer** pushes to the official repo.
* The maintainer can accept commits from **anyone without giving them write access**. That makes it safe for large, organic teams, **including untrusted third parties**.
* The branching model usually follows **Gitflow**-style feature branches.

---

### 3. How It Works — Step by Step
1. A developer **forks** the official server-side repo (creating their own server-side copy).
2. They **clone** that copy to their machine.
3. They **add a remote** for the official repo.
4. They create a **local feature branch**.
5. They make changes on it…
6. …and **commit** them.
7. They **push** the branch to **their own fork**.
8. They open a **pull request** from that branch to the official repo.
9. The PR is **approved and merged** into the official repo.

* **Maintainer's side:** pull the contributor's changes locally → check that nothing breaks → merge into local `main` → **push `main`** to the official repo. Everyone else then **pulls from the official repo** to sync.
* The "official" repo is just a **convention**: it's official only because it's the **maintainer's public repo**.

```
          ┌──────────────── Official repo (maintainer) ◀── only maintainer pushes
          │ fork                     ▲
          ▼                          │ pull request
   Your fork (origin) ───────────────┘
      ▲      │ clone
 push │      ▼
   Your local repo ◀── git pull upstream main (stay in sync)
```

---

### 4. Forking vs. Cloning
* **Forking isn't a special Git operation.** A fork is just a **server-side `git clone`**, usually managed by a hosting service like Bitbucket/GitHub.
* There's **no `git fork` command**. A clone is simply a copy of a repo and its history.

---

### 5. Branching in the Forking Workflow
* Personal public repos are really just a **convenient way to share branches**.
* You still **isolate features in branches**, just like in the Feature Branch and Gitflow workflows. The only difference is **how branches are shared**:

| Workflow | Branches are shared by… |
|---|---|
| Feature Branch / Gitflow | **Pushing** to the official repo |
| Forking | Being **pulled** into another developer's (the maintainer's) repo |

---

### 6. Hands-On

#### Fork the repository
* Use your host's **Fork** button (or SSH into the server and `git clone` to another location).

#### Clone your fork
```bash
git clone https://user@bitbucket.org/user/repo.git
```

#### Add the upstream remote
* You need **two remotes**. By convention:

| Remote | Points to | Created by |
|---|---|---|
| **`origin`** | **Your fork** | `git clone` (automatic) |
| **`upstream`** | The **official** repo | You, manually |

```bash
git remote add upstream https://bitbucket.org/maintainer/repo
# private upstream? include a username (you'll be asked for a password):
git remote add upstream https://user@bitbucket.org/maintainer/repo.git
```

#### Work in a branch
```bash
git checkout -b some-feature
# edit some code
git commit -a -m "Add first draft of some feature"
```
* Changes stay **private** until you push them.
* Keep up with the official project:
```bash
git pull upstream main    # usually a fast-forward, since you work on a feature branch
```

#### Make a pull request
```bash
git push origin feature-branch    # origin = YOUR fork, not the main codebase
```
* Then click **"Pull request"** on the host and choose your `feature-branch` → **upstream's `main`**.

---

### 7. High-Level Example
1. You want to contribute to `bitbucket.org/userA/open-project`.
2. You **fork** it to `bitbucket.org/YourName/open-project`.
3. `git clone https://bitbucket.org/YourName/open-project`
4. Create a **feature branch**.
5. Do the work and `git commit`.
6. **Push** the branch to your fork.
7. Open a **pull request** against `bitbucket.org/userA/open-project`.

---

### Summary Takeaway
In the Forking Workflow, every contributor gets a **public server-side fork** plus a private local clone. You push to **your fork (`origin`)**, stay in sync with the **official repo (`upstream`)**, and propose changes through **pull requests**. The maintainer never has to hand out write access. It's a **"pull"-style** model, ideal for **open source**, and also useful for companies that want **tight control over what gets merged** (e.g. deploy managers, strict release cycles).
