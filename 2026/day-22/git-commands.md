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

---

## Viewing Changes

| Command | Description |
|--------|-------------|
| `git log` | Full commit history |
| `git log --oneline` | Compact one-line commit history |
| `git log --oneline --graph` | Visual branch graph in terminal |
| `git diff` | Show unstaged changes |
| `git diff <branch1> <branch2>` | Compare two branches |

