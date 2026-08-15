# Exploiting vsftpd on Metasploitable
Gaining  a Remote shell by Exploiting the VSFTPD Backdoor Vulnerability on a Metasploitable Machine Using the Metasploitable Framework.

# Router Device Scan
![Router device Scan](screenshots/router_device_scan0.1.png)
<p align = "center">router_device_scan0.1.png</p>

### Command
```bash
nmap -sn -n 192.168.0.0/24
```
### Parameters
- `-sn` : Ping scan only — checks which hosts are alive, skips port scanning.
- `-n`  : Skips DNS resolution, making the scan faster.
- `192.168.0.0/24` : Target subnet in CIDR notation, covering all 256
  possible addresses in that network range.

### Observation
The scan completed in 4.37 seconds and found 3 hosts up out of 256
addresses scanned:
 
| IP Address    | MAC Address        | Likely Role                  |
|---------------|---------------------|-------------------------------|
| 192.168.0.1   | —                   | Router/Gateway                |
| 192.168.0.128 | 00:0C:29:FA:DD:34 (VMware) | Target (Metasploitable2) |
| 192.168.0.254 | 00:50:56:EC:66:4E (VMware) | Another VM on the network |
 
Both `.128` and `.254` show VMware MAC address prefixes, confirming they
are virtual machines on the host-only/NAT network. I identified
`192.168.0.128` as the Metasploitable2 target based on this.

### Why Is This Important?
The purpose of running this command was to identify which machines are
actually alive on the network before doing anything else. Scanning a
/24 subnet means 256 possible addresses — without this step, I would be
guessing at IPs or wasting time scanning dead/non-existent hosts. This
scan immediately narrowed the target search down from 256 possibilities
to just 3 real, reachable machines, and the MAC address vendor info
(VMware) helped me distinguish which hosts were virtual machines relevant
to my lab, versus the physical router.

### What I Learned
Reconnaissance should always start broad and narrow down gradually. A
lightweight ping sweep is the right first move because it's fast (under 5
seconds here) and doesn't waste time probing ports on dead hosts. I also
learned that MAC address vendor identification (like the "VMware" tag) is
a useful clue for pinpointing which host is actually my intended lab
target when multiple machines respond on the same network.
 
---
# Target IP Port Scan 
![Target IP Port Scan](screenshots/port_&_version_scan0.2.png)
<p align= "center">port_&_version_scan0.2.png</p>

### command
```bash
nmap -sV 192.168.0.128
```
### Parameters
- `-sV` : Probes open ports to detect the **service and version** running
  on each (e.g., vsftpd 2.3.4, Apache 2.2.8).
- `192.168.0.128` : The confirmed target IP from the previous host
  discovery step.
- Default scan covers the top 1000 common ports --- full port scan wasn't needed since the target already showed 23 open ports here.

### Observation 
- The scan found the host up with 23 open port out of the 1000 scanned (977 closed).
- The port that mattered most to me port 21, running vsftpd 2.3.4. This is the exact version I came here to exploit.
- Full list of open ports:

| Port | Service | Version Detected |
|------|---------|-------------------|
| 21   | ftp     | **vsftpd 2.3.4** |
| 22   | ssh     | OpenSSH 4.7p1 Debian 8ubuntu1 |
| 23   | telnet  | Linux telnetd |
| 25   | smtp    | Postfix smtpd |
| 53   | domain  | ISC BIND 9.4.2 |
| 80   | http    | Apache httpd 2.2.8 (Ubuntu) DAV/2 |
| 139/445 | netbios-ssn | Samba smbd 3.X - 4.X |
| 1099 | java-rmi | GNU Classpath grmiregistry |
| 1524 | bindshell | **Metasploitable root shell** |
| 2121 | ftp     | ProFTPD 1.3.1 |
| 3306 | mysql   | MySQL 5.0.51a-3ubuntu5 |
| 5432 | postgresql | PostgreSQL DB 8.3.0 - 8.3.7 |
| 5900 | vnc     | VNC (protocol 3.3) |
| 6000 | X11     | (access denied) |

### Why Is This Important?
The purpose of this scan was to find out exactly what services are running
on the target and their exact versions. Version numbers are critical —
they're what let you match a service to a known, publicly documented
vulnerability. Without this step, I wouldn't have known that vsftpd
2.3.4 specifically was present, or that it's the backdoored version. The
scan also revealed the broader attack surface (23 open services), showing
just how many potential entry points a misconfigured or intentionally
vulnerable system can expose.

### What I Learned
I learned that service version detection is the bridge between recon and
exploitation — it turns "a port is open" into "this port is exploitable."
I also learned that a single machine can expose an unusually large number
of services (like Telnet, rsh, and even a service literally named
"Metasploitable root shell"), and that outdated software versions are the
biggest giveaway when hunting for a way in. This scan gave me a full map
of the attack surface before I committed to exploiting any one service.
 
---
 


  



