# Day 04 – Linux Practice: Processes and Services

## Today’s goal is to _practice Linux fundamentals with real commands_.

### - _Checking running processes_

- **ps** : Show snapshot of currently running processes. It displays key information including Process ID (PID), terminal (TTY), CPU time used (TIME), and the command name (CMD). Often used with grep e.g., `ps aux`

- **ps aux** : Shows every single process running on this system right now, who is running it, and how much CPU and memory it is using

- **pgrep 'Process_Name'** : Shows the PID of all running processes with this name

- **top** : Display sorted information about processes

- **htop** : A beautiful, interactive, real-time process manager where you can monitor, search, filter and kill processes easily


### - *Inspecting systemd services*

- **systemctl** : Control the systemd system and service manager 

- **systemctl status 'service_name'** : It provides a detailed, real-time snapshot of the current state of a specific background service (or unit). 

- **systemctl start 'service_name'** : Immediately activate (start) a background service. 

- **sudo systemctl stop 'service_name'** : Immediately halts a running systemd service in Linux, making it inactive or dead. 

- **systemctl list-units** : Display a list of all currently loaded and active systemd units that the system manager is aware of. 

- **systemctl is-active 'service_name'** :  Prints: active or inactive

- **systemctl is-enabled 'service_name'** :  Prints: enabled or disabled

- **journalctl** : Print log entries from the systemd journal

- **journalctl -u 'service_name' / 'unit_name'** : Show messages for the specified systemd unit UNIT (such as a service unit), or for any of the units matched by PATTERN.


### - *Capture a small troubleshooting flow*

#### *Full Troubleshooting Flow Summary:*
```
systemctl status nginx        → ❌ Failed
        ↓
systemctl is-enabled nginx    → disabled
        ↓
journalctl -u nginx -n 50     → port 80 already in use
        ↓
lsof -i :80                   → apache2 is using port 80
        ↓
systemctl stop apache2        → free the port
        ↓
systemctl start nginx         → start nginx
        ↓
systemctl status nginx        → ✅ active (running)
        ↓
journalctl -u nginx -f        → ✅ no errors in live logs
        ↓
systemctl enable nginx        → ✅ will auto-start on reboot


In short: Troubleshooting flow = status → logs → find cause → fix → verify → enable
```






