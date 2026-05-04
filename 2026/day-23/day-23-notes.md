# Day 23 – Git Branching & Working with GitHub

## Task

Now that we know how to create repos, stage, and commit — it's time to learn the most powerful concept in Git: **branching**. Branches let us work on features, fixes, and experiments in isolation without breaking our main code. We'll also push our work to GitHub for the first time.

---

## Understanding Branches
1. What is a branch in Git?
    - A branch in Git is a lightweight, movable pointer to a specific commit in our project's history. 
    - It allows us to diverge from the main line of development to work on new features, fix bugs, or experiment in an isolated environment without affecting the stable "main" code.
    - It also allows multiple team members to work on different tasks simultaneously without interfering with each other.


2. Why do we use branches instead of committing everything to `main`?
    - To keep the production code stable
    - Direct commits to main risk breaking production code, while branches keep the main codebase clean and deployable at any time.
    - If an experiment fails, the branch can be deleted without affecting the main project.


3. What is `HEAD` in Git?
    - `HEAD` is a special pointer in Git that references the currently checked-out branch or commit. 


4. What happens to your files when you switch branches?
    - When we switch branches in Git, our working directory is updated to match the snapshot of the branch we are moving to. 
    - Git replaces, removes, or creates files to match the new branch's commit, updating timestamps. 


---

## Branching Commands — Hands-On

1. List all branches in your repo
    - `git branch` list all the branches present in local
    - `git branch -r` list all the branches present at remote
    

2. Create a new branch called `feature-1`
    - `git branch feature-1`        : Create branch named feature-1    
    

3. Switch to `feature-1`
    - `git switch feature-1`    : Switch to target branch from current branch
    - `git checkout feature-1`  : Switch to target branch from current branch


4. Create a new branch and switch to it in a single command — call it `feature-2`
    - `git switch -c feature-2`     : Create and Switch to branch named feature-2
    - `git checkout -b feature-2`   : Create and checkout from current branch to new branch named feature-2
    - `git checkout -b feature-2 base-branch` : Creates from a specific point or branch
 

5. Try using `git switch` to move between branches — how is it different from `git checkout`?
    - `git switch branch_name` – New and does one thing only that is to switch between branches.
    - `git checkout branch_name` – Old and multipurpose command. Can switch branches and restore files. Does too many things, which caused confusion.


6. Make a commit on `feature-1` that does **not** exist on `main`
    - After making commit , check by using `git log` or `git log --oneline`


7. Switch back to `main` — verify that the commit from `feature-1` is not there
    - After switching to `main` branch , verify that the commit we did in branch feature-1 is not present in main branch --> by using `git log` or `git log --oneline` you'll see that the head is pointing at the last commit of main branch , and there we won't see feature-1 branch name
    - `49c3906 (HEAD -> master, origin/master, feature-2) chore: new file added`  : something like this 
    ```bash
    905bc60 (HEAD -> feature-1) Chore: New script added
    49c3906 (origin/master, master, feature-2) chore: new file added
    ``` 
    - In feature-1 branch we can see that it is ahead by 1 commit  

    


8. Delete a branch you no longer need
    - `git branch -d branchname`    : Deletes the branch


9. Add all branching commands to your `git-commands.md`

    ```
    # Branching Commands

    git branch                              :     List local branches 
    git branch -r                           :     List remote branches 
    git branch -a                           :     List all branches (local + remote) 
    git branch -vv                          :     List branches with tracking info 
    git branch name                         :     Create a new branch 
    git switch branch_name                  :     Switch to a branch 
    git switch -c branch_name               :     Create and switch to new branch 
    git checkout branch_name                :     Switch to a branch (older syntax) 
    git checkout -b branch_name             :     Create and switch (older syntax) 
    git branch -d branch_name               :     Delete branch safely (merged only) 
    git branch -D branch_name               :     Force delete branch 
    git push origin --delete branch_name    :     Delete branch from remote 
    git branch -M main                      :     Rename current branch to main 
    ```

---

## Push to GitHub

1. Create a **new repository** on GitHub (do NOT initialize it with a README)
    - Go to github
    - Click on create New Repository
    - Do as it says , you'll land on a page where it ask to configure and also mention link for HTTP and SSH method

