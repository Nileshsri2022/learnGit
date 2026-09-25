Here is a clear breakdown of the key points about **working with Git and Perforce side by side (integration workflow)** from the text:

> Source: https://www.atlassian.com/git/tutorials/git-p4
> *(Site title: "Working with Git and Perforce: integration workflow". The colon is replaced with " - " in the file name.)*

---

### 1. The Scenario
* **Your team uses Git**, but another part of the organization **stays on Perforce (P4)** for parts of the **same code base** and has no plans to migrate.
* You need an ongoing **two-way code-sharing process** so improvements made on either side reach the other, without slowing your team down too much.
* This is the real process used by **TomTom's NavApps unit**, where the sync runs **once per release** (about every **6 weeks**).
* **Assumptions:** you know **basic Git** and the **feature branch workflow**. ⚠️ There are subtleties, so be careful when running these integrations.

---

### 2. Installing `git p4`
Check whether it's already available:
```bash
git p4
```
If not:
1. Download **`git-p4.py`** (from the Git source tree) into a folder on your `PATH`, e.g. `~/bin` (**Python** is required).
2. Make it executable:
```bash
chmod +x git-p4.py
```
3. Add an alias in `~/.gitconfig`:
```ini
[alias]
    p4 = !~/bin/git-p4.py
```
4. Run `git p4` again. No errors means it's installed.

> 📝 *Extra note (not in the original article):* The article's alias says `bit-p4.py`, which is a typo for `git-p4.py`. Most modern Git installations already include `git p4`, so you often don't need to install anything.

---

### 3. Branches & Refs Used

| Name | Role |
|---|---|
| `remotes/p4/main` | Git's view of the **Perforce** history (created by `git p4`) |
| **`p4-integ`** | The branch that **talks directly to Perforce** |
| `main` / `develop` | Normal Git development branches |
| **`tag/last-p4-integ`** | Tag marking the **last integration point** with P4 |
| `cleanup`, `p4mergebranch` | **Temporary** helper branches |

```
      Perforce depot
           ▲   │  git p4 submit / git p4 sync
           │   ▼
  remotes/p4/main ◀──▶ p4-integ ◀── cherry-pick ── main/develop (Git side)
                                  ── squash merge ──▶
```

---

### 4. Initial Clone
* P4 histories can be **huge**, so pick a **cut-off changelist** to start from (saves space and time):
```bash
git p4 clone //depot/path/project@<earlier-cutoff-point>,<latest-changelist>
```
* Import any new P4 changes as Git commits:
```bash
git p4 sync
```
* Create the integration branch from the P4 history:
```bash
git checkout -b p4-integ origin/p4/main
```
> 📝 *Extra note (not in the original article):* `git p4` stores the Perforce history under `refs/remotes/p4/main`, so on a fresh clone the start point is usually just `p4/main`. The article's `origin/p4/main` only applies if that ref was pushed to and fetched from `origin`.

---

### 5. Follow-Up Fast Sync ("Bait and Switch")
The simple way works but is **slow**:
```bash
git checkout p4-integ
git p4 sync
```
**Faster:** recreate refs identical to the last integration. This also ensures a **new integrator starts at the right commit/changelist**.
```bash
# (optional) remove stale p4 refs
git symbolic-ref -d refs/remotes/p4/HEAD
git update-ref -d refs/remotes/p4/main

# create "fake" p4 refs pointing at the last p4-integ on origin
git update-ref refs/remotes/p4/main remotes/origin/p4-integ
git symbolic-ref refs/remotes/p4/HEAD refs/remotes/p4/main

# the only catch: you must name the branch explicitly from now on
git p4 sync --branch=refs/remotes/p4/main
```

#### How `git p4` maps commits ↔ changelists
* It annotates commits with metadata, e.g.:
```
[git-p4: depot-paths = "//depot-mobile/project/": change = 1794239]
```
* Newer versions of `git p4` store this in a **git note** instead of the commit message. TomTom found that slightly less convenient for looking up changelist numbers.

---

### 6. Moving Changes: Git → Perforce

