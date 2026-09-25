Here is a clear breakdown of the key points about **Migrating from Perforce to Git** (step-by-step guide) from the text:

> Source: https://www.atlassian.com/git/tutorials/perforce-git-migration

---

### 1. Overview — The 8 Steps
If you have years of history in Perforce, you're probably weighing the **cost of switching**. The migration is broken into 8 steps:

| # | Step |
|---|---|
| 1 | Moving Perforce data |
| 2 | Mapping users and permissions |
| 3 | Managing large binary files |
| 4 | Managing complex dependencies |
| 5 | Structuring your team during the migration |
| 6 | Mirroring data |
| 7 | Choosing your ALM tools |
| 8 | Defining success |

---

### 2. Step 1 — Moving Perforce Data

#### A fundamental difference
| Perforce | Git |
|---|---|
| One server holds **tens or hundreds of projects**, each with its own branching model | One repo normally holds **one project** + its branches and tags |
| Developers define a **"view"** to choose files for their working copy | You **clone** the repo (and maybe check out submodules/subtrees) |
| Branches appear as **extra directories** in the tree | Branches are pointers to commits |

So the problem has two parts: **extract** data from Perforce, and **translate** it into an equivalent set of Git repos.

#### Option 1 — Git Fusion (preserve history)
1. Install **Git Fusion**.
2. Set up the correct **views**, including the branching structure.
3. **Clone** from Git Fusion with any Git client.
4. **Push** the repo to Bitbucket.

**Hands-on example** — a Perforce project at `//depot/acme/...` with branches `main`, `r1.0`, `r1.1`. Create a repo config file and submit it to `//.git-fusion/repos/acme/p4gf_config`:
```ini
[@repo]
description = Acme project
charset = utf8

[main]
git-branch-name = main
view = //depot/acme/main/... ...

[r1.0]
git-branch-name = r1.0
view = //depot/acme/r1.0/... ...

[r1.1]
git-branch-name = r1.1
view = //depot/acme/r1.1/... ...
```
Create an empty `acme` project in Bitbucket, then:
```bash
git clone https://<git-fusion-server>/acme
cd acme
git remote add bitbucket <bitbucket-repo-url>
git push -u --all bitbucket
git push --tags bitbucket
```
* Not always **100% faithful** — some Perforce operations (e.g. **partial merges**) have no Git equivalent — but you get most history with little effort.
* Keeping 10 years of history ≠ keeping the old workflow. Consider moving to **feature branch workflows like Git Flow**.

#### Option 2 — Start over (tip only)
* Extract only the **head (tip) of each branch** and commit it into a **new, empty Git repo**.

```bash
p4 sync                                  # in your Perforce workspace: fetch latest revision
git init .
git remote add origin <bitbucket-repo-url>
git push -u --all origin
git push --tags origin
```
* The latest snapshot becomes the **first commit** of the new project.

#### Comparing the options
| | Git Fusion | Start over |
|---|---|---|
| Effort / runtime | ❌ Most setup work & runtime | ✅ Fast and simple |
| History | ✅ Preserves the most history → can **shut down** Perforce | ❌ Keep old Perforce server in **read-only** mode for archaeology (no license cost, but server stays alive) |
| Branching model | Keeps the **legacy** model in history | ✅ Chance to **redesign** branching & workflow — no baggage |

---

### 3. Step 2 — Mapping Users & Permissions
* **Users:** LDAP saves time; otherwise export with `p4 users -o` and add them to Bitbucket project by project.
* **Permissions:** Perforce permissions are **granular and complex** (can exclude individual files) — one reason Perforce servers **bog down**.
* ✅ Faster approach: have project leads define a **simpler** scheme using Bitbucket's **project / repo / branch-level** permissions.
  * e.g. Perforce restricted **branch creation**; in Bitbucket you may only need to restrict **push access to `main`**.

---

### 4. Step 3 — Managing Large Binary Files
* Don't **blindly push large blobs** into a Git repo.
* Options: **Git LFS**, or a regular **artifact management system**.

---

### 5. Step 4 — Managing Complex Dependencies
* A Perforce working copy may map **read-only copies of several modules**.
* In Git, use **submodules**, **subtrees**, or **CI/CD / artifact management** systems. Some import tools can model **submodule** relationships.

---

### 6. Step 5 — Structuring Your Team
* ❌ Don't schedule one maintenance window for "100 projects from 10 teams" — **don't boil the ocean in a day**. Switching SCM is about **people, process, and schedule**, not just data.

| Strategy | Description |
|---|---|
| **Team-by-team, project-by-project** | Start each at the beginning of a **sprint / program increment** |
| **Incremental** | Import everything over a weekend; teams switch gradually; re-run import tools to pick up **deltas**. Good when teams depend on each other and early adopters need a recent snapshot for CI/CD |
| **Both systems at once** | Two-way exchange via Git Fusion — feasible but "**not for the faint of heart**"; avoid complex operations that confuse the translator |

* **Communicate** the motivation, the *why*, and the *how*.
* Pick an experienced **early adopter team** as a model; find **Git champions** to help others.
* Make **small, understandable, iterative** changes.

---

### 7. Step 6 — Mirroring Data
* Perforce has mirroring for remote sites and read-only clustering.
* Latency matters less with Git, but for **worldwide** teams use **Bitbucket Data Center** for **clustering and mirroring** → much faster clones.

---

### 8. Step 7 — Choosing ALM Tools
* 🎉 Good news: almost **every developer/ALM tool works with Git**.
* Bitbucket integrates with **Jira** and **Bamboo**; explore Bamboo **Plan Branches**, which leverage a feature branch workflow.

---

### 9. Step 8 — Defining Success
* ❌ Don't obsess over **bit-for-bit history fidelity** — it's practically impossible from a centralized SCM.
* ✅ Use **CI/CD for verification**: after switching your pipeline to Git —
  * Do all **tests still pass**?
  * Can you still **deploy**?
  * Do important **older builds** still pass?
  → If yes, **declare victory!**

---

### Summary Takeaway
Migrate Perforce data either with **Git Fusion** (keeps history, more work) or by **starting over** from branch tips (fast, clean, keep Perforce read-only). Then **simplify permissions**, handle **binaries** with Git LFS/artifact stores, model **dependencies** with submodules/subtrees/CI, migrate **people gradually** with champions and early adopters, use **Bitbucket Data Center** for global mirroring, and measure success by whether your **CI/CD pipeline still builds, tests, and deploys**.
