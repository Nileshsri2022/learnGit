Here is a clear breakdown of the key points about **Git subtree — the alternative to Git submodule** from the text:

> Source: https://www.atlassian.com/git/tutorials/git-subtree (by Nicola Paolucci, Atlassian Developer Advocate)

---

### 1. Why Look for an Alternative to Submodules?
* The Internet is full of articles on why you **shouldn't use Git submodules** — useful for a few cases, but they have several drawbacks.
* Two tools can track dependency history while still using Git:
  1. **`git subtree`** ← focus of this article
  2. **Google repo**
* `git subtree` is an **improvement over submodules — albeit not perfect**.

---

### 2. What Is Git Subtree?
* Lets you **nest one repository inside another as a sub-directory**.
* One of several ways to manage **project dependencies** in Git.

```
my-project/                 my-project/
├── src/          ─────▶    ├── src/
└── README                  ├── README
                            └── .vim/bundle/tpope-vim-surround/   ← another repo's content, as a plain folder
```

---

### 3. Pros & Cons

| ✅ Why consider `git subtree` | ❌ Drawbacks (largely acceptable) |
|---|---|
| Simple workflows are **easy to manage** | You must learn a **new merge strategy** |
| Supports **old Git versions** (even < v1.5.2) | **Contributing back upstream** is slightly more complicated |
| Sub-project code is available **right after cloning** the super-project | **You** are responsible for not mixing super- and sub-project code in the same commits |
| Users of the repo **don't need to learn anything new** — they can ignore it | |
| **No new metadata files** (unlike submodule's `.gitmodules`) | |
| Module contents can be **modified in place** without a separate copy of the dependency repo | |

---

### 4. Availability
* Ships with stock Git since **May 2012 (v1.7.11+)**. Homebrew's Git on macOS has it wired up; some platforms may need extra install steps.

---

### 5. How to Use It (Example: tracking a vim plugin)

#### a) Quick & dirty — no remote tracking
```bash
# Add the subtree at a prefix folder
git subtree add --prefix .vim/bundle/tpope-vim-surround \
  https://bitbucket.org/vim-plugins-mirror/vim-surround.git main --squash

# Later: update from upstream
git subtree pull --prefix .vim/bundle/tpope-vim-surround \
  https://bitbucket.org/vim-plugins-mirror/vim-surround.git main --squash
```
* `--squash` squashes the sub-project's **entire history into one commit** (common practice). Omit it to **preserve full history**.
* The result is two commits:
```
1bda0bd (HEAD) Merge commit 'ca1f4da…' as '.vim/bundle/tpope-vim-surround'
ca1f4da        Squashed '.vim/bundle/tpope-vim-surround/' content from commit 02199ea
```
* Fast and painless, but the commands are **long and hard to remember** →

#### b) Add the sub-project as a remote (shorter commands)
```bash
git remote add -f tpope-vim-surround https://bitbucket.org/vim-plugins-mirror/vim-surround.git
git subtree add --prefix .vim/bundle/tpope-vim-surround tpope-vim-surround main --squash

# Update later
git fetch tpope-vim-surround main
git subtree pull --prefix .vim/bundle/tpope-vim-surround tpope-vim-surround main --squash
```

#### c) Contributing back upstream
* Commit fixes to the sub-project locally, **fork** the upstream project, add your fork as a remote, then **`subtree push`**:
```bash
git remote add durdn-vim-surround ssh://git@bitbucket.org/durdn/vim-surround.git
git subtree push --prefix=.vim/bundle/tpope-vim-surround/ durdn-vim-surround main
```
* Then open a **pull request** to the package maintainer.

---

### 6. Doing It Without the `git subtree` Command
* `git subtree` (the command) ≠ the **subtree merge strategy**. You can use the strategy even if the command isn't available:

```bash
# 1. Add the dependency as a normal remote
git remote add -f tpope-vim-surround https://bitbucket.org/vim-plugins-mirror/vim-surround.git

# 2. Record a merge (keep our tree) so the plugin's history is tracked
git merge -s ours --no-commit tpope-vim-surround/main
#   → "Automatic merge went well; stopped before committing as requested"

# 3. Read the plugin's latest tree into the prefix folder
git read-tree --prefix=.vim/bundle/tpope-vim-surround/ -u tpope-vim-surround/main

# 4. Commit — a merge commit preserving the tree's history
git commit -m "[subtree] adding tpope-vim-surround"

# 5. Update later using the subtree merge strategy
git pull -s subtree tpope-vim-surround main
```

---

### 7. Quick Command Reference
| Goal | Command |
|---|---|
| Add a sub-project | `git subtree add --prefix <dir> <repo> <branch> [--squash]` |
| Update from upstream | `git subtree pull --prefix <dir> <repo> <branch> [--squash]` |
| Push changes upstream | `git subtree push --prefix <dir> <remote> <branch>` |
| Manual merge strategy | `git merge -s ours --no-commit` → `git read-tree --prefix=<dir> -u` → `git commit` → `git pull -s subtree` |

---

### Summary Takeaway
`git subtree` nests another repository **as a normal sub-directory**, solving many submodule pain points: **no `.gitmodules`**, code is present **immediately after clone**, and teammates **don't need to learn anything new**. Use `add`/`pull`/`push` with `--prefix` (and `--squash` to keep history light), add the sub-project as a **remote** for shorter commands, and fall back to the **subtree merge strategy** if the command isn't available. There's a learning curve, but it's a great alternative to submodules.
