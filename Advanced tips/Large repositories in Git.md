Here is a clear breakdown of the key points about **Handling big repositories with Git** from the text:

> Source: https://www.atlassian.com/git/tutorials/big-repositories (by Nicola Paolucci, Atlassian Developer Advocate)

---

### 1. Two Kinds of Big Repositories
Repositories grow massive for two main reasons (or both):
1. **Very long history** — the project has accumulated baggage over many years.
2. **Huge binary assets** tracked alongside the code.

* The second problem is often worsened by **old, deprecated binaries still stored in history** (fixable — see `filter-branch` below).
* The techniques for each are **different but sometimes complementary**.

---

## PART A — Repositories with a Very Long History
Long histories can't always be avoided (e.g. **legal or regulatory** reasons), but they're painful to clone.

### 2. Simple Solution — Shallow Clone
* Download only the **latest *n* commits**:
```bash
git clone --depth [depth] [remote-url]
```
* **Real example — Jira** (11-year-old code base, 47,000+ commits):

| | Size / Time |
|---|---|
| Full clone | **677 MB** repo (+ 320 MB working dir) |
| Full clone time | **4 min 24 s** |
| Shallow clone time | **29.5 s** |

* Savings grow with the amount of binary assets swallowed over time.
* 💡 **CI/build systems** benefit from shallow clones too.
* Since **Git 1.9+**, you can properly **pull and push** from shallow clones.

### 3. Surgical Solution — `git filter-branch`
* Walks the **entire history**, filtering out/modifying/skipping files by pattern — ideal for removing **binary cruft committed by mistake** or obsolete assets. Helper scripts can identify big objects.
```bash
git filter-branch --tree-filter 'rm -rf [/path/to/spurious/asset/folder]'
```
* ⚠️ **Rewrites the entire history** → **all commit IDs change** → every developer must **re-clone**.
* Plan it: **alert the team → short freeze → run cleanup → tell everyone to re-clone**.

> 📝 *Extra note (not in the original article):* current Git documentation discourages `git filter-branch` and recommends the faster, safer **`git filter-repo`** tool for history rewriting.

### 4. Alternative — Clone Only One Branch (Git 1.7.10+)
```bash
git clone [remote url] --branch [branch_name] --single-branch [folder]
```
* Useful with **long-running, divergent branches**, or many branches when you only need a few. Little benefit if branches barely differ.

---

## PART B — Repositories with Huge Binary Assets
Game teams (3D models), web teams (raw images), CAD teams (binary deliverables)… many teams face this.
* Git isn't especially **bad** at binaries — but not especially **good** either: by default it compresses and stores **every full version** of each binary.

### 5. Basic Tuning Tips
| Tip | Why |
|---|---|
| Run **`git gc`** | Packs "duplicated" loose objects into a single pack file (only helps if files compress) |
| Use **`delta off`** in `.gitattributes` for binaries that change significantly | Delta compression is useless for them; skip the wasted repack work |
| Set **`core.compression 0`** / **`core.loosecompression 0`** | Such files don't zlib-compress well either — but it's **global**, so best when binaries live in a **separate repo** |
| Tune **`core.bigFileThreshold`** | Files > **512 MB** are never delta-compressed anyway; adjust as needed |

* First **reflect on the nature of your binary assets** — it determines the winning approach.

### 6. Big Folder Trees — Sparse Checkout (Git 1.7.0+)
* Populate your **working directory** with only the folders you need.
* ⚠️ Doesn't shrink the **local repository** size — just keeps the working tree manageable.
```bash
git clone <url>                                   # clone the full repo once
git config core.sparsecheckout true               # enable the feature
echo src/ > .git/info/sparse-checkout             # list folders you want (skip asset folders)
git read-tree -m -u HEAD                          # re-read the tree as specified
```
* Afterwards, use Git normally — only the listed folders appear.

> 📝 *Extra note:* modern Git (2.25+) offers the simpler **`git sparse-checkout set <dirs>`** command for the same purpose.

### 7. Controlling When Large Files Update — Submodules
* Put large assets in a **separate repository** included as a **submodule**, so you decide **when** to update them.

### 8. ⭐ Best Option Today — Git LFS
* **Git LFS**, co-developed by **Atlassian and GitHub in 2015** (an open-source contribution to Git).
* Stores **pointers** to large files in the repo; actual files live on a **remote server** → dramatically **faster clones**.
* Supported by **Bitbucket** and **GitHub**. Especially helpful for teams with **designers, videographers, musicians, CAD users**.

---

### 9. Cheat Sheet

| Problem | Technique | Command |
|---|---|---|
| Long history, need fast clone | Shallow clone | `git clone --depth N <url>` |
| Many branches, need one | Single-branch clone | `git clone <url> --branch B --single-branch` |
| Junk binaries in history | Rewrite history | `git filter-branch --tree-filter 'rm -rf <path>'` |
| Huge folder tree | Sparse checkout | `core.sparsecheckout` + `.git/info/sparse-checkout` |
| Binaries bloating repo | Tune compression | `delta off`, `core.compression 0`, `core.bigFileThreshold` |
| Large files updated regularly | **Git LFS** | `git lfs track "<pattern>"` |

---

### Summary Takeaway
Don't give up Git because of a **long history** or **huge files** — both have workable solutions. For long histories use **shallow** or **single-branch clones** and surgically clean history with **`filter-branch`** (coordinate re-clones!). For big binaries, tune **compression/delta** settings, use **sparse checkout** or **submodules**, and — best of all — adopt **Git LFS**.
