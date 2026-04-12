# Day 15 – Networking Concepts: DNS, IP, Subnets & Ports

### Task : Build on Day 14 by understanding the building blocks of networking every DevOps engineer must know.


# DNS – How Names Become IPs

### 1. Explain in 3–4 lines: what happens when you type `google.com` in a browser?
    -   
```
You type: google.com
              │
              ▼
┌─────────────────────────────┐
│ Step 1: Browser Cache       │  "Have I visited before?"
│ → checks its own memory     │  ✅ found → skip all below
└─────────────────────────────┘
              │ not found
              ▼
┌─────────────────────────────┐
│ Step 2: OS Cache            │  checks /etc/hosts
│ → systemd-resolved          │  127.0.0.53:53
└─────────────────────────────┘
              │ not found
              ▼
┌─────────────────────────────┐
│ Step 3: Root DNS Server     │  "I don't know google.com
│ → 13 root servers globally  │   but .com is handled here →"
└─────────────────────────────┘
              │
              ▼
┌───────────────────────────────┐
│ Step 4: TLD Server (.com)     │  "google.com is handled
│ → Top Level Domain(TLD) server│   by Google's nameserver →"
└───────────────────────────────┘
              │
              ▼
┌─────────────────────────────┐
│ Step 5: Google's Nameserver │  "google.com = 142.250.x.x"
│ → authoritative answer      │  ✅ final answer!
└─────────────────────────────┘
              │
              ▼
        IP returned to browser
        142.250.x.x
              │
              ▼
        TCP + TLS + HTTP starts
        (DNS job is done ✅)
```


2. What are these record types? Write one line each:
    - `A`, `AAAA`, `CNAME`, `MX`, `NS`
        - A -> Address ---> Maps domain → IPv4 address
        - AAAA -> IPv6 Address ---> Maps domain → IPv6 address
        - CNAME -> Canonical Name ---> Maps domain → another domain (alias)
        - MX -> Mail Exchange ---> Tells where to deliver emails for this domain
        - NS -> Name Server ---> Tells which server is authoritative for this domain

            ```
            A/AAAA  →  house address        (where to find it)
            CNAME   →  nickname/alias       (another name for same place)
            MX      →  post office          (where to deliver mail)
            NS      →  directory authority  (who knows this area)
            ```
        - `dig A/AAAA/CNAME/MX/NS google.com` will tell you specifically 
        - `dig ANY google.com` will tell you everything at once

3. Run: `dig google.com` — identify the A record and TTL from the output
    - TTL = 300 seconds
        - → DNS resolver caches this answer for 300 seconds
        - → After 300s it asks again for fresh answer
        - → High TTL (86400) = cache for 1 day
        - → Low TTL  (60)    = refresh every 1 minute
    
    ```
    google.com. 300    IN    A    142.250.77.46
     │           │      │    │         │
     │           │      │    │         └── IP address
     │           │      │    └──────────── record type
     │           │      └───────────────── IN = Internet class
     │           └──────────────────────── TTL (seconds)
     └──────────────────────────────────── domain name
    ```

---


# IP Addressing

## 1. What is an IPv4 address? How is it structured? (e.g., `192.168.1.10`)
    - An IPv4 address is a unique number assigned to every device on a network so that data knows WHERE to go — like a home address for your computer.

    ```
        192  .  168  .  1  .  10
         ↑       ↑      ↑     ↑
        8bit    8bit   8bit  8bit

       └─────────────────────────┘
               4 octets
         (4 x 8 = 32 bits total)

        
        Each octet = 8 bits
        Range      = 0 to 255

        So valid IP:   0.0.0.0  →  255.255.255.255
    ``` 


## 2. Difference between **public** and **private** IPs — give one example of each

- A public IP address is used to identify your network on the global internet, while a private IP address is used to identify individual devices within a local network (like your home or office). 

### Example : 
    ```
    Private (not routable on internet):
    10.0.0.0/8        → big internal networks
    172.16.0.0/12     → AWS default VPC ← your server!
    192.168.0.0/16    → home routers
    
    Public (routable on internet):
    8.8.8.8           → Google DNS
    142.250.77.46     → google.com
    ```

## 3. What are the private IP ranges?

### `10.x.x.x`, `172.16.x.x – 172.31.x.x`, `192.168.x.x`

- **Ans :** 
    - Private IPs are reserved ranges that are NOT routable on the internet — only work within a local network (home, office, cloud VPC)

    ```
    10.x.x.x — Class A Private

    Range:   10.0.0.0  →  10.255.255.255
    Subnet:  /8
    Total IPs: 2²⁴ = 16,777,216 (16 million!)

    Used for:
    → Large enterprise networks
    → Kubernetes pod networks
    → Big cloud internal networks
    → Data centers
    ```

    ```
    172.16.x.x – 172.31.x.x — Class B Private

    Range:   172.16.0.0  →  172.31.255.255
    Subnet:  /12
    Total IPs: 2²⁰ = 1,048,576 (1 million!)
    
    Used for:
    → AWS default VPC  ← YOUR server 172.31.12.210!
    → Medium networks
    → Docker default bridge 172.17.0.0/16    
    ```

    ```
    192.168.x.x — Class C Private

    Range:   192.168.0.0  →  192.168.255.255
    Subnet:  /16
    Total IPs: 2¹⁶ = 65,536

    Used for:
    → Home WiFi routers      ← most common
    → Office networks
    → Small business networks
    ```
