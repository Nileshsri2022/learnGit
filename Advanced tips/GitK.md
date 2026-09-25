Here is a clear breakdown of the key points about **Gitk** from the text:

> Source: https://www.atlassian.com/git/tutorials/gitk

---

### 1. What Is Gitk?
* A **graphical repository browser**, the **first of its kind**.
* Think of it as a **GUI wrapper for `git log`**: it's for **exploring and visualizing** a repo's history.
* Written in **Tcl/Tk**, so it's **portable** across operating systems.
* Maintained by **Paul Mackerras** as a project **separate from Git core**, but stable versions **ship with Git** for convenience.
* A great **learning aid** for Git newcomers and people coming from other VCSs like Subversion.
* Other popular GUIs: **git-gui** and Atlassian's **Sourcetree**.

---

### 2. The Gitk Window
Run it just like `git log`:
```bash
gitk
```

```
┌──────────────────────────────┬─────────────────────────┐
│  Commit history (newest top) │  (authors / dates)      │  ← upper panes
├──────────────────────────────┼─────────────────────────┤
│  Commit details + full diff  │  Files changed in the   │  ← lower panes
│                              │  selected commit        │
└──────────────────────────────┴─────────────────────────┘
```

| Pane | Shows |
|---|---|
| **Upper left** | List of commits, **latest on top** |
| **Lower left** | **Commit details** and the **full diff** |
| **Lower right** | **Files affected** by the selected commit. Click one to jump to its part of the diff |

* Gitk shows the repo **as it was when launched**. If you change things on the command line (e.g. switch branches), **reload** with **File → Reload**.

---

### 3. Options
```bash
gitk [<options>] [<revision range>] [--] [<path>…]
```
Most options **limit which commits** are shown:

| Option | Effect |
|---|---|
| `<revision range>` | e.g. `from..to`: commits reachable from `to` but not from `from`. A single revision also works |
| `<path>…` | Only commits touching these paths (use `--` to separate paths from revisions) |
| `--all` | Show **all** branches, tags, and refs |
| `--branches[=<pattern>]` / `--tags[=<pattern>]` / `--remotes[=<pattern>]` | Show those refs as if they were mainline commits, optionally filtered by a pattern |
| `--since=<date>` / `--until=<date>` | Commits **newer** / **older** than a date |
| `--date-order` | Sort commits **by date** |
| `--merge` | Commits that touch files **conflicted during a merge** |
| `--left-right` | Mark which side of a range each commit comes from: `<` for left, `>` for right |
| `--ancestry-path` | With a range `commit1..commit2`, show only commits **on the direct ancestry chain** (descendants of `commit1` **and** ancestors of `commit2`) |
| `-L<start>,<end>:<file>` | **Trace the history of a line range** in a file |

---

### 4. Examples

#### Demo repo
```bash
mkdir gitkdemo && cd gitkdemo && git init . &&
echo "hello world" > index.txt &&
git add index.txt &&
git commit -m "added index.txt with hello world content"
gitk
```

#### Compare two commits
```bash
echo "prpended content to index" >> index.txt &&
git commit -am "prepended content to index"
# in gitk: File → Reload
```
1. In the history pane, **click** the base commit.
2. **Right-click** the second commit.
3. Choose **"Diff this → selected"** or **"Diff selected → this"**.
4. The diff appears in the lower-left pane (here: the new line added to `index.txt`).

#### Compare two branches
```bash
git checkout -b new_branch &&
echo "new branch content" > new_branch_file.txt &&
git add new_branch_file.txt &&
git commit -m "new branch commit with new file and prepended content" &&
echo "new branch index update" >> index.txt &&
git commit -am "new branch commit to index.txt with new content"
```
* `new_branch` is now **2 commits ahead** of `main`. After a reload, Gitk shows them as a **straight line**, with no visible fork.
* 💡 That's a lesson in how Git branches work: **a branch is just a pointer to a commit** that moves as you commit. Creating a branch **doesn't change the repo's structure**. It only adds a pointer. A fork only appears once **both** branches get new commits.
* To see **only the commits that differ**:
```bash
gitk main..new_branch
```

---

### 5. Gitk vs. Git Gui
| | **Gitk** | **Git Gui** (`git gui`) |
|---|---|---|
| Built with | Tcl/Tk | Tcl/Tk |
| Focus | **Navigating and visualizing history** | **Crafting individual commits** (staging, committing), single-file annotation (blame) |
| Shows project history? | ✅ Yes | ❌ No, but it has a menu item to **launch Gitk** |

> 📝 *Extra note (not in the original article):* If `gitk` isn't found on Linux, it's often a separate package (e.g. `sudo apt install gitk`). It's included with Git for Windows and usually with macOS Git installs.

---

### Summary Takeaway
Gitk is a **graphical `git log`**. It shows commits, diffs, and changed files in three panes, and accepts the same kind of **revision ranges, paths, and filters** (`--all`, `--since`, `-L`, `main..branch`). Use it to **compare commits** (right-click → Diff) and **branches** (`gitk main..new_branch`), and to **learn how Git works internally**, such as branches being simple pointers. Remember to **File → Reload** after command-line changes.
