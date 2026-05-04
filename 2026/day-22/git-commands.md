# Git Commands Reference

## Setup & Config

| Command | Description |
|--------|-------------|
| `git config --global user.name "Your Name"` | Set your global username |
| `git config --global user.email "you@example.com"` | Set your global email |
| `git config --list` | View all config settings |
| `git init` | Initialize a new local repository |
| `git clone <url>` | Clone a remote repository locally |
| `git remote add origin <url>` | Link local repo to remote |
| `git remote -v` | View linked remote URLs |
| `git remote set-url origin <url>` | Change existing remote URL |
| `git remote set-head origin main` | Set default remote branch |

---

## Basic Workflow

| Command | Description |
|--------|-------------|
| `git status` | Check current state of working directory |
| `git add <file>` | Stage a specific file |
| `git add .` | Stage all changed files |
| `git commit -m "message"` | Commit staged changes with a message |
| `git commit -a -m "message"` | Stage and commit tracked files at once |
| `git push origin <branch>` | Push local branch to remote |
| `git push -u origin <branch>` | Push and set upstream (first time) |
| `git push origin <branch> --force` | Force push (overwrites remote history) |
| `git pull origin <branch>` | Fetch and merge remote changes |
| `git pull origin main --allow-unrelated-histories` | Merge two unrelated histories |
| `git fetch` | Download remote changes without merging |
| `git log HEAD..origin/feature-1 --oneline` | Shows new commits on remote that you don't have locally yet |
| `git log --oneline --graph origin/feature-1` | Shows remote branch commit visually |
| `git diff --name-only HEAD origin/feature-1` | Shows file names that have been changed |
| `git diff HEAD origin/feature-1` | See exactly what lines changed | 

---

## Viewing Changes

| Command | Description |
|--------|-------------|
| `git log` | Full commit history |
| `git log --oneline` | Compact one-line commit history |
| `git log --oneline --graph` | Visual branch graph in terminal |
| `git diff` | Show unstaged changes |
| `git diff <branch1> <branch2>` | Compare two branches |


---

### Branching Commands

| Command | Description |
|--------|-------------|
| `git branch` | List local branches |
| `git branch -r` | List remote branches |
| `git branch -a` | List all branches (local + remote) |
| `git branch -vv` | List branches with tracking info |
| `git branch <name>` | Create a new branch |
| `git switch <branch>` | Switch to a branch |
| `git switch -c <branch>` | Create and switch to new branch |
| `git checkout <branch>` | Switch to a branch (older syntax) |
| `git checkout -b <branch>` | Create and switch (older syntax) |
| `git branch -d <branch>` | Delete branch safely (merged only) |
| `git branch -D <branch>` | Force delete branch |
| `git push origin --delete <branch>` | Delete branch from remote |
| `git branch -M main` | Rename current branch to main |


### Merging & Rebasing

| Command | Description |
|--------|-------------|
| `git merge <branch>` | Merge branch into current branch |
| `git merge <branch> -m "message"` | Merge with custom commit message |
| `git merge <branch> --ff-only` | Only merge if fast-forward is possible |
| `git merge <branch> --squash` | Squash all commits into one (must commit after!) |
| `git rebase <branch>` | Rebase current branch onto another |
| `git rebase -i HEAD~<n>` | Interactively edit last N commits |
| `git rebase --abort` | Cancel an ongoing rebase |
| `git rebase --continue` | Continue rebase after resolving conflict |

### Stashing

| Command | Description |
|--------|-------------|
| `git stash` | Save uncommitted changes temporarily |
| `git stash list` | View all stashed changes |
| `git stash pop` | Restore stash and remove it from list |
| `git stash apply` | Restore stash but keep it in list |
| `git stash drop stash@{0}` | Delete a specific stash |
| `git stash clear` | Delete all stashes |


### Cherry-Picking

| Command | Description |
|--------|-------------|
| `git cherry-pick <hash>` | Apply a specific commit to current branch |
| `git cherry-pick <hash1> <hash2>` | Apply multiple specific commits |
| `git cherry-pick <hash1>..<hash2>` | Apply a range of commits |
| `git cherry-pick --no-commit <hash>` | Apply changes but don't commit yet |
| `git cherry-pick --abort` | Cancel cherry-pick in progress |
| `git cherry-pick --continue` | Continue after resolving conflict |
| `git cherry-pick -x <hash>` | Apply commit and mention original hash in message |