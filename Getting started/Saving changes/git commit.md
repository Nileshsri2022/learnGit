Here is a clear breakdown of the key points about **`git commit`** from the text:

> Source: https://www.atlassian.com/git/tutorials/saving-changes/git-commit

---

### 1. What Does `git commit` Do?
* Captures a **snapshot of the currently staged changes**.
* Commits are **"safe" versions** of your project. Git **never changes them** unless you explicitly ask it to.
* `git add` (to stage) and `git commit` (to save) are two of the **most frequently used** Git commands.

---

### 2. `git commit` vs. `svn commit`
Same name, **completely different** behavior:

| | `svn commit` | `git commit` |
|---|---|---|
| Model | Centralized | Distributed |
| Where it goes | Pushes changes to the **remote central** repo | Saves to your **local** repo only |
| Network needed | ✅ Yes | ❌ No. Push to any remote **later** |

---

### 3. How It Works
* Git is a **timeline management utility**. **Commits are the building blocks** of the timeline: snapshots or milestones.
* Two "buffers" in Git:
  * **Staging area** = buffer between the **working directory** and **project history**.
  * **Local repository** = buffer between **your contributions** and the **central repository**.
* Accumulating commits locally lets you:
  * Split a feature into **atomic commits**
  * Keep **related commits together**
  * **Clean up** local history before publishing
  * Work **in isolation** and integrate when convenient
* ⚠️ Still, teams benefit most from integrating **frequently and in small units**.

---

### 4. Snapshots, Not Differences
| SVN | Git |
|---|---|
| Stores **diffs** against the original file | Stores the **entire content** of each file in every commit (a snapshot) |
| A version must be **assembled** from diffs | Every full revision is **immediately available** → faster |

* The snapshot model affects **everything**: branching, merging, and collaboration workflows.

---

### 5. Common Options

| Command | Effect |
|---|---|
| `git commit` | Commit the staged snapshot. Opens an **editor** for the message |
| `git commit -a` | Commit **all changes to tracked files** (skips `git add` for them; new untracked files are **not** included) |
| `git commit -m "message"` | Commit with an **inline message** (no editor) |
| `git commit -am "message"` | Power-user combo of `-a` + `-m` |
| `git commit --amend` | **Modify the last commit**: add staged changes to it and edit its message |

---

### 6. Example — Saving Changes
```bash
git add hello.py
git status
# On branch main
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#       new file: hello.py

git commit   # opens your editor
```
The editor shows:
```
# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# On branch main
# Changes to be committed:
#       modified: hello.py
```
* Lines starting with `#` are ignored. An **empty message aborts** the commit.

---

### 7. Writing Good Commit Messages
Git doesn't enforce a format, but the **canonical** style is:
```
Change the message displayed by hello.py          ← subject: < 50 characters

- Update the sayHello() function to output the user's name    ← body: details
- Change the sayGoodbye() function to a friendlier message
```
* **First line = subject** (like an email subject), then a **blank line**, then the **body**.
* Many developers write in the **present/imperative tense** ("Change…", "Add…"). It reads like an action on the repository and makes history-rewriting operations more intuitive.

---

### 8. Amending a Commit
```bash
# more edits to hello.py
git add hello.py
git commit --amend
```
* The editor opens **pre-filled with the previous message**. You're **editing the last commit**, not creating a new one.

---

### Summary Takeaway
`git commit` saves a **snapshot of staged changes** to your **local** repository (not to a server like SVN). Stage first with `git add`, then commit with a clear message (**short subject, blank line, detailed body**). Use `-m` for inline messages, `-a` to include all tracked changes, and **`--amend`** to fix the last commit.
