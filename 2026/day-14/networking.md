# Day 14 – Networking Fundamentals & Hands-on Checks

### Task : Get comfortable with core networking concepts and the commands you’ll actually run during troubleshooting.

---

## Quick Concepts

---

- OSI layers (L1–L7) vs TCP/IP stack (Link, Internet, Transport, Application)
  - TCP/IP was built around real protocols (TCP, IP, UDP); OSI is protocol-independent
  - OSI is stricter — each layer talks only to adjacent layers; TCP/IP is more flexible

```
    OSI                         TCP/IP
─────────────────────────────────────
L7 Application   ┐
L6 Presentation  ├──>  Application
L5 Session       ┘
L4 Transport     ────>  Transport
L3 Network       ────>  Internet
L2 Data Link     ┐
L1 Physical      ┴──>  Link
```

---

- Where **IP**, **TCP/UDP**, **HTTP/HTTPS**, **DNS** sit in the stack
  - IP sits on L3 or on Network Level , TCP/UDP sits on L4 or on Transport Level , HTTP/HTTPS and DNS sits on L7 or on Application Level

```
TCP/IP Layer       OSI Layer        Protocols
─────────────────────────────────────────────────
Application        L7 Application   HTTP, HTTPS, DNS, FTP, SMTP
                   L6 Presentation  SSL/TLS (part of HTTPS)
                   L5 Session       (handled by OS)
Transport          L4 Transport     TCP, UDP
Internet           L3 Network       IP (IPv4, IPv6)
Link               L2 Data Link     Ethernet, MAC
                   L1 Physical      Cables, WiFi signals
```

### Full Request Flow (visiting google.com)

```
You type google.com
        ↓
DNS (L7/UDP) → resolves to IP
        ↓
TCP Handshake (L4) → SYN, SYN-ACK, ACK
        ↓
TLS Handshake (L6) → for HTTPS
        ↓
HTTP GET request (L7)
        ↓
IP routes packets (L3)
        ↓
Ethernet/WiFi delivers frames (L2/L1)
```

---

- One real example: “`curl https://example.com` = App layer over TCP over IP”

[realWorldExample](/2026/day-14/realWorldExample.txt)

---

## Hands-on Checklist 

- **Identity:** `hostname -I` (or `ip addr show`) — note your IP.
  - `hostname -I` --> tells AWS EC2 primary network ip and other running services ip too (Shows all ip addresses)
  - `ip addr show` --> Show network interfaces and associated IP addresses

- **Reachability:** `ping <target>` — mention latency and packet loss.
  - `ping google.com` : Send ICMP(Internet Control Message Protocol) echo request packets and waits for replies to diagnose connectivity.
  - 0% packet loss & time 9180ms

- **Path:** `traceroute <target>` (or `tracepath`) — note any long hops/timeouts. 
   - `traceroute scandinee.vercel.app` : 30 hops max & 13.872 ms  
   - `tracepath scandinee.vercel.app` :
  ```
  1: ip-172-31-0-1.ec2.internal 0.090ms pmtu 1500
  1: no reply
  2: no reply
  3: no reply
  4: no reply
  5: 241.0.11.200 0.364ms
  6: 240.0.160.9 0.367ms
  7: 240.0.184.74 1.200ms asymm 4
  8: 240.0.184.48 0.838ms asymm 5
  9: 240.0.184.52 0.716ms
  10: no reply
  ```

- **Ports:** `ss -tulpn` (or `netstat -tulpn`) — list one listening service and its port.
    - Run `ss -tulpn` or `netstat -tulpn` with sudo to see Program Names
    - `sudo ss -tulpn` `tcp   LISTEN   0   511   0.0.0.0:80   0.0.0.0:*   users:(("nginx",pid=1745,fd=5),("nginx",pid=1744,fd=5),("nginx",pid=1743,fd=5))`
    - `sudo netstat -tulpn` `tcp6       0      0 :::80                   :::                    LISTEN      1743/nginx: master`

- **Name resolution:** `dig <domain>` or `nslookup <domain>` — record the resolved IP.
    - `dig google.com`: Perform DNS lookup (List all the ip addressses mapped to the domain name)
    - `nslookup google.com` : Query DNS server (Do similar task as dig < domain >)


- **HTTP check:** `curl -I <http/https-url>` — note the HTTP status code.
    - fetch only the HTTP headers, not the actual page body/HTML content.


- **Connections snapshot:** `netstat -an | head` — count ESTABLISHED vs LISTEN (rough).
    - `netstat -an | head` : Show a quick snapshot of all network connections on the working machine right now


---

## Mini Task: Port Probe & Interpret

### 1) Identify one listening port from `ss -tulpn` (e.g., SSH on 22 or a local web app).  
   - tcp   LISTEN &nbsp;   0  &nbsp;  511  &nbsp;  0.0.0.0:**80**  &nbsp;   0.0.0.0:*  &nbsp;  ("nginx",pid=1745,fd=5)  
   - Nginx is running on port 80        

### 2) From the same machine, test it: `nc -zv localhost <port>` (or `curl -I http://localhost:<port>`).  

   - After getting this from ss -tulpn

```
tcp     LISTEN   0        4096            127.0.0.53%lo:53             0.0.0.0:*      users:(("systemd-resolve",pid=342,fd=15))                                        
tcp     LISTEN   0        4096               127.0.0.54:53             0.0.0.0:*      users:(("systemd-resolve",pid=342,fd=17))                                        
tcp     LISTEN   0        511                   0.0.0.0:80             0.0.0.0:*      users:("nginx",pid=1745,fd=5)  
tcp     LISTEN   0        4096                  0.0.0.0:22             0.0.0.0:*      users:(("sshd",pid=1355,fd=3),("systemd",pid=1,fd=95))                           
tcp     LISTEN   0        4096                127.0.0.1:33005          0.0.0.0:*      users:(("containerd",pid=740,fd=9))                                              
tcp     LISTEN   0        511                      [::]:80                [::]:*      users:("nginx",pid=1745,fd=6)
```

- I checked every port **80**,**22**,**9999**,**33005** and **53** (For port 53 The service is at 127.0.0.53:53, not 127.0.0.1:53! || So we have to run this command `nc -zv 127.0.0.53 53`)


### 3) Write one line: is it reachable? If not, what’s the next check? (e.g., service status, firewall).

- Yes , it is reachable. If it it's not reachable then weather we're checking on the wrong ip or maybe the service is not running 

   - If we were checking on the wrong ip  
        ```
        127.0.0.1:8080   ← only localhost
        0.0.0.0:8080     ← all interfaces
        172.31.x.x:8080  ← specific interface only
        ```
   - If service is not running or something 

        ```
        systemctl status nginx
        systemctl status docker
        systemctl status mysql
        ```

   - Check for Logs 
        
        ```
        journalctl -u nginx --no-pager | tail -20
        journalctl -u mysql --no-pager | tail -20

        ```

---

## Reflection 

- Which command gives you the fastest signal when something is broken?
    - curl -I https://yoursite.com &nbsp; *Why it's fastest — becoz this one command tells you :*
        - Is the server reachable? (network)
        - Is DNS resolving? (DNS)
        - Is port 443 open? (firewall)
        - Is the app responding? (application)
        - What HTTP status? (app health)         

- What layer (OSI/TCP-IP) would you inspect next if DNS fails? If HTTP 500 shows up?
    - If DNS fails I'll inspect L3 and L7 &nbsp; *Why these two layers?*
        - L7 → DNS is an APPLICATION protocol (like HTTP) the query/response format, resolver config all happen at application layer
        - L3 → Even if DNS app is fine, if network can't ROUTE packets to DNS server it still means DNS fails

    - If HTTP 500 shows up , I'll inspect L7 Only &nbsp; *Why ?*
        - HTTP 500 means all lower layers are FINE:
        ```
          L1 Physical     ✅ cables/wifi working
          L2 Data Link    ✅ MAC/frames delivered
          L3 Network      ✅ IP routing worked
          L4 Transport    ✅ TCP connected on port 443
          L6 Presentation ✅ TLS handshake succeeded
          L7 Application  ❌ YOUR APP BROKE HERE
        ```

- Two follow-up checks you’d run in a real incident.
    - First check     ping 8.8.8.8 (For DNS Issue)      app logs (For HTTP error)
    - Second check    dig google.com (For DNS)          DB status (For HTTP)
