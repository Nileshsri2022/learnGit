Here is a clear breakdown of the key points about **Git aliases** from the text:

> Source: https://www.atlassian.com/git/tutorials/git-alias
> *(Shown in the site's sidebar under "Setting up a repository".)*

---

### 1. What Is an Alias?
* **Alias = shortcut.** A short command that maps to a longer one (a common pattern, e.g. in the `bash` shell).
* Saves keystrokes and makes workflows faster. Example: map `git co` → `git checkout`.

---

### 2. There Is No `git alias` Command
* Aliases are created with **`git config`** and stored in Git's **config files**.
* Like other config values, they can be **local** (one repo) or **global** (your user).

```bash
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
```
* The original commands **still work** (`git checkout` is still available alongside `git co`).
* With `--global`, they're saved in `~/.gitconfig`:
```ini
[alias]
    co = checkout
    br = branch
    ci = commit
    st = status
```

---

### 3. Creating New "Commands" with Aliases
Aliases can wrap commands **with options**, which gives you new, easy-to-remember commands:
```bash
git config --global alias.unstage 'reset HEAD --'
```
These two are now equivalent:
```bash
git unstage fileA
git reset HEAD -- fileA
```

---

### 4. Two Ways to Create Aliases

| Method | How |
|---|---|
| **Edit config files directly** | Add to the `[alias]` section of `$HOME/.gitconfig` (global) or `.git/config` (local) |
| **Use `git config`** | `git config --global alias.co checkout` writes the same thing to the file for you |

---

### 5. Handy Alias Ideas (from this and related pages)
| Alias | Expands to |
|---|---|
| `git st` | `git status` |
| `git co` | `git checkout` |
| `git br` | `git branch` |
| `git ci` | `git commit` |
| `git up` | `git rebase` |
| `git unstage <file>` | `git reset HEAD -- <file>` |
| `git amend` | `git ci --amend` (an alias built from another alias) |

---

### Summary Takeaway
Git aliases are **shortcuts** for frequently used commands. They can also wrap a command plus its options into a **new faux Git command**. There's no `git alias` command: you create them with **`git config [--global] alias.<name> <command>`** or by editing the **`[alias]`** section of your config file.
