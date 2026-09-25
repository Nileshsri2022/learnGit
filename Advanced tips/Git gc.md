Here is a clear breakdown of the key points about **`git gc`** from the text:

> Source: https://www.atlassian.com/git/tutorials/git-gc

---

### 1. What Is `git gc`?
* A **repository maintenance** command. **gc = garbage collection**: it tells Git to *"clean up the mess it's made"* in the current repo.
* The term comes from interpreted languages, where GC recovers memory the program can no longer reach.

#### Git's two kinds of "garbage"
| Garbage | Where it comes from | What `git gc` does |
|---|---|---|
| **Orphaned / unreachable commits** | History-altering commands like `git reset` and `git rebase` | Deletes them **after an expiry period** |
| **Loose, uncompressed objects** | Normal day-to-day work | **Compresses** similar objects into **packs** (like zip files) in `.git/objects/pack` |

* Git **doesn't delete detached commits immediately**, so it can preserve history and avoid data loss. Until then they can still be checked out, cherry-picked, and viewed in `git log`.

---

### 2. Configuration Checked Before Running

| Setting | Default | Controls |
|---|---|---|
| `gc.reflogExpire` | **90 days** | How long **reachable** reflog entries are kept |
| `gc.reflogExpireUnreachable` | **30 days** | How long **unreachable** reflog entries are kept |
| `gc.aggressiveWindow` | **250** | Time spent in delta compression with `--aggressive` |
| `gc.aggressiveDepth` | **50** | Delta-chain depth `git repack` uses with `--aggressive` |
| `gc.pruneExpire` | **"2 weeks ago"** | How long unreachable **objects** are kept before pruning |
| `gc.worktreePruneExpire` | **"3 months ago"** | How long a **stale working tree** is kept before deletion |

```bash
git config gc.reflogExpire "60 days"   # example: change a threshold
```

---

### 3. What It Actually Runs
* `git gc` is a **bundle of internal subcommands**, including **`git prune`**, **`git repack`**, **`git pack`**, and **`git rerere`** (gc).
* Together they find objects **outside the configured thresholds** and then **compress** or **prune** them.

```
git gc
 ├── reflog expire   (drop old reflog entries)
 ├── repack / pack   (compress loose objects into packs)
 ├── prune           (delete unreachable objects older than gc.pruneExpire)
 └── rerere gc       (clean old recorded conflict resolutions)
```

---

### 4. Best Practices & FAQ

#### It already runs automatically
* Frequently used commands trigger it: **`git pull`, `git merge`, `git rebase`, `git commit`**.
* How often to run it **manually** depends on activity: a **solo** repo needs it far less often than a busy **multi-user** repo.

#### `git gc` vs. `git prune`
| `git gc` | `git prune` |
|---|---|
| **Parent** command | **Child** command |
| Does full housekeeping | Only removes objects deemed **unreachable** by the gc config |
| Triggers `git prune` internally | Rarely run on its own |

#### `git gc --aggressive`
* Spends **more time optimizing**: slower, but **saves more disk space**.
* Its effects are **persistent**, so you only need it **after a large volume of changes**.

#### `git gc --auto`
* First checks **whether housekeeping is needed** (thresholds for loose objects and pack counts). If not, it **exits without doing anything**.
* Many Git commands run `git gc --auto` **implicitly** after they finish.
> 📝 *Extra note (not in the original article):* The thresholds are `gc.auto` (default **6700** loose objects) and `gc.autoPackLimit` (default **50** packs). Set `gc.auto 0` to disable automatic gc.

---

### 5. Getting Started
```bash
git gc                 # manual run; prints what it did
git gc --aggressive    # deeper optimization after big changes
git gc --auto          # only if needed
git count-objects -vH  # (extra) see loose/packed object counts and sizes
```

---

### Summary Takeaway
`git gc` is Git's **housekeeper**. It **expires old reflog entries**, **prunes unreachable objects** (after grace periods like **30/90 days** for reflogs and **2 weeks** for objects), and **packs loose objects** to save space by running `prune`, `repack`, and friends. You're probably already using it through **`git gc --auto`** after common commands. Run it manually on busy repos, and use **`--aggressive`** only after large changes.
