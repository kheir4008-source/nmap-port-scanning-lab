---

## Phase 1: Host Discovery

**Goal:** Identify live hosts on the local network without triggering port scans.

**Command:**
```bash
nmap -sn 192.168.64.0/24 -oN discovery.txt
```

**Flag breakdown:**
- `-sn` — Ping scan only, no port scanning
- `-oN` — Save output in normal format

**Findings:**

| IP | Role |
|---|---|
| 192.168.64.1 | UTM virtual network gateway |
| 192.168.64.12 | Kali Linux attacker machine |

**Notes:** Only 2 hosts live on this isolated virtual network. Host discovery scans are low-noise — they don't touch ports and are less likely to trigger IDS alerts.

---

## Phase 2: Basic Port Scan — Local Gateway

**Command:**
```bash
nmap -sS 192.168.64.1
```

**Flag breakdown:**
- `-sS` — SYN stealth scan. Sends SYN, receives SYN/ACK, replies with RST — never completes the handshake. Faster and quieter than a full TCP connect scan.

**Findings:** Port 53 (DNS) is the only open TCP port. Expected — the gateway handles DNS resolution for the virtual network.

| Flag | Type | Speed | Noise | Requires Root |
|---|---|---|---|---|
| `-sS` | SYN Stealth | Fast | Low | Yes |
| `-sT` | TCP Connect | Slow | High | No |# Nmap Port Scanning Lab

**Tool:** Nmap 7.98
**Platform:** Kali Linux (UTM/Apple Virtualization — M1 MacBook Air)
**Environment:** Local virtual network (192.168.64.0/24) + scanme.nmap.org
**Related TryHackMe Rooms:** Nmap Live Host Discovery · Nmap Basic Port Scans · Nmap Advanced Port Scans · Nmap Post Port Scans

---

## Objective

Demonstrate practical use of Nmap for host discovery, port scanning, service/version detection, OS fingerprinting, and the Nmap Scripting Engine (NSE). All scans were performed on authorized targets: a local UTM virtual network and Nmap's official practice server (scanme.nmap.org).

---

## Environment Setup

| Component | Details |
|---|---|
| Attacker machine | Kali Linux 7.98 on UTM (Apple Virtualization engine) |
| Network | 192.168.64.0/24 (UTM virtual network) |
| Local target | 192.168.64.1 (UTM gateway) |
| Remote target | scanme.nmap.org (45.33.32.156) — authorized practice server |
---

## Phase 3: UDP Scan — Local Gateway

**Command:**
```bash
nmap -sU 192.168.64.1
```

**Flag breakdown:**
- `-sU` — UDP scan. No handshake — Nmap sends packets and waits for responses.

**Findings:**

| Port | Service | Significance |
|---|---|---|
| 53/udp | DNS | Name resolution |
| 67/udp | DHCP Server | Assigns IP addresses to network devices |
| 137/udp | NetBIOS-NS | Windows name resolution |
| 1900/udp | UPnP | Device auto-discovery protocol |
| 5353/udp | mDNS | Local network device discovery |

**Notes:** UDP revealed 5 additional services invisible to TCP scans. Always run both TCP and UDP scans in a real engagement.

---

## Phase 4: Full Scan — Service, Version and OS Detection

**Command:**
```bash
nmap -sS -sV -O -sC -oA gateway_scan 192.168.64.1
```

**Flag breakdown:**
- `-sV` — Probe open ports to detect service versions
- `-O` — Enable OS detection
- `-sC` — Run default NSE scripts
- `-oA` — Save output in all three formats

**Key findings:**
- Port 53 open — DNS service
- OS detected: Apple macOS 11 (Big Sur) (Darwin 20.6.0)

**Notes:** OS detection correctly identified the underlying macOS host running UTM.
---

## Phase 5: Remote Target — scanme.nmap.org

**Command:**
```bash
nmap -sS -sV -sC -oA scanme_scan scanme.nmap.org
```

**Findings:**

| Port | Service | Version | Notes |
|---|---|---|---|
| 22/tcp | SSH | OpenSSH 6.6.1p1 Ubuntu | Old version — known CVEs exist |
| 80/tcp | HTTP | Apache 2.4.7 (Ubuntu) | Old version — known CVEs exist |
| 9929/tcp | nping-echo | — | Nmap test service |
| 31337/tcp | Elite | — | Legacy port, intentionally open |

**Notes:** OpenSSH 6.6.1p1 and Apache 2.4.7 are both outdated. In a real engagement these version numbers feed directly into CVE searches on nvd.nist.gov and exploit-db.com.

---

## Phase 6: Nmap Scripting Engine (NSE)

**HTTP Title:**
```bash
nmap --script http-title scanme.nmap.org -p 80
# Result: Go ahead and ScanMe!
```

**HTTP Headers:**
```bash
nmap --script http-headers scanme.nmap.org -p 80
# Result: Server: Apache/2.4.7 (Ubuntu)
```

**Finding:** The Server header reveals exact web server software and version — an information disclosure misconfiguration. A hardened server would suppress this header.

**NSE Categories:**

| Category | Use case |
|---|---|
| `auth` | Authentication bypass, default credentials |
| `brute` | Brute force attacks |
| `vuln` | Known vulnerability detection |
| `discovery` | Service and network enumeration |
| `safe` | Non-intrusive reconnaissance |---

## Output Formats

| Extension | Format | Use case |
|---|---|---|
| `.nmap` | Normal text | Human-readable review |
| `.xml` | XML | Import into Metasploit |
| `.gnmap` | Grepable | Parse with grep/awk |

---

## Pentest Workflow Summary

```bash
