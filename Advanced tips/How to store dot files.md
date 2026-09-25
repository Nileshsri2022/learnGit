Here is a clear breakdown of the key points about **Storing dotfiles in a bare Git repository** from the text:

> Source: https://www.atlassian.com/git/tutorials/dotfiles (by Nicola Paolucci)

---

### 1. What Are Dotfiles & Why Version Them?
* **Dotfiles** are configuration files in your home folder whose names start with a dot — e.g. `.bashrc`, `.vimrc`, `.gitconfig`, `.zshrc`.
* Tracking them in Git lets you **back them up**, **see their history**, and **replicate your setup** on any new machine.
* The author admits the title ("best way") is slightly **hyperbolic** — other solutions exist — but this technique (from Hacker News user **StreakyCobra**) is very **elegant**.

---

### 2. Why This Technique?
Only prerequisite: **Git**. Benefits:
* ✅ **No extra tooling**
* ✅ **No symlinks**
* ✅ Files are tracked in **version control**
* ✅ **Different branches for different computers**
* ✅ Easy to **replicate your configuration** on a new install

#### The core idea
* Store a **bare Git repository** in a "side" folder (e.g. `$HOME/.cfg`).
* Use a special **alias** so Git commands run against **that** repo with **`$HOME` as the work tree** — instead of a normal `.git` folder in `$HOME`, which would **interfere with every other Git repo** under your home directory.

---

### 3. Starting from Scratch

```bash
git init --bare $HOME/.cfg
alias config='/usr/bin/git --git-dir=$HOME/.cfg/ --work-tree=$HOME'
config config --local status.showUntrackedFiles no
echo "alias config='/usr/bin/git --git-dir=$HOME/.cfg/ --work-tree=$HOME'" >> $HOME/.bashrc
```

| Line | What it does |
|---|---|
| 1 | Creates `~/.cfg` — a **bare repo** that tracks your files |
| 2 | Creates the **`config` alias** — use it instead of `git` for your dotfiles repo |
| 3 | Hides files you're **not explicitly tracking**, so `config status` isn't flooded with every file in `$HOME` |
| 4 | Saves the alias in `.bashrc` so it's available in future shells |

#### Daily usage — just replace `git` with `config`
```bash
config status
config add .vimrc
config commit -m "Add vimrc"
config add .bashrc
config commit -m "Add bashrc"
config push
```
* (Add a remote first, e.g. `config remote add origin <repo-url>`, to be able to push.)

---

### 4. Installing Your Dotfiles on a New System (or Migrating)

**Before you start**, make sure your dotfiles repo:
* Has the **alias committed** in your `.bashrc`/`.zshrc`.
* **Ignores** the folder you'll clone into (avoids weird recursion):
```bash
echo ".cfg" >> .gitignore
```

**Steps on the new machine:**
```bash
# 1. Clone as a BARE repo into a dot folder
git clone --bare <git-repo-url> $HOME/.cfg

# 2. Define the alias in the current shell
alias config='/usr/bin/git --git-dir=$HOME/.cfg/ --work-tree=$HOME'

# 3. Check out the files into $HOME
config checkout
```

#### If checkout fails
```
error: The following untracked working tree files would be overwritten by checkout:
    .bashrc
    .gitignore
Please move or remove them before you can switch branches.
Aborting
```
* Your `$HOME` already has **stock config files**. **Back them up** (or delete them if you don't care). A shortcut to move them all to a backup folder:
```bash
mkdir -p .config-backup && \
config checkout 2>&1 | egrep "\s+\." | awk {'print $1'} | \
xargs -I{} mv {} .config-backup/{}
```
```bash
# 4. Re-run the checkout
config checkout

# 5. Hide untracked files for this repo
config config --local status.showUntrackedFiles no
```
* ✅ Done — manage your dotfiles with `config add/commit/push` from now on.

---

### 5. Automating Setup with a Script
The author's complete install script (tested on fresh **Alpine Linux** containers):
```bash
git clone --bare https://bitbucket.org/durdn/cfg.git $HOME/.cfg
function config {
   /usr/bin/git --git-dir=$HOME/.cfg/ --work-tree=$HOME $@
}
mkdir -p .config-backup
config checkout
if [ $? = 0 ]; then
  echo "Checked out config.";
else
  echo "Backing up pre-existing dot files.";
  config checkout 2>&1 | egrep "\s+\." | awk {'print $1'} | xargs -I{} mv {} .config-backup/{}
fi;
config checkout
config config status.showUntrackedFiles no
```
* The article also suggests storing such scripts as a snippet and running them via `curl … | /bin/bash`.
  > ⚠️ *Personal caution:* only pipe scripts into your shell from sources **you control and trust** — read them first.

---

### 6. Key Concepts Recap
| Concept | Role in this setup |
|---|---|
| **Bare repository** (`--bare`) | Holds Git data only, no working directory of its own |
| `--git-dir=$HOME/.cfg/` | Tells Git where the repo data lives |
| `--work-tree=$HOME` | Tells Git your home folder is the working directory |
| `status.showUntrackedFiles no` | Keeps `config status` clean |
| **Branches** | Different configs for different machines (e.g. `laptop`, `work`) |

---

### Summary Takeaway
Create a **bare repo** in `~/.cfg` and an alias **`config`** = `git --git-dir=$HOME/.cfg/ --work-tree=$HOME`. Your home folder becomes the working tree without a `.git` folder that would clash with other repos. Hide untracked files, then **`config add/commit/push`** your dotfiles. On a new machine: **clone `--bare`**, define the alias, **`config checkout`** (backing up conflicting files), and your whole environment is restored — **no symlinks, no extra tools**.
