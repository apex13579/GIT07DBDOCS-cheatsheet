# 🛡️ Master Security & Network Cheatsheet

> **Role:** Senior Pen Tester · Network Specialist · Developer  
> **Scope:** Zero Trust · OPNsense · Cisco CLI · Bash · SQL · Python · Markdown · Git  
> **Model:** Zero Trust Architecture · No Exposed Ports · Defense in Depth

---

## Table of Contents

1. [Zero Trust Architecture](#1-zero-trust-architecture)
2. [OPNsense Firewall](#2-opnsense-firewall)
3. [Cisco CLI](#3-cisco-cli)
4. [Bash](#4-bash)
5. [SQL](#5-sql)
6. [Python](#6-python)
7. [Markdown](#7-markdown)
8. [Git](#8-git)

---

## 1. Zero Trust Architecture

> **Core Principle:** Never trust, always verify. Every device, user, and packet is untrusted by default regardless of network position. Authenticate everything. Authorize explicitly. Minimize blast radius.

### 1.1 The Six Pillars

| Pillar | Implementation |
|---|---|
| **Deny All by Default** | Default firewall policy is `BLOCK ALL`. Every rule is an explicit allow. No rule = traffic dies. |
| **No Exposed Ports** | Zero listening services on WAN. All remote access via WireGuard VPN only. No SSH, no GUI, no ICMP on WAN. |
| **Micro-Segmentation** | Each service lives in its own VLAN. Inter-VLAN traffic denied unless explicitly permitted per-flow. |
| **Least Privilege** | Every rule allows the minimum source, destination, and port required. Host-to-host over subnet-to-subnet. |
| **Continuous Verification** | Suricata IDS/IPS on every interface. Zeek for behavioral analysis. All logs to SIEM. |
| **Encrypted Everywhere** | TLS 1.3 minimum on all traffic. DNS over TLS. No plaintext protocols on any segment. |

---

### 1.2 VLAN Segmentation Model

| VLAN | Name | Purpose | Trust Level |
|---|---|---|---|
| VLAN 10 | Management | OPNsense GUI, switches, APs, Proxmox. No internet access. | 🔴 CRITICAL |
| VLAN 20 | Servers | Vaultwarden, Gitea, ntfy, Portainer. Outbound only. | 🟠 HIGH |
| VLAN 30 | Trusted | Primary workstations. Access to servers via explicit rules only. | 🟡 MEDIUM |
| VLAN 40 | IoT | IP cameras, printers, smart devices. No lateral movement. | 🔴 UNTRUSTED |
| VLAN 50 | Guest | Internet only. Isolated from all VLANs. Filtered DNS. | 🔴 UNTRUSTED |
| VLAN 60 | VPN | WireGuard tunnel endpoint. Treated as Trusted post-auth. | 🟢 VERIFIED |
| VLAN 70 | Sandbox | Pentesting, experimental VMs. Fully isolated. | 🔴 QUARANTINE |

---

### 1.3 Firewall Rule Order

> ⚠️ **Critical:** OPNsense evaluates rules top-to-bottom and stops at first match. BLOCK rules for RFC1918 inter-VLAN must come before any ALLOW rules. Every interface ruleset ends with an explicit block-all logging rule.

| Priority | Action | Rule | Notes |
|---|---|---|---|
| 1 | `BLOCK` | Anti-lockout bypass | Block GUI access from non-management VLANs |
| 2 | `BLOCK` | RFC1918 inter-VLAN catch-all | Block all traffic between VLANs by default |
| 3 | `ALLOW` | Explicit inter-VLAN | Specific host-to-host allows only — listed individually |
| 4 | `ALLOW` | DNS to Unbound resolver only | Block all other DNS — prevent DNS exfiltration |
| 5 | `ALLOW` | NTP UDP 123 | Internal NTP server only |
| 6 | `ALLOW` | WireGuard UDP 51820 inbound WAN | Only open port on WAN interface |
| 7 | `ALLOW` | Trusted → Internet 443 | Via HAProxy reverse proxy only |
| 8 | `ALLOW` | IoT → specific update IPs | By IP, not domain. Explicit list only. |
| 9 | `ALLOW` | Guest → Internet 80/443 | No RFC1918 destinations permitted |
| 10 | `BLOCK + LOG` | Everything else | Every hit is a forensic data point |

---

### 1.4 WAN Interface Zero Exposure

```bash
# WAN interface — absolute minimum exposure

# Block RFC1918 arriving on WAN (anti-spoofing)
BLOCK IN  WAN  src:10.0.0.0/8       → DROP + LOG
BLOCK IN  WAN  src:172.16.0.0/12    → DROP + LOG
BLOCK IN  WAN  src:192.168.0.0/16   → DROP + LOG

# Block all ICMP on WAN — no ping response to internet
BLOCK IN  WAN  proto:ICMP           → DROP

# ONLY open port — WireGuard VPN
ALLOW IN  WAN  proto:UDP  dst-port:51820  → WireGuard tunnel

# Block everything else inbound on WAN
BLOCK IN  WAN  src:any  dst:any     → DROP + LOG

# Outbound NAT — only from internal RFC1918 ranges
ALLOW OUT WAN  src:10.0.0.0/8  dst:!RFC1918  → NAT
```

---

## 2. OPNsense Firewall

> FreeBSD-based open source firewall/router. GUI at `https://192.168.1.1`. Always export config backup before major changes: **System → Configuration → Backups**

### 2.1 Initial Hardening — Do This First

| Setting | Where | Why |
|---|---|---|
| Disable WebGUI on WAN | System → Settings → Administration | GUI must never be reachable from internet |
| Change default admin password | System → Access → Users | Default creds are public knowledge |
| Enable HTTPS only | System → Settings → Administration | Never allow HTTP access to GUI |
| Restrict GUI to VLAN 10 | System → Settings → Administration → Listen Interfaces | Limits GUI to management VLAN only |
| Enable SSH key-only auth | System → Settings → Administration → SSH | Disable password SSH entirely |
| Disable SSH on WAN | System → Settings → Administration | SSH never reachable from internet |
| Enable bogon blocking | Interfaces → WAN → Block bogon networks | Drops unroutable source IPs |
| Enable Suricata IDS/IPS | Services → Intrusion Detection | Active threat detection and blocking |
| Enable Unbound DNS | Services → Unbound DNS | Local recursive resolver — no DNS leaks |
| Enable DNS-over-TLS | Services → Unbound DNS → DNS over TLS | Encrypts all upstream DNS queries |
| Set NTP server | Services → NTP | Internal time sync — do not use public pool on management VLAN |

---

### 2.2 VLAN Configuration

| Task | Path | Notes |
|---|---|---|
| Create VLAN | Interfaces → Other Types → VLAN | Set parent interface, VLAN tag, description |
| Assign VLAN interface | Interfaces → Assignments | Assign VLAN to interface, then configure IP |
| Enable DHCP per VLAN | Services → DHCPv4 → [Interface] | Set range, reserve IPs for servers by MAC |
| Set firewall rules per VLAN | Firewall → Rules → [Interface] | Each VLAN interface has its own ruleset |
| Enable inter-VLAN routing | Firewall → Rules | Add explicit allow rules between VLANs — never use a blanket allow |

---

### 2.3 WireGuard VPN Setup

| Step | Path | Notes |
|---|---|---|
| Install WireGuard | System → Firmware → Plugins → os-wireguard | Reboot after install |
| Create server instance | VPN → WireGuard → Local | Generate keypair, set listen port 51820, set tunnel IP |
| Add peer (client) | VPN → WireGuard → Peers | Paste client pubkey, set allowed IPs = client tunnel IP |
| Assign WG interface | Interfaces → Assignments | Assign wg0, enable, no IP needed — tunnel handles it |
| WAN firewall rule | Firewall → Rules → WAN | Allow UDP 51820 inbound — only WAN rule needed |
| WG firewall rules | Firewall → Rules → WireGuard | Allow VPN clients to reach approved VLANs only |

**Client config template:**
```ini
[Interface]
PrivateKey = <client_private_key>
Address = 10.60.0.2/24
DNS = 10.10.0.1  # Point to OPNsense Unbound — internal DNS only

[Peer]
PublicKey = <server_public_key>
Endpoint = your.domain.com:51820
AllowedIPs = 10.0.0.0/8  # Route only internal traffic through VPN
PersistentKeepalive = 25
```

---

### 2.4 Suricata IDS/IPS

| Setting | Value | Notes |
|---|---|---|
| Mode | IPS (inline) | IDS detects. IPS blocks. Always use IPS. |
| Rulesets | ET Open + ET Pro + Feodo Tracker | ET Open is free. ET Pro is worth paying for. |
| Pattern matcher | Hyperscan | Fastest engine on modern hardware |
| Interfaces | WAN + all VLANs | Inspect all segments, not just WAN |
| Update interval | Every 12 hours | Threat signatures must stay current |
| Log alerts | Yes — send to syslog | Forward to central SIEM |

---

### 2.5 HAProxy Reverse Proxy

> ✅ **Why HAProxy:** Instead of exposing port 443 per service, HAProxy terminates TLS and routes to internal services by hostname/SNI. One listener, zero exposed service ports, everything internal.  
> Install: **System → Firmware → Plugins → os-haproxy**

| Component | Setting | Notes |
|---|---|---|
| Backend | Internal service IP:port | e.g. Vaultwarden at `10.20.0.5:8080` |
| Frontend | Listen on VPN VLAN IP:443 | Only reachable after WireGuard authentication |
| SSL offloading | ACME cert via Let's Encrypt | Services → ACME Client for auto-renewal |
| Health checks | Enabled per backend | HAProxy stops routing to dead backends automatically |
| ACL routing | `hdr(host) -i service.internal` | Route by hostname to correct backend |

---

### 2.6 Diagnostics

| Tool | Path | What It Shows |
|---|---|---|
| Live firewall log | Firewall → Log Files → Live View | Real-time allow/block decisions |
| States table | Firewall → Diagnostics → States | All active connections through the firewall |
| Packet capture | Interfaces → Diagnostics → Packet Capture | tcpdump-level capture per interface |
| DHCP leases | Services → DHCPv4 → Leases | All clients and their assigned IPs |
| Routes | System → Routes → Status | Active routing table |
| System log | System → Log Files → General | Auth events, service starts, errors |
| ARP table | Interfaces → Diagnostics → ARP Table | IP to MAC mappings on local segments |

---

## 3. Cisco CLI

> **IOS Prompt Reference:**  
> `>` = User EXEC (read-only) | `#` = Privileged EXEC | `(config)#` = Global Config  
> `(config-if)#` = Interface Config | `(config-router)#` = Router Config

### 3.1 Navigation & Basics

| Command | What It Does | Mode |
|---|---|---|
| `enable` | Enter privileged EXEC mode | `>` |
| `configure terminal` | Enter global config mode | `#` |
| `exit` | Go back one mode level | any |
| `end` | Return directly to privileged EXEC | any config |
| `do [cmd]` | Run EXEC command from config mode | config |
| `?` | Context-sensitive help | any |
| `[partial]?` | Tab-complete or list completions | any |
| `no [command]` | Negate / undo a configuration command | config |

---

### 3.2 Show Commands

| Command | What It Shows |
|---|---|
| `show version` | IOS version, uptime, hardware model, license |
| `show running-config` | Active config in RAM — what the device is doing right now |
| `show startup-config` | Config in NVRAM — what loads on reboot |
| `show interfaces` | All interfaces — status, errors, bandwidth, duplex |
| `show ip interface brief` | Quick summary — all interfaces, IP, status, protocol |
| `show ip route` | Routing table |
| `show ip route ospf` | Only OSPF-learned routes |
| `show ip ospf neighbor` | OSPF adjacency state |
| `show vlan brief` | All VLANs and assigned ports (switch) |
| `show interfaces trunk` | Trunk status, allowed VLANs, native VLAN |
| `show spanning-tree` | STP topology, root bridge, port states |
| `show mac address-table` | MAC to port mappings on a switch |
| `show ip arp` | ARP cache — IP to MAC mappings |
| `show cdp neighbors detail` | Connected Cisco devices — IP, model, interface |
| `show crypto isakmp sa` | VPN IKE phase 1 SA state |
| `show crypto ipsec sa` | VPN IPSec phase 2 SA + packet counters |
| `show access-lists` | All ACLs with hit counters |
| `show logging` | System log buffer |
| `show processes cpu` | CPU utilization — spot runaway processes |
| `show processes memory` | Memory usage per process |

---

### 3.3 Interface Configuration

```cisco
interface GigabitEthernet0/0
 description WAN_Uplink
 ip address 203.0.113.1 255.255.255.0
 no shutdown

interface GigabitEthernet0/1
 description LAN_Trusted
 ip address 10.30.0.1 255.255.255.0
 no shutdown
```

| Command | What It Does |
|---|---|
| `interface GigabitEthernet0/0` | Enter interface config mode |
| `ip address [IP] [mask]` | Set IP address and subnet mask |
| `no shutdown` | Bring interface up |
| `shutdown` | Administratively disable interface |
| `description [text]` | Label the interface — always do this |
| `duplex full` | Force full duplex |
| `speed 1000` | Force gigabit — avoid auto-negotiation mismatches |
| `interface loopback 0` | Create logical loopback — stable for OSPF router ID |

---

### 3.4 VLANs & Trunking (Switch)

```cisco
! Create VLANs
vlan 10
 name MANAGEMENT
vlan 20
 name SERVERS
vlan 30
 name TRUSTED

! Access port — end device
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 30
 switchport nonegotiate
 spanning-tree portfast

! Trunk port — carries multiple VLANs
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 switchport trunk native vlan 999
 switchport nonegotiate
```

> ⚠️ `switchport nonegotiate` disables DTP. Never auto-negotiate trunking — it's a VLAN hopping vector.  
> ⚠️ Native VLAN should always be set to an unused VLAN ID (999) to prevent VLAN hopping attacks.

---

### 3.5 OSPF Routing

```cisco
router ospf 1
 router-id 1.1.1.1
 passive-interface default
 no passive-interface GigabitEthernet0/1
 network 10.0.0.0 0.0.0.255 area 0
 network 10.10.0.0 0.0.0.255 area 0
 default-information originate
```

| Command | What It Does |
|---|---|
| `router ospf 1` | Enable OSPF process 1 |
| `router-id 1.1.1.1` | Set stable router ID — use loopback IP |
| `network [IP] [wildcard] area [n]` | Advertise network into OSPF area |
| `passive-interface default` | Make ALL interfaces passive by default |
| `no passive-interface [int]` | Enable OSPF hellos on specific interface only |
| `default-information originate` | Advertise default route to OSPF neighbors |

---

### 3.6 Access Control Lists

> ⚠️ **Standard ACLs** match source IP only — apply close to destination.  
> ⚠️ **Extended ACLs** match src + dst + port — apply close to source.  
> ⚠️ Implicit deny-all at end of every ACL. Always verify with `show access-lists`.

```cisco
! Extended named ACL — block IoT from reaching management
ip access-list extended BLOCK_IOT_TO_MGMT
 deny ip 10.40.0.0 0.0.0.255 10.10.0.0 0.0.0.255
 permit ip any any

! Apply to interface
interface GigabitEthernet0/2
 ip access-group BLOCK_IOT_TO_MGMT in

! Verify hit counters
show access-lists BLOCK_IOT_TO_MGMT
```

---

### 3.7 Save & Maintenance

| Command | What It Does |
|---|---|
| `copy running-config startup-config` | Save config to NVRAM — persists reboot |
| `write memory` | Shorthand for the above |
| `copy running-config tftp` | Back up config to TFTP server |
| `reload` | Reboot the device |
| `erase startup-config` | Factory reset — wipe NVRAM config |
| `debug ip ospf events` | Live OSPF event debug — use with extreme caution on prod |
| `undebug all` | Turn off ALL debugs immediately |

---

## 4. Bash

### 4.1 Navigation & File Operations

| Command | What It Does |
|---|---|
| `pwd` | Print current working directory |
| `ls -lah` | List with permissions, sizes, hidden files |
| `cd -` | Jump to previous directory |
| `find / -name "*.conf" 2>/dev/null` | Find files by name, suppress errors |
| `find / -perm -4000 2>/dev/null` | Find SUID binaries — key pentest enumeration step |
| `find / -writable -type f 2>/dev/null` | Find world-writable files |
| `cp -r src/ dest/` | Recursive copy |
| `rsync -av --delete src/ dest/` | Sync directories — deletes files removed from source |
| `tar -czvf archive.tar.gz /path` | Create compressed archive |
| `tar -xzvf archive.tar.gz` | Extract compressed archive |
| `chmod 700 script.sh` | Owner execute only — no group/world access |
| `chown user:group file` | Change file ownership |
| `ln -s /target /link` | Create symbolic link |
| `stat file` | Full file metadata — permissions, timestamps, inode |

---

### 4.2 Text Processing

| Command | What It Does |
|---|---|
| `grep -r "password" /etc/ 2>/dev/null` | Recursive credential search in config files |
| `grep -E "PATTERN1\|PATTERN2"` | Extended regex — match multiple patterns |
| `grep -v "exclude"` | Invert match — show lines NOT matching |
| `grep -i "pattern"` | Case-insensitive search |
| `awk '{print $1,$3}'` | Print specific columns |
| `awk -F: '{print $1}' /etc/passwd` | Extract usernames — field delimiter colon |
| `sed -i 's/old/new/g' file` | In-place find and replace |
| `cut -d: -f1 /etc/passwd` | Cut fields by delimiter |
| `sort -u` | Sort and deduplicate |
| `uniq -c` | Count occurrences of repeated lines |
| `wc -l file` | Count lines |
| `tr -d '\r'` | Remove carriage returns — fix Windows line endings |
| `cat file \| jq '.'` | Pretty-print JSON |
| `strings binary` | Extract readable strings from binary |
| `xxd file \| head` | Hex dump — inspect binary file headers |

---

### 4.3 Network & Recon

| Command | What It Does |
|---|---|
| `sudo ss -tulpn` | All listening ports + process binding each |
| `sudo lsof -i :80` | What process owns port 80 |
| `ip addr show` | All interfaces and IPs |
| `ip route show` | Routing table |
| `ip neigh show` | ARP cache — local segment neighbors |
| `curl -I http://target` | HTTP headers only — service fingerprint |
| `curl -v https://target` | Full TLS negotiation details |
| `nc -zv host 1-1024` | TCP port scan with netcat |
| `nc -lvnp 4444` | Listen for reverse shell |
| `ssh -L 8080:internal:80 user@pivot` | Local port forward through SSH tunnel |
| `ssh -R 4444:localhost:4444 user@host` | Remote port forward |
| `ssh -D 1080 user@host` | SOCKS5 proxy through SSH |
| `tcpdump -i eth0 -w capture.pcap` | Capture traffic to file |
| `tcpdump -i eth0 port 80` | Live capture filtered to port 80 |

---

### 4.4 Process & System Enumeration

| Command | What It Does |
|---|---|
| `ps aux` | All running processes with CPU/RAM |
| `kill -9 [PID]` | Force kill process |
| `systemctl status [service]` | Service status, logs, PID |
| `systemctl disable --now [service]` | Stop and prevent auto-start |
| `journalctl -u [service] -f` | Follow live service logs |
| `journalctl -p err -b` | Only errors since last boot |
| `crontab -e` | Edit user cron jobs |
| `cat /etc/crontab` | System cron — check for misconfigs |
| `uname -a` | Kernel version — key for exploit research |
| `id` | Current user, UID, groups |
| `sudo -l` | What sudo commands current user can run |
| `cat /etc/shadow` | Password hashes — requires root |
| `env` | Environment variables — often contains credentials |
| `history` | Command history — goldmine for creds and paths |

---

### 4.5 Scripting Reference

```bash
#!/bin/bash
# Always start with error handling
set -euo pipefail
trap 'echo "Error on line $LINENO"' ERR

# Variables
TARGET="10.0.0.1"
PORT=80

# Conditionals — always use [[ ]] not [ ] for strings
if [[ "$VAR" == "value" ]]; then
  echo "match"
elif [[ -f "/path/to/file" ]]; then
  echo "file exists"
else
  echo "no match"
fi

# For loop — ping sweep
for ip in 10.0.0.{1..254}; do
  ping -c 1 -W 1 "$ip" &>/dev/null && echo "$ip UP"
done

# While loop — polling
while true; do
  RESULT=$(curl -s --max-time 5 "http://$TARGET")
  [[ -n "$RESULT" ]] && echo "[+] Host up"
  sleep 10
done

# Functions
check_port() {
  local host="$1" port="$2"
  nc -z -w2 "$host" "$port" 2>/dev/null && \
    echo "[+] $host:$port OPEN" || \
    echo "[-] $host:$port CLOSED"
}

# Command substitution
CURRENT_IP=$(ip route get 1 | awk '{print $7; exit}')

# Arrays
HOSTS=("10.0.0.1" "10.0.0.2" "10.0.0.3")
for host in "${HOSTS[@]}"; do
  check_port "$host" 22
done

# Redirect stderr
command 2>/dev/null       # Discard errors
command 2>&1 | tee log    # Capture both stdout and stderr to file
command > /dev/null 2>&1  # Suppress all output

# Cron syntax: minute hour day month weekday
# 0 2 * * *    = 2:00 AM every day
# */5 * * * *  = every 5 minutes
# 0 */6 * * *  = every 6 hours
```

---

## 5. SQL

> ⚠️ **NEVER** concatenate user input into SQL queries. Always use parameterized queries / prepared statements. SQL injection is in the OWASP Top 10 for a reason.

### 5.1 Core CRUD

```sql
-- SELECT
SELECT * FROM users;
SELECT id, name, email FROM users WHERE active = 1;
SELECT * FROM users ORDER BY name ASC LIMIT 10 OFFSET 20;
SELECT COUNT(*) FROM users;
SELECT DISTINCT role FROM users;

-- INSERT
INSERT INTO users (name, email, role) VALUES ('Kyle', 'k@lab.com', 'admin');

-- UPDATE -- always use WHERE or you update every row
UPDATE users SET active = 0 WHERE id = 5;

-- DELETE -- always use WHERE or you delete every row
DELETE FROM users WHERE id = 5;
```

---

### 5.2 Filtering & Pattern Matching

```sql
-- WHERE operators
SELECT * FROM users WHERE age > 18 AND active = 1;
SELECT * FROM users WHERE role IN ('admin', 'operator');
SELECT * FROM users WHERE name LIKE 'Sw%';     -- starts with Sw
SELECT * FROM users WHERE email LIKE '%@lab%'; -- contains @lab
SELECT * FROM users WHERE last_login IS NULL;
SELECT * FROM users WHERE id BETWEEN 10 AND 50;
```

---

### 5.3 Joins

```sql
-- INNER JOIN — only matching rows in both tables
SELECT u.name, r.role_name
FROM users u
INNER JOIN roles r ON u.role_id = r.id;

-- LEFT JOIN — all users, NULL if no matching role
SELECT u.name, r.role_name
FROM users u
LEFT JOIN roles r ON u.role_id = r.id;

-- Multiple joins
SELECT u.name, r.role_name, d.dept_name
FROM users u
JOIN roles r ON u.role_id = r.id
JOIN departments d ON u.dept_id = d.id
WHERE u.active = 1;
```

---

### 5.4 Aggregation & Grouping

```sql
-- Aggregate functions
SELECT COUNT(*) FROM users;
SELECT AVG(salary), MAX(salary), MIN(salary) FROM employees;
SELECT SUM(bytes) FROM logs WHERE date = '2026-05-01';

-- GROUP BY
SELECT role, COUNT(*) AS user_count
FROM users
GROUP BY role
ORDER BY user_count DESC;

-- HAVING — filter after grouping (WHERE filters before)
SELECT role, COUNT(*) AS user_count
FROM users
GROUP BY role
HAVING COUNT(*) > 5;
```

---

### 5.5 Subqueries & CTEs

```sql
-- Subquery
SELECT name FROM users
WHERE id IN (SELECT user_id FROM logs WHERE action = 'failed_login');

-- CTE (Common Table Expression) — cleaner than nested subqueries
WITH failed_logins AS (
  SELECT user_id, COUNT(*) AS attempts
  FROM logs
  WHERE action = 'failed_login'
  GROUP BY user_id
  HAVING COUNT(*) > 10
)
SELECT u.name, fl.attempts
FROM users u
JOIN failed_logins fl ON u.id = fl.user_id;
```

---

### 5.6 Schema Management

```sql
-- Create table
CREATE TABLE users (
  id       INTEGER PRIMARY KEY AUTOINCREMENT,
  name     TEXT NOT NULL,
  email    TEXT UNIQUE NOT NULL,
  role     TEXT DEFAULT 'viewer',
  active   INTEGER DEFAULT 1,
  created  DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Add column
ALTER TABLE users ADD COLUMN last_login DATETIME;

-- Create index — speeds up queries on frequently searched columns
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_logs_user_date ON logs(user_id, created_at);

-- Drop table
DROP TABLE IF EXISTS old_table;
```

---

### 5.7 SQLite-Specific Commands

| Command | What It Does |
|---|---|
| `sqlite3 database.db` | Open or create a database file |
| `.tables` | List all tables |
| `.schema [table]` | Show CREATE statement for a table |
| `.mode column` | Format output as aligned columns |
| `.headers on` | Show column names in output |
| `.quit` | Exit the SQLite shell |
| `PRAGMA integrity_check;` | Check database for corruption |
| `PRAGMA table_info(table);` | Show columns, types, constraints |
| `VACUUM;` | Reclaim disk space, defragment file |
| `.backup backup.db` | Hot backup while service is running |
| `.dump > dump.sql` | Export entire database as SQL text |

---

### 5.8 SQL Injection — What Attackers Look For

```sql
-- UNSAFE — never do this
query = "SELECT * FROM users WHERE name = '" + user_input + "'"

-- Attacker input: ' OR '1'='1
-- Resulting query — returns ALL users:
SELECT * FROM users WHERE name = '' OR '1'='1'

-- Attacker input: '; DROP TABLE users; --
-- Resulting query — destroys the table:
SELECT * FROM users WHERE name = ''; DROP TABLE users; --'

-- SAFE — always use parameterized queries
-- Python example:
cursor.execute("SELECT * FROM users WHERE name = ?", (user_input,))
```

---

## 6. Python

### 6.1 HTTP Requests

```python
import requests

# GET with timeout
r = requests.get("https://target.com", timeout=10)
print(r.status_code)   # 200
print(r.headers)       # response headers
print(r.json())        # parse JSON body

# POST with JSON payload
r = requests.post(
    "https://api.target.com/login",
    json={"user": "admin", "pass": "password"},
    timeout=10
)
r.raise_for_status()   # raises exception on 4xx/5xx

# Session — persists cookies between requests
s = requests.Session()
s.get("https://target.com/login")
s.post("https://target.com/login", data={"user": "admin", "pass": "pass"})
r = s.get("https://target.com/dashboard")  # authenticated

# Custom headers
headers = {"Authorization": "Bearer TOKEN", "User-Agent": "CustomAgent/1.0"}
r = requests.get("https://api.target.com", headers=headers)
```

---

### 6.2 File & System Operations

```python
import subprocess
import os
import pathlib
import json

# Run shell command safely — never use shell=True with user input
result = subprocess.run(
    ["nmap", "-sV", "10.0.0.0/24"],
    capture_output=True,
    text=True,
    check=True
)
print(result.stdout)

# Read environment variables safely
api_key = os.environ.get("API_KEY", "default_fallback")

# File operations — modern approach
path = pathlib.Path("/etc/hosts")
content = path.read_text()
path.write_text("new content")

# JSON handling
data = json.loads('{"key": "value"}')
output = json.dumps(data, indent=2)

# Context manager — always use for file I/O
with open("output.txt", "w") as f:
    f.write("data")
```

---

### 6.3 Network & Security Scripts

```python
import socket
import ipaddress
import subprocess
from concurrent.futures import ThreadPoolExecutor

# Port scanner
def scan_port(host: str, port: int) -> bool:
    try:
        with socket.create_connection((host, port), timeout=1):
            return True
    except (socket.timeout, ConnectionRefusedError, OSError):
        return False

# Threaded subnet scanner
def scan_host(ip: str) -> None:
    if scan_port(str(ip), 22):
        print(f"[+] {ip}:22 OPEN")

network = ipaddress.ip_network("10.0.0.0/24")
with ThreadPoolExecutor(max_workers=50) as executor:
    executor.map(scan_host, network.hosts())

# Parse /etc/passwd
def get_users() -> list[str]:
    with open("/etc/passwd") as f:
        return [line.split(":")[0] for line in f if not line.startswith("#")]

# Send ntfy alert
def alert(topic: str, message: str, priority: str = "default") -> None:
    import requests
    requests.post(
        f"http://localhost/{topic}",
        data=message.encode(),
        headers={"Priority": priority, "Title": "Lab Alert"}
    )
```

---

### 6.4 Patterns for Security Scripts

```python
import argparse
import logging
import sys

# Logging — always use this instead of print
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(message)s",
    handlers=[
        logging.StreamHandler(sys.stdout),
        logging.FileHandler("script.log")
    ]
)
log = logging.getLogger(__name__)

# CLI argument parsing
def parse_args() -> argparse.Namespace:
    parser = argparse.ArgumentParser(description="Security tool")
    parser.add_argument("--target", required=True, help="Target IP or hostname")
    parser.add_argument("--port", type=int, default=80, help="Port (default: 80)")
    parser.add_argument("--verbose", action="store_true", help="Verbose output")
    return parser.parse_args()

# Error handling — never let scripts die silently
try:
    result = risky_operation()
except requests.exceptions.Timeout:
    log.error("Request timed out")
    sys.exit(1)
except requests.exceptions.ConnectionError as e:
    log.error(f"Connection failed: {e}")
    sys.exit(1)
except Exception as e:
    log.exception(f"Unexpected error: {e}")
    sys.exit(1)

# Entry point guard
if __name__ == "__main__":
    args = parse_args()
    log.info(f"Starting scan against {args.target}:{args.port}")
```

---

### 6.5 Parameterized SQL in Python

```python
import sqlite3

conn = sqlite3.connect("lab.db")
cursor = conn.cursor()

# SAFE — always parameterized
cursor.execute("SELECT * FROM users WHERE name = ?", (user_input,))
cursor.execute(
    "INSERT INTO users (name, email) VALUES (?, ?)",
    (name, email)
)
conn.commit()

# Fetch results
row = cursor.fetchone()
rows = cursor.fetchall()

# Context manager — auto-commits and closes
with sqlite3.connect("lab.db") as conn:
    conn.execute("UPDATE users SET active=0 WHERE id=?", (user_id,))

conn.close()
```

---

## 7. Markdown

### 7.1 Text Formatting

```markdown
# H1 Heading
## H2 Heading
### H3 Heading
#### H4 Heading

**bold text**
*italic text*
***bold and italic***
~~strikethrough~~
`inline code`

> Blockquote — use for notes, warnings, philosophy statements

> ⚠️ **Warning:** Use callout emoji for visual emphasis in GitHub Markdown.
> ✅ **Success:** Green check for confirmed working.
> 🔴 **Danger:** Red for critical / destructive.
> ℹ️ **Info:** Blue for background context.
```

---

### 7.2 Lists

```markdown
<!-- Unordered -->
- Item one
- Item two
  - Nested item
  - Another nested

<!-- Ordered -->
1. First step
2. Second step
   1. Sub-step
   2. Sub-step

<!-- Task list — renders checkboxes on GitHub -->
- [x] Completed task
- [ ] Incomplete task
- [ ] Another task
```

---

### 7.3 Code Blocks

````markdown
<!-- Inline code -->
Use `sudo ss -tulpn` to check ports.

<!-- Fenced code block with language -->
```bash
docker stop $(docker ps -aq)
sudo systemctl restart docker
```

```python
import requests
r = requests.get("https://target.com", timeout=10)
```

```sql
SELECT * FROM users WHERE active = 1;
```

```cisco
show ip interface brief
```
````

---

### 7.4 Tables

```markdown
| Column 1 | Column 2 | Column 3 |
|---|---|---|
| Left | Center | Right |
| Value | Value | Value |

<!-- Alignment -->
| Left | Center | Right |
|:---|:---:|---:|
| text | text | text |
```

---

### 7.5 Links & Images

```markdown
<!-- Links -->
[Link text](https://url.com)
[Link with title](https://url.com "Hover title")
[Internal anchor link](#section-heading)
[Reference link][ref-id]

[ref-id]: https://url.com

<!-- Images -->
![Alt text](image.png)
![Alt text](image.png "Title")

<!-- Linked image -->
[![Alt text](image.png)](https://url.com)
```

---

### 7.6 GitHub-Specific Markdown

```markdown
<!-- Collapsible section -->
<details>
<summary>Click to expand</summary>

Content hidden by default. Useful for large code blocks or optional reading.

</details>

<!-- Alerts (GitHub Markdown only) -->
> [!NOTE]
> Useful information.

> [!WARNING]
> Critical information requiring attention.

> [!IMPORTANT]
> Key information users need to know.

> [!TIP]
> Optional helpful advice.

> [!CAUTION]
> Negative potential consequences.

<!-- Mention and reference -->
@username          — mention a user
#123               — reference an issue or PR
SHA                — reference a commit

<!-- Keyboard shortcuts -->
Press <kbd>Ctrl</kbd>+<kbd>C</kbd> to copy.

<!-- Footnotes -->
This is a statement.[^1]
[^1]: This is the footnote text.
```

---

### 7.7 README Structure Template

```markdown
# Project Name

> One-line description of what this does and why.

## Quick Start

```bash
git clone https://github.com/user/repo.git
cd repo
./install.sh
```

## Requirements

- Requirement one
- Requirement two

## Usage

Explain how to use it with examples.

## Configuration

| Variable | Default | Description |
|---|---|---|
| `VAR_NAME` | `default` | What it does |

## License

MIT — see [LICENSE](LICENSE)
```

---

## 8. Git

### 8.1 Setup & Config

```bash
# Identity — set before first commit
git config --global user.name "Kyle Sweatt"
git config --global user.email "kyle@sweattlabs.com"
git config --global core.editor "nano"
git config --global init.defaultBranch main

# View config
git config --list
git config --global --list
```

---

### 8.2 Core Workflow

```bash
# Initialize
git init
git clone https://github.com/user/repo.git
git clone git@github.com:user/repo.git  # SSH — preferred

# Stage & commit
git status                         # what's changed
git add file.txt                   # stage specific file
git add .                          # stage all changes
git add -p                         # stage interactively — review each chunk
git commit -m "feat: add firewall rules"
git commit --amend                 # modify the last commit (before pushing)

# Push & pull
git push origin main
git push -u origin main            # set upstream — then just git push
git pull                           # fetch + merge
git fetch origin                   # fetch only — don't merge yet
git pull --rebase origin main      # rebase instead of merge — cleaner history
```

---

### 8.3 Branching

```bash
# Create and switch
git branch feature/zero-trust      # create branch
git checkout feature/zero-trust    # switch to branch
git checkout -b feature/zero-trust # create and switch in one command
git switch -c feature/zero-trust   # modern syntax — same result

# View branches
git branch                         # local branches
git branch -r                      # remote branches
git branch -a                      # all branches

# Merge and delete
git checkout main
git merge feature/zero-trust
git branch -d feature/zero-trust   # delete merged branch
git branch -D feature/zero-trust   # force delete unmerged branch
git push origin --delete feature/zero-trust  # delete remote branch
```

---

### 8.4 Stash — Save Work Without Committing

```bash
git stash                          # stash all uncommitted changes
git stash push -m "wip: firewall rules"  # stash with a name
git stash list                     # view all stashes
git stash pop                      # apply most recent stash and remove it
git stash apply stash@{2}          # apply specific stash, keep it in list
git stash drop stash@{0}           # delete specific stash
git stash clear                    # delete all stashes
```

---

### 8.5 History & Diff

```bash
git log                            # full commit history
git log --oneline                  # condensed — one line per commit
git log --oneline --graph --all    # visual branch tree
git log --author="Kyle"            # filter by author
git log --since="1 week ago"       # filter by date
git log -- filename                # history for specific file

git diff                           # unstaged changes
git diff --staged                  # staged changes vs last commit
git diff main..feature/branch      # diff between branches
git show [commit-SHA]              # show a specific commit
```

---

### 8.6 Undoing Changes

```bash
# Undo unstaged changes
git restore file.txt               # discard working directory changes
git restore .                      # discard all unstaged changes

# Unstage
git restore --staged file.txt      # unstage a file (keep changes)
git reset HEAD file.txt            # older syntax — same result

# Undo commits — SAFE (keeps changes)
git revert [commit-SHA]            # create new commit that undoes specified commit
git reset --soft HEAD~1            # undo last commit, keep changes staged
git reset --mixed HEAD~1           # undo last commit, keep changes unstaged

# Undo commits — DESTRUCTIVE (loses changes)
git reset --hard HEAD~1            # undo last commit, discard changes entirely
git reset --hard origin/main       # reset to match remote — destroys local changes
```

> ⚠️ `git reset --hard` is destructive and cannot be undone. Never use on shared branches.

---

### 8.7 Remote Management

```bash
git remote -v                      # list remotes with URLs
git remote add origin git@github.com:user/repo.git
git remote set-url origin git@github.com:user/new-repo.git  # change URL
git remote remove origin           # remove remote

# Sync fork with upstream
git remote add upstream https://github.com/original/repo.git
git fetch upstream
git merge upstream/main
```

---

### 8.8 .gitignore

```gitignore
# Secrets — NEVER commit these
.env
*.key
*.pem
*_rsa
secrets.yaml
credentials.json

# OS files
.DS_Store
Thumbs.db

# Editor files
.vscode/
.idea/
*.swp

# Build artifacts
__pycache__/
*.pyc
*.pyo
node_modules/
dist/
build/

# Logs
*.log
logs/
```

---

### 8.9 Commit Message Convention

```
<type>(<scope>): <short summary>

<optional body — explain WHY, not WHAT>

<optional footer — references, breaking changes>
```

| Type | When to Use |
|---|---|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Formatting — no logic change |
| `refactor` | Code restructure — no feature or fix |
| `perf` | Performance improvement |
| `test` | Adding or updating tests |
| `chore` | Maintenance — deps, config, build |
| `security` | Security fix or hardening |

**Examples:**
```
feat(firewall): add zero trust inter-VLAN block rules
fix(vpn): correct WireGuard peer allowed IPs
security(ssh): disable password authentication on all hosts
docs(readme): add VLAN segmentation table
```

---

### 8.10 SSH Key Setup for GitHub

```bash
# Generate key — Ed25519 is current best practice
ssh-keygen -t ed25519 -C "kyle@sweattlabs.com" -f ~/.ssh/github_ed25519

# Add to SSH agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/github_ed25519

# Copy public key — paste into GitHub Settings → SSH Keys
cat ~/.ssh/github_ed25519.pub

# Test connection
ssh -T git@github.com

# ~/.ssh/config — manage multiple keys
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/github_ed25519
  AddKeysToAgent yes
```

---

*Sweatt Labs · Master Security Cheatsheet · Updated 2026*  
*Stay curious. Stay secure.* 🚀
