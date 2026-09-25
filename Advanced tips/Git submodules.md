Here is a clear breakdown of the key points about **Git submodules** from the text:

> Source: https://www.atlassian.com/git/tutorials/git-submodule

---

### 1. What Is a Git Submodule?
* Lets you keep **one Git repository as a subdirectory of another**.
* A submodule is simply a **reference to another repository at a specific commit (snapshot)**.
* It lets the host repo **incorporate external code and track its version history**.

---

### 2. Ways to Include External Code

| Method | Downside |
|---|---|
| **Copy & paste** the code into your repo | You **lose upstream changes** |
| **Package manager** (npm, Ruby Gems…) | Requires installation and version management **everywhere the code is deployed** |
| **Git submodule** | Points to an **exact commit** of the external repo and tracks it |

* The first two **can't track edits** to the external repository.

---

### 3. How Submodules Behave
* A submodule is a **record in the host repo pointing to a specific commit** in an external repo.
* **Very static:**
  * It tracks **specific commits**, **not** refs or branches.
  * It is **not automatically updated** when the host repo updates.
* Adding one creates a **`.gitmodules`** file that maps each submodule's **URL ↔ local directory** (one entry per submodule).

```ini
[submodule "awesomelibrary"]
    path = awesomelibrary
    url = https://bitbucket.org/jaredw/awesomelibrary
```

---

### 4. When to Use Submodules
They're useful when you need **strict version control over external dependencies**:
* An external component **changes too fast** or **upcoming changes will break its API**, so you **lock it to a specific commit**.
* A component is **rarely updated** and you want to track it as a **vendor dependency**.
* You **delegate part of the project to a third party** and want to integrate their work **at a specific time/release** (works when updates are infrequent).

---

### 5. Common Commands

#### Add a submodule
```bash
mkdir git-submodule-demo && cd git-submodule-demo
git init
git submodule add https://bitbucket.org/jaredw/awesomelibrary   # clones it immediately
git status
#   new file: .gitmodules
#   new file: awesomelibrary
git add .gitmodules awesomelibrary/
git commit -m "added submodule"
#   create mode 160000 awesomelibrary   ← 160000 = a "gitlink" (commit pointer), not a normal folder
```

#### Clone a repo that has submodules
```bash
git clone /url/to/repo/with/submodules
git submodule init
git submodule update
```
> 📝 *Extra note (not in the original article):* Shortcuts: `git clone --recurse-submodules <url>` does all three steps at once. In an existing clone, run `git submodule update --init --recursive`.

#### `git submodule init`
* By default it **copies the mappings from `.gitmodules` into `.git/config`**.
* It also accepts **specific module names**, so you can **activate only the submodules you need** (handy when a repo has many).

| Command | Purpose |
|---|---|
| `git submodule add <url> [path]` | Add a new submodule |
| `git submodule init [name…]` | Register submodules in `.git/config` |
| `git submodule update` | Check out the commit the parent repo expects |

---

### 6. Working Inside a Submodule
* Once initialized, a submodule works **like a standalone repo**, with **its own branches and history**.
```bash
cd awesomelibrary/
git checkout -b new_awesome
echo "new awesome file" > new_awesome.txt
git add new_awesome.txt
git commit -m "added new awesome textfile"

cd ..
git status
#   modified: awesomelibrary (new commits)
```
* The parent only knows there are **new commits**. The details are the submodule's business. The parent's only job is **pinning the submodule to a commit**.
* Update the pin in the parent:
```bash
git add awesomelibrary
git commit -m "update awesomelibrary"
```

---

### 7. ⚠️ The #1 Submodule Mistake: Forgetting to Push
* If you push the **parent** but **not the submodule**, teammates pull a parent that points to a submodule commit **they can't fetch**, which **breaks their local repo**.
* ✅ **Always commit and push both**: first the **submodule**, then the **parent**.

```
1. cd submodule → commit → git push      (publish the submodule commit)
2. cd ..        → git add submodule → commit → git push   (publish the new pin)
```
> 📝 *Extra note (not in the original article):* `git push --recurse-submodules=check` refuses to push the parent if a submodule commit hasn't been pushed. `=on-demand` pushes the submodules for you.

---

### 8. Submodules vs. Subtree (see *Git subtree* notes)
| | Submodule | Subtree |
|---|---|---|
| Stores | A **pointer** to a commit (+ `.gitmodules`) | The **actual code** merged into the repo |
| After clone | Needs `init`/`update` | Code is **there immediately** |
| Learning curve | Higher (easy to forget pushes/updates) | Users don't need to learn anything new |

---

### Summary Takeaway
Git submodules embed another repository as a subdirectory **pinned to an exact commit**, recorded in **`.gitmodules`**. They're great for **strict dependency versioning** (fast-changing APIs, vendor code, third-party deliverables), but they're an **advanced feature** with a learning curve. Use `git submodule add`, `init`, and `update`, treat each submodule as its own repo, and **always push the submodule before the parent**.
