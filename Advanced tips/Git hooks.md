Here is a clear breakdown of the key points about **Git hooks** from the text:

> Source: https://www.atlassian.com/git/tutorials/git-hooks
> *(Python examples below are adapted to Python 3; the original article uses Python 2 `print` syntax.)*

---

### 1. What Are Git Hooks?
* **Scripts that run automatically** whenever a particular event occurs in a Git repository.
* They let you **customize Git's internal behavior** and trigger actions at key points in the development life cycle.
* **Common uses:**
  * Encourage a **commit policy** (e.g. message format)
  * Alter the project environment depending on repo state
  * Implement **continuous integration** workflows
  * …virtually anything — scripts are infinitely customizable.

---

### 2. Installing Hooks
* Hooks live in **`.git/hooks/`**. `git init` fills it with **sample scripts**:
```
applypatch-msg.sample   pre-push.sample
commit-msg.sample       pre-rebase.sample
post-update.sample      prepare-commit-msg.sample
pre-applypatch.sample   update.sample
pre-commit.sample
```
* The **`.sample`** extension disables them. To **install** a hook: **remove `.sample`**, or create a new file with the hook's exact name.
* Hooks must be **executable**:
```bash
chmod +x .git/hooks/prepare-commit-msg
```
* Minimal example — `prepare-commit-msg`:
```sh
#!/bin/sh
echo "# Please include a useful commit message!" > $1
```
* The sample scripts are great references — they document the **parameters** each hook receives.

---

### 3. Any Scripting Language
* Samples are mostly **shell** and **Perl**, but any language works if it's executable. The **shebang** line decides the interpreter.
```python
#!/usr/bin/env python3
import sys
commit_msg_filepath = sys.argv[1]      # instead of $1
with open(commit_msg_filepath, 'w') as f:
    f.write("# Please include a useful commit message!")
```

---

### 4. Scope of Hooks (Important for Teams!)
* Hooks are **local** to a repository and are **NOT copied by `git clone`**, nor under version control.
* Anyone with repo access can **change or delete** them.
* Consequences:
  1. You need a way to keep hooks **up to date** across the team.
  2. You **can't force** developers to follow rules with local hooks — only **encourage** them.
* **Solutions:**
  * Store hooks **in the project directory** (version-controlled) and **symlink/copy** them into `.git/hooks`.
  * Use Git's **template directory** — its contents are copied into `.git` on every `git init` / `git clone`.
* Treat local hooks as a **convenient developer tool**, not a strict policy. For real enforcement → **server-side hooks**.

---

### 5. Local Hooks

| Hook | When it runs | Args | Can abort? | Typical use |
|---|---|---|---|---|
| `pre-commit` | Before asking for a message / creating the commit | None | ✅ non-zero exit | Run tests, lint, whitespace checks |
| `prepare-commit-msg` | After `pre-commit`, to pre-fill the editor | 1–3 | ✅ | Auto-insert issue number, tweak merge/squash messages |
| `commit-msg` | After the user writes the message | 1 (msg file) | ✅ | Validate message against team standards |
| `post-commit` | Right after the commit is created | None | ❌ | Notifications |
| `post-checkout` | After a successful `git checkout` | 3 | ❌ | Clean generated files, set up per-branch env |
| `pre-rebase` | Before `git rebase` changes anything | 2 | ✅ | Prevent dangerous rebases |

* **`pre-` hooks** can alter/abort the action; **`post-` hooks** are for **notifications only**.

#### a) `pre-commit` — block commits with whitespace errors
```sh
#!/bin/sh
# Compare against HEAD, or an empty tree for the very first commit
if git rev-parse --verify HEAD >/dev/null 2>&1
then
    against=HEAD
else
    against=4b825dc642cb6eb9a060e54bf8d69288fbee4904   # magic ID of the empty tree
fi

if ! git diff-index --check --cached $against
then
    echo "pre-commit: Aborting commit due to whitespace errors"
    exit 1
else
    echo "pre-commit: No whitespace errors :)"
    exit 0
fi
```
* `git rev-parse --verify HEAD` checks if `HEAD` exists; `git diff-index --check --cached` compares the index to a commit and flags whitespace errors.
* You can do anything here: run a test suite, check style with a linter, etc.

#### b) `prepare-commit-msg` — auto-add the issue number from the branch name
Arguments:
1. Temp file containing the message (edit it in place)
2. Commit type: `message` (`-m`/`-F`), `template` (`-t`), `merge`, or `squash`
3. SHA-1 of the relevant commit (only with `-c`, `-C`, or `--amend`)

```python
#!/usr/bin/env python3
import sys, re
from subprocess import check_output

commit_msg_filepath = sys.argv[1]
branch = check_output(['git', 'symbolic-ref', '--short', 'HEAD']).decode().strip()

if branch.startswith('issue-'):
    issue_number = re.match('issue-(.*)', branch).group(1)
    with open(commit_msg_filepath, 'r+') as f:
        content = f.read()
        f.seek(0, 0)
        f.write("ISSUE-%s %s" % (issue_number, content))
```
* On branch `issue-224` the message starts with `ISSUE-224`.
* ⚠️ It also runs with `git commit -m` (user can't edit the result) — check if argument 2 == `message`.
* Users can still edit the message → a **convenience**, not enforcement. For enforcement use `commit-msg`.

#### c) `commit-msg` — enforce the message format
```python
#!/usr/bin/env python3
import sys, re
from subprocess import check_output

commit_msg_filepath = sys.argv[1]
branch = check_output(['git', 'symbolic-ref', '--short', 'HEAD']).decode().strip()

if branch.startswith('issue-'):
    required = "ISSUE-%s" % re.match('issue-(.*)', branch).group(1)
    with open(commit_msg_filepath) as f:
        if not f.read().startswith(required):
            print("commit-msg: ERROR! The commit message must start with '%s'" % required)
            sys.exit(1)
```
* Keep it focused on **checking the message**; notifications belong in `post-commit`.

#### d) `post-commit` — notifications
* Exit status doesn't matter. Get the new commit with `git rev-parse HEAD` or `git log -1 HEAD`.
* Example idea: email your boss the `git log -1 --stat HEAD` output after every commit (probably not a great idea 😄).
* For CI, prefer the **server-side `post-receive`** hook (runs for **every developer's** push).

#### e) `post-checkout` — clean up after switching branches
Arguments: previous `HEAD` ref, new `HEAD` ref, and a flag (`1` = branch checkout, `0` = file checkout).
```python
#!/usr/bin/env python3
import sys, os

if sys.argv[3] == "0":
    sys.exit(0)   # file checkout, nothing to do

for root, dirs, files in os.walk('.'):          # hooks run from the repo root
    for filename in files:
        if os.path.splitext(filename)[1] == '.pyc':
            os.unlink(os.path.join(root, filename))
```
* Solves stale **`.pyc`** files confusing the Python interpreter after switching branches.
* Also useful to build things only on certain branches (e.g. binaries needed only on a `plugins` branch).

#### f) `pre-rebase` — prevent disasters
Arguments: the upstream branch, and the branch being rebased (empty if current branch).
```sh
#!/bin/sh
echo "pre-rebase: Rebasing is dangerous. Don't do it."
exit 1
```
```
pre-rebase: Rebasing is dangerous. Don't do it.
The pre-rebase hook refused to rebase.
```
* The bundled `pre-rebase.sample` is smarter: it blocks rebasing topic branches **already merged** into `next`.

---

### 6. Server-Side Hooks
* Live in **server-side repos** (central repo or a developer's public repo) and react to stages of **`git push`**.
* On the official repo they can **enforce policy** by **rejecting** commits.
* Their output is **piped to the pusher's console** — but they block the terminal until done, so avoid long-running tasks.

| Hook | When | Input | Scope | Use |
|---|---|---|---|---|
| `pre-receive` | Before **any** ref is updated | **stdin**: `<old-value> <new-value> <ref-name>` per ref | Non-zero exit rejects **all** pushed refs | Enforce policy: who pushes, message format, content |
| `update` | After `pre-receive`, **once per ref** | Args: ref name, old SHA, new SHA | Reject **individual** refs | Case-by-case branch rules |
| `post-receive` | After a **successful** push | Same stdin as `pre-receive` | Can't reject | Email developers, **trigger CI** |

#### `pre-receive` example
```python
#!/usr/bin/env python3
import fileinput
for line in fileinput.input():
    print("pre-receive: Trying to push ref: %s" % line)
# sys.exit(1)  # uncomment to abort the push
```
```
b6b36c697eb2d24302f89aa22d9170dfe609855b 85baa88c22b52ddd24d71f05db31f4e46d579095 refs/heads/main
```
* Common checks: reject changes involving an **upstream rebase**, prevent **non-fast-forward** merges, verify user **permissions**.
* You can't stop developers from making malformed commits locally, but you **can keep them out of the official codebase**.

#### `update` example
```python
#!/usr/bin/env python3
import sys
branch, old_commit, new_commit = sys.argv[1], sys.argv[2], sys.argv[3]
print("Moving '%s' from %s to %s" % (branch, old_commit, new_commit))
# sys.exit(1)  # abort pushing only this branch
```

#### `post-receive`
* Better place for notifications than `post-commit`, because changes are now on a **public server**.

---

### Summary Takeaway
Git hooks are **ordinary executable scripts in `.git/hooks`** that let you plug into the whole development life cycle. **Local hooks** (`pre-commit`, `prepare-commit-msg`, `commit-msg`, `post-commit`, `post-checkout`, `pre-rebase`) help developers follow guidelines but **can't enforce** them (they're not cloned and can be edited). **Server-side hooks** (`pre-receive`, `update`, `post-receive`) run on push and **can enforce policy** and **trigger CI**. With a little scripting, you can automate almost anything.
