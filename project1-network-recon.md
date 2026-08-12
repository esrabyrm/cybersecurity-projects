# Project 1: Home Lab Setup and Basic Network Reconnaissance

**Date:** August 11, 2026
**Environment:** VirtualBox — Kali Linux 2026.2 (NAT Network)
**Objective:** Set up an isolated virtual lab environment, perform port/service scanning with Nmap, and practice basic vulnerability research.

---

## 1. Executive Summary

In this project, an isolated virtual machine (Kali Linux) was set up, network connectivity was verified, and Nmap was used to perform port scanning against a target system. In the default state (firewall active), all ports were observed as filtered. A web service (Apache) was then deliberately enabled to create a real "open port" scenario, followed by vulnerability (CVE) research on the discovered service version.

## 2. Lab Environment

| Component | Detail |
|---|---|
| Hypervisor | Oracle VirtualBox |
| Test Machine | Kali Linux 2026.2 (amd64) |
| Network Mode | NAT Network (isolated, not exposed to host network) |
| Kali IP | 10.0.2.15/24 |

## 3. Methodology and Findings

### 3.1 Connectivity Verification
```
ip a
ping -c 4 google.com
```
Result: The virtual machine could reach the internet through NAT, while remaining isolated from the host's local network (confirming secure isolation).

### 3.2 Initial Port Scan (Default State)
```
nmap 10.0.2.15
```
**Result:** `1000 filtered tcp ports (no-response)`

**Analysis:** A "filtered" state differs from "closed" — it indicates a firewall is silently dropping packets rather than actively rejecting them. This was expected given Kali's default firewall configuration.

### 3.3 Re-scan with a Controlled Service
To observe a real "open port" scenario, the Apache web service was started:
```
sudo systemctl start apache2
nmap -sV 10.0.2.15
```

**Result:**
```
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.68 ((Debian))
```

**Analysis:** The `-sV` flag revealed the exact service version, enabling the first step of vulnerability research.

### 3.4 Vulnerability Research
```
searchsploit apache 2.4.68
```

**Result:** Keyword-matched results were returned, but manual review determined that most were unrelated (belonging to different components such as PHP, mod_ssl, or Tomcat, many dating back to the early 2000s).

**Analysis:** No known/current exploit was found specific to Apache httpd 2.4.68 — an expected outcome given the version's currency. This step demonstrated that raw scan tool output must be manually validated rather than trusted automatically.

## 4. Key Concepts Learned

- Setting up an isolated virtual lab environment (VirtualBox + NAT Network)
- Port/service scanning with Nmap (`-sV`, basic scan)
- The distinction between "filtered," "closed," and "open" port states
- Vulnerability research with `searchsploit` and the need to manually validate results