| # | Step | Commands |
|---|---|---|
| 1 | Bring `p4-integ` up to date with P4 | `git checkout p4-integ` · `git p4 rebase` |
| 2 | Update `main` from `develop` | `git checkout main` · `git merge develop` |
| 3 | Get the latest tags | `git fetch --tags` |
| 4 | Make a temporary **`cleanup`** branch off `main` and **linearize** history since the last integration (dropping commits already in P4) | `git checkout -b cleanup` · `git rebase -s recursive -X theirs tag/last-p4-integ` *(or interactively: `git rebase -i tag/last-p4-integ`)* |
| 5 | **Cherry-pick** the new commits onto `p4-integ` (`main`/`develop` aren't assumed to be ancestors of `p4-integ`) | `git checkout p4-integ` · `git cherry-pick tag/last-p4-integ..cleanup` |
| 6 | **Submit to P4** and resync | `git p4 submit` · `git p4 sync --branch=refs/remotes/p4/main` · `git reset --hard refs/remotes/p4/main` |
| 7 | Delete the temp branch | `git branch -D cleanup` |
| 8 | Remove the old integration tag (local + remote) | `git tag -d tag/last-p4-integ` · `git push origin :refs/tags/tag/last-p4-integ` |
| 9 | Re-tag the new integration point and push | `git checkout develop` · `git tag -a tag/last-p4-integ -m "tag pointer to last develop commit integrated with p4"` · `git push origin main` · `git push origin tag/last-p4-integ` · `git push origin p4-integ` |
| 10 | ✅ **Run tests on the P4 code base** | — |

---

### 7. Moving Changes: Perforce → Git
Do this **after** the Git → P4 push, once the **P4 tests pass**.

```bash
# 1. Get the latest P4 changes onto p4-integ
git checkout p4-integ
git p4 sync --branch=refs/remotes/p4/main
git p4 rebase

# 2. Robust "theirs" merge, squashed into ONE commit on main
git checkout -b p4mergebranch        # branching off p4-integ
git merge -s ours main               # record main as merged, keep p4-integ's content
git checkout main
git merge p4mergebranch --squash
git commit -m "Type your integration message"
git branch -D p4mergebranch
```
* 💡 **Why this trick works:** `-s ours` creates a merge on `p4mergebranch` whose **content = P4's version** but whose history includes `main`. Squash-merging that into `main` then makes `main` **match P4 exactly** in a **single commit**. It's a "theirs" strategy that Git doesn't offer directly.

```bash
# 3. Merge into develop
git checkout develop
git merge main

# 4. Move the integration tag to main (NOT to the develop merge commit)
git tag -d tag/last-p4-integ
git push origin :refs/tags/tag/last-p4-integ
git checkout main
git tag -a tag/last-p4-integ -m "tag pointer to last develop commit integrated with p4"

# 5. Push everything
git push origin main develop p4-integ
git push origin tag/last-p4-integ
```
> 📝 *Extra note (not in the original article):* On the live page, the "merge changes to `develop`" code block shows unrelated HTML (pasted by mistake), and the final push commands are missing. Steps 3 and 5 above reconstruct them from the surrounding text. If `develop` has moved on since you picked changes from it, merge those changes first.

---

### 8. Conclusions
* This process **evolved over time at TomTom** and has run **without major problems** for a long time.
* ⚠️ But it has **a lot of maintenance overhead**. **If you can, migrate fully to Git** (see *Migrating from Perforce to Git*).

---

### Summary Takeaway
When Git and Perforce teams must share one code base, use **`git p4`** with a dedicated **`p4-integ`** branch and a **`tag/last-p4-integ`** marker. **Git → P4:** rebase `p4-integ`, linearize new work on a `cleanup` branch, **cherry-pick** it onto `p4-integ`, then **`git p4 submit`**. **P4 → Git:** **`git p4 sync` + `rebase`**, then squash in P4's state with the **`-s ours` + `--squash`** trick, merge into `develop`, and move the tag. Use the **"bait and switch"** ref trick for fast syncs. It works, but a **full migration to Git** is far simpler.
