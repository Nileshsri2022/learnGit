Here is a clear breakdown of the key points about **`git clean`** from the text:

> Source: https://www.atlassian.com/git/tutorials/undoing-changes/git-clean

---

### 1. What Is `git clean`?
* An "undo" command for **untracked files**: files created in the working directory but **never added** with `git add`.
* Complements `git reset` and `git checkout`, which work on **tracked** files.

| Command | Operates on |
|---|---|
| `git reset`, `git checkout` | **Tracked** files (in the index) |
| `git clean` | **Untracked** files |

Setup example:
```bash
mkdir git_clean_test && cd git_clean_test
git init .
echo "tracked" > ./tracked_file && git add ./tracked_file
echo "untracked" > ./untracked_file
mkdir ./untracked_dir && touch ./untracked_dir/file
git status
# new file: tracked_file
# Untracked files: untracked_dir/  untracked_file
```

---

### 2. Safety First ⚠️
```bash
$ git clean
fatal: clean.requireForce defaults to true and neither -i, -n, nor -f given; refusing to clean
```
* Git **refuses** to clean without `-f`, `-n`, or `-i` (controlled by `clean.requireForce`).
* **`git clean` is NOT undoable.** It performs a **hard filesystem deletion**, like `rm`. Deleted files are **not** in Git's history, so they can't be recovered.
* By default it **doesn't recurse into directories**, another safety net.

---

### 3. Common Options

| Option | Effect |
|---|---|
| `-n` | **Dry run**: show what *would* be removed. ✅ **Always do this first** |
| `-f` / `--force` | **Actually delete** untracked files in the current directory (not directories, not ignored files) |
| `-f <path>` | Delete a **specific** untracked file |
| `-d` | Also include untracked **directories** |
| `-x` | Also include **ignored** files (e.g. build output, **and** things like `.idea/`) |
| `-i` | **Interactive** mode |

```bash
$ git clean -n
Would remove untracked_file          # untracked_dir not listed (no -d)

$ git clean -f
Removing untracked_file

$ git clean -dn
Would remove untracked_dir/
$ git clean -df
Removing untracked_dir/

$ git clean -xf                      # also removes .gitignore'd files — dry-run first!
```
* Options **combine**: `-dn`, `-df`, `-xf`, `-dxf`…
* ⚠️ `-x` removes **all** ignored files, not just build artifacts. That can include IDE config you wanted to keep.

---

### 4. Interactive Mode — `git clean -i`
```
$ git clean -di
Would remove the following items:
  untracked_dir/  untracked_file
*** Commands ***
    1: clean   2: filter by pattern   3: select by numbers   4: ask each   5: quit   6: help
What now>
```

| Command | What it does |
|---|---|
| `1: clean` | Delete the listed items |
| `2: filter by pattern` | **Exclude** items with patterns (e.g. `*_file` leaves only `untracked_dir/`) |
| `3: select by numbers` | Choose items to delete **by number** |
| `4: ask each` | Confirm each deletion with a `y/N` prompt (like `rm -i`) |
| `5: quit` | Stop without cleaning |
| `6: help` / `?` | Explain the commands / prompt help |

---

### Summary Takeaway
`git clean` **permanently deletes untracked files** from your working directory. It's a shortcut for `git status` plus manual `rm`. Always run a **dry run (`-n`)** first. Then use **`-f`** to delete, add **`-d`** for directories and **`-x`** for ignored files, or use **`-i`** to pick interactively. Together with **`git reset --hard`** it can fully return a repo to a clean state.
