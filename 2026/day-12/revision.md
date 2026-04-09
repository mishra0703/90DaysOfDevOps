# Day 12 – Breather & Revision (Days 01–11)


### 1) Which 3 commands save you the most time right now, and why?  

- 1. tail -n on log files
    - I can see the recent logs of a service and decide what to do next

- 2. id to verify users and groups instantly
    - I can check which user is in which group so that no wrong group will be assigned to wrong user

- 3. usermod -aG to manage group membership
    - I can modify usre's group so that I can select my choice of team in a group for a particular work

---

### 2) How do you check if a service is healthy? List the exact 2–3 commands you’d run first.  

- `top` , `htop` or `ps aux | grep serviceName`
- `systemctl status service`
- `systemctl is-active service` or `systemctl is-enabled service`
- `journalctl -u service`

---

###  3) How do you safely change ownership and permissions without breaking access? Give one example command.  
- By never giving ownership to other user , because any other user can lock out the actual owner so always give ownership to yourself first not untill you trust someone  
- Ex : If I did `chown otherUser:otherGroup dirName` you'll ended up in a situation where you locked out yourself if the otherUser is not giving you ownership back

---

###  4) What will you focus on improving in the next 3 days?

- Will do more and more practice as doing it right now 
- Will do 90DaysOfDevOps challenge to practice and learn by hands-on experience not just theoritically or watching lecture like a copy-paster

