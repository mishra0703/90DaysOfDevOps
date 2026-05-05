# Day 26 – GitHub CLI: Manage GitHub from Your Terminal

## Task

Every time we switch to the browser to create a PR, check an issue, or manage a repo — we lose context. The **GitHub CLI (`gh`)** lets us do all of that without leaving our terminal. For DevOps engineers, this is essential — especially when we start automating workflows, scripting PR reviews, and managing repos at scale.

---


## Install and Authenticate

1. Install the GitHub CLI on your machine
    - For Ubuntu bases systems : `sudo apt update sudo apt install gh -y`    
    - For Windows : `winget install GitHub.cli`  
    - Verify that it installed succesfully : `gh --version`

2. Authenticate with your GitHub account
    - `gh auth login`
    - Follow the steps and log in to github account via https or ssh 

3. Verify you're logged in and check which account is active
    - `gh auth status` 
    - This command will show the status weather we're logged in or not
    ```
    # Something like this 

    github.com
    ✓ Logged in to github.com account mishra0703 (/home/ubuntu/.config/gh/hosts.yml)
    - Active account: true
    - Git operations protocol: ssh
    - Token: gho_************************************
    - Token scopes: 'admin:public_key', 'gist', 'read:org', 'repo'    
    ```

4. Answer in your notes: What authentication methods does `gh` support?
    - Web Browser (OAuth) `gh auth login` → opens browser → login via GitHub → token saved automatically
    - Personal Access Token (PAT) → manually create a token on GitHub → paste it into `gh auth login` when prompted
    - Secure Shell (SSH) → Use existing SSH key pair → `gh` uses it to authenticate with GitHub


---

## Working with Repositories

1. Create a **new GitHub repo** directly from the terminal — make it public with a README
    - `gh repo create`  :  To create a new repository interactively:


2. Clone a repo using `gh` instead of `git clone`
    - `gh repo clone username/repository`   :  To clone a repo to local machine


3. View details of one of your repos from the terminal
    - `gh repo view`   :    To view information about the current repository:


4. List all your repositories
    - `gh repo list`    :  List all our repositories with their desc. and public/private status and last update time
    

5. Open a repo in your browser directly from the terminal
    -  `gh repo view mishra0703/scandine --web` : Opens a repo in browser direct;y from the terminal (Only work with default cmd/shell not with git bash)


6. Delete the test repo you created (be careful!)
     - `gh repo delete repo_name`  : Delete a repository 
     - But before that we must have admin rights to Repository. This API operation needs the "delete_repo" scope. To request it, run:  `gh auth refresh -h github.com -s delete_repo`
     - Then again authenticate via Browser or SSH and give permission to delete a repo
     - Then run `gh repo delete repo_name`



---

## Issues

1. Create an issue on one of your repos from the terminal — give it a title, body, and a label
    - `gh issue create`  :  To start an interactive issue creation process 
    - `gh issue create --title "Bug in login" --body "Login fails on invalid token"`  :  To create an issue quickly with a title and body:


2. List all open issues on that repo
    - `gh issue list`  :  Lists open issues in repo 


3. View a specific issue by its number
    - `gh issue view 1`  :  View a specific issue by its number:


4. Close an issue from the terminal
    - `gh issue close 1`  :  Once an issue is resolved, close it using its number: 


5. Answer in your notes: How could you use `gh issue` in a script or automation?
    - `gh issue` can output `JSON` format → perfect for scripting
    - We can auto-assign issues based on labels
    - We can even Generate issue reports / summaries
    - We can Close stale issues in bulk
    - We can Create issues from CI/CD pipeline (e.g., when tests fail)
    ```
    # In GitHub Actions / CI script

    gh issue create \
      --title "Build failed on main" \
      --body "Automated: Build failed at $(date)" \
      --label "bug" \
      --assignee "dev-username"
    ```
    - Every time build fails → issue auto-created on GitHub


---

## Pull Requests

1. Create a branch, make a change, push it, and create a **pull request** entirely from the terminal
    - `gh pr create` : To create a pull request interactively
    - `gh pr create --base main --head feature-branch --title "Add login feature" --body "Implemented login logic"`  :  To create pull request quickly 

2. List all open PRs on a repo
    - `gh pr list`  :   List active pull requests
    - `gh pr list --state open` :  List open pull requests
    - `gh pr list --state closed`   :  List closed pull requests
    - `gh pr list --state merged`   :  List merged pull requests
    - `gh pr list --state all`  :  List all pull requests


3. View the details of your PR — check its status, reviewers, and checks
    - `gh pr view` or `gh pr view pr_number`  :  auto-detects current branch's PR , and shows it's details (Title, body, checks, reviewers)
    - `gh pr status`  :  shows all PRs relevant to YOU in the repo  , Unlike gh pr view which shows one specific PR. It shows ---> PRs you created ; PRs requesting your review ; PRs assigned to you Current branch's PR status (if there is any)


4. Merge your PR from the terminal
    - `gh pr merge pr_number`  :  Merge a pull request


5. Answer in your notes:
- What merge methods does `gh pr merge` support?   
    - `--merge` → standard merge commit — keeps full history, creates a merge commit
    - `--squash` → squashes all PR commits into one single commit → clean history
    - `--rebase` → replays PR commits on top of base branch → linear history, no merge commit


- How would you review someone else's PR using `gh`?
    - `gh pr list`  :  First See all open PRs
    - `gh pr view 5`  :  View PR details
    - `gh pr checkout 5`  :  Checkout their code locally & test it
    - `gh pr review 5 --comment --body "Looks good, but check edge cases"`  :  Add a review comment
    - `gh pr review 5 --approve` or `gh pr review 5 --request-changes --body "Fix the login bug first"` : Approve or Request Changes


