Here is a clear breakdown of the key points about **Git LFS (Large File Storage)** from the text:

> Source: https://www.atlassian.com/git/tutorials/git-lfs

---

### 1. What Is Git LFS?
* **The problem:** Git is distributed — cloning downloads **every version of every file**. Large, frequently modified files make clones painfully slow.
* **The solution:** Git LFS, an extension built by **Atlassian, GitHub**, and open-source contributors, downloads large files **lazily** — during **checkout**, not during clone/fetch.

#### How it works — pointer files
1. **`git add`** → LFS replaces the file's content with a tiny **pointer** and stores the real content in a **local LFS cache**.
2. **`git push`** → LFS files referenced by the pushed commits go from your local cache to the **remote LFS store**.
3. **`git checkout`** → pointers are replaced with real files from the local cache or **downloaded** from the remote store.

A pointer file looks like:
```
version https://git-lfs.github.com/spec/v1
oid sha256:3b6124b8b01d601fa20b47f5be14e1be3ea7759838c1aac8f36df4859164e4cc
size 21647
```

* **Seamless:** you only ever see real content; keep using `checkout`, `add`, `commit` as usual.
* **Faster** `clone` and `pull` — you only download versions of large files for commits you **actually check out**.
* **Requirements:** an **LFS-aware host** (Bitbucket Cloud/Data Center, GitHub…) and the **LFS CLI** or an LFS-aware GUI like **Sourcetree** (its inventor, Steve Streeting, is a major LFS contributor).

---

### 2. Installing & Setting Up

```bash
# Install via package manager (Homebrew, MacPorts, dnf, packagecloud),
# the Git LFS website, or bundled with Sourcetree. Then, once per system:
git lfs install
```
* After that, LFS bootstraps **automatically** when you clone an LFS repo.

#### New LFS repository
```bash
mkdir Atlasteroids && cd Atlasteroids
git init
git lfs install      # installs a pre-push hook that uploads LFS files on git push
```
* Enabled automatically on Bitbucket Cloud; on Bitbucket Data Center, enable it in repo settings.

---

### 3. Cloning & Pulling

| Task | Command | Notes |
|---|---|---|
| Clone | `git clone <url>` | LFS files download **one at a time** during checkout |
| **Faster** clone | `git lfs clone <url>` | Waits for checkout, then downloads LFS files in a **parallel batch** (fewer HTTP requests — great on Windows) |
| Pull | `git pull` | Needed LFS files download automatically |
| Fix a failed checkout | `git lfs pull` | Downloads missing LFS content for the current commit |

#### Faster pulls (batch download)
```bash
git -c filter.lfs.smudge= -c filter.lfs.required=false pull && git lfs pull

# Save it as an alias
git config --global alias.plfs "!git -c filter.lfs.smudge= -c filter.lfs.required=false pull && git lfs pull"
git plfs
```

---

### 4. Tracking Files

```bash
git lfs track "*.ogg"      # ⚠️ keep the quotes!
```
* Without quotes, the **shell expands** the wildcard and tracks each existing file individually — probably not what you want.

#### Pattern examples (same syntax as `.gitignore`)
| Pattern | Tracks |
|---|---|
| `"*.ogg"` | All `.ogg` files anywhere |
| `"music.ogg"` | Files named `music.ogg` anywhere |
| `"Assets/**"` | Everything in `Assets/` **and subdirectories** |
| `"Assets/*"` | Files in `Assets/` but **not** subdirectories |
| `"Assets/Audio/*.ogg"` | `.ogg` files in `Assets/Audio` |
| `"**/Music/*.ogg"` | `.ogg` files in any `Music` directory |
| `"*xxhdpi*.png"` | PNGs with "xxhdpi" in the name |

* Patterns are **relative to where you run the command** → run from the **repo root**.
* **No negative patterns** (unlike `.gitignore`).

#### `.gitattributes`
* `git lfs track` creates/updates **`.gitattributes`**, binding patterns to the LFS filter — **you must commit it yourself**:
```bash
git lfs track "*.ogg"
git add .gitattributes
# +*.ogg filter=lfs diff=lfs merge=lfs -text
git commit -m "Track ogg files with Git LFS"
```

| Command | Effect |
|---|---|
| `git lfs track` | List all tracked patterns and which `.gitattributes` defines them |
| `git lfs untrack "*.ogg"` | Stop tracking (then commit `.gitattributes`) |

---

### 5. Committing & Pushing
* Commit and push **as normal**; `git push` shows extra LFS transfer output:
```
Git LFS: (3 of 3 files) 4.68 MB / 4.68 MB
```
* If the LFS transfer fails, the push is **aborted** — just retry safely.
* LFS storage is **content-addressable** (key = **SHA-256** of the content), so you can **never overwrite** a file with the wrong version.

