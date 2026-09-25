Here is a clear breakdown of the key points about **Inspecting a repository** (`git status` & `git log`) from the text:

> Source: https://www.atlassian.com/git/tutorials/inspecting-a-repository

---

### 1. The Two Inspection Commands at a Glance

| Command | Looks at | Shows |
|---|---|---|
| `git status` | **Working directory + staging area** | Staged, unstaged, and untracked files |
| `git log` | **Committed history only** | Commits: list, filter, search |

```
Working Directory ── Staging Area ──│── Commit History
      └──────── git status ─────────┘   └── git log ──┘
```

#### Related commands
* **`git tag`** — tags are refs pointing to specific points in history, usually used to mark releases (e.g. `v1.0.1`).
* **`git blame`** — shows **author metadata** for each committed line of a file; answers *what, how, and why* code was added.
* **`git log`** — displays committed snapshots (covered below).

---

### 2. `git status`
* Shows what has been happening with `git add` and `git commit`.
* Status output also includes **helpful hints** for staging/unstaging.

```bash
git status   # list staged, unstaged and untracked files
```

#### The three main categories in the output
```
# On branch main
# Changes to be committed:            ← STAGED
#   (use "git reset HEAD <file>..." to unstage)
#       modified: hello.py
#
# Changes not staged for commit:      ← MODIFIED but NOT staged
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#       modified: main.py
#
# Untracked files:                    ← NEW, Git doesn't track them yet
#   (use "git add <file>..." to include in what will be committed)
#       hello.pyc
```

---

### 3. Ignoring Files with `.gitignore`
* Untracked files are usually either:
  1. **New files** not yet committed → useful to see.
  2. **Compiled binaries** (`.pyc`, `.obj`, `.exe` …) → noise that hides what's really going on.
* Put paths in a special **`.gitignore`** file, **one pattern per line**; `*` works as a wildcard.

```gitignore
*.pyc
```

---

### 4. Example — Status Before & After Committing
* **Good practice:** check status **before committing** so you don't commit something by accident.

```bash
# Edit hello.py
git status   # hello.py → "Changes not staged for commit"
git add hello.py
git status   # hello.py → "Changes to be committed"
git commit
git status   # "nothing to commit (working directory clean)"
```

* Some commands (e.g. **`git merge`**) require a **clean working directory** so you don't overwrite changes.

---

### 5. `git log` — Common Options

| Command | What it shows |
|---|---|
| `git log` | Full history (default format). `Space` to scroll, `q` to quit |
| `git log -n <limit>` | Only the last `<limit>` commits (e.g. `-n 3`) |
| `git log --oneline` | Each commit on **one line** — quick overview |
| `git log --stat` | Files changed + number of lines added/deleted |
| `git log -p` | The full **patch/diff** of each commit (most detailed view) |
| `git log --author="<pattern>"` | Commits by an author (string or regex) |
| `git log --grep="<pattern>"` | Commits whose **message** matches (string or regex) |
| `git log <since>..<until>` | Commits between two refs (commit ID, branch, `HEAD`…) |
| `git log <file>` | Only commits that touched `<file>` |
| `git log --graph --decorate --oneline` | Text **graph** of history + branch/tag names, one line each |

---

### 6. Understanding Commit IDs and References
```
commit 3157ee3718e180a9476bf2e5cab8e3f1e78a73b7
Author: John Smith
```
* The **40-character string** is a **SHA-1 checksum** of the commit's contents. It serves two purposes:
  1. **Integrity** — a corrupted commit would produce a different checksum.
  2. **Unique ID** — used to reference the commit (a short prefix is enough, e.g. `3157e`).

#### Ways to refer to a commit
| Reference | Meaning |
|---|---|
| `3157e` | A commit by (abbreviated) hash |
| `main`, `some-feature` | The tip of a branch |
| `HEAD` | The **current** commit (branch or detached commit) |
| `3157e~1` | The **parent** of `3157e` |
| `HEAD~3` | The **great-grandparent** of the current commit |

---

### 7. Combining Options

```bash
# Full diff of every change John Smith made to hello.py
git log --author="John Smith" -p hello.py

# Commits that are in some-feature but NOT in main (great for comparing branches)
git log --oneline main..some-feature
```

---

### Summary Takeaway
Use **`git status`** constantly to see what's **staged, unstaged, and untracked** (and keep noise out with **`.gitignore`**). Use **`git log`** to explore the **committed history** — filter by author, message, file, or range, and reference commits by **hash, branch, `HEAD`, or `~` relative notation**.
