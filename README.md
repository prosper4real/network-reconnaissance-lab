# Network Reconnaissance & Service Enumeration Lab

## Overview

This project demonstrates a structured network reconnaissance and service enumeration assessment against an intentionally vulnerable Metasploitable 2 virtual machine in a controlled lab environment.

The goal was to identify the full attack surface of the target by combining multiple reconnaissance techniques, accurately fingerprinting services, and translating technical findings into clear security recommendations.

## Lab Environment

| Component              | Details                          |
|------------------------|----------------------------------|
| Attacker Machine       | Kali Linux                       |
| Target                 | Metasploitable 2                 |
| Target IP              | 10.10.10.2                       |
| Primary Tool           | Nmap 7.94                        |
| Additional Tools       | curl, Firefox                    |
| Network                | Isolated lab environment         |

## Methodology

The assessment followed a deliberate multi-stage approach:

1. Initial TCP port discovery (default Nmap ports)
2. Service and version enumeration
3. Full TCP port scan (all 65,535 ports)
4. Targeted service enumeration of newly discovered ports
5. HTTP header and technology fingerprinting
6. Web content discovery using NSE scripts
7. Manual verification of critical web findings

## Reconnaissance Results

### 1. Initial Port Discovery


nmap 10.10.10.2 -oN scans/initial-scan.txt


- **23 open TCP ports** identified

### 2. Service & Version Enumeration


nmap -sV 10.10.10.2 -oN scans/service-enumeration.txt


| Port      | Service              | Detected Version / Notes                  |
|-----------|----------------------|-------------------------------------------|
| 21/tcp    | FTP                  | vsftpd 2.3.4                              |
| 22/tcp    | SSH                  | OpenSSH 4.7p1 Debian                      |
| 23/tcp    | Telnet               | Linux telnetd                             |
| 25/tcp    | SMTP                 | Postfix smtpd                            |
| 53/tcp    | DNS                  | ISC BIND 9.4.2                            |
| 80/tcp    | HTTP                 | Apache 2.2.8 (Ubuntu) DAV/2               |
| 139/445   | SMB                  | Samba smbd 3.X – 4.X                      |
| 1524/tcp  | Bind Shell           | Metasploitable root shell                 |
| 2121/tcp  | FTP                  | ProFTPD 1.3.1                             |
| 3306/tcp  | MySQL                | MySQL 5.0.51a                             |
| 5432/tcp  | PostgreSQL           | PostgreSQL 8.3.x                          |
| 5900/tcp  | VNC                  | VNC protocol 3.3                          |
| 6667/tcp  | IRC                  | UnrealIRCd                                |
| 8180/tcp  | HTTP                 | Apache Tomcat/Coyote JSP engine           |

### 3. Full TCP Port Scan


nmap -p- 10.10.10.2 -oN scans/full-port-scan.txt


- **30 open TCP ports** discovered
- 7 additional ports found beyond the default scan

**Newly discovered ports:**
- 3632/tcp, 6697/tcp, 8787/tcp, 33927/tcp, 36312/tcp, 43694/tcp, 50303/tcp

### 4. Additional Service Enumeration


nmap -sV -p 3632,6697,8787,33927,36312,43694,50303 10.10.10.2 -oN scans/additional-services.txt


| Port       | Identified Service          |
|------------|-----------------------------|
| 3632/tcp   | distccd v1                  |
| 6697/tcp   | UnrealIRCd                  |
| 8787/tcp   | Ruby DRb RMI                |
| 33927/tcp  | Java RMI Registry           |
| 36312/tcp  | RPC Status                  |
| 43694/tcp  | mountd                      |
| 50303/tcp  | nlockmgr                    |

### 5. Web Service Enumeration

**HTTP Headers & Technology Discovery**

nmap -p 80 --script http-title,http-headers 10.10.10.2 -oN scans/web-enumeration.txt


- Page Title: Metasploitable2 - Linux  
- Server: Apache/2.2.8 (Ubuntu) DAV/2  
- PHP: PHP/5.2.4-2ubuntu5.10

**Web Content Discovery**

nmap -p 80 --script http-enum 10.10.10.2 -oN scans/web-content-enumeration.txt


Notable findings:
- `/phpinfo.php` – Publicly accessible
- `/phpMyAdmin/` – Login interface exposed
- `/tikiwiki/` – TikiWiki application detected
- Directory listings on `/doc/` and `/icons/`

**Manual Verification**
- Confirmed `/phpinfo.php` returns HTTP 200 and discloses detailed PHP configuration
- Confirmed `/phpMyAdmin/` is reachable and presents a login page

## Key Security Findings

### 1. Large Exposed Attack Surface
30 open TCP ports including remote administration, databases, file sharing, and legacy protocols significantly increase the attack surface.

**Recommendation:** Disable unnecessary services and enforce network segmentation / firewall rules.

### 2. Detailed Version Information Disclosure
Multiple services revealed specific outdated versions (Apache 2.2.8, PHP 5.2.4, OpenSSH 4.7p1, MySQL 5.0.51a, etc.).

**Recommendation:** Keep software updated and minimize version disclosure where possible.

### 3. Insecure Legacy Services
Telnet and r-services were exposed.

**Recommendation:** Disable legacy clear-text remote access protocols and enforce SSH-only access.

### 4. Publicly Accessible phpinfo.php
The PHP information page was reachable without authentication.

**Recommendation:** Remove diagnostic pages from production systems.

### 5. Exposed phpMyAdmin Interface
Database administration interface was accessible from the network.

**Recommendation:** Restrict administrative interfaces to trusted networks and enforce strong authentication.

### 6. Importance of Full Port Scanning
Default Nmap scan missed 7 open ports that were only discovered during the full 65,535-port scan.

## Skills Demonstrated

- Methodical network reconnaissance
- Nmap port scanning & service version detection
- Full TCP port range analysis
- HTTP enumeration with NSE scripts
- Manual verification of findings
- Attack surface analysis
- Clear technical documentation and remediation recommendations

## Evidence

All scan outputs are available in the `/scans` directory.  
Screenshots are available in the `/screenshots` directory.

## Disclaimer

All testing was performed against Metasploitable 2, an intentionally vulnerable virtual machine, inside a fully isolated lab environment for educational and portfolio purposes only. No unauthorized systems were targeted.
