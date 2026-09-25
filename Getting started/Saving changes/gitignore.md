Here is a clear breakdown of the key points about **`.gitignore`** from the text:

> Source: https://www.atlassian.com/git/tutorials/saving-changes/gitignore
> *(This file is named `gitignore.md` without the leading dot so it isn't hidden.)*

---

### 1. Three Kinds of Files
| State | Meaning |
|---|---|
| **Tracked** | Previously staged or committed |
| **Untracked** | Not staged or committed yet |
| **Ignored** | Git has been **explicitly told to ignore** it |

---

### 2. What Usually Gets Ignored
Build artifacts and machine-generated files that can be **derived from source** or **shouldn't be committed**:

| Category | Examples |
|---|---|
| Dependency caches | `/node_modules`, `/packages` |
| Compiled code | `.o`, `.pyc`, `.class` |
| Build output | `/bin`, `/out`, `/target` |
| Runtime files | `.log`, `.lock`, `.tmp` |
| Hidden system files | `.DS_Store`, `Thumbs.db` |
| Personal IDE config | `.idea/workspace.xml` |

* Rules live in a **`.gitignore`** file, usually at the **repo root**. It's checked in like any other file.
* There's **no `git ignore` command**. You edit and commit `.gitignore` by hand.

---

### 3. Ignore Patterns (Globbing)

| Pattern | Matches | Doesn't match | Rule |
|---|---|---|---|
| `**/logs` | `logs/debug.log`, `logs/monday/foo.bar`, `build/logs/debug.log` | | `**/` matches a directory **anywhere** |
| `**/logs/debug.log` | `logs/debug.log`, `build/logs/debug.log` | `logs/build/debug.log` | Match by file name **and** parent dir name |
| `*.log` | `debug.log`, `foo.log`, `.log`, `logs/debug.log` | | `*` = zero or more characters |
| `*.log` + `!important.log` | `debug.log`, `trace.log` | `important.log`, `logs/important.log` | `!` **negates** a pattern (un-ignores) |
| `*.log` + `!important/*.log` + `trace.*` | `debug.log`, `important/trace.log` | `important/debug.log` | Patterns **after** a negation **re-ignore** files |
| `/debug.log` | `debug.log` | `logs/debug.log` | Leading `/` = **repo root only** |
| `debug.log` | `debug.log`, `logs/debug.log` | | By default, matches in **any** directory |
| `debug?.log` | `debug0.log`, `debugg.log` | `debug10.log` | `?` = exactly **one** character |
| `debug[0-9].log` | `debug0.log`, `debug1.log` | `debug10.log` | `[range]` = one character from a range |
| `debug[01].log` | `debug0.log`, `debug1.log` | `debug2.log`, `debug01.log` | `[set]` = one character from a set |
| `debug[!01].log` | `debug2.log` | `debug0.log`, `debug1.log`, `debug01.log` | `[!set]` = any character **except** those |
| `debug[a-z].log` | `debuga.log`, `debugb.log` | `debug1.log` | Ranges can be numeric or alphabetic |
| `logs` | `logs`, `logs/debug.log`, `logs/latest/foo.bar`, `build/logs`, `build/logs/debug.log` | | No trailing slash = matches **files and directories** |
| `logs/` | `logs/debug.log`, `logs/latest/foo.bar`, `build/logs/foo.bar`, `build/logs/latest/debug.log` | | Trailing `/` = **directory** (everything inside is ignored) |
| `logs/` + `!logs/important.log` | `logs/debug.log`, **`logs/important.log`** | | ⚠️ You **can't** un-ignore a file inside an **ignored directory** (a performance quirk) |
| `logs/**/debug.log` | `logs/debug.log`, `logs/monday/debug.log`, `logs/monday/pm/debug.log` | | `/**/` = zero or more directories |
| `logs/*day/debug.log` | `logs/monday/debug.log`, `logs/tuesday/debug.log` | `logs/latest/debug.log` | Wildcards work in directory names |
| `logs/debug.log` | `logs/debug.log` | `debug.log`, `build/logs/debug.log` | A path with a directory is **relative to the root** (a leading `/` adds nothing) |

\* These assume `.gitignore` is at the repo root. For a nested `.gitignore`, read "repo root" as "the directory containing that `.gitignore`".

> 📝 *Extra note (not in the original article):* The table on the live page has some rows misaligned (explanations shifted by one row). The table above restores the intended pattern ↔ explanation pairs.

#### Comments & escaping
```gitignore
# ignore all logs
*.log

# ignore the file literally named foo[01].txt
foo\[01\].txt
```

---

### 4. Where Ignore Rules Can Live

| Location | Versioned & shared? | Scope | Use for |
|---|---|---|---|
| `.gitignore` (root, or per directory) | ✅ Yes, committed | That repo (patterns relative to the file's directory) | Patterns that **benefit the whole team** |
| `.git/info/exclude` | ❌ No | That repo, **only you** | Personal tools, custom logging output |
| Global file via `core.excludesFile` | ❌ No | **All repos** on your machine | OS files (`.DS_Store`, `thumbs.db`), editor temp files |

```bash
touch ~/.gitignore
git config --global core.excludesFile ~/.gitignore
```
* Convention: a **single root `.gitignore`** is simplest.
* Be careful with global patterns: different projects need different files.

---

### 5. Ignoring a Previously Committed File
Adding a rule isn't enough. You also have to **remove the file from the repo**:
```bash
echo debug.log >> .gitignore
git rm --cached debug.log      # delete from repo, KEEP in working directory
git commit -m "Start ignoring debug.log"
```
* Omit `--cached` to delete it from **both** the repo and your disk.

---

### 6. Committing an Ignored File
```bash
git add -f debug.log            # force-add despite *.log
git commit -m "Force adding debug.log"
```
✅ **Better:** add an **exception** so teammates can see it clearly:
```bash
echo '!debug.log' >> .gitignore   # quotes stop the shell from treating ! specially
# .gitignore now:
#   *.log
#   !debug.log
git add debug.log
git commit -m "Adding debug.log"
```

---

### 7. Stashing Ignored Files
* `git stash` skips ignored files by default. Use **`git stash --all`** to include ignored **and** untracked files.

---

### 8. Debugging — `git check-ignore`
```bash
$ git check-ignore -v debug.log
.gitignore:3:*.log  debug.log
# <file with pattern>:<line number>:<pattern>   <file name>
```
* Shows **which rule** ignores a file. Accepts multiple names, and the files **don't even have to exist**.

---

### Summary Takeaway
`.gitignore` tells Git which files to **leave alone**: build output, caches, logs, OS/IDE files. Learn the **glob patterns** (`*`, `**`, `?`, `[]`, `!`, leading/trailing `/`). Put **team rules** in `.gitignore`, **personal rules** in `.git/info/exclude`, and **machine-wide rules** in a global `core.excludesFile`. Use `git rm --cached` to stop tracking an already-committed file, and `git check-ignore -v` to debug.
