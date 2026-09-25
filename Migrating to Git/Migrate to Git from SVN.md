Here is a clear breakdown of the key points about **Migrating from SVN to Git** (the process overview) from the text:

> Source: https://www.atlassian.com/git/tutorials/migrating-overview

---

### 1. The 5-Step Migration Process

| # | Step | What happens |
|---|---|---|
| 1 | **Prepare** | Set up your environment for the migration |
| 2 | **Convert** | Turn the SVN repository into a **local Git repository** |
| 3 | **Synchronize** | Keep the local Git repo updated as the SVN repo keeps changing |
| 4 | **Share** | Push the Git repo to a host (e.g. **Bitbucket**) so developers can access it |
| 5 | **Migrate** | Move active development from SVN to Git |

---

### 2. Phase 1 — Create the Git Repository (Steps 1–3)
* **Prepare, convert, and synchronize** take the SVN commit history and turn it into a Git repository.
* Designate one team member as the **migration lead** (probably you, if you're reading the guide).
* All three steps are performed on the **migration lead's local computer**.
* After synchronizing, the lead can easily keep the local Git repo **up to date** with its SVN counterpart.

```
SVN repo ──prepare──▶ ──convert (clone)──▶ Local Git repo ◀──sync── SVN repo (keeps changing)
                                          (migration lead's machine)
```

---

### 3. Phase 2 — Share the Repository (Step 4)
* The migration lead **pushes** the local Git repo to **Bitbucket** (a Git hosting service).
* Other developers can then:
  * **Clone** the converted repo locally,
  * **Explore its history** with Git commands,
  * Start **integrating it into build processes**.

#### ⚠️ One-way synchronization (SVN → Git)
* Until the team is ready for a pure Git workflow, sync **only from SVN to Git**.
* Everyone treats their Git repo as **read-only** and keeps **committing to SVN**.
* The **only** changes to the Git repo come from the migration lead syncing and pushing updates to Bitbucket.

| Who | Commits to | Git repo role |
|---|---|---|
| Developers | **SVN** (as usual) | Read-only: learn, explore, integrate builds |
| Migration lead | — | Syncs SVN → Git and pushes to Bitbucket |

* This gives a **clear-cut transition period** — the team gets comfortable with Git **without interrupting** the existing SVN workflow.

---

### 4. Phase 3 — Migrate Development (Step 5)
* When developers are ready: **freeze the SVN repository** and **start committing with Git**.
* The switch should feel **natural**, because:
  * The entire Git workflow is **already in place**, and
  * Developers have had **plenty of time** to get comfortable with it.

---

### Summary Takeaway
Migrating from SVN to Git is broken into **5 steps: prepare → convert → synchronize → share → migrate**. A single **migration lead** builds and syncs the Git repo locally, then shares it on **Bitbucket** while the team keeps committing to SVN (**one-way sync, Git is read-only**). Once everyone is comfortable, **freeze SVN** and switch all development to Git.
