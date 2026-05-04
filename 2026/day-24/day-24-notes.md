# Day 24 – Advanced Git: Merge, Rebase, Stash & Cherry Pick

## Task

We know how to branch and push to GitHub. Now it's time to learn how branches come back together — and what to do when we're in the middle of something and need to context-switch. These are the Git skills that separate beginners from confident practitioners.

---


## Git Merge — Hands-On

1. Create a new branch `feature-login` from `main`, add a couple of commits to it
    -  `git switch -c feature-login`
    - Did some commits

2. Switch back to `main` and merge `feature-login` into `main`
    - `git switch main`
    - `git merge feature-login` : This will merge the feature-login branch into main


3. Observe the merge — did Git do a **fast-forward** merge or a **merge commit**?
    - Git did a **fast-forward merge**
    ```bash
    Updating 7e8ef17..c3e5fb2
    Fast-forward
     fake_file | 2 ++
     1 file changed, 2 insertions(+)
    ```


4. Now create another branch `feature-signup`, add commits to it — but also add a commit to `main` before merging
    - `git switch -c feature-signup`
    - Added a file and did `git add` then commited it 
    - `git switch main`
    - Did some changes and commit that too on branch `main`


5. Merge `feature-signup` into `main` — what happens this time?
    - `git merge feature-signup`
    - An editor will gets open and ask for the `merge commit` message
    - Add any merge commit msg you want to add then save it and a new git began
    - This commit is also known as `Merge Commit`


6. Answer in your notes:

- What is a fast-forward merge?
    - When our main branch has no new commits since we branched off — Git simply moves the pointer forward. No merge commit is created. That commit is fast-forward commit
    ```bash

    ---------- Example of Fast-forward commit ----------
    # Before merge

    main:      A → B
                    \
    feature-1:       C → D


    # After merge

    main:      A → B → C → D
    ```

- When does Git create a merge commit instead?
    - When Both branches have new commits after the point they split — Git can't just slide forward, so it creates a new commit that joins both.
    ```bash

    ---------- Example of Merge commit ----------
    # Before merge

    main:      A → B → E
                    \
    feature-1:       C → D


    # After merge

    main:      A → B → E → M
                    \     /
    feature-1:       C → D
    ```


- What is a merge conflict? (try creating one intentionally by editing the same line in both branches)
    - A merge conflict happens when two branches edit the same line of the same file and Git doesn't know which version to keep — so it stops and will ask us to decide.
    ```
    # In both branch we editted the same file and commit it , and then now we are trying to merging these branches , so this creates a conflict that which edit to keep and which to remove 
    #  So git let this up to us(user) so we have to fix this conflict manually by editting the file which created the conflict
    
    main:           A → B (edited signup_file → "Added owner credits")
                     \
    feature-signup:   C (edited signup_file → "Added credits")
    ```

    - By running `git diff --name-only --diff-filter=U` we will no exactly which files have conflict
    - Or `git status` will also tell this

- How to solve this conflict ? 
    - Open the conflicted file — we'll see:
    ```
    <<<<<<< HEAD
    Added owner credits        ← main branch version
    =======
    Added credits              ← feature-signup version
    >>>>>>> feature-signup
    ```
    - Manually fix it — delete the markers, keep what you want:
    - Then Stage and commit the fix



---

## Git Rebase — Hands-On

1. Create a branch `feature-dashboard` from `main`, add 2-3 commits
    - Did it


2. While on `main`, add a new commit (so `main` moves ahead)
    - Did it


3. Switch to `feature-dashboard` and rebase it onto `main`
    - `git rebase main`


4. Observe your `git log --oneline --graph --all` — how does the history look compared to a merge?
    - After merge , It shows a V shape (branching and joining), merge commit visible
    - After rebase , It shows a straight line (clean, linear history), no merge commit


5. Answer in your notes:
- What does rebase actually do to your commits?
    - Takes our feature-dashboard commits and replants them on top of latest main
    - Like saying "pretend I branched off from here (latest main), not from the old point"


- How is the history different from a merge?
    - After merge , Histoty shows a V shape (branching and joining), merge commit visible
    - Merge History looks messy and sometimes hard to understand 
    - Rebase makes the history linear and easy to understand for humans as Rebase rewrites commit history 
 

- Why should you **never rebase commits that have been pushed and shared** with others?
    - Never rebase a shared/public branch — it confuses teammates
    - `rebase` re-writes the whole history


- When would you use rebase vs merge?
    - We should Use merge when:
        - Working on a shared/team branch
        - We want to preserve exact history (who did what, when)
        - Merging a finished feature into main
        - Working on open source contributions

    - We should Use rebase when:
        - Working on your own local branch
        - We want clean, linear history
        - Before opening a Pull Request (to clean up messy commits)
        - Syncing our branch with latest main before merging


---

## Squash Commit vs Merge Commit

1. Create a branch `feature-profile`, add 4-5 small commits (typo fix, formatting, etc.)
    - Did it


2. Merge it into `main` using `--squash` — what happens?
    - `git merge --squash feature-profile`
    ```
    Fast-forward
    Squash commit -- not updating HEAD
     profile_page | 7 +++++++
     1 file changed, 7 insertions(+)
     create mode 100644 profile_page
    ```
    - Now we have to commit manually to add a single commit instead of 4 different commits (from feature-profile)


3. Check `git log` — how many commits were added to `main`?
    - Before we committed manually there was no commit added 
    - Then we add 1 final commit in place of 4 different commits

    
