Here is a clear breakdown of the key points about **Syncing with `git remote`** from the text:

> Source: https://www.atlassian.com/git/tutorials/syncing

---

### 1. Centralized (SVN) vs. Distributed (Git) Collaboration
| | SVN | Git |
|---|---|---|
| Hub | One **central repository** | Every developer has a **full repository** (own history + branches) |
| What gets shared | Single **changesets** between working copy ↔ server | Series of **commits / entire branches** between repositories |

* `git remote` is one piece of the syncing system. The remotes it registers are used by **`git fetch`**, **`git push`**, and **`git pull`**.

---

### 2. What Is a Remote?
* `git remote` lets you **create, view, and delete connections** to other repositories.
* Remotes are **bookmarks**, not live links — convenient **short names** for long, inconvenient URLs.
  * e.g. `origin` → the central repo, `john` → a teammate's repo.
* Remote entries are stored in the repo's **`./.git/config`** file; `git remote` is a helper for editing it (you could edit the file by hand with the same result).
* Git gives each developer an **isolated environment** — nothing syncs automatically; you must explicitly **pull** or **push**.

---

### 3. Viewing Remotes

```bash
git remote      # list remote names
git remote -v   # list names + fetch/push URLs ("verbose")
```

```
$ git remote -v
origin            git@bitbucket.com:origin_user/reponame.git (fetch)
origin            git@bitbucket.com:origin_user/reponame.git (push)
upstream          https://bitbucket.com/upstream_user/reponame.git (fetch)
upstream          https://bitbucket.com/upstream_user/reponame.git (push)
other_users_repo  https://bitbucket.com/other_users_repo/reponame (fetch)
other_users_repo  https://bitbucket.com/other_users_repo/reponame (push)
```

---

### 4. Creating & Modifying Remotes

| Command | Effect |
|---|---|
| `git remote add <name> <url>` | Create a connection; use `<name>` as a shortcut for `<url>` |
| `git remote rm <name>` (or `remove`) | Delete the connection (and its remote-tracking branches & settings) |
| `git remote rename <old> <new>` | Rename it (remote-tracking branches & settings updated too) |

#### What `git remote add` writes to `.git/config`
```ini
[remote "remote_test"]
    url = https://bitbucket.com/upstream_user/reponame.git
    fetch = +refs/heads/*:refs/remotes/remote_test/*
```
* After `git remote rm remote_test`, that block is gone.

---

### 5. The `origin` Remote
* `git clone` **automatically** creates a remote named **`origin`** pointing back to the cloned repo.
* That's why most Git projects call their central repository **origin**.

---

### 6. Repository URLs
| Protocol | Example | Access |
|---|---|---|
| **HTTP(S)** | `http://host/path/to/repo.git` | Easy **anonymous, read-only** access (pushing generally not allowed) |
| **SSH** | `ssh://user@host/path/to/repo.git` | **Authenticated read-write** access (needs an SSH account on the host) |

* Hosting services like Bitbucket provide these URLs for you.

---

### 7. All `git remote` Subcommands

| Subcommand | What it does | Options |
|---|---|---|
| `add <name> <url>` | Add a remote record | `-f` → fetch immediately; `--tags` → fetch & import all tags |
| `rename <old> <new>` | Rename a remote | — |
| `remove` / `rm <name>` | Remove a remote | — |
| `get-url <name>` | Print a remote's URL(s) | `--push` → push URLs; `--all` → all URLs |
| `show <name>` | High-level info about the remote | — |
| `prune <name>` | Delete local remote-tracking branches that no longer exist on the remote | `--dry-run` → list only, don't delete |

---

### 8. Examples

#### Connect to a teammate's repo
```bash
git remote add john http://dev.example.com/john.git
```
* Lets small teams collaborate **outside** the central repository.

#### Inspect a remote
```
$ git remote show upstream
* remote upstream
   Fetch URL: https://bitbucket.com/upstream_user/reponame.git
   Push URL: https://bitbucket.com/upstream_user/reponame.git
   HEAD branch: main
   Remote branches:
      main tracked
      simd-deprecated tracked
      tutorial tracked
   Local ref configured for 'git push':
      main pushes to main (fast-forwardable)
```

---

### 9. Using Remotes with Other Commands
* **Read** from a remote: `git fetch <remote>` or `git pull <remote>`.
* **Write** to a remote:
```bash
git push <remote-name> <branch-name>   # upload local branch state to the remote
```

---

### Summary Takeaway
`git remote` manages **named bookmarks** to other repositories (stored in `.git/config`). `git clone` creates **`origin`** for you; use **`add`, `rename`, `rm`, `show`, `get-url`, and `prune`** to manage connections, then pass remote names to **`fetch`, `pull`, and `push`** to actually sync commits. Prefer **SSH** for read-write, **HTTP** for anonymous read-only access.
