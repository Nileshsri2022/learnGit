Here is a clear breakdown of the key points about **Git merge strategies** from the text:

> Source: https://www.atlassian.com/git/tutorials/using-branches/merge-strategy

---

### 1. Why Merge Strategies Matter
* When work is complete, tested, and ready to merge back into the main line, your team has **policy choices** to make.
* A merge takes **two (or more) commit pointers** and finds a **common base commit**. The different ways Git finds that base are called **merge strategies**.
* A **merge commit** is just a regular commit that happens to have **two (or more) parents**.
* `git merge` and `git pull` accept **`-s <strategy>`**. If omitted, Git **picks the most appropriate strategy** automatically.

---

### 2. The Merge Strategies (`-s`)

| Strategy | Command | Heads | What it does / when to use it |
|---|---|---|---|
| **Recursive** | `git merge -s recursive <branch>` | 2 | Default when pulling/merging **one** branch (in older Git). Detects and handles **renames**, but can't use detected copies |
| **Resolve** | `git merge -s resolve <branch>` | 2 | Classic **3-way merge**. Carefully detects **criss-cross merge** ambiguities. Safe and fast |
| **Octopus** | `git merge -s octopus b1 b2 b3 …` | **>2** | **Default for more than two heads**. **Refuses** if manual conflict resolution is needed. Used to bundle similar feature branches |
| **Ours** | `git merge -s ours b1 b2 …` | N | Result is **always the current `HEAD`'s tree**. It **ignores all changes** from other branches but records their history. Used to combine the history of similar branches |
| **Subtree** | `git merge -s subtree A B` | 2 | Extension of recursive. If B is a **subtree** of A, B's tree (and the common ancestor's) is first adjusted to match A's structure |

> 📝 *Extra note (not in the original article):* Since **Git 2.34**, the default 2-head strategy is **`ort`** ("Ostensibly Recursive's Twin"). It's a faster rewrite of `recursive` that accepts the same options. In recent Git versions, `-s recursive` is simply redirected to `ort`.

---

### 3. Types of Merges (Team Policy)

| Type | Description | Trade-off |
|---|---|---|
| **Explicit merge** *(default)* | Creates a **new merge commit** that shows **where** a merge happened and **which commits** were its parents | Great traceability, but some teams see merge commits as **"noise"** |
| **Implicit merge** (via **rebase** or **fast-forward**) | No merge commit. History stays linear | *(section is empty on the live page)* |
| **Squash on merge** (usually without an explicit merge) | All branch commits are condensed into **one** commit on the target | *(section is empty on the live page)* |

> 📝 *Extra note (not in the original article):* The live page has headings for the last two types but no text. In short: **rebase/fast-forward** gives a linear history but loses the "this was a feature branch" context. **Squash** (`git merge --squash <branch>` + `git commit`) gives one tidy commit per feature but throws away the individual commits.

---

### 4. Recursive Strategy Options (`-X <option>`)
Sub-options for the `recursive` (and `ort`) strategy, passed as `git merge -X <option> <branch>`:

| Option | Effect |
|---|---|
| `ours` | Auto-resolve **conflicts** by favoring **our** version. Non-conflicting changes from "theirs" are still merged. ⚠️ **Not** the same as `-s ours` |
| `theirs` | The opposite: favor **their** (the merged branch's) version in conflicts |
| `patience` | Spend extra time avoiding mis-merges on unimportant matching lines (like braces). Best when branches have **diverged a lot** |
| `diff-algorithm=<alg>` | Choose the diff algorithm to use |
| `ignore-space-change`, `ignore-all-space`, `ignore-space-at-eol`, `ignore-cr-at-eol` | Treat lines with only these **whitespace** changes as unchanged |
| `renormalize` | Run a virtual check-out/check-in of all three trees during the 3-way merge. For branches with different **line-ending/filter** (checkin/checkout) settings |
| `no-renormalize` | Disable `renormalize` (overrides `merge.renormalize`) |
| `no-renames` | **Ignore renames** during the merge |
| `find-renames[=<n>]` | *(default)* Detect renames. `<n>` = similarity threshold |
| `subtree[=<path>]` | Like the subtree strategy, but works on **path metadata** to make the trees match |

> 📝 *Extra note (not in the original article):* The article says the default `find-renames` threshold is 100%. According to the official Git docs it's **50%**.

```bash
git merge -X theirs feature        # prefer feature's side in conflicts
git merge -X ignore-all-space dev  # ignore whitespace-only differences
git merge -s ours old-experiment   # record the merge, keep our tree unchanged
```

---

### 5. Atlassian's Merge Policy
* Atlassian **strongly prefers explicit merges**, because they provide **great traceability and context** about the features being merged.
* A **local clean-up rebase** before sharing a feature branch for review is **encouraged**. It **augments** the policy rather than replacing it.

---

### Summary Takeaway
Git picks a **merge strategy** automatically: `recursive`/`ort` for two heads, `octopus` for more. You can force one with **`-s`** (`resolve`, `octopus`, `ours`, `subtree`) and tune recursive/ort with **`-X`** options (`ours`, `theirs`, `patience`, whitespace, rename, and normalization options). For team policy, choose between **explicit merge commits** (traceable, and Atlassian's preference), **rebase/fast-forward** (linear), and **squash** (one commit per feature).
