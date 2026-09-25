Here is a clear breakdown of the key points about **Git merge conflicts** from the text:

> Source: https://www.atlassian.com/git/tutorials/using-branches/merge-conflicts

---

### 1. Why Conflicts Happen
* Version control manages contributions from **many authors**. When two developers edit **the same content**, a conflict can occur.
* Working in **separate isolated branches** reduces conflicts. `git merge` combines branches and resolves conflicting edits.

#### A conflict arises when…
* Two people changed **the same lines** in a file, or
* One developer **deleted a file** while another **modified** it.

* Git **can't decide automatically**, so it marks the file as conflicted and **halts the merge**.
* Only the **developer doing the merge** is affected. The rest of the team doesn't see it.

---

### 2. Two Types of Merge Conflicts

| When | Cause | Error message | Fix |
|---|---|---|---|
| **Merge fails to START** | **Pending local changes** (working directory/staging area) that the merge could overwrite. Not a conflict with other developers | `error: Entry '<fileName>' not uptodate. Cannot merge. (Changes in working directory)` | Stabilize local state: `git stash`, `git checkout`, `git commit`, or `git reset` |
| **Merge fails DURING** | A real conflict between the **current branch** and the **branch being merged** (another developer's code) | `error: Entry '<fileName>' would be overwritten by merge. Cannot merge. (Changes in staging area)` | Resolve the conflicted files manually |

---

### 3. Creating a Conflict (Hands-on)
```bash
mkdir git-merge-test && cd git-merge-test
git init .
echo "this is some content to mess with" > merge.txt
git add merge.txt
git commit -am "we are commiting the inital content"

# Branch overwrites the file
git checkout -b new_branch_to_merge_later
echo "totally different content to merge later" > merge.txt
git commit -am "edited the content of merge.txt to cause a conflict"

# main appends to the same file
git checkout main
echo "content to append" >> merge.txt
git commit -am "appended content to merge.txt"

git merge new_branch_to_merge_later
# Auto-merging merge.txt
# CONFLICT (content): Merge conflict in merge.txt
# Automatic merge failed; fix conflicts and then commit the result.
```
💥 **BOOM, a conflict.**

---

### 4. Identifying Conflicts
```bash
$ git status
On branch main
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)
Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both modified:   merge.txt
```
```bash
$ cat merge.txt
<<<<<<< HEAD
this is some content to mess with
content to append
=======
totally different content to merge later
>>>>>>> new_branch_to_merge_later
```

| "Conflict divider" | Meaning |
|---|---|
| `<<<<<<< HEAD` | Start of the **current branch's** version (`main`, where HEAD points) |
| `=======` | The **center** of the conflict |
| `>>>>>>> new_branch_to_merge_later` | End of the **merging branch's** version |

---

### 5. Resolving on the Command Line
1. **Edit** the file: keep what you want and **remove all the dividers**.
```
this is some content to mess with
content to append
totally different content to merge later
```
2. **Stage** it: `git add merge.txt`
3. **Commit** to finish the merge:
```bash
git commit -m "merged and resolved the conflict in merge.txt"
```

---

### 6. Helpful Commands

| Situation | Command | Purpose |
|---|---|---|
| **General** | `git status` | Find conflicted files |
| | `git log --merge` | List the commits that **conflict** between the merging branches |
| | `git diff` | See differences between states and predict or prevent conflicts |
| **Merge fails to start** | `git checkout` | Undo changes to files or switch branches |
| | `git reset --mixed` | Undo changes in the working directory/staging area |
| **Conflict during merge** | `git merge --abort` | **Exit the merge** and return to the pre-merge state |
| | `git reset` | Reset conflicted files to a known good state |

* Many **third-party merge tools** also make resolving conflicts easier (see `merge.tool` in the git config notes).

---

### Summary Takeaway
Merge conflicts happen when **two branches change the same lines** or one **deletes a file** the other edited. They're normal in team work, and Git handles most merges automatically. When it can't, find the files with `git status`, fix the **`<<<<<<<` / `=======` / `>>>>>>>`** sections, then `git add` and `git commit`. If things go wrong, **`git merge --abort`** takes you back to where you started.
