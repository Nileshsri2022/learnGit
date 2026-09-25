Here is a clear breakdown of the key points about the **Share** phase of migrating from SVN to Git, from the text:

> Source: https://www.atlassian.com/git/tutorials/migrating-share
> *(Step 4 of 5: Prepare → Convert → Synchronize → **Share** → Migrate)*

---

### 1. SVN vs. Git Collaboration

| SVN (centralized) | Git (distributed) |
|---|---|
| Commit from a **working copy** straight to the **central repo** | Each developer has a **full local repository** and commits **locally** |
| Others **update** from the central repo | Share by **pushing** to a public server repo. Others **pull** from it |

* A **complete repository per developer** is the heart of distributed version control. It enables many **workflows** (see the Comparing workflows notes).

---

### 2. Goal of This Phase
* So far the Git repo exists **only on the migration lead's machine**. Now you **push it to a hosted repo** (Bitbucket) so the team can **experiment with Git** without affecting active SVN development.
* ⚠️ Until the switch: **treat the shared Git repositories as READ-ONLY.** All development still goes to **SVN**.

---

### 3. Create a Bitbucket Account & Repository
* **Account:** free hosting for up to **5 users**.
* **Repository:** click **Create repository** and fill in the form:

| Field | Recommended value |
|---|---|
| Name / Description | Your project |
| Access level | ✅ **Private** (only designated developers can clone) |
| Forking | **Allow only private forks** |
| Repository type | **Git** |
| Project management | Any tools you want |
| Language | Your project's primary language |

* After creating it, a **Next steps** page shows import commands, which are covered below.

---

### 4. Add an `origin` Remote
* A **remote** is a shortcut name for a URL. The official codebase is conventionally called **`origin`**.
```bash
git remote add origin https://<username>@bitbucket.org/<workspace-id>/<repo>.git
```
* Replace `<username>`, `<workspace-id>`, and `<repo>`, or copy the full URL from Bitbucket.

---

### 5. Push the Local Repository
```bash
git push -u origin --all   # all branches; -u sets up upstream tracking
git push --tags            # tags aren't included in --all
```
| Option | Effect |
|---|---|
| `-u` | Track upstream branches, so Git can tell you if you're **ahead/behind** the remote |
| `--all` | Push **all local branches** |
| `--tags` | Push **all local tags** |

* The Bitbucket repo is now essentially a **clone of your local repo**, and you can browse all history in the web UI.

---

### 6. Share with Your Team
* Give teammates the **repository URL** (copy it from the repo home page).
* For **private** repos, grant access under **Administration → Access management** (users and groups).
* Or use Bitbucket's **invitation** feature to invite developers to **fork** it (access is granted automatically).
* Teammates then clone it:
```bash
git clone https://<username>@bitbucket.org/<workspace-id>/<repo>.git
```

---

### 7. Keep Committing with SVN, NOT Git ⚠️
* The **only** changes to the Git repos should come **from SVN through the sync process** (previous phase).
* All Git repos, **local and remote**, are effectively **read-only**:
  * ✅ Experiment freely and start integrating Git into the **build process**.
  * ❌ Don't commit **permanent** changes with Git yet.

```
Developers ──commit──▶ SVN ──sync──▶ migration lead's Git repo ──push──▶ Bitbucket
                                                                          │
                                           teammates clone/experiment ◀───┘ (read-only)
```

---

### Summary Takeaway
**Share** = create a **(private) Bitbucket repo**, add it as **`origin`**, and push everything with **`git push -u origin --all`** and **`git push --tags`**. Then give your team access so they can **clone and experiment**. Until everyone is ready to switch, **SVN stays the source of truth** and all Git repos are **read-only**, updated only by the SVN → Git sync. Next step: **Migrate**.
