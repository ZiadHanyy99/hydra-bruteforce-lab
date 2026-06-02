# FTP & SSH Brute-Force Lab — Hydra

> Controlled brute-force attack simulation against FTP and SSH services using Hydra and Nmap in an isolated lab environment. Demonstrates how weak credentials are exploited and documents hardening recommendations to prevent such attacks.

**Author:** Ziad Hany Mohamed Salem  
**Department:** Cybersecurity  
**Environment:** Kali Linux + Target VM (VirtualBox)  
**Date:** 2026

---

## Objective

Simulate a real-world credential brute-force attack against two common network services (FTP and SSH) in a fully controlled lab environment. The goal is to understand the attacker's perspective and derive practical defensive recommendations.

---

## Lab Environment

```
VirtualBox Host
├── Kali Linux   (192.168.1.4) — attacker machine
└── Alice VM     (192.168.1.2) — target machine

Network: Host-Only Adapter — fully isolated, no internet exposure
```

---

## Tools Used

| Tool | Purpose |
|------|---------|
| Hydra | Automated brute-force attack tool |
| Nmap | Host discovery and open port identification |
| nano / cat | Wordlist creation and verification |

---

## Methodology

### Phase 1 — Host Discovery & Enumeration

Assigned IP addresses to both machines and verified connectivity before scanning.

```bash
# Check IP configuration
sudo ifconfig

# Verify connectivity to target
ping -c 4 192.168.1.2

# Discover live hosts on the network
nmap -sn 192.168.1.0/24

# Scan target for open ports and services
nmap -sV 192.168.1.2
```

**Result:** Target at `192.168.1.2` identified with two open ports:

| Port | Service | Version |
|------|---------|---------|
| 21/tcp | FTP | vsftpd |
| 22/tcp | SSH | OpenSSH |

---

### Phase 2 — Wordlist Preparation

Created username and password wordlists for the brute-force attack.

```bash
# Create wordlist files
touch username password

# Edit with potential usernames and passwords
nano username
nano password

# Verify contents
cat username
cat password
```

---

### Phase 3 — FTP Brute-Force with Hydra

```bash
hydra -L username -P password ftp://alice
```

**Result:** Hydra successfully cracked the FTP credentials.

| Field | Value |
|-------|-------|
| Username | `alice` |
| Password | `alice` |
| Time | Seconds |

This demonstrates the critical risk of using default or weak credentials — a username and password identical to the hostname is cracked instantly.

---

### Phase 4 — SSH Brute-Force Attempt

```bash
hydra -L username -P password ssh://192.168.1.2
```

**Result:** Hydra returned an error during the SSH brute-force attempt, likely caused by SSH rate limiting, connection throttling, or firewall restrictions on the target system — demonstrating that SSH hardening measures can effectively prevent this type of attack.

---

## Security Analysis

### Why This Attack Succeeded on FTP

- Default credentials (`alice:alice`) were in use
- No login attempt limit was configured
- No IP-based blocking or rate limiting was active
- FTP transmits credentials in plaintext — easily captured alongside the brute-force attempt

### Why SSH Resisted the Attack

- SSH connection throttling or firewall rules blocked rapid repeated attempts
- This demonstrates that even basic hardening significantly raises the cost of brute-force attacks

---

## Hardening Recommendations

| # | Recommendation | Implementation |
|---|---------------|----------------|
| 1 | Use strong passwords | Minimum 12 characters, mixed case, numbers, symbols — never use default credentials |
| 2 | Limit login attempts | Configure `fail2ban` to block IPs after 3–5 failed attempts |
| 3 | Disable FTP entirely | Replace with SFTP or FTPS — FTP transmits credentials in plaintext |
| 4 | SSH key authentication | Disable password-based SSH login, require public key authentication |
| 5 | Multi-factor authentication | Add TOTP or hardware key as a second factor for SSH |
| 6 | Network segmentation | Restrict access to FTP/SSH ports to specific trusted IP ranges via firewall |
| 7 | Keep services updated | Patch vsftpd and OpenSSH regularly to eliminate known vulnerabilities |

---

## Key Takeaway

A username and password identical to the hostname (`alice:alice`) was cracked by Hydra in seconds. This is not a sophisticated attack — it is the simplest possible brute-force attempt. The most effective defense is also the simplest: **never use default or weak credentials**.

---

## Full Report

Complete lab documentation is available in [`report.pdf`](./report.pdf).

---

## Disclaimer

This attack was performed exclusively in a controlled, isolated VirtualBox lab environment against a virtual machine set up specifically for this exercise. No real systems, networks, or users were targeted or affected. This project is intended solely for educational purposes to understand offensive techniques and improve defensive security posture.
