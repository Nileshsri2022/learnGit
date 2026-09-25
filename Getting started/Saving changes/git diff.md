Here is a clear breakdown of the key points about **`git diff`** from the text:

> Source: https://www.atlassian.com/git/tutorials/saving-changes/git-diff

---

### 1. What Is `git diff`?
* **Diffing** takes two data sets and outputs the **changes between them**.
* `git diff` runs a diff on Git data sources: **commits, branches, files, the index**, and more.
* Often used together with **`git status`** and **`git log`** to analyze the state of a repo.

---

### 2. Reading Diff Output
Setup:
```bash
mkdir diff_test_repo && cd diff_test_repo
echo "this is a git diff test example" > diff_test.txt
git init .
git add diff_test.txt
git commit -am "add diff test file"
echo "this is a diff example" > diff_test.txt   # modify the file
git diff
```
Output:
```diff
diff --git a/diff_test.txt b/diff_test.txt
index 6b0c6cf..b37e70a 100644
--- a/diff_test.txt
+++ b/diff_test.txt
@@ -1 +1 @@
-this is a git diff test example
+this is a diff example
```

| # | Line | Meaning |
|---|---|---|
| 1 | `diff --git a/… b/…` | **Comparison input**: the two sources being diffed |
| 2 | `index 6b0c6cf..b37e70a 100644` | **Metadata**: internal Git object hashes + file mode (rarely needed) |
| 3 | `--- a/…` / `+++ b/…` | **Legend**: `-` marks lines from source **a**, `+` marks lines from source **b** |
| 4 | `@@ -1 +1 @@` … | **Chunks (hunks)**: only the changed sections of the file |

#### Chunk headers
* `@@ -34,6 +34,8 @@` → in version **a**, **6 lines** starting at line **34**. In version **b**, **8 lines** starting at line **34**.
* Each changed line starts with **`-`** (from a / removed) or **`+`** (from b / added).

---

### 3. Highlighting Changes
| Method | What it does |
|---|---|
| `git diff --color-words` | Splits lines into **words** (by whitespace) and highlights only the **changed words** |
| `diff-highlight` (Perl script in Git's `contrib/` folder) | Pairs matching lines and highlights the changed **sub-word fragments** |

```bash
git diff --color-words
git diff | /path/to/git-core/contrib/diff-highlight/diff-highlight
```

---

### 4. Diffing Binary Files
By default the output is unhelpful:
```
Binary files a/script.pdf and b/script.pdf differ
```
Fix it with a **`textconv` filter** that converts binary content to text before diffing:

1. Define the filter in `.git/config` (one repo) or `~/.gitconfig` (global):
```ini
[diff "pdfconv"]
    textconv = pdftohtml -stdout
```
2. Map file patterns to it in **`.gitattributes`** at the repo root:
```
*.pdf diff=pdfconv
```

Other converters:
| File type | Converter idea |
|---|---|
| zip, jar, archives | `unzip -l` (shows added/removed paths) |
| Images | `exiv2` (metadata changes like dimensions) |
| Documents (.odf, .doc…) | Doc-to-text conversion tools |
| Anything else | `strings` works in a pinch |

---

### 5. Common Comparisons Cheat Sheet

| Command | Compares |
|---|---|
| `git diff` | **Working directory vs. index**: changes **not yet staged** |
| `git diff ./path/to/file` | Same as above, limited to one file |
| `git diff --cached [file]` (= `--staged`) | **Index vs. last commit**: what's **staged** for the next commit |
| `git diff HEAD [file]` | **Working directory vs. last commit**: all uncommitted changes (staged + unstaged) |
| `git diff <commitA> <commitB>` | Two commits (IDs from `git log --pretty=oneline`, or refs like `HEAD`, tags, branches) |
| `git diff branch1..branch2` (or `branch1 branch2`) | The **tips** of both branches |
| `git diff branch1...branch2` | The **common ancestor** of both vs. the tip of `branch2` (i.e. "what did branch2 add since it split off?") |
| `git diff main new_branch ./diff_test.txt` | One **file** across two branches |

> 📝 *Extra note (not in the original article):* The article says `git diff HEAD ./path/to/file` is the same as `git diff ./path/to/file`. It isn't. Without `HEAD`, Git compares against the **index** (unstaged changes only). With `HEAD`, it compares against the **last commit** (staged + unstaged). They only give the same output when nothing is staged.

```
   Working Dir ──── git diff ────▶ Index ──── git diff --cached ────▶ HEAD
        └──────────────────────── git diff HEAD ─────────────────────────┘
```

---

### 6. Two Dots vs. Three Dots
```
          A───B───C  branch2
         /
    D───E───F───G    branch1

git diff branch1..branch2   →  G vs C   (tip vs tip)
git diff branch1...branch2  →  E vs C   (common ancestor vs tip of branch2)
```

---

### Summary Takeaway
`git diff` shows **what changed** between any two Git data sources. Learn to read its output (**inputs, metadata, `---`/`+++` legend, `@@` chunks**). Use `--color-words` for word-level detail, `textconv` for binary files, and the right form for the job: **unstaged** (`git diff`), **staged** (`--cached`), **commits**, or **branches** (`..` vs `...`).
