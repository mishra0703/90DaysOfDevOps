# Day 11 – File Ownership Challenge (chown & chgrp)

### Task : Master file and directory ownership in Linux.


## Files & Directories Created

- `Devops.txt`
- `notes.txt`
- `/project`
- `script.sh`
- `/bank-heist`
    - `access-codes.txt`
    - `blueprints.pdf`
    - `escape-plan.txt`
- `project-config.yaml`  
- `team-notes.txt`
- `/app-logs`    
- `/heist-project`     
    - `/plans`
        - `strategy.conf`  
    - `/vault`
        - `gold.txt`




## Ownership Changes

- Before :  -r--r--r-- 1 ubuntu  ubuntu   37 Apr  9 12:45 Devops.txt
- After :  -r--r--r-- 1 tokyo  ubuntu   37 Apr  9 12:45 Devops.txt
---
- Before : -rw-r----- 1 ubuntu ubuntu   44 Apr  9 12:46 notes.txt
- After : -rw-r----- 1 berlin ubuntu   44 Apr  9 12:46 notes.txt
---
- Before : -rw-rw-r-- 1 ubuntu ubuntu   21 Apr  9 14:06 team-notes.txt 
- After : -rw-rw-r-- 1 ubuntu heist-team   21 Apr  9 14:06 team-notes.txt
---
- Before :  -rw-rw-r-- 1 ubuntu ubuntu    0 Apr  9 14:14 project-config.yaml
- After :  -rw-rw-r-- 1 professor heist-team    0 Apr  9 14:14 project-config.yaml
---
- Before : drwxrwxr-x 2 ubuntu  ubuntu 4096 Apr  9 14:15 app-logs 
- After : drwxrwxr-x 2 berlin  heist-team 4096 Apr  9 14:15 app-logs
---
- **Before** : 

heist-project:

drwxrwxr-x 2 ubuntu ubuntu 4096 Apr  9 14:20 plans
drwxrwxr-x 2 ubuntu ubuntu 4096 Apr  9 14:19 vault

heist-project/plans:

-rw-rw-r-- 1 ubuntu ubuntu 0 Apr  9 14:20 strategy.conf

heist-project/vault:

-rw-rw-r-- 1 ubuntu ubuntu 0 Apr  9 14:19 gold.txt

- **After** : we ran `sudo chown -R professor:planners heist-project/`

heist-project:

drwxrwxr-x 2 professor planners 4096 Apr  9 14:20 plans
drwxrwxr-x 2 professor planners 4096 Apr  9 14:19 vault

heist-project/plans:

-rw-rw-r-- 1 professor planners 0 Apr  9 14:20 strategy.conf

heist-project/vault:

-rw-rw-r-- 1 professor planners 0 Apr  9 14:19 gold.txt



## Commands Used

### View ownership
`ls -l filename`

### View ownership of files & directories inside of a directory 
`ls -lR directoryName`  : will show ownerships of files and folders inside directoryName

### Change owner only
`sudo chown newowner filename`

### Change group only
`sudo chgrp newgroup filename`

### Change both owner and group
`sudo chown owner:group filename`

### Recursive change (directories)
`sudo chown -R owner:group directory/`

### Change only group with chown
`sudo chown :groupname filename`



## What I Learned

- I learned that if you want to share any file or directory then it should be present in root not in any user's home directory like /home/ubuntu/Practice/file you can't access this by other user you've to change group or owner permission for ubuntu

- I learned that we can simultaneously change owner and group of a file or directory at a same time using `sudo chown user:group fileName/dirName`

- I learned that using -R flag we can recursively change owner and group of files and directory inside a parent directory. Ex :  `sudo chown -R professor:planners heist-project/`

- I learned that we can view permissions of all files and subdirectories inside a directory using : `ls -lR heist-project/` command