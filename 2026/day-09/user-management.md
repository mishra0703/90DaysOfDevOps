# Day 09 – Linux User & Group Management Challenge



### Task : *Today's goal is to **practice user and group management** by completing hands-on challenges.*



## Users & Groups Created
- Users: tokyo, berlin, professor, nairobi
    - To create an user : `sudo useradd -m Username` -m makes a home directory for the user
    - `sudo useradd Username` also creates an user but doesn't create it's home dierctory
    - By default new user profile will be locked , to use it you have to set a password first by using `sudo passwd Username`
    - To check weather new user is created successfully or not , run `cat /etc/passwd` or go to `cd /home` you'll see the new user's name there
    - So the flow is always : 
        - sudo useradd -m Prem      # Create user (locked by default)
        - sudo passwd Prem           # You MUST do this to enable login
        - su - Prem                  # Now login works
    - Also by running `id Username` we get the information about the particular user , like it's UID(user Id) , it's GID (group Id) and the group it is part of  

- Groups: developers, admins, project-team
    - To create a group : `sudo groupadd Groupname` 
    - To add a user in the group : `sudo usermod -aG group_name username`
    - To check weather the group is created or not , run `cat /etc/group`
    - To Rename a group	`sudo groupmod -n new_name old_name`
    - Remove a user	`sudo gpasswd -d username group_name`
    - Delete a group `sudo groupdel group_name`


## Group Assignments
[List who is in which groups]

- To check this run `cat /etc/group`
    - developers : tokyo & berlin
    - admins : professor & berlin
    - project-team : nairobi & tokyo


## Directories Created
[List directories with permissions]

- To give permission to a particular group for a particular file/folder we use 
    - `chgrp group_name file_name` changes group ownership (Give the ownership of the file to the group)
    - `chown user_name:group_name file_name` changes user and group ownership at the same time (Gives the ownership of the file to the user and the group at the same time) 
    - `chown :group_name file_name` is same as `chgrp group_name file_name`

- opt/dev-projects : drwxrwxr-x 2 root developers   4096 Apr  8 23:32 dev-projects
    - user and group has every permission (read , write & execute)
    - Owner is root
    - Group is developers (Members of developers (tokyo & berlin) can do read , write and execute)

- opt/team-workspace : drwxrwxr-x 2 root project-team 4096 Apr  8 23:40 team-workspace
    - user and group has every permission (read , write & execute)
    - Owner is root
    - Group is project-team (Members of project-team (nairobi and tokyo) can do read , write and execute)


## What I Learned
[3 key points]

- I learned about user management , like how to create new user with and without /home directory , how to delete it how to add them in a group

- I learned about group management too , like how to create groups and how to add a user in a group and how to give permission of a particular file/folder to a group

- I learned about how different users can work on a same shared directory or file and how can we protect the same shared directory/file from other users at the same time