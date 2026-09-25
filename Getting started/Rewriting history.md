Here is a clear breakdown of the key points about **Rewriting history in Git** (`commit --amend`, `rebase`, `reflog`) from the text:

> Source: https://www.atlassian.com/git/tutorials/rewriting-history

---

### 1. Why Rewrite History?
* Git's main job is to make sure you **never lose a committed change** — but it also gives you **total control** over what your project history looks like.
* That control comes with a **disclaimer**: history-rewriting commands **can lose content** if misused.
* Three main mechanisms covered:
  1. **`git commit --amend`** — fix the last commit.
  2. **`git rebase`** (esp. `-i`) — change older or multiple commits.
  3. **`git reflog`** — the safety net to recover from mistakes.

---

### 2. Changing the Last Commit — `git commit --amend`
* Combines **staged changes** with the previous commit instead of creating a new one, or just edits the **commit message**.
* ⚠️ Amending **replaces** the commit entirely — the result is a **brand-new commit with a new ID** (shown as `*` in diagrams).

#### Fix the last commit message
```bash
git commit --amend                               # nothing staged → just edit the message in the editor
git commit --amend -m "an updated commit message" # pass the new message inline
```

#### Add a forgotten file
```bash
# Edit hello.py and main.py
git add hello.py
git commit
# Oops — forgot main.py
git add main.py
git commit --amend --no-edit   # --no-edit keeps the existing message
```
* Result: looks like `hello.py` and `main.py` were committed together in **one snapshot**.

#### 🚫 Don't amend public commits
* The old commit disappears from your branch — the same consequences as **resetting a public snapshot**.
* Teammates who built on it face **confusing, lengthy merge conflicts**.

---

### 3. Changing Older or Multiple Commits — `git rebase`
* **Standard rebase** re-applies your branch's commits on top of another branch head — literally rewriting history.
* ⚠️ Never rebase commits that were **pushed publicly** — it will look like project history disappeared.
* **Interactive rebase (`git rebase -i`)** lets you alter **individual commits** as they are replayed.

#### Useful interactive commands
| Command | Effect |
|---|---|
| `pick` | Keep the commit as-is (may still get a new ID if earlier commits changed) |
| `edit` / `e` | **Pause** on this commit so you can amend it |
| `squash` / `s` | **Merge** this commit into the previous one and combine messages |
| (reword etc.) | Modify commit messages during playback |

#### Pausing to edit a commit
```
Stopped at 5d025d1... formatting
You can amend the commit now, with

    git commit --amend

Once you are satisfied with your changes, run

    git rebase --continue
```

#### Squash for a clean history
* `squash` is where rebase really shines: fold small "fix typo" / "WIP" commits into meaningful ones.
* Git opens your editor to **combine the commit messages**.
* Rewritten commits get **new IDs**.
* Hosting tools like **Bitbucket** offer **auto-squash on merge** via the UI.

---

### 4. The Safety Net — `git reflog`
* **Reference logs** record every update to the **tips of branches** and other refs — switching branches, pulling, rewriting history, or adding commits.
* Lets you return to commits that **no branch or tag references** anymore.

```bash
git reflog                  # show the local reflog
git reflog --relative-date  # with relative dates, e.g. "2 weeks ago"
```

#### Example: recovering from an accidental hard reset
```
0a2e358 HEAD@{0}: reset: moving to HEAD~2
0254ea7 HEAD@{1}: checkout: moving from 2.2 to main
c10f740 HEAD@{2}: checkout: moving from main to 2.2
```
* `HEAD@{0}` is the **latest** action — a hard reset that dropped 2 commits.
* `main` pointed to `0254ea7` before the reset, so:
```bash
git reset --hard 0254ea7   # main is back where it was
```

#### Reflog limitations
* Only helps for changes that were **committed locally**.
* Tracks only movements of **branch tips / refs**.
* Entries **expire** — default is **90 days**.

---

### Summary Takeaway
* There are **many ways** to rewrite history in Git.
* **`git commit --amend`** → change the latest message or add/remove changes in the most recent commit.
* **`git rebase`** → combine commits and modify a branch's history; **`git rebase -i`** gives fine-grained control (edit, squash, reword).
* **`git reflog`** → your **safety net** to undo history rewrites.
* **Golden rule:** only rewrite history that **hasn't been shared** with others.