---

### 6. Moving an LFS Repo Between Hosts
```bash
git clone --bare git@github.com:kannonboy/atlasteroids.git
cd atlasteroids
git remote add bitbucket git@bitbucket.org:tpettersen/atlasteroids.git
git remote add github git@github.com:kannonboy/atlasteroids.git
git lfs fetch --all github       # fetch ALL LFS content from the old host
git push --mirror bitbucket      # push all Git data
git lfs push --all bitbucket     # push all LFS data
```

---

### 7. Fetching Extra LFS History
| Command / Setting | Effect |
|---|---|
| `git lfs fetch --recent` | Download LFS content for **recent branches/tags** (e.g. before a flight ✈️) |
| `git config lfs.fetchrecentrefsdays 10` | "Recent" = refs with commits in last N days (default **7**) |
| `git config lfs.fetchrecentcommitsdays 3` | Also fetch **earlier commits** on recent refs (default: tip only). ⚠️ Can download **huge** amounts |
| `git lfs fetch --all` | Fetch **all** LFS content ever referenced |

---

### 8. Deleting Local LFS Files — `git lfs prune`
* Deletes **old** files from the local LFS cache. A file is "old" if it's **not** referenced by:
  * the currently checked-out commit,
  * an **unpushed** commit (checked against origin or `lfs.pruneremotetocheck`),
  * a **recent** commit — default **10 days** = `lfs.fetchrecentrefsdays` (7) + `lfs.pruneoffsetdays` (3).
* ⚠️ LFS content is **not** pruned automatically (unlike Git's GC) → run it **regularly**.

| Command | Effect |
|---|---|
| `git lfs prune` | Prune old local LFS files |
| `git lfs prune --dry-run` | Preview how many files would be pruned |
| `git lfs prune --dry-run --verbose` | Also list the **OIDs** (SHA-256 hashes) to be pruned |
| `git lfs prune --verify-remote` | Only prune files confirmed to exist on the server (slower, safer) |
| `git config lfs.pruneoffsetdays 21` | Keep content for 4 weeks (7 + 21) |
| `git config --global lfs.pruneverifyremotealways true` | Always verify remote before pruning |

---

### 9. Deleting LFS Files from the Server
* The LFS CLI **can't** prune server files — it depends on the host. On **Bitbucket Cloud**: *Repository Settings → Git LFS*.
* Files are listed by **SHA-256 OID** only (paths aren't shown — too slow to look up). To identify one: check the preview/file type, download it, or search commits for its OID.

---

### 10. Finding What References an LFS Object
```bash
# Which commits (any branch) add/remove a pointer with this OID?
git log --all -p -S <OID>

# Which path references this OID in HEAD / on a branch?
git grep <OID> HEAD
git grep <OID> power-ups
```

---

### 11. Including / Excluding LFS Files
* Useful for e.g. **CI builds** that only need source code.

| Command | Effect |
|---|---|
| `git lfs fetch -X "Assets/**"` | **Exclude** a pattern/dir (`--exclude`) |
| `git lfs fetch -I "*.ogg,*.wav"` | **Include** only these (`--include`) — e.g. for an audio engineer |
| `git lfs fetch -I "Assets/**" -X "*.gif"` | Everything in Assets **except** GIFs (must match include **and** not exclude) |
| `git config lfs.fetchinclude "Assets/**"` / `lfs.fetchexclude "*.gif"` | Make it permanent (add `--global` for all repos) |

---

### 12. Locking LFS Files
* Binary merge conflicts **can't be resolved easily** → **lock** files to prevent overwrites.
```bash
git lfs track "*.psd" --lockable
# .gitattributes: *.psd filter=lfs diff=lfs merge=lfs -text lockable

git lfs lock images/foo.psd      # register the lock on the server
git lfs unlock images/foo.psd    # release it
git lfs unlock images/foo.psd --force   # override — only if you really know what you're doing!
```

---

### Summary Takeaway
Git LFS keeps repos fast by storing large files as **tiny pointers** in Git while the real content lives in an **LFS store**, downloaded **only when checked out**. Install once with `git lfs install`, choose files with **`git lfs track "<pattern>"`** (and commit `.gitattributes`), then work normally. Use `git lfs clone`/`git lfs pull` for **batch speed**, `fetch --recent/--all` for offline work, **`prune`** to reclaim disk space, `-I`/`-X` to fetch subsets, and **`lock`/`unlock`** to avoid binary merge conflicts.
