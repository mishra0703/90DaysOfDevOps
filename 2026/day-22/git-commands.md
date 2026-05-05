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

---

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

---

### Stashing

| Command | Description |
|--------|-------------|
| `git stash` | Save uncommitted changes temporarily |
| `git stash list` | View all stashed changes |
| `git stash pop` | Restore stash and remove it from list |
| `git stash apply` | Restore stash but keep it in list |
| `git stash drop stash@{0}` | Delete a specific stash |
| `git stash clear` | Delete all stashes |

---

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

---

### Undoing Changes

| Command | Description |
|--------|-------------|
| `git revert <commit>` | Create new commit that undoes a commit (safe) |
| `git reset --soft HEAD~1` | Undo last commit, keep changes staged |
| `git reset --mixed HEAD~1` | Undo last commit, keep changes unstaged (default reset behavior) |
| `git reset --hard HEAD~1` | Undo last commit and delete all changes |
| `git reset --hard <commit-hash>` | Jump to a specific commit and delete everything after it |
| `git restore <file>` | Discard uncommitted changes in a file |
| `git checkout -- <file>` | Discard changes (older syntax) |
| `git reset HEAD~N` | Undo last N commits, keep changes unstaged |
| `git revert HEAD~3..HEAD` | Revert last 3 commits safely (creates 3 new commits) |
| `git restore --staged <file>` | Unstage a file but keep changes in working directory |
| `git clean -fd` | Delete all untracked files and folders permanently |
| `git reflog` | View history of all HEAD movements (recovery tool after hard reset) |


---

## GH CLI Commands

### GH CLI — Auth

| Command | Description |
|--------|-------------|
| `gh auth login` | Authenticate with GitHub via browser, PAT, or SSH |
| `gh auth status` | Check which account is active and token scopes |
| `gh auth refresh -h github.com -s delete_repo` | Request additional permission scope |

---

### GH CLI — Repositories

| Command | Description |
|--------|-------------|
| `gh repo create` | Create a new GitHub repo interactively |
| `gh repo clone username/repository` | Clone a repo using gh instead of git clone |
| `gh repo fork username/repository` | Fork a repo using gh instead of doing it on remote |
| `gh repo view` | View details of current repository |
| `gh repo view username/repo --web` | Open repo directly in browser |
| `gh repo list` | List all your repositories with details |
| `gh repo delete repo_name` | Delete a repository (needs delete_repo scope) |
| `gh auth refresh -h github.com -s delete_repo` | This is the scope for Requesting additional permission scope |


---

### GH CLI — Issues

| Command | Description |
|--------|-------------|
| `gh issue create` | Create an issue interactively |
| `gh issue create --title "title" --body "body" --label "bug"` | Create issue quickly with flags |
| `gh issue list` | List all open issues in repo |
| `gh issue view 1` | View a specific issue by number |
| `gh issue close 1` | Close a specific issue by number |

---

### GH CLI — Pull Requests

| Command | Description |
|--------|-------------|
| `gh pr create` | Create a pull request interactively |
| `gh pr create --base main --head feature --title "title" --body "body"` | Create PR quickly with flags |
| `gh pr list` | List all open PRs |
| `gh pr list --state open/closed/merged/all` | Filter PRs by state |
| `gh pr view` | View current branch's PR details |
| `gh pr view 5` | View specific PR by number |
| `gh pr view --web` | Open PR in browser |
| `gh pr status` | Dashboard of all PRs relevant to you |
| `gh pr checks` | See only CI/CD checks status |
| `gh pr checkout 5` | Switch locally to PR #5's branch |
| `gh pr review 5 --approve` | Approve a PR |
| `gh pr review 5 --request-changes --body "reason"` | Request changes on a PR |
| `gh pr review 5 --comment --body "comment"` | Leave a comment on a PR |
| `gh pr merge 5 --merge` | Merge PR with standard merge commit |
| `gh pr merge 5 --squash` | Merge PR squashing all commits into one |
| `gh pr merge 5 --rebase` | Merge PR using rebase (linear history) |
| `gh pr merge 5 --squash --delete-branch` | Merge and auto-delete feature branch |


---

### Useful Commands

| Command | Description |
|--------|-------------|
| `gh status` | Provides a quick overview of your work in the current repository. |
| `gh help` | Accesses the built-in documentation for any command. |
|`-web` | Most commands support this flag to transition to the browser when visual detail is needed. |


---

### Custom Aliases


*You can create custom shortcuts for frequently used commands*

| Command | Description |
|--------|-------------|
|`gh alias set rl "repo list"` | Now, instead of gh repo list, you can simply run: |
| `gh rl` | Will show repo list | 