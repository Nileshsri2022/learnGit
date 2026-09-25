Here is a clear breakdown of the key points about **Advanced Git log** from the text:

> Source: https://www.atlassian.com/git/tutorials/git-log

---

### 1. Why Master `git log`?
* Version control lets you see **who contributed what**, find **where bugs were introduced**, and **revert** problematic changes — but history is useless if you can't **navigate** it.
* Advanced `git log` features fall into **two categories**:
  1. **Formatting** — *how* each commit is displayed.
  2. **Filtering** — *which* commits are included.
* 💡 Don't like the default format? Save your favorite combo as a **`git config` alias**.

---

## PART A — Formatting Log Output

### 2. `--oneline`
* One line per commit: **short ID + first line of the message** → high-level overview.
```
0e25143 Merge branch 'feature'
ad8621a Fix a bug in the feature
16b36c6 Add a new feature
23ad9ad Add the initial code base
```

### 3. `--decorate`
* Shows all **refs** (branches, tags, `HEAD`) pointing to each commit.
```bash
git log --oneline --decorate
```
```
0e25143 (HEAD, main) Merge branch 'feature'
ad8621a (feature) Fix a bug in the feature
16b36c6 Add a new feature
23ad9ad (tag: v0.9) Add the initial code base
```
* Branches + tags + `HEAD` + history ≈ almost everything in a repo → a complete view of its **logical structure**.

### 4. Diffs: `--stat` and `-p`
| Option | Shows |
|---|---|
| `--stat` | Insertions/deletions **per file** (a modified line = 1 insertion + 1 deletion). `+`/`-` bars show relative change size |
| `-p` | The **full patch** for each commit |

```
 hello.py | 105 ++++++++++++++++++++++++-----------------
 1 file changed, 67 insertion(+), 38 deletions(-)
```
* Full patches get long — if you're hunting for a specific change, use the **pickaxe** (`-S`, see below).

### 5. `git shortlog`
* Special version of `git log` for **release announcements**: groups commits **by author**, showing each first message line.
```
Mary (2):
      Fix a bug in the feature
      Fix a serious security hole in our framework

John (3):
      Add the initial code base
      Add a new feature
      Merge branch 'feature'
```
* Sorted by author name by default; `-n` sorts by **number of commits**.

### 6. `--graph`
* Draws an **ASCII graph** of the branch structure; usually combined with `--oneline --decorate`.
```bash
git log --graph --oneline --decorate
```
```
*   0e25143 (HEAD, main) Merge branch 'feature'
|\
| * 16b36c6 Fix a bug in the new feature
| * 23ad9ad Start a new feature
* | ad8621a Fix a critical security issue
|/
* 400e4b7 Fix typos in the documentation
* 160e224 Add the initial code base
```
* The `*` shows which branch a commit is on. For heavily branched projects, prefer **`gitk`** or **Sourcetree**.

### 7. Custom Formatting — `--pretty=format:"<string>"`
* `printf`-style placeholders let you display commits **exactly how you want**.
```bash
git log --pretty=format:"%cn committed %h on %cd"
# John committed 400e4b7 on Fri Jun 24 12:30:04 2014 -0500
```
| Placeholder | Meaning |
|---|---|
| `%cn` | Committer name |
| `%h` | Abbreviated commit hash |
| `%cd` | Committer date |

* Full list: "Pretty Formats" in the `git log` man page.
* Especially useful when **piping** `git log` output into other commands.

---

## PART B — Filtering the Commit History
*(All filters can be combined with any formatting option.)*

### 8. Filter Reference

| Filter | Example | Notes |
|---|---|---|
| **By amount** | `git log -3` | Only the 3 most recent commits |
| **By date** | `git log --after="2014-7-1"` | Inclusive; many date formats |
| | `git log --after="yesterday"` | Relative dates like `"1 week ago"` |
| | `git log --after="2014-7-1" --before="2014-7-4"` | Between two dates |
| **By author** | `git log --author="John"` | Regex; name only needs to **contain** the phrase; matches **email** too |
| | `git log --author="John\|Mary"` | Either author |
| | `--committer` | Same, for committers (if your workflow separates them) |
| **By message** | `git log --grep="JRA-224:"` | Regex on commit message (e.g. issue IDs); add `-i` to ignore case |
| **By file** | `git log -- foo.py bar.py` | `--` means "what follows are file paths, not branches" (omit if unambiguous) |
| **By content (pickaxe)** | `git log -S"Hello, World!"` | Commits that **add or remove** that string |
| | `git log -G"<regex>"` | Same, using a regex |
| **By range** | `git log main..feature` | Commits in `feature` but **not** in `main` |
| **Hide merges** | `git log --no-merges` | Skip merge commits |
| **Only merges** | `git log --merges` | Commits with **≥ 2 parents** |

### 9. Deep-Dive Notes
* **Pickaxe (`-S`)** is a **powerful debugging tool** — it finds every commit affecting a particular line of code, even when it was **copied or moved** to another file.
* **Ranges** `<since>..<until>`:
  * `main..feature` → how far `feature` has progressed since forking from `main`.
  * `feature..main` → commits in `main` not in `feature`.
  * If **both** return commits → your history has **diverged**.
* **Merge filtering:** teams with an **always-merge** policy (merging upstream into topic branches instead of rebasing) collect many extraneous merge commits — `--no-merges` hides them.

---

### Summary Takeaway
`git log` becomes a superpower when you combine **formatting** (`--oneline`, `--decorate`, `--stat`, `-p`, `--graph`, `--pretty=format`, `git shortlog`) with **filtering** (`-n`, `--after/--before`, `--author`, `--grep`, `-- <file>`, `-S`/`-G`, `a..b` ranges, `--merges`/`--no-merges`). Once you find the commit you need, hand it to **`git checkout`**, **`git revert`**, or other history tools.
