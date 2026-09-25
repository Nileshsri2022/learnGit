Here is a clear breakdown of the key points about **`git config`** from the text:

> Source: https://www.atlassian.com/git/tutorials/setting-up-a-repository/git-config

---

### 1. What Is `git config`?
* A convenience command for setting Git configuration values at the **local, global, or system** level.
* Each level is a plain **text file**. Running `git config` just **edits that file** for you.
* Covers user info, editor, merge tool, colors, aliases, whitespace rules, and more.

---

### 2. Reading & Writing Values
* Config names are **dot-delimited**: `section.key` (e.g. `user.email`).

```bash
git config user.email                                   # READ: show the current value
git config --global user.email "your_email@example.com" # WRITE: set it for your OS user
```

---

### 3. Configuration Levels & Files

| Level | Flag | Applies to | File |
|---|---|---|---|
| **Local** | `--local` *(default)* | The current repository | `.git/config` |
| **Global** | `--global` | The current OS **user** | `~/.gitconfig` (Unix) · `C:\Users\<username>\.gitconfig` (Windows) |
| **System** | `--system` | **All users and repos** on the machine | `$(prefix)/etc/gitconfig` (Unix) · `C:\ProgramData\Git\config` (Windows Vista+) · `C:\Documents and Settings\All Users\Application Data\Git\config` (XP) |

* **Priority: local → global → system.** Git looks at the local level first and "bubbles up", so local values override global ones, and global overrides system.

---

### 4. Setting the Editor — `core.editor`
Many commands (e.g. `git commit` without `-m`) open an editor.

| Editor | Command |
|---|---|
| Atom | `git config --global core.editor "atom --wait"` |
| emacs | `git config --global core.editor "emacs"` |
| nano | `git config --global core.editor "nano -w"` |
| vim | `git config --global core.editor "vim"` |
| Sublime Text (Mac) | `git config --global core.editor "subl -n -w"` |
| Sublime Text (Win, 32-bit) | `git config --global core.editor "'c:/program files (x86)/sublime text 3/sublimetext.exe' -w"` |
| Sublime Text (Win, 64-bit) | `git config --global core.editor "'c:/program files/sublime text 3/sublimetext.exe' -w"` |
| TextMate | `git config --global core.editor "mate -w"` |
| *VS Code (extra, not in the article)* | `git config --global core.editor "code --wait"` |

---

### 5. Merge Tools
* On a merge conflict, Git can launch a **merge tool**. The default is a minimal built-in diff viewer, and many third-party tools exist.
```bash
git config --global merge.tool kdiff3
```

---

### 6. Colored Output

#### `color.ui` — the master switch
| Value | Behavior |
|---|---|
| `auto` *(default)* | Color in the terminal. **No** color codes when output is redirected to a file or pipe |
| `always` | Color codes **even** in files/pipes (can break programs that don't expect them) |
| `false` | Disable all colored output |

```bash
git config --global color.ui false
```

#### Color values
* Named colors: `normal`, `black`, `red`, `green`, `yellow`, `blue`, `magenta`, `cyan`, `white`, plus **hex** (`#ff0000`) or **ANSI 256** values if your terminal supports them.
* Granular settings also accept `false`, `auto`, or `always`.

#### Granular color settings
| Setting | Colors… | Slots |
|---|---|---|
| `color.branch[.<slot>]` | `git branch` output | `current`, `local`, `remote`, `upstream`, `plain` |
| `color.diff[.<slot>]` | `git diff`, `git log`, `git show` | `context`/`plain`, `meta`, `frag` (hunk header), `old` (removed), `new` (added), `commit`, `whitespace` |
| `color.decorate.<slot>` | `git log --decorate` | `branch`, `remoteBranch`, `tag`, `stash`, `HEAD` |
| `color.grep[.<slot>]` | `git grep` | `context`, `filename`, `function`, `linenumber`, `match`, `matchContext`, `matchSelected`, `selected`, `separator` |
| `color.interactive[.<slot>]` | Interactive prompts (`git add -i`, `git clean -i`) | `prompt`, `header`, `help`, `error` |
| `color.pager` | Color when a pager is used | — |
| `color.showBranch` | `git show-branch` | — |
| `color.status[.<slot>]` | `git status` | `header`, `added`/`updated`, `changed`, `untracked`, `branch`, `nobranch`, `unmerged` |

---

### 7. Aliases
* **Custom shortcuts** for frequently used commands, stored in Git config files.
```bash
git config --global alias.ci commit            # git ci  → git commit
git config --global alias.amend "ci --amend"   # aliases can build on other aliases
```

---

### 8. Formatting & Whitespace (`core.whitespace`)
Whitespace problems are highlighted in `git diff` using `color.diff.whitespace`.

| Feature | Default | Highlights |
|---|---|---|
| `blank-at-eol` | ✅ On | Trailing whitespace at line ends |
| `space-before-tab` | ✅ On | A space before a tab in the indent |
| `blank-at-eof` | ✅ On | Blank lines at the end of a file |
| `indent-with-non-tab` | ❌ Off | Lines indented with spaces instead of tabs |
| `tab-in-indent` | ❌ Off | A tab used in the initial indent (flagged as an error) |
| `trailing-space` | ❌ Off | Shorthand for `blank-at-eol` + `blank-at-eof` |
| `cr-at-eol` | ❌ Off | Carriage return at line ends |
| `tabwidth=<n>` | 8 | How many columns a tab takes (1–63) |

---

### Summary Takeaway
`git config` is a **shortcut for editing Git's config files**. Values are resolved **local → global → system**. Use it to set your **identity** (`user.name`, `user.email`), your **editor** (`core.editor`), a **merge tool**, **colors** (`color.*`), **aliases** (`alias.*`), and **whitespace rules**, and build a Git setup that fits the way you work.
