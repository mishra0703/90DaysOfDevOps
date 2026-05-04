# Day 25 – Git Reset vs Revert & Branching Strategies

## Task

We'll learn how to **undo mistakes** safely — one of the most important skills in Git. We'll also explore **branching strategies** used by real engineering teams to manage code at scale.

---

## Git Reset — Hands-On

1. Make 3 commits in your practice repo (commit A, B, C)
    - Did it

2. Use `git reset --soft` to go back one commit — what happens to the changes?
    - Moves the branch pointer back one commit
    - Our changes from that commit are kept and moved to the staging area (ready to commit again)
    - Working directory files remain untouched
    - Useful when you want to redo or reword(rename) a commit


3. Re-commit, then use `git reset --mixed` to go back one commit — what happens now?
    - Moves the branch pointer back one commit
    - Our changes are kept but moved to the working directory (unstaged/one step more backward) 
    - Staging area is cleared
    - This is the default behavior of git reset 


4. Re-commit, then use `git reset --hard` to go back one commit — what happens this time?
    - Moves the branch pointer back one commit
    - Our changes are completely deleted — both from staging and working directory
    - Cannot be recovered easily (unless via git reflog)
    - Use with caution — destructive operation


5. Answer in your notes:
- What is the difference between `--soft`, `--mixed`, and `--hard`?
    - `--soft` → uncommits, keeps changes in staging area
    - `--mixed` → uncommits, keeps changes in working directory (unstaged)
    - `--hard` → uncommits, deletes changes entirely


- Which one is destructive and why?
    - `--hard` is destructive — it wipes our code changes permanently with no undo (unless git reflog)


- When would you use each one?
    - `--soft` → want to recommit with changes already staged
    - `--mixed` → want to re-stage selectively before recommitting
    - `--hard` → want to completely discard a bad commit and its changes


- Should you ever use `git reset` on commits that are already pushed?
    - No — it rewrites history, which breaks other people's branches
    - Use git revert instead — it adds a new commit that undoes changes safely
    - Only `reset` , pushed commits if you're the sole developer and force-push with caution: `git push --force`


---

## Git Revert — Hands-On

1. Make 3 commits (commit X, Y, Z)
    - Did it


2. Revert commit Y (the middle one) — what happens?
    - creates a new commit that undoes the changes of the specified commit
    - Original commit is still there in history


3. Check `git log` — is commit Y still in the history?
    - Yes — both the original and the new "revert commit" appear in history
    - History is never rewritten


4. Answer in your notes:
- How is `git revert` different from `git reset`?
    - `revert` → adds a new undo commit, keeps history intact
    - `reset` → removes commits, rewrites history


- Why is revert considered **safer** than reset for shared branches?
    - `revert` doesn't rewrite history → no conflict with teammates' local copies
    - Everyone can simply git pull the revert commit normally
    - But `reset` on shared branches forces others to re-clone or force-pull


- When would you use revert vs reset?
    - `revert` → commit is already pushed / working on a shared branch
    - `reset` → commit is local only / working alone and want clean history


---

## Reset vs Revert — Summary

Comparison :

| | `git reset` | `git revert` |
|---|---|---|
| What it does | Moves branch pointer back, removes commits | Creates a new commit that undoes changes |
| Removes commit from history? | Yes | No |
| Safe for shared/pushed branches? | No | Yes |
| When to use | Local-only commits, cleaning up before push | Already pushed commits, shared branches |

---

## Branching Strategies

Research the following branching strategies and document each in your notes with:

### 1. *GitFlow* — develop, feature, release, hotfix branches

- How it works 
    - Has multiple long-lived branches: `main`, `develop`, `feature/*`, `release/*`, `hotfix/*`
    - Features are built on `feature/` branches → merged into `develop` → then into `release/` → finally into `main`

- A simple diagram or flow
    ```
    main
     └── develop
           ├── feature/login
           ├── feature/payment
           └── release/v1.0
                 └── hotfix/fix-crash → main
    ```

- When/where it's used
    - Large teams with scheduled releases (e.g., banks, enterprise software)

- Pros and cons
    - Pros : Very structured, clear separation of work
    - Cons : Complex, slow — too much overhead for small teams

---

### 2. *GitHub Flow* — simple, single main branch + feature branches

- How it works 
    - Only one main branch + short-lived feature branches
    - Feature branch → Pull Request → review → merge into main → deploy

- A simple diagram or flow 
    ```
    main
    ├── feature/login → PR → main → deploy
    ├── feature/dark-mode → PR → main → deploy
    ```

- When/where it's used
    - Startups, small-medium teams shipping continuously
    
- Pros and cons
    - Pros : Simple, fast, easy to understand
    - Cons : Needs good CI/CD — broken code , hits main fast


---

### 3. **Trunk-Based Development** — everyone commits to main, short-lived branches

- How it works 
    - Everyone commits directly to main (trunk) or uses very short-lived branches (< 1 day)
    - Heavy use of feature flags to hide incomplete features 

- A simple diagram or flow 
    ```
    main ← dev1 commits
    main ← dev2 commits
    main ← dev3 commits (feature hidden via flag)
    # deploy continuously
    ```

- When/where it's used
    - Big tech companies (Google, Meta) with strong CI/CD pipelines

- Pros and cons
    - Pros : Fastest delivery, no merge conflicts
    - Cons : Needs discipline + feature flags + strong automated testing


---

### 4. Answer:

- Which strategy would you use for a startup shipping fast?
    - Startup shipping fast → GitHub Flow
        - Simple, quick PRs, deploy often, no overhead

- Which strategy would you use for a large team with scheduled releases?
    - Large team with scheduled releases → GitFlow
        - Structured, controlled, handles multiple versions


- Which one does your favorite open-source project use? (check any repo on GitHub)
    - React (facebook/react) — GitHub Flow
    - Kubernetes — GitFlow
    - Linux Kernel — Trunk-Based
    

---

## Hints
- `git reflog` is your safety net — it shows everything Git has done, even after a hard reset

