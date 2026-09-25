Here is a clear breakdown of the key points about the **Prepare** phase of migrating from SVN to Git, from the text:

> Source: https://www.atlassian.com/git/tutorials/migrating-prepare
> *(Step 1 of 5: **Prepare** → Convert → Synchronize → Share → Migrate)*

---

### 1. Goal of This Phase
* Set up the **migration lead's local machine** with the tools needed to convert an SVN repo to Git.
* Everything in this phase runs on **the migration lead's computer**, not on the server or team machines.

---

### 2. Download the Migration Scripts
* Download **`svn-migration-scripts.jar`** from Atlassian's Bitbucket (`bitbucket.org/atlassian/svn-migration-scripts/downloads`) into your **home directory**.
* ⚠️ The scripts need a **case-sensitive filesystem**:
  * They **don't work on NTFS** (Windows).
  * **Linux is recommended.**
  * On macOS you'll create a case-sensitive disk image (see below).

---

### 3. Verify Your Tools
```bash
java -jar ~/svn-migration-scripts.jar verify
```
Checks that the required software is installed:

| Requirement | Why |
|---|---|
| **Java Runtime (JRE)** | Runs the `.jar` scripts |
| **Git** | Target VCS |
| **Subversion** | Reads the SVN repo |
| **git-svn** | Bridges SVN ↔ Git during conversion |

* If there's a warning about **version detection**, set the locale and re-run:
```bash
export LANG=C      # *nix
SET LANG=C         # Windows
```
* On **OS X**, it also warns that the filesystem is **case-insensitive**. The next step fixes that.

---

### 4. Create a Case-Sensitive Workspace

| OS | Command |
|---|---|
| **Linux / other case-sensitive OS** | `mkdir ~/GitMigration` |
| **OS X** | `java -jar ~/svn-migration-scripts.jar create-disk-image 5 GitMigration` |

* The `create-disk-image` arguments are:
  * **Size in GB** (make it **bigger than your SVN repo**)
  * **Name** of the image
* The image is **mounted at `~/GitMigration`**.

---

### 5. Build the Authors File
* **SVN stores only usernames** for commits, but **Git needs a full name and email**. You need a mapping.
```bash
cd ~/GitMigration
java -jar ~/svn-migration-scripts.jar authors <svn_repo_uri> > authors.txt
```
* This produces one line per SVN user, e.g.:
```
j.doe = j.doe <j.doe@mycompany.com>
```
* **Edit each line** so it has the real name and email:
```
j.doe = John Doe <john.doe@atlassian.com>
```
* This file is used in the **Convert** phase so every Git commit gets the correct author.

---

### Checklist
| ✅ | Task |
|---|---|
| ☐ | `svn-migration-scripts.jar` downloaded to `~` |
| ☐ | `verify` passes (JRE, Git, SVN, git-svn) |
| ☐ | Case-sensitive `~/GitMigration` created (disk image on OS X) |
| ☐ | `authors.txt` generated and edited with real names/emails |

---

### Summary Takeaway
**Prepare** = get the **migration lead's machine** ready: download **`svn-migration-scripts.jar`**, **`verify`** that Java, Git, Subversion, and git-svn are installed, create a **case-sensitive `~/GitMigration`** workspace (a disk image on macOS), and generate and edit an **`authors.txt`** that maps SVN usernames to Git names and emails. Next comes **Convert**.
