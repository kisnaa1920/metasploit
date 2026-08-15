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