2. Connect your local `devops-git-practice` repo to the GitHub remote

    - In our terminal (inside our local project folder), run: 

    ````bash
    `# Using HTTPS
    `git remote add origin https://github.com/your-username/devops-git-practice.git
    `
    `# OR using SSH
    `git remote add origin git@github.com:your-username/devops-git-practice.git
    ````
 
---

### - SSH Setup (One-time)
 

- Go to local/terminal and follow these commands

    ```bash
    cd                          # Go to home directory
    cd .ssh                     # Navigate to .ssh folder (create it if it doesn't exist)
    ssh-keygen   # Generate SSH key
                                # When prompted, give it a name (e.g., github-key)
    ```

- Two files will be generated:
    - `github-key` → private key (never share this)
    - `github-key.pub` → public key (this goes to GitHub)
 
    ```bash
    cat github-key.pub          # Print the public key
                                # Copy the entire output
    ```


- Now add the key to GitHub:
    - Go to **GitHub → Settings → SSH and GPG Keys → New SSH Key**
    - Give it a name (e.g., `my-laptop`)
    - Paste the copied key → Click **"Add SSH Key"**


- If on pushing the code to remote it doesn't work , throws an error then do :
    ```bash
    # In .ssh directory 
    vim config

    # Add this in the file 

    ****************
    Host github.com
        AddKeysToAgent yes
        IdentityFile ~/.ssh/new-key(Location to the key)
    ****************
    # It says that use this as default key not the old one
    ```
 
---
 
### - HTTPS Setup
 
- No key setup needed. 
- When do `git push origin branch_name`
- Git will prompt for our **GitHub username** and a **Personal Access Token (PAT)** as password.
- GitHub no longer accepts plain passwords — we must use a PAT.
 
- To generate a PAT:
    - Go to **GitHub → Settings → Developer Settings → Personal Access Tokens → Tokens (classic)**
    - Click **"Generate new token"** → Select scope: `repo` → Generate & copy it

    ```bash
    git remote add origin https://github.com/your-username/devops-git-practice.git
    git push -u origin main     # It will prompt for username and PAT 
                                # Type github username and paste the copied PAT
    ```
 
- But then on every push we have to type our **Username** and paste the **PAT**
- So, the solution is 
    - The **PAT** we copied use it and run
    - `git remote set-url https://<PAT>@github.com/your-username/devops-git-practice.git`   after `//` paste the copied PAT and put `@` and run it 
    - This will set our PAT in HTTP link so we don't have to type or paste it on every `git push`


---

3. Push your `main` branch to GitHub    
    - git push -u origin main    
    - The `-u` flag sets `origin/main` as the upstream — after this, just `git push` works.
 

4. Push `feature-1` branch to GitHub
    - `git switch feature-1`            : Switch to feature-1 branch
    - `git push -u origin feature-1`    : Will push `feature-1` branch only


5. Verify both branches are visible on GitHub
    - Go to your GitHub repo page
    - Click the **branch dropdown** (top-left, shows "main" by default)
    - You should see both `main` and `feature-1` listed ✅


6. Answer in your notes: What is the difference between `origin` and `upstream`?

    | Term | Meaning |
    |------|---------|
    | `origin` | Our **own fork or remote repo** on GitHub (the one we cloned/pushed to) |
    | `upstream` | The **original source repo** we forked from (someone else's repo) |
    
    - In simple personal projects (means no forking), we usually only have `origin`.
 

---

## Pull from GitHub
1. Make a change to a file **directly on GitHub** (use the GitHub editor)
    - Go to repo
    - Open any file/directory in the repo
    - Do any change , like edit it , delete it , add any new file/folder
    - Then save it 


2. Pull that change to your local repo
    - `git pull origin branch_name` branch name should be the branch on which you edit,delete or add file on remote


3. Answer in your notes: What is the difference between `git fetch` and `git pull`?
    - `git pull` : Fetch and merge remote changes
    - `git fetch` : Download remote changes without merging
        - We can see the remote changes by various methods : 

| Command | Description |
|--------|-------------|
| `git log HEAD..origin/feature-1 --oneline` | Shows new commits on remote that you don't have locally yet |
| `git log --oneline --graph origin/feature-1` | Shows remote branch commit visually |
| `git diff --name-only HEAD origin/feature-1` | Shows file names that have been changed |
| `git diff HEAD origin/feature-1` | See exactly what lines changed | 

**Note** : Flow  ::  fetch → diff → review → merge 


---

## Clone vs Fork

1. **Clone** any public repository from GitHub to your local machine
    - Creates a new folder with the repo name
    - Downloads all files, branches & commit history
    - Sets origin as the remote automatically


2. **Fork** the same repository on GitHub, then clone your fork
    - Makes a copy of someone else's repo into our GitHub account. Now we own that copy — we can change it freely without affecting the original.
    - 


3. Answer in your notes:

- What is the difference between clone and fork?
    - Fork :  
        - Copy a repo on GitHub cloud 
    - Clone : 
        - Copy on your PC (Local) 
    - *Note*  :    We can clone without forking (just to use/read code) But we fork when we want our own GitHub copy to push changes to


- When would you clone vs fork?
    - Fork : when we want to contribute or experiment with someone's project
    - Clone :  when we want to actually write/edit code locally
    
- After forking, how do you keep your fork in sync with the original repo?
    - By adding original repo as upstream 
        - `git remote add upstream https://github.com/original-owner/repo.git`

    - Then fetch everytime before starting new work , so that the repo will be up-to-date
        - `git fetch upstream` 

    - Then we can merge this upstream changes (original repo changes) into our local main branch
        - git merge upstream/main

    - Then we can push the updated main on our forked repo 
        - `git push origin main`


