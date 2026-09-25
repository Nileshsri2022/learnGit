Here is a clear breakdown of the key points about **Perforce to Git: Why make the move** from the text:

> Source: https://www.atlassian.com/git/tutorials/perforce-git

---

### 1. Context
* **Git** is the leading SCM (source code management) tool — popular from indie devs to enterprises, and in critical open-source projects like **Android** and the **Linux kernel**. Interest has grown steadily since its **2005** release.
* **Perforce** is a **commercial, centralized** SCM that still appeals to **game developers** and some other niches.
* The article explains **why Git surpassed Perforce** for general development, and why game dev has been **slower to switch**.

---

### 2. A Bit of History (1995 → Today)
| Tool | Profile |
|---|---|
| **CVS** | Free — and "worth every penny" feature-wise |
| **ClearCase** | Very expensive but powerful (up to **64-way merges**, global teams, multi-module projects) |
| **Perforce** | Cheaper than ClearCase, less powerful, but **fast** and gets the job done → the recipe for commercial success |
| **Subversion** | Stagnated |
| **Git** | Now the **top SCM** for software developers |

---

### 3. How Git "Ate the World"

#### a) Distributed speed
* Every developer has the **full history locally**.
  * Initial **clone is slower** (unless using Smart Mirroring)…
  * …but `commit`, `blame`, `diff`, `merge`, `log` are **dramatically faster**.
* **Perforce** mostly needs a **server connection** even to view history. The single central server becomes a **bottleneck** as teams grow — even these need **write access** to the server:

| Action | Perforce command |
|---|---|
| View history | `p4 changes` |
| Create a tag | `p4 label` / `p4 tag` |
| Make a branch | `p4 integ` |
| Make a file writable | `p4 edit` |

#### b) Cost
| Perforce | Git |
|---|---|
| Several **hundred dollars per user** + annual renewals; expensive central server hardware | **Open source & free**; hosting (e.g. Bitbucket Cloud) is affordable |

#### c) Workflow & collaboration
* Git offers **simple, cheap branching** → many workflows: **task branching, Git Flow, forked repos**, plus strong code review tools.
* Easy **cross-company collaboration**; even without network access, **`git patch`/`git bundle`** make sharing simple.
* **Perforce problems:**
  * Tracks branches **per file** (Git: **per commit**) → huge metadata per branch → performance issues → admins often **restrict branch creation**.
  * Needing **permission to create a task branch** leads to unstable code on main or huge "done" commits, no CI/CD on task branches, less granular WIP tracking → **lower productivity** (some devs secretly use Git on the side).
  * Branches are **shared** → no private task branches with periodic rebasing.
  * **Overly complicated merge algorithms** (e.g. renamed files or changed attributes).
  * Sharing between Perforce servers = passing **tar files with no common history**. Git clones and shares history anywhere.

#### d) Mind share & community
* Created by **Linus Torvalds** for Linux kernel development; now standard for **Linux, Android, OpenStack** and most major open-source projects.
* Hiring managers can assume new engineers **already know Git**.
* Vibrant community → rapid evolution (e.g. **Git LFS**), you can contribute fixes yourself, and **no vendor lock-in**.

#### e) GUIs & developer tools
* Early Git lacked GUIs — non-technical collaborators (e.g. **game artists**) preferred Perforce's **Windows Explorer plugin**.
* Today: **Sourcetree** (point-and-click), shell integrations, IDE plugins, and Bitbucket's code review, PRs, forking, and online browsing.

---

### 4. Why Game Developers Are "Special"
It comes down to the **type of data** and **project complexity**.

#### a) Large binary files (textures, audio, huge datasets)
| Problem | Solution / Counterpoint |
|---|---|
| **Can't be merged** → a central **locking** mechanism helps (Perforce has one) | Perforce locks only work on a **single branch** (implies a very restricted workflow). **Git LFS** file locking coordinates locks **across branches**. Locking is also a **coordination problem** — solved by PRs and real-time team chat. |
| **Slow Git down** as repo size grows | **Git LFS** stores large files elsewhere while Git tracks pointers |

* For **Big Data** (terabyte-scale test data), no SCM fits — you need CI/CD pipelines with artifacts on **HDFS or S3**.

#### b) Large, multi-module projects (engine, UI, art, video…)
* Perforce's monolithic repo lets users pick parts into their workspace.
* This advantage is now **largely moot**:
  * Git supports **submodules** and **subtrees**.
  * Projects like **Android** show how to compose complex projects with higher-level tools.
  * CI/CD tools (**Bamboo**, **Bitbucket Pipelines**) model cross-project dependencies and manage artifacts.
* Follows the **Unix philosophy** — tools that do **one job well** — and the modern trend toward **microservices** over monoliths.

---

### 5. What About Git Fusion?
* Perforce built **Git Fusion** to expose part of a Perforce repo as a Git repo.
* Layering Git onto a centralized system is hard: **mixing usage models can corrupt data**; not mixing them makes the commercial backend pointless.
* The real trend is the **reverse**: bringing the few useful centralized features **into Git**.

---

### Summary Takeaway
Git beat Perforce on **speed (distributed), cost (free), workflows (cheap per-commit branching), community, and tooling**. The remaining reasons game developers stuck with Perforce — **large binary files** and **multi-module projects** — are now addressed by **Git LFS** (storage + cross-branch locking), **submodules/subtrees**, and modern **CI/CD** tools. The next step is figuring out **how to migrate** (covered in *Migrating from Perforce to Git*).
