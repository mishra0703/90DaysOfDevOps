# Day 22 – Introduction to Git: Your First Repository

## Task

Today marks the beginning of my Git journey. Git is the backbone of modern DevOps — every tool, pipeline, and workflow revolves around version control. Before diving into advanced concepts, I need to get comfortable with the basics by :

- Understand what Git is and why it matters
- Setting up my first Git repository from scratch
- Starting building a living document of Git commands

---


## Challenge Tasks

### Installing and Configure Git
1. Verify Git is installed on your machine
    - From `https://git-scm.com/install/windows` download git for suitable OS ; Like for windows click on `Git for Windows/x64 Setup.`
    - Install Git on your OS by continuing the steps
    - Can take help of yt videos if facing difficulties

2. Set up your Git identity — name and email
    - In git bash terminal , setup username and email by using `git config` command
    - `git config --global user.name "YOUR_NAME"`
    - `git config --global user.email "YOUR_EMAIL_ID"`

3. Verify your configuration
    - Verify the details using   : `git config --global --list`    
    - It will show the details if added any

---

### Create Your Git Project
1. Create a new folder called `devops-git-practice`
    - `mkdir devops-git-practice`

2. Initialize it as a Git repository
    - `cd devops-git-practice`
    - `git init`

3. Check the status — read and understand what Git is telling you
    - `git status`  :   It says I am on master branch , I didn't commit any file yet and If I want to track any file or folder I have to use `git add` command

4. Explore the hidden `.git/` directory — look at what's inside
    - It has some folders and some files
    - `HEAD`  -d `branches`  `config`  `description`  -d `hooks` -d `info` -d `objects` -d `refs`     

---

### Create Your Git Commands Reference
1. Create a file called `git-commands.md` inside the repo
    - touch `/devops-git-practice/git-commands.md`

2. Add the Git commands you've used so far, organized by category:
   - **Setup & Config**
   - **Basic Workflow**
   - **Viewing Changes**

---
---
---

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

---
---
---


3. For each command, write:
   - What it does (1 line)
   

---

### Stage and Commit
1. Stage your file
    - `git add git-commands.md`

2. Check what's staged
    - `git status`

3. Commit with a meaningful message
    - `git commit -m "chore: git cheat-sheet"`

4. View your commit history
    - `git log` or `git log --oneline`

---

### Make More Changes and Build History
1. Edit `git-commands.md` — add more commands as you discover them
    
2. Check what changed since your last commit
    - `git diff "commit_1" "commit_2"`

3. Stage and commit again with a different, descriptive message
    - `git commit -a -m "any_msg"` do the staging and commit of a *Already Tracked File* (Not works on Untrack File)    

4. Repeat this process at least **3 times** so you have multiple commits in your history

5. View the full history in a compact format
    - `git log --oneline`

---

### Understand the Git Workflow
Answer these questions in your own words (add them to a `day-22-notes.md` file):
1. What is the difference between `git add` and `git commit`?
    - `git add` added file to stage (It takes a snapshot of current condition of the file)
    - `git commit` starts to track the file from now on , whatever changes will happen to that file git will show it and ask it to commit again after some new changes or we can say Git is officially "aware" of the file now and will actively monitor it for any future changes, renames, or deletions.

2. What does the **staging area** do? Why doesn't Git just commit directly?
    - Because not everything we change has to be in the same commit , sometime we only want to commit specific files or folder only not everything in the main dir 

3. What information does `git log` show you?
    - It shows commit history , shows detailed information of each and every commit we made
    
4. What is the `.git/` folder and what happens if you delete it?
    - It's a hidden folder that Git creates when we run git init. It contains everything that Git knows about our project.
    - Basically our entire Git history lives inside `.git/` , our actual project files are outside it.
    - Delete it and our project becomes a regular folder with zero history, as if Git never existed

5. What is the difference between a **working directory**, **staging area**, and **repository**?
    - **Working Directory** : 
        - Where we actually write and edit our files
        - Git watches this folder but doesn't track changes yet

    - **Staging Area** :
        - A middle zone between working directory and repository
        - We manually choose what goes here using `git add`        
 
    - **Repository** : 
        - Where Git permanently stores all our committed history
        - Stores files only those which we committed using `git commit -m "msg"` 

        
---
