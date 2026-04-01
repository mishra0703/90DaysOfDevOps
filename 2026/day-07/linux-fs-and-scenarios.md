# Day 07 – Linux File System Hierarchy & Scenario-Based Practice


### *Part 1: Linux File System Hierarchy*

- `/` : Root Directory
    - The top of the entire filesystem tree. 
    - Every single file, folder, and device on the system lives under this. 
    - There is only one root on a Linux system.

- `/home` : User Home Directories
    - Every regular user gets their own folder here. 
    - Personal files, shell configs, downloads — all live here. 
    - On AWS EC2, the default user is ec2-user or ubuntu, so their home is /home/ec2-user or /home/ubuntu.

- `/root` : Root User's Home
    - The superuser (root)'s personal home directory. 
    - It is deliberately kept separate from /home for security — even if /home is on a broken/unmounted disk, root can still log in and fix things.
    
- `/etc` : Configuration Files
    - The brain of the system. 
    - All system-wide configuration files live here — no binaries, just plain text config files. 
    - If you want to change how any service or system behaves, you come here.

- `/var/log` : Log files
    - var stands for variable data — files that constantly grow and change. 
    - /var/log is where every service writes its logs. 
    - Very Usefull for Debugging issues.

- `/tmp` : Temporary Files
    - A scratch space for temporary files. 
    - Any user or process can write here. 
    - Files here are automatically deleted on reboot (or sometimes after a few days). 
    - Never store anything important here.
    - ⚠️ A common **security concern** — `/tmp` is world-writable, so on production servers it's often mounted with `noexec` in `/etc/fstab` to prevent attackers from running malicious scripts from here.

![alt text](image.png) 