4. Now create another branch `feature-settings`, add a few commits
    - Did it


5. Merge it into `main` **without** `--squash` (regular merge) — compare the history
    - All commits from feature-profile come into main as-is — every single commit is visible in history.

    ```
    Regular merge ---> "Bring every commit as it is"
    Squash merge ---> "Convert every commit into single one"
    ```


6. Answer in your notes:
- What does squash merging do?
     - Takes all **commits** from a feature branch and combines them into one single commit
     - Git stages the changes but waits for us to commit manually


- When would you use squash merge vs regular merge?
    - When to use Squash merge:
        - When Feature branch has messy/WIP commits like "fix", "typo", "testing", "temp"
        - When we want clean, readable main history
        - When Small features or bug fixes where individual commits don't matter
    - When to use Regular merge:
        - When each and every commit is meaningful and well-written
        - When we need full traceability (who changed what, when, why)
        - When working on a large feature where history matters
        - When Team follows strict commit history standards


- What is the trade-off of squashing?
    - Instead of a long and messy Commit history we get a Clean history
    - As we get only single commit for all changes It's hard to debug specific changes later
    - In New commit history We get less noise in log history
    - Old commits Loses author credit for each commit 
    - New commit is easier to revert as it is a single commit



---

## Git Stash — Hands-On

1. Start making changes to a file but **do not commit**
    - Did it


2. Now imagine you need to urgently switch to another branch — try switching. What happens?
    - Same work will reflect in this branch too , as we didn't commit the work 
    - Did git 

    - Went into branch `feature-settings` and did some work there
    - Made commits related to the work
    - Came back to the main branch


3. Use `git stash` to save your work-in-progress
    - Did `git stash` in WIP(work in progress) branch 
    - Everything / Every change since the last commit get saved in stash 
     
4. Switch to another branch, do some work, switch back
    - Did it


5. Apply your stashed changes using `git stash pop`
    - Came back to the initial branch 
    - Did `git stash pop` 
    - No changes get affected due to this branch 


6. Try stashing multiple times and list all stashes
    ```
    stash@{0}: WIP on master: 0936f5d chore : feature udpated
    stash@{1}: WIP on master: 0936f5d chore : feature udpated
    stash@{2}: WIP on master: 0936f5d chore : feature udpated
    stash@{3}: WIP on master: 0936f5d chore : feature udpated
    ```
    - Each and Every stash gets saved to the last commit that this branch has made


7. Try applying a specific stash from the list
    - `git stash apply stash@{index}`
    - `apply` takes out the stash copy from the `stash list` , means we will come back to the position where we saved the file but the stash won't get deleted from the list we can use it again in other branches 
    - `pop` takes out the stash and delete it from the `stash list` 
    - `drop` deletes a stash from stash list


8. Answer in your notes:
- What is the difference between `git stash pop` and `git stash apply`?
    - git stash pop
        - Applies the stash AND removes it from stash list
        - Use when we're done with that stash and don't need it anymore
        - Most commonly used

    - git stash apply
        - Applies the stash but KEEPS it in stash list
        - Use when we want to apply same stash to multiple branches
        - Safer — stash stays as backup


- When would you use stash in a real-world workflow?
    - When Urgent hotfix needed mid-work
    - On wrong branch situation : When we realize that we've been coding on main instead of feature branch so we will do stash then switch to correct branch and then there we will pop that stash
    - We have uncommitted changes but need to git pull latest , so we should do `git stash` first then `git pull` and then `git stash pop`
    - When Trying something risky, and want a safety net , then `stash` the current work first and then do the experiment and if it fails, do `git stash pop` and we'll come back to the point where we started


---

## Cherry Picking
1. Create a branch `feature-hotfix`, make 3 commits with different changes
    - Did it 
    - Created 3 files and did 3 different commit


2. Switch to `main`
    - `git switch main`


3. Cherry-pick **only the second commit** from `feature-hotfix` onto `main`
    - `git cherry-pick b38914f(commit hash/id)`
    - Picked only 2nd commit and added the file that file that I created in 2nd commit


4. Verify with `git log` that only that one commit was applied
    ```
    # In branch feature-hotfix 

    cb4ad29 (HEAD -> feature-hotfix) chore: Created file_3      # 3rd commit in feature-hotfix 
    b38914f chore: Created file_2                               # 2nd commit in feature-hotfix 
    136d890 chore: Created file_1                               # 1st commit in feature-hotfix 
    a7575d1 (main) Tried git stash      # main branch's last commit


    # In branch main

    a54a006 (HEAD -> main) chore: Created file_2       # Cherry-picked only 2nd commit from feature-hotfix branch
    a7575d1 Tried git stash                            # main branch's last commit
    ```


5. Answer in your notes:

- What does cherry-pick do?
    - Picks one specific commit from any branch and applies it to the current branch
    - Creates a new commit with same changes and even same message but different hash


- When would you use cherry-pick in a real project?
    - When we fixed a critical bug on main and we need same fix on feature branch too → cherry-pick that fix commit 
    - When a branch got cancelled but had one useful commit → cherry-pick just that commit 
    - Not all features are ready for release → cherry-pick only the ready ones to release branch 
    - Committed on main instead of feature → cherry-pick to correct branch → revert from main 
 

- What can go wrong with cherry-picking?
    - Cherry-picked code may conflict with current branch
    - Same change appears twice with different hashes — confusing history
    - Picked commit may depend on other commits you didn't pick
    - Hard to know where commit originally came from