Here is a clear breakdown of the key points about **`git tag`** from the text:

> Source: https://www.atlassian.com/git/tutorials/inspecting-a-repository/git-tag

---

### 1. What Is a Tag?
* A **ref that points to a specific point in history**, usually used to mark a **release version** (e.g. `v1.0.1`).
* **A tag is like a branch that doesn't change.** Once created, it has **no further commit history**.

---

### 2. Two Types of Tags

| | **Annotated** | **Lightweight** |
|---|---|---|
| Stored as | A **full object** in the Git database | Just a **name + pointer** to a commit (a "bookmark") |
| Metadata | Tagger **name, email, date**, plus a **message** | None |
| GPG signing | ✅ Can be signed & verified | ❌ |
| Best for | **Public** releases | **Private**/temporary quick links |
| Create with | `git tag -a v1.4` | `git tag v1.4-lw` (no `-a`, `-s`, or `-m`) |

* **Best practice:** prefer **annotated** tags so you keep all the metadata.

```bash
git tag -a v1.4                     # opens your editor for the tag message
git tag -a v1.4 -m "my version 1.4" # inline message (like git commit -m)
git tag v1.4-lw                     # lightweight tag
```

---

### 3. Listing Tags
```bash
git tag              # list all tags
git tag -l '*-rc*'   # filter with a wildcard, e.g. release candidates
```

---

### 4. Tagging Old Commits
* By default, `git tag` tags the commit **`HEAD`** points to. Pass a commit SHA to tag an older one:
```bash
git log --pretty=oneline
# 15027957951b64cf874c3557a0f3547bd83b3ff6 Merge branch 'feature'
# a6b4c97498bd301d84096da251c98a07c7723e65 add update method for thing

git tag -a v1.2 15027957951b64cf874c3557a0f3547bd83b3ff6
```

---

### 5. Re-tagging / Replacing a Tag
* Reusing an existing name gives an error:
```
fatal: tag 'v0.4' already exists
```
* Use **`-f`** (force) to move it. This **overwrites** the existing tag:
```bash
git tag -a -f v1.4 15027957951b64cf874c3557a0f3547bd83b3ff6
```

---

### 6. Sharing Tags — Pushing to a Remote
* **`git push` does NOT push tags by default.** Push them explicitly:
```bash
git push origin v1.4     # one tag
git push origin --tags   # all tags
```
* Others get the tags when they **clone or pull**.

---

### 7. Checking Out a Tag
```bash
git checkout v1.4
```
* Puts you in a **detached `HEAD`** state. New commits **won't update the tag**, won't belong to any branch, and are reachable **only by SHA**.
* ✅ **Best practice:** create a **new branch** before making changes (e.g. `git checkout -b fix-from-v1.4 v1.4`).

---

### 8. Deleting Tags
```bash
git tag -d v1
```
> 📝 *Extra note (not in the original article):* `-d` only deletes the tag **locally**. To remove it from the remote as well, run `git push origin --delete v1`.

---

### Cheat Sheet
| Task | Command |
|---|---|
| Lightweight tag | `git tag <name>` |
| Annotated tag | `git tag -a <name> -m "msg"` |
| Tag an old commit | `git tag -a <name> <sha>` |
| List / filter | `git tag` / `git tag -l '<pattern>'` |
| Replace a tag | `git tag -a -f <name> <sha>` |
| Push one / all | `git push origin <name>` / `git push origin --tags` |
| View tagged state | `git checkout <name>` (detached HEAD) |
| Delete (local) | `git tag -d <name>` |

---

### Summary Takeaway
Tags are **fixed snapshots** in history, typically **semantic version numbers** for releases. `git tag` creates, lists, modifies, and deletes them. **Annotated** tags (with tagger info, message, and optional GPG signature) are the better practice for public releases. Remember that tags must be **pushed explicitly**, and checking one out gives you a **detached HEAD**.
