# 🛠️ Git & GitHub Crash Course Cheat Sheet

This cheat sheet covers the 20% of Git commands you will use 80% of the time in your career as a Data Engineer.

---

## 🔄 The Daily Loop (Save & Sync)

You will run these commands multiple times a day as you write code.

| Command | What it does | When to use it |
|---|---|---|
| **`git status`** | Shows modified, deleted, or untracked files. | Run this first to see what you've changed. |
| **`git add .`** | Stages ALL changes in the directory. | Puts all your edits into the "shopping cart." |
| **`git add <file>`** | Stages only one specific file. | Puts only that file into the cart. |
| **`git commit -m "msg"`** | Creates a local save point (commit). | Locks your staged changes into history. |
| **`git push`** | Uploads your local commits to GitHub. | Syncs your local history to the cloud. |

---

## 🔎 Inspecting Your Work

Commands for checking your project history and finding differences.

*   **`git log --oneline`**
    *   *Use case:* Shows a clean, one-line timeline of your past save points.
*   **`git diff`**
    *   *Use case:* Shows exactly what lines of code you added or deleted since your last commit.
*   **`git diff --staged`**
    *   *Use case:* Shows changes in files that you have already run `git add` on.

---

## 🌳 Branching (Parallel Universes)

Branches allow you to work on new features without breaking your main, working code.

```text
main branch:    ●──────────●──────────● (Production code)
                            \
feature branch:              ●────────● (Testing new transformations)
```

*   **`git branch`**
    *   *Use case:* Lists all branches in your repository. Tells you which one you are currently on.
*   **`git checkout -b <branch-name>`**
    *   *Use case:* Creates a new branch and switches to it. (e.g., `git checkout -b feature-date-dimension`).
*   **`git checkout <branch-name>`**
    *   *Use case:* Switches back to an existing branch (e.g., `git checkout main`).
*   **`git merge <branch-name>`**
    *   *Use case:* Merges changes from a feature branch into your current branch (always switch to `main` before merging).

---

## 👥 Collaboration (Multiplayer Mode)

Commands for working with remote code bases.

*   **`git clone <url>`**
    *   *Use case:* Downloads an entire existing GitHub repository onto your computer for the first time.
*   **`git pull`**
    *   *Use case:* Downloads and merges the latest changes from GitHub into your local folder. (Always do this before you start coding if someone else on your team is editing files).
*   **`git fetch`**
    *   *Use case:* Downloads the latest changes from GitHub but *does not* merge them yet. Just lets you see what's new.

---

## 🚨 Fixing Mistakes (The Undo Buttons)

What to do when things go wrong.

### 1. Discard uncommitted changes
You edited a file, broke it completely, and want to discard all edits since your last commit:
```bash
git checkout -- <filename>
```

### 2. Unstage a file (Remove from shopping cart)
You ran `git add .` but realized you don't want to save a specific file in the next commit:
```bash
git restore --staged <filename>
```

### 3. Undo your last commit (Keep your code changes)
You committed, but forgot to add a file or made a typo in your commit message. This undoes the commit but leaves your code edits intact in your staging area:
```bash
git reset --soft HEAD~1
```

### 4. Nuke the last commit completely (Destructive!)
This deletes your last commit AND deletes all code changes you made in that commit. **Warning: There is no undo for this.**
```bash
git reset --hard HEAD~1
```

---

## 🚫 The `.gitignore` File

In Data Engineering, you **never** want to commit raw data files (like 2GB CSVs), passwords, API keys, or database config files to public GitHub.

To prevent this:
1. Create a file named `.gitignore` in your root folder.
2. Write the names of files or directories you want Git to ignore.

**Example `.gitignore` content:**
```text
# Ignore raw data folders
data/
*.csv
*.xlsx

# Ignore environment variables / secrets
.env
secret_keys.json

# Ignore system configuration
.DS_Store
Thumbs.db
```
Git will completely ignore these files, and they will never show up in `git status` or get pushed to GitHub.
