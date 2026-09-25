Here is a clear breakdown of the key points about **`git blame`** from the text:

> Source: https://www.atlassian.com/git/tutorials/inspecting-a-repository/git-blame

---

### 1. What Is `git blame`?
* A versatile **troubleshooting utility** that shows **author metadata for each line** of a file: **who last modified the line, when, and in which commit**.
* Helps answer **what, how, and why** a piece of code was added.
* Often used through a **GUI**: hosting sites like Bitbucket have **blame views**, and most IDEs with Git integration have dynamic blame views. These are commonly referenced in pull request discussions.

---

### 2. How It Works
Example repo with commits from several authors:
```bash
git clone https://kevzettler@bitbucket.org/kevzettler/git-blame-example.git && cd git-blame-example
git log      # see the commit history
```
* `git blame` works on **individual files only**. Without a file path it just prints its help.
```bash
git blame README.md
```
```
82496ea3 (kevzettler     2018-02-28 13:37:02 -0800  1) # Git Blame example
89feb84d (Albert So      2018-03-01 00:54:03 +0000  3) This repository is an example of a project with multiple contributors making commits.
eb06faed (Juni Mukherjee 2018-03-01 19:53:23 +0000  9) Annotates each line in the given file with information from the revision...
548dabed (Juni Mukherjee 2018-03-01 19:55:15 +0000 11) Creating a line to support documentation needs for git blame.
```

#### Reading one line
| Commit ID | Author | Timestamp | Line # | Line content |
|---|---|---|---|---|
| `89feb84d` | Albert So | 2018-03-01 00:54:03 +0000 | 3 | This repository is an example of a project with multiple contributors making commits. |

* **Author** is usually the most valuable column, followed by the **timestamp**. The **line content** shows *what* changed.

---

### 3. Common Options

| Option | Effect |
|---|---|
| `-L 1,5` | Only show lines **1 to 5** |
| `-e` | Show author **email** instead of username |
| `-w` | **Ignore whitespace** changes (e.g. tabs → spaces wouldn't hide the real author) |
| `-M` | Detect lines **moved/copied within the same file** → shows the **original** author |
| `-C` | Detect lines **moved/copied from other files** → shows the **original** author |

```bash
git blame -L 1,5 README.md
git blame -e README.md
git blame -w README.md
git blame -M README.md
git blame -C README.md
```

---

### 4. `git blame` vs. `git log -S`
* `git blame` shows the **last** person who touched a line. Finding when a line was **originally added** is awkward (you'd need `-w -C -M`).
* **Easier:** use `git log -S"<code>"` (the "pickaxe") to list **every commit that added or removed** that text:
```bash
$ git log -S"CSS3D and WebGL renderers." --pretty=format:'%h %an %ad %s'
e339d3c85 Mario Schuettel Tue Oct 13 16:51:06 2015 +0200 reverted README.md to original content
509c2cc35 Daniel          Tue Sep 8 13:56:14 2015 +0200 Updated README
cb20237cc Mr.doob         Mon Dec 31 00:22:36 2012 +0100 Removed DOMRenderer. Now with the CSS3DRenderer it has become irrelevant.
```
* This text was touched **3 times by 3 authors**. It was **originally added** in `cb20237cc` by Mr.doob (the oldest entry).
* `--pretty=format:'%h %an %ad %s'` = short hash, author name, date, subject.

| Question | Tool |
|---|---|
| Who **last** changed this line? | `git blame` |
| When was this code **first introduced**, and every time it changed? | `git log -S"<code>"` |

---

### Summary Takeaway
`git blame` annotates a file **line by line** with the **commit, author, and date** of the last change. Narrow it with `-L`, see emails with `-e`, and cut through noise with **`-w`, `-M`, `-C`**. For the full history of a snippet (including when it was first added), combine it with **`git log -S`**. GUI blame views on Bitbucket/GitHub and in IDEs make this even easier.
