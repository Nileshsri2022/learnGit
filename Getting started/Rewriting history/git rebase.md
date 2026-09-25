Here is a clear breakdown of the key points about **`git rebase`** from the text:

> Source: https://www.atlassian.com/git/tutorials/rewriting-history/git-rebase

---

### 1. What Is `git rebase`?
* One of **two** Git tools for integrating changes from one branch into another. The other is **`git merge`**.

| `git merge` | `git rebase` |
|---|---|
| Always a **forward-moving** record | Powerful **history-rewriting** features |

* **Rebasing = moving or combining a sequence of commits onto a new base commit.**
* It makes it look as if you'd **created your branch from a different commit**.
* ⚠️ Internally Git creates **brand-new commits** and applies them to the new base. The branch looks the same, but it's made of **entirely new commits** (new SHAs).

```
Before:              A───B───C  feature
                    /
          D───E───F───G  main

git checkout feature && git rebase main

After:                       A'──B'──C'  feature
                            /
          D───E───F───G  main
```

---

### 2. Why Rebase? — A Linear, Clean History
* Get the latest `main` updates into your feature branch **as if you'd been working off the latest `main` all along**, which later allows a **clean merge** back.
* **Real-world benefit:** hunting a regression.
  1. A bug appears in `main`.
  2. With a clean history, `git log` is easy to reason about.
  3. If that's not enough, `git bisect` has a **refined set of commits** to search.
  4. The bad commit is found quickly.

| Integrate feature by… | Result |
|---|---|
| **Merging directly** | 3-way merge + **merge commit** |
| **Rebasing, then merging** | **Fast-forward** merge + perfectly **linear** history |

* For pulling upstream changes, merge creates a **superfluous merge commit** each time. Rebase says: *"I want to base my changes on what everybody has already done."*

---

### 3. ⚠️ Don't Rebase Public History
* **Never rebase commits that have been pushed to a public repository.** The rebase replaces old commits with new ones, so part of the history seems to **vanish** for everyone else.

---

### 4. Standard vs. Interactive Mode
```bash
git checkout -b feature_branch main
# edit files
git commit -a -m "Adds new feature"
```

| Mode | Command | Behavior |
|---|---|---|
| **Standard** | `git rebase <base>` | Automatically replays your branch's commits on top of `<base>` (commit ID, branch, tag, or relative ref like `HEAD~3`) |
| **Interactive** | `git rebase -i <base>` (`--interactive`) | Opens an editor so you can **alter each commit** as it's replayed. It's "`git commit --amend` on steroids" |

#### Interactive commands
```
pick 2231360 some old commit
pick ee2adc2 Adds new feature

# Commands:
# p, pick   = use commit
# r, reword = use commit, but edit the commit message
# e, edit   = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup  = like "squash", but discard this commit's log message
# x, exec   = run command (the rest of the line) using shell
# d, drop   = remove commit
```
* You can also **reorder** the lines to reorder commits.

| Command | Use |
|---|---|
| `pick` | Keep the commit as is (message and content unchanged) |
| `reword` | Keep it, but change the message |
| `edit` | Stop so you can amend/split it |
| `squash` / `fixup` | Combine into the previous commit (keep / discard the message) |
| `exec` | Run a shell command after a commit (e.g. **run the test suite** to spot regressions) |
| `drop` | Discard the commit |

> 📝 *Extra note (not in the original article):* The article writes these as `git rebase -- d`, `-- p`, `-- x`, but they are **interactive todo-list commands** (`d`, `p`, `x`), not command-line flags. The real CLI flag is `git rebase -x "<cmd>" <base>`, which adds an `exec` line after every commit.

#### Why interactive rebase?
* Commit a **"messy" history** while coding, then **clean it up** before merging: squash trivial commits, drop obsolete ones.
* To everyone else, the feature looks like it was built in **a single series of well-planned commits**.

---

### 5. Configuration Options (`git config`)

| Setting | Effect |
|---|---|
| `rebase.stat` | *(default `false`)* Show a diffstat of what changed since the last rebase |
| `rebase.autoSquash` | Enable `--autosquash` behavior by default |
| `rebase.missingCommitsCheck` | `warn` (warn about removed commits in interactive mode) · `error` (stop the rebase) · `ignore` *(default)* |
| `rebase.instructionFormat` | A `git log` format string for the interactive todo list |

---

### 6. Advanced — `git rebase --onto`
```bash
git rebase --onto <newbase> <oldbase> [<branch>]
```
Example: `featureB` was branched from `featureA`, but it doesn't actually depend on it:
```
o---o---o---o---o  main
     \
      o---o---o---o---o  featureA
           \
            o---o---o  featureB
```
```bash
git rebase --onto main featureA featureB
```
```
                      o---o---o  featureB
                     /
    o---o---o---o---o  main
     \
      o---o---o---o---o  featureA
```
* Takes the commits in `featureB` that **aren't in `featureA`** and replays them onto `main`.

---

### 7. Dangers of Rebasing & How to Handle Them

| Danger | Remedy |
|---|---|
| **More frequent merge conflicts**, especially on long-lived branches that drifted from `main` | Rebase against `main` **often** and commit frequently. Use `git rebase --continue` / `--abort` while resolving |
| **"Lost" commits** after `squash`/`drop` in interactive mode | They're not really gone: use **`git reflog`** to restore them or undo the whole rebase |
| **Rewriting + force-pushing a shared branch** (the real danger) | **Avoid it.** It can overwrite teammates' work when they pull |

---

### 8. Recovering from an Upstream Rebase
* If someone rebased and **force-pushed** a branch you built on, a `git pull` can overwrite your commits based on the old tip.
* Fix: use the **reflog of the remote branch** to find its ref **before** the rebase, then rebase your work with **`--onto`** (new tip, old ref).

---

### Summary Takeaway
`git rebase` **moves your commits onto a new base**, producing new commits and a **clean, linear history** that's easier to read, bisect, and fast-forward merge. Use **standard** mode to update a branch, **interactive** (`-i`) mode to reword, squash, reorder, or drop commits, and **`--onto`** for advanced re-parenting. Resolve conflicts with `--continue`/`--abort`, rescue mistakes with **`git reflog`**, and **never rebase or force-push shared history**.
