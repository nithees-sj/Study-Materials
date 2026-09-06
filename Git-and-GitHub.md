# Git & GitHub — Complete CLI Guide

> From `git init` to pushing, branching, rollbacks, and running GitHub entirely from your terminal.
> This guide covers everything from setting up Git for the first time to what happens **after** you push your code to GitHub.

---

## Table of Contents

1. [What Is Git vs. GitHub?](#1-what-is-git-vs-github)
2. [Installing & Configuring Git](#2-installing--configuring-git)
3. [Initializing a Project](#3-initializing-a-project)
4. [Staging & Committing Changes](#4-staging--committing-changes)
5. [Connecting to GitHub & Pushing](#5-connecting-to-github--pushing)
6. [Branching & Merging](#6-branching--merging)
7. [Cloning, Fetching & Pulling](#7-cloning-fetching--pulling)
8. [After the Push: Working on GitHub](#8-after-the-push-working-on-github)
9. [Version Control & Rollbacks](#9-version-control--rollbacks)
10. [Using GitHub From the Terminal (gh CLI)](#10-using-github-from-the-terminal-gh-cli)
11. [Complete Command Cheat Sheet](#11-complete-command-cheat-sheet)
12. [Best Practices](#12-best-practices)

---

## 1. What Is Git vs. GitHub?

People often use **Git** and **GitHub** interchangeably, but they are two different things working together.

### Git
Git is **version control software** that runs on your own computer. It tracks every change you make to your files over time, lets you save "snapshots" of your project (commits), and lets you go back to any earlier snapshot whenever you need to. Git works completely offline — you don't need the internet to use it.

### GitHub
GitHub is a **website/cloud service** that hosts Git repositories online. It gives your local Git project a home on the internet so you can back it up, share it with others, collaborate, and track issues, pull requests, and project history through a web interface (or its own terminal tool).

> 💡 **In short:** Git = the tool that tracks changes. GitHub = the website that stores and shares your Git project.

### How a Typical Project Flows

1. **Initialize** a Git repository in your project folder
2. **Stage** the files you want Git to track
3. **Commit** the staged files as a saved snapshot
4. **Connect** your local repository to a GitHub repository
5. **Push** your commits up to GitHub
6. **Continue working** — branch, merge, roll back mistakes, pull updates, and manage everything from GitHub or the terminal

---

## 2. Installing & Configuring Git

### Check if Git Is Installed

```bash
git --version
```

### Install Git

| OS | Command / Method |
|---|---|
| Windows | Download from git-scm.com and run the installer |
| macOS | `brew install git` |
| Ubuntu/Debian Linux | `sudo apt update && sudo apt install git` |

### First-Time Setup (Run Once)

Before your first commit, tell Git who you are — this information is attached to every commit you make.

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

### Useful Configuration Options

```bash
# Set your default branch name to "main"
git config --global init.defaultBranch main

# Set your default text editor (e.g., VS Code)
git config --global core.editor "code --wait"

# View all current settings
git config --list
```

> 📝 **Note:** `--global` applies the setting to every repository on your machine. Drop it to set something for just one project.

---

## 3. Initializing a Project

### Turn a Folder Into a Git Repository

```bash
cd my-project
git init
```

This creates a hidden `.git` folder that stores all version history. Your files are not tracked automatically — you still need to stage them.

### Check Repository Status

This is the command you will run constantly. It shows you what has changed, what's staged, and what isn't tracked yet.

```bash
git status
```

### Create a .gitignore File

Use this to tell Git which files or folders to **never** track (e.g., dependency folders, secrets, build files).

```bash
# Example .gitignore contents
node_modules/
.env
*.log
__pycache__/
```

---

## 4. Staging & Committing Changes

### Stage Files

Staging means marking a file as "ready to be included in the next commit."

```bash
git add filename.txt        # stage one file
git add folder/             # stage a whole folder
git add .                   # stage everything changed
git add -p                  # stage changes interactively, chunk by chunk
```

### Commit Staged Changes

A commit is a permanent snapshot of your staged changes, saved with a message describing what changed.

```bash
git commit -m "Add login page and validation"
```

```bash
# Stage all tracked file changes AND commit in one step
git commit -am "Fix typo in header"
```

### View Commit History

```bash
git log                            # full history
git log --oneline                  # compact, one line per commit
git log --oneline --graph --all    # visual branch history
```

### See Exactly What Changed

```bash
git diff                    # unstaged changes
git diff --staged           # staged changes not yet committed
```

> 💡 **Good habit:** Commit small, logical chunks of work with clear messages instead of one giant commit at the end of the day. It makes rollbacks and reviews far easier later.

---

## 5. Connecting to GitHub & Pushing

### Step A — Create a Repository on GitHub

On GitHub.com, click **New Repository**, name it, choose public/private, and skip adding a README if you already have local files.

### Step B — Link Your Local Repo to GitHub

```bash
git remote add origin https://github.com/username/repo-name.git
```

`origin` is just the default nickname Git uses for your main remote repository.

### Step C — Push Your Code

```bash
# First push: sets "origin main" as the default upstream branch
git push -u origin main
```

After this first push with `-u`, future pushes only need:

```bash
git push
```

### Useful Remote Commands

```bash
git remote -v                              # list remote URLs (fetch/push)
git remote set-url origin <new-url>        # change the remote URL
git remote remove origin                   # disconnect from a remote
git push origin branch-name                # push a specific branch
git push --tags                            # push local tags to GitHub
```

> ⚠️ **Authentication note:** GitHub no longer accepts your account password for HTTPS pushes. Use a **Personal Access Token** or set up **SSH keys** (Settings → Developer settings → SSH and GPG keys on GitHub).

---

## 6. Branching & Merging

Branches let you work on new features or fixes without touching the main, stable codebase.

### Create & Switch Branches

```bash
git branch feature-login              # create a branch
git checkout feature-login            # switch to it
git checkout -b feature-login         # create + switch in one step
git switch feature-login              # modern alternative to checkout
git switch -c feature-login           # modern create + switch
```

### List, Rename, Delete Branches

```bash
git branch                     # list local branches
git branch -a                  # list local + remote branches
git branch -m new-name         # rename current branch
git branch -d feature-login    # delete a merged branch
git branch -D feature-login    # force-delete (unmerged)
```

### Merge a Branch Into Main

```bash
git checkout main
git merge feature-login
```

### Push a New Branch to GitHub

```bash
git push -u origin feature-login
```

> ⚠️ **Merge conflicts:** If Git can't automatically combine changes, it marks the conflicting lines in the file with `<<<<<<<`, `=======`, and `>>>>>>>`. Edit the file to keep the correct code, then run `git add <file>` and `git commit` to finish the merge.

---

## 7. Cloning, Fetching & Pulling

### Clone an Existing GitHub Repo

Downloads a full copy of a repository, including all its history, to your machine.

```bash
git clone https://github.com/username/repo-name.git
```

### Fetch vs. Pull

| Command | What It Does |
|---|---|
| `git fetch` | Downloads new commits from GitHub but does **not** merge them into your working files |
| `git pull` | Downloads **and** merges new commits into your current branch (fetch + merge combined) |

```bash
git fetch origin
git pull origin main
```

> 💡 **Tip:** Always `git pull` before you start new work each day to make sure you have the latest changes from GitHub.

---

## 8. After the Push: Working on GitHub

Once your code is on GitHub, most day-to-day collaboration happens around these features.

### Pull Requests (PRs)

A Pull Request proposes merging one branch into another (e.g., `feature-login` into `main`). It's where teammates review code, leave comments, and approve changes before they're merged.

1. **Push your branch** to GitHub with `git push -u origin branch-name`
2. **Open GitHub** — it will show a banner to "Compare & pull request"
3. **Add a title/description**, request reviewers, and click **Create pull request**
4. **Merge** once approved, using "Merge," "Squash and merge," or "Rebase and merge"

### Issues

Issues track bugs, tasks, and feature requests. You can link a commit to an issue by including `Fixes #12` or `Closes #12` in a commit message — GitHub will auto-close that issue when the commit is merged.

### Forks

A fork is your own personal copy of someone else's repository on GitHub, used to contribute to projects you don't have direct write access to.

```bash
# After forking on GitHub's website, add the original repo as "upstream"
git remote add upstream https://github.com/original-owner/repo-name.git
git fetch upstream
git merge upstream/main
```

### Releases & Tags

Tags mark specific commits as important milestones, like version numbers.

```bash
git tag v1.0.0
git push origin v1.0.0
```

On GitHub, you can turn a tag into a formal "Release" with notes and downloadable files.

### GitHub Actions

A workflow automation system that can automatically run tests, build, or deploy your code whenever you push or open a PR. Defined by `.yml` files inside a `.github/workflows/` folder in your repo.

---

## 9. Version Control & Rollbacks

This is Git's core strength: safely undoing mistakes. Here's how, from least to most drastic.

### 1. Undo Changes in a File (Not Yet Staged)

```bash
git restore filename.txt
# (older Git versions: git checkout -- filename.txt)
```

### 2. Unstage a File (Keep the Edits)

```bash
git restore --staged filename.txt
# (older Git versions: git reset filename.txt)
```

### 3. Edit the Last Commit

Forgot a file or made a typo in your last commit message?

```bash
git add forgotten-file.txt
git commit --amend -m "Corrected commit message"
```

> ⚠️ **Caution:** Only amend commits that haven't been pushed yet. Amending a pushed commit rewrites history and can cause problems for anyone who already pulled it.

### 4. Roll Back to an Older Commit (Keep History)

`git revert` creates a **new** commit that undoes a previous one. This is the safest way to roll back on a shared/pushed branch, since nothing is deleted.

```bash
git revert <commit-hash>
git push
```

### 5. Move the Branch Pointer Back (Rewrites History)

`git reset` moves your branch backward to an earlier commit. Use with caution — avoid on commits already pushed and shared with others.

```bash
git reset --soft <commit-hash>     # keep changes, staged
git reset --mixed <commit-hash>    # keep changes, unstaged (default)
git reset --hard <commit-hash>     # DELETE changes entirely
```

> ⚠️ **Danger:** `git reset --hard` permanently discards uncommitted work in that range. Double-check `git status` and `git log` before running it.

### 6. Recover "Lost" Commits

Even after a hard reset, Git usually keeps a safety log of where your branch has been.

```bash
git reflog
# find the commit hash you want, then:
git reset --hard <commit-hash-from-reflog>
```

### 7. Roll Back Code on GitHub Itself

You don't have to use the terminal — on GitHub.com you can:
- Open a file's **History** and view/restore an older version
- Revert a merge commit directly from a Pull Request's **"Revert"** button
- Restore a deleted branch from the repository's **"Branches"** page (shortly after deletion)

### Quick Comparison

| Command | Effect | Safe on Shared Branches? |
|---|---|---|
| `git restore` | Undo local uncommitted edits | ✅ Yes |
| `git commit --amend` | Edit the very last commit | ⚠️ Only if not pushed |
| `git revert` | New commit that undoes an old one | ✅ Yes |
| `git reset --soft/mixed` | Move branch back, keep changes | ❌ No |
| `git reset --hard` | Move branch back, discard changes | ❌ No |

---

## 10. Using GitHub From the Terminal (gh CLI)

The official **GitHub CLI** (`gh`) lets you do almost everything GitHub's website does — creating repos, opening PRs, managing issues — without leaving your terminal.

### Install & Log In

```bash
# macOS
brew install gh

# Ubuntu/Debian
sudo apt install gh

# Authenticate your account
gh auth login
```

### Repository Commands

```bash
gh repo create my-project --public --source=. --push   # create + push in one step
gh repo clone username/repo-name
gh repo view --web                                       # open repo in browser
```

### Pull Request Commands

```bash
gh pr create --title "Add login page" --body "Implements login form"
gh pr list
gh pr view 12
gh pr checkout 12          # pull down someone else's PR to test locally
gh pr merge 12
```

### Issue Commands

```bash
gh issue create --title "Bug: login fails on mobile"
gh issue list
gh issue close 8
```

> 💡 **Why use gh:** No context switching to a browser. You can script repo creation, PR review, and issue triage as part of your normal terminal workflow.

---

## 11. Complete Command Cheat Sheet

**Setup & Init**

| Command | Purpose |
|---|---|
| `git init` | Start tracking a folder |
| `git clone <url>` | Copy a remote repo locally |
| `git config --global user.name/email` | Set author identity |

**Everyday Workflow**

| Command | Purpose |
|---|---|
| `git status` | See current changes/state |
| `git add <file> / .` | Stage changes |
| `git commit -m "msg"` | Save a snapshot |
| `git push` | Upload commits to GitHub |
| `git pull` | Download + merge latest changes |
| `git log --oneline` | View commit history |

**Branching**

| Command | Purpose |
|---|---|
| `git branch` | List branches |
| `git switch -c <name>` | Create + switch branch |
| `git merge <branch>` | Merge branch into current one |
| `git branch -d <name>` | Delete a branch |

**Undo / Rollback**

| Command | Purpose |
|---|---|
| `git restore <file>` | Discard local edits |
| `git commit --amend` | Fix last commit |
| `git revert <hash>` | Undo a commit safely |
| `git reset --hard <hash>` | Force branch back (destructive) |
| `git reflog` | Recover lost commits |

**Remote / GitHub**

| Command | Purpose |
|---|---|
| `git remote add origin <url>` | Link to a GitHub repo |
| `git push -u origin main` | First push + set upstream |
| `gh pr create` | Open a pull request from terminal |
| `gh issue create` | Open an issue from terminal |

---

## 12. Best Practices

- **Commit often, in small units** — each commit should represent one logical change
- **Write clear commit messages** in the present tense: "Fix login bug," not "fixed bug"
- **Pull before you push** to avoid unnecessary conflicts
- **Use branches** for every new feature or fix instead of committing straight to `main`
- **Never rewrite history** (`reset --hard`, force-push) on a branch others are using
- **Use `.gitignore`** to keep secrets, build artifacts, and dependencies out of the repo
- **Review before merging** — use Pull Requests even when working solo; it creates a clean history and review trail
- **Tag releases** so you can always find and roll back to a known-stable version

---

<p align="center"><em>Git & GitHub Command Reference — Compiled Guide</em></p>
