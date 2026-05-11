# Day 28 – Revision Day: Everything from Day 1 to Day 27

## Task

You've covered a lot of ground in 27 days — DevOps fundamentals, Linux deep dives, Shell scripting, Python basics, Git & GitHub, and even your developer branding. Today, **stop and revise**. No new concepts. Just solidify what you've learned.

The goal is to identify gaps, revisit topics you struggled with, and make sure you can confidently explain and use everything covered so far.

---

## What You've Covered So Far

| Days | Topic | Key Concepts |
|------|-------|-------------|
| 1 | DevOps & Cloud Intro | What is DevOps, SDLC, Cloud basics |
| 2–7 | Linux Fundamentals | Architecture, commands, processes, systemd, file system hierarchy, troubleshooting, text files |
| 8 | Cloud Server Setup | Docker, Nginx, web deployment |
| 9–11 | Users, Permissions & Ownership | User/group management, file permissions, chown/chgrp |
| 12 | Revision Day 1 | Days 1–11 recap |
| 13 | Volume Management | LVM — physical volumes, volume groups, logical volumes |
| 14–15 | Networking | Fundamentals, DNS, IP, subnets, ports, hands-on checks |
| 16–18 | Shell Scripting | Basics, loops, arguments, error handling, functions |
| 19–20 | Shell Scripting Projects | Log rotation, backup, crontab, log analyzer |
| 21 | Shell Scripting Cheat Sheet | Personal reference guide |
| 22–25 | Git & GitHub | Init, branching, merge, rebase, stash, cherry pick, reset, revert, branching strategies |
| 26 | GitHub CLI | Managing GitHub from the terminal |
| 27 | GitHub Profile | Profile README, repo organization, developer branding |

---

## Challenge Tasks

### Task 1: Self-Assessment Checklist
Go through the checklist below. For each item, mark yourself honestly:
- **Can do confidently**  (D : Done)  
- **Need to revisit**     (R : Revise)
- **Haven't done yet**    (N : Not done yet)  

#### Linux
- [ D ] Navigate the file system, create/move/delete files and directories
- [ D ] Manage processes — list, kill, background/foreground
- [ D ] Work with systemd — start, stop, enable, check status of services
- [ D ] Read and edit text files using vi/vim or nano
- [ D ] Troubleshoot CPU, memory, and disk issues using top, free, df, du
- [ R ] Explain the Linux file system hierarchy (/, /etc, /var, /home, /tmp, etc.)
- [ D ] Create users and groups, manage passwords
- [ D ] Set file permissions using chmod (numeric and symbolic)
- [ D ] Change file ownership with chown and chgrp
- [ D ] Create and manage LVM volumes
- [ R ] Check network connectivity — ping, curl, netstat, ss, dig, nslookup
- [ R ] Explain DNS resolution, IP addressing, subnets, and common ports

#### Shell Scripting
- [ D ] Write a script with variables, arguments, and user input
- [ D ] Use if/elif/else and case statements
- [ R ] Write for, while, and until loops
- [ D ] Define and call functions with arguments and return values
- [ D ] Use grep, awk, sed, sort, uniq for text processing
- [ D ] Handle errors with set -e, set -u, set -o pipefail, trap
- [ D ] Schedule scripts with crontab

#### Git & GitHub
- [ D ] Initialize a repo, stage, commit, and view history
- [ D ] Create and switch branches
- [ D ] Push to and pull from GitHub
- [ D ] Explain clone vs fork
- [ D ] Merge branches — understand fast-forward vs merge commit
- [ D ] Rebase a branch and explain when to use it vs merge
- [ D ] Use git stash and git stash pop
- [ D ] Cherry-pick a commit from another branch
- [ D ] Explain squash merge vs regular merge
- [ D ] Use git reset (soft, mixed, hard) and git revert
- [ R ] Explain GitFlow, GitHub Flow, and Trunk-Based Development
- [ D ] Use GitHub CLI to create repos, PRs, and issues

---

### Task 2: Revisit Your Weak Spots
1. Pick **3 topics** from the checklist where you marked "Need to revisit"
2. Go back to that day's challenge and redo the hands-on tasks
3. Document what you re-learned in `day-28-notes.md`

---

### Task 3: Quick-Fire Questions
Answer these from memory (no Googling). Then verify your answers:

1. What does `chmod 755 script.sh` do?
2. What is the difference between a process and a service?      R
3. How do you find which process is using port 8080?    
4. What does `set -euo pipefail` do in a shell script?
5. What is the difference between `git reset --hard` and `git revert`?
6. What branching strategy would you recommend for a team of 5 developers shipping weekly?  Learn
7. What does `git stash` do and when would you use it?      R
8. How do you schedule a script to run every day at 3 AM?   
9. What is the difference between `git fetch` and `git pull`?   
10. What is LVM and why would you use it instead of regular partitions?    R

---

### Task 4: Organize Your Work
1. Make sure all your daily submissions (day-1 through day-27) are committed and pushed
2. Check that your `git-commands.md` is up to date
3. Check that your shell scripting cheat sheet is complete
4. Verify your GitHub profile and repos are clean (from Day 27)

---

### Task 5: Teach It Back
Pick **one topic** you've learned and write a short explanation (5-10 lines) as if you're teaching it to someone who has never heard of it. Add it to your `day-28-notes.md`.

Examples:
- Explain Git branching to a non-developer
- Explain file permissions to a new Linux user
- Explain what a crontab is and why sysadmins use it

Teaching is the best test of understanding.

---

## Submission
1. Add your `day-28-notes.md` to `2026/day-28/`
2. Push to your fork
3. Make sure all previous days are pushed and up to date

---

## Learn in Public

Share your self-assessment results or your "teach it back" explanation on LinkedIn. Be honest about what you found easy and what you need to work on.

`#90DaysOfDevOps` `#DevOpsKaJosh` `#TrainWithShubham`

Happy Learning!
**TrainWithShubham**
