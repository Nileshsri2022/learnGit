Here is a clear, cheat-sheet-style guide to installing and configuring Git across **macOS, Windows, and Linux** based on the provided text.

---

### 1. Mandatory Post-Installation Setup (All Platforms)
Once Git is installed, you must configure your username and email. These details are attached to every commit you make:

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

To verify your installation and check the installed version:
```bash
git --version
```

---

### 2. Installing Git on macOS

#### Option A: Stand-alone Installer (Easiest)
1. Download the official **Git for Mac installer**.
2. Run the installer package and follow the on-screen prompts.

#### Option B: Using Homebrew (Recommended for Developers)
```bash
brew install git
```

#### Option C: Using MacPorts
```bash
sudo port selfupdate
sudo port install git +bash_completion +credential_osxkeychain +doc
```

#### Option D: GUI Client (Sourcetree)
Download and install **Atlassian Sourcetree**, which comes with Git pre-bundled.

---

### 3. Installing Git on Windows

#### Option A: Stand-alone Installer (Git for Windows)
1. Download and run the official **Git for Windows** setup wizard.
2. Follow the setup wizard using the default recommended settings (includes **Git Bash** and the command-line tools).

#### Option B: GUI Client (Sourcetree)
Download **Atlassian Sourcetree for Windows**, which includes a bundled version of Git.

---

### 4. Installing Git on Linux

#### Debian / Ubuntu (`apt-get`)
```bash
sudo apt-get update
sudo apt-get install git
```

#### Fedora / RHEL / CentOS (`dnf` or `yum`)
```bash
# On modern Fedora:
sudo dnf install git

# On older Fedora / CentOS / RHEL:
sudo yum install git
```

---

### 5. Managing Passwords (Credential Helpers)
To avoid entering your username and password every time you push or pull over HTTPS:

* **macOS (OS X Keychain):**
  ```bash
  git config --global credential.helper osxkeychain
  ```
* **Windows:** Use the built-in **Git Credential Manager for Windows** during the installer setup.

---

### 6. Summary Comparison: Which Method Should You Use?

| Platform | Recommended Method | Why? |
|---|---|---|
| **macOS** | **Homebrew** (`brew install git`) | Keeps Git up-to-date easily (Apple’s built-in version lags behind). |
| **Windows** | **Git for Windows Installer** | Includes Git CLI, Git GUI, Git Bash, and the Credential Manager. |
| **Linux** | **Package Manager** (`apt` or `dnf`) | Fast, native, and automatically handles system dependencies. |
| **Non-CLI Users** | **Atlassian Sourcetree** | Full visual interface with Git already bundled inside. |