**Important :**
    
    How Private IPs reach Internet (NAT)

    Your laptop                    Internet
    192.168.1.10  →  Router  →  103.21.x.x (public)
         ↑               ↑            ↑
      private IP        NAT        public IP

    NAT = Network Address Translation
        = converts private → public when going out        


## 4. Run: `ip addr show` — identify which of your IPs are private

- I got 

    ```
    172.31.12.210  ✅ PRIVATE
    │
    └── falls in 172.16.0.0 – 172.31.255.255 range
        → AWS default VPC
        → NOT reachable from internet directly
        → AWS NAT Gateway converts to public IP

    172.17.0.1     ✅ PRIVATE
    │
    └── falls in 172.16.0.0 – 172.31.255.255 range
        → Docker internal bridge
        → only containers talk on this

    127.0.0.1      ⚠️ SPECIAL (not private, not public)
    │
    └── Loopback — only THIS machine talks to itself
        → never leaves your machine
    ```

- See ONLY IPv4 addresses : `ip -4 addr show`
    

---

# CIDR & Subnetting

## 1. What does `/24` mean in `192.168.1.0/24`?

**Ans :** 

- /24 means first 24 bits are the NETWORK part and remaining 8 bits (out of 32) are for DEVICES (hosts)

```
192.168.1.0/24

192.168.1  =  Street name      (fixed, network part)
.0 – .255  =  House numbers    (changes, host part)

/24 means:
→ First 24 bits = street (same for everyone)
→ Last  8 bits  = house number (unique per device)
```



## 2. How many usable hosts in a `/24`? a `/16`? a `/28`?

- `/24` has 254 usable hosts
- `/16` has 65534 usable hosts
- `/28` has 14 usable hosts
 
- Shortcut to calculate no. of usable hosts :  
    - `sudo apt install ipcalc`
    - Run : `ipcalc 192.168.1.0/24` (x.x.x.x/0-32)
 

## 3. Explain why do we subnet?

- Every device needs an individual IP and in this world there are almost 15+ Billion devices and if we calculate x.x.x.x where x = 0-255 we only have 4.2 Billion IPs and that's not enough so we do subnet that divides existing IPs into smaller groups
   
*Basically :*

```
IPv4 only has 4.2B public IPs which isn't
enough for 15B devices.
We solve this with NAT — one public IP
shared by many private devices.
We subnet to ORGANISE those private IPs
into smaller groups for security,
performance and efficiency —
```
***Subnetting solves organisation, security and traffic management.***


## 4. Quick exercise — filled in:

```
| CIDR | Subnet Mask    | Total IPs | Usable Hosts |
|------|----------------|-----------|--------------|
| /24  |255.255.255.0   | 256       | 254          |
| /16  |255.255.0.0     | 65536     | 65534        |
| /28  |255.255.255.240 | 16        | 14           |
```

# Ports – The Doors to Services

1. What is a port? Why do we need them?

- A port is a number that tells the operating system which application should receive the incoming network data

```
Without ports:
Data arrives at 192.168.1.10
OS confused → "which app gets this??"
Browser? SSH? Database? Game? 

With ports:
Data arrives at 192.168.1.10:80
OS knows → "port 80 = give to Nginx"
Data arrives at 192.168.1.10:22
OS knows → "port 22 = give to SSH"
```

2. Document these common ports:

```
| Port | Service |
|------|---------|
| 22   | ssh     |
| 80   | nginx   |
| 443  | https   |
| 53   | DNS     |
| 3306 | MySQL   |
| 6379 | Redis   |
| 27017| MongoDB |
```

3. Run `ss -tulpn` — match at least 2 listening ports to their services

- 0.0.0.0:22                   0.0.0.0:*         users:("sshd",pid=1355,fd=3)
- 0.0.0.0:80                   0.0.0.0:*         users:("nginx",pid=15384,fd=5)

---


# Putting All Together

Answer in 2–3 lines for each:

- You run `curl http://myapp.com:8080` — what networking concepts from today are involved?
    ```
    DNS     → "find the IP of myapp.com"
    IP      → "find the right machine"
    Port    → "find the right app on that machine"
    TCP     → "make sure connection is reliable"
    HTTP    → "ask the app for data"
    ```

- Your app can't reach a database at `10.0.1.50:3306` — what would you check first?

    - First I'll check is the database service running ? 
        - `systemctl status mysql` or `systemctl status postgresql`

    -  Then I'll check is it even reachable ? 
        ```
        Test the exact IP and port
        nc -zv 10.0.1.50 3306

        Two possible results:
          succeeded  → network OK, problem is app config
          refused    → port closed or DB not running
          timed out  → firewall blocking
        ``` 
    - Then I'll ping it : `ping 10.0.1.50`
        - If ping works → machine alive, port/app issue
        - If ping fails → routing/network issue or ICMP blocked

    - Then I'll check is DB Bound to right IP ? 
        `127.0.0.1:3306  ← only localhost can connect`

    - If issue still not found then I'll start debugging it....