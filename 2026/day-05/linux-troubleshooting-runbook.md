# Day 05 – Linux Troubleshooting Drill: CPU, Memory, and Logs

## Go to Runbook

## - *Know about your Environment*

- `uname -a` : Shows everything about the system like — kernel version, hostname, architecture and OS in one line

- `cat /etc/os-release` : Displays detailed information about the Linux distribution installed on your system — such as the OS name, version, and ID.

## - *Create your runbook folder*

- `mkdir /tmp/runbook-demo`
- `cp /etc/hosts /tmp/runbook-demo/hosts-copy`
- `ls -l /tmp/runbook-demo`


## - *CPU & Memory*

- `ps -o pid,pcpu,pmem,comm -p <PID>` → CPU: 0.0%, MEM: 0.1% (normal)
- `free -h` → 512MB used out of 1.8GB (healthy)
- ⚠️ Simulated spike using `yes` → CPU hit 90%
- Fixed by → `killall yes`



## - *Disk & IO*

- `df -h` → Root partition 26% used (healthy)
- `du -sh /var/log` → Tells the total size of the /var/log folder in a readable format ; if <50M then it's (normal)


## - *Network*

- `ss -tulpn` → Shows all open ports and which service is listening on them ; Ex : `sshd listening on port 22`

- `ping google.com` → avg 1.2ms (healthy)


## - *Logs*

- `journalctl -u ssh -n 50` → Check recent logs for the error  

- `tail -n 50 /var/log/auth.log` → Check for any suspicious login attempt made recently. Like : 2 failed login attempts from unknown IP 


## - *Quick Findings*

### - Write status for checks , like : 
    - Service is healthy and running normally
    - CPU spike was simulated and resolved
    - Minor: 2 failed SSH login attempts in auth.log (could be bots)


## - *If any one of the following happens & Worsens*

### - *Then follow this*

- If CPU stays high → `ps aux --sort=-%cpu` to find the culprit process and kill it
- If disk fills up → `du -sh /var/log/*` to find largest log files and rotate/delete them
- If SSH fails to connect → `systemctl restart ssh` and check `journalctl -u ssh -f` for live errors
