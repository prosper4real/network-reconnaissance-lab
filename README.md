# Network Reconnaissance and Service Enumeration Lab

## Objective

The objective of this project was to perform network reconnaissance and
service enumeration against an intentionally vulnerable Metasploitable 2
virtual machine in a controlled lab environment.

The assessment focused on identifying open TCP ports, determining the
services and versions running on those ports, discovering services outside
Nmap's default scan range, and investigating the exposed HTTP attack surface.

## Lab Environment

- Attacker Machine: Kali Linux
- Target Machine: Metasploitable 2
- Target IP: 10.10.10.2
- Primary Tool: Nmap 7.94
- Additional Tool: curl
- Environment: Local isolated cybersecurity lab

## Methodology

The assessment was performed in several stages:

1. Initial TCP port discovery
2. Service and version enumeration
3. Full 65,535 TCP port scan
4. Enumeration of additional discovered services
5. HTTP header and technology enumeration
6. Web content discovery
7. Manual verification of selected HTTP findings

## Reconnaissance Results

### Initial Port Discovery

An initial Nmap scan was performed to identify commonly exposed TCP services on the target.


nmap 10.10.10.2 -oN initial-scan.txt
The initial scan identified 23 open TCP ports.
Service and Version Enumeration
Service detection was then performed to identify the software running behind the discovered ports.

nmap -sV 10.10.10.2 -oN service-enumeration.txt
Notable services identified included:
| Port | Service | Detected Version / Information |
|------|---------|--------------------------------|
| 21 | FTP | vsftpd 2.3.4 |
| 22 | SSH | OpenSSH 4.7p1 Debian |
| 23 | Telnet | Linux telnetd |
| 25 | SMTP | Postfix smtpd |
| 53 | DNS | ISC BIND 9.4.2 |
| 80 | HTTP | Apache 2.2.8 (Ubuntu) DAV/2 |
| 139/445 | SMB | Samba smbd 3.X - 4.X |
| 1524 | Bind Shell | Metasploitable root shell |
| 2121 | FTP | ProFTPD 1.3.1 |
| 3306 | MySQL | MySQL 5.0.51a |
| 5432 | PostgreSQL | PostgreSQL 8.3.x |
| 5900 | VNC | VNC protocol 3.3 |
| 6667 | IRC | UnrealIRCd |
| 8180 | HTTP | Apache Tomcat/Coyote JSP engine |

Full TCP Port Scan
Because Nmap's default scan does not examine every possible TCP port, a full scan of all 65,535 TCP ports was performed.
nmap -p- 10.10.10.2 -oN full-port-scan.txt

The full scan identified 30 open TCP ports, revealing additional services that were not present in the initial default scan.
Additional ports discovered included:
- 3632/tcp
- 6697/tcp
- 8787/tcp
- 33927/tcp
- 36312/tcp
- 43694/tcp
- 50303/tcp
Additional Service Enumeration
The newly discovered ports were examined using Nmap service detection.
nmap -sV -p 3632,6697,8787,33927,36312,43694,50303 10.10.10.2 -oN additional-services.txt

This identified:
| Port | Identified Service |
|------|--------------------|
| 3632 | distccd v1 |
| 6697 | UnrealIRCd |
| 8787 | Ruby DRb RMI |
| 33927 | Java RMI Registry |
| 36312 | RPC Status |
| 43694 | mountd |
| 50303 | nlockmgr |

This demonstrated why full-port scanning and service enumeration are separate but complementary reconnaissance steps. Some high-numbered services were not discovered during the default Nmap scan, and several could only be accurately identified after version detection was performed.
## Web Service Enumeration

Port 80 was identified as running Apache HTTP Server 2.2.8. Additional HTTP enumeration was performed to gather information about the web server and discover exposed web resources.

### HTTP Headers and Technology Discovery

nmap -p 80 --script http-title,http-headers 10.10.10.2 -oN web-enumeration.txt

The scan identified:
- Page title: Metasploitable2 - Linux
- Web server: Apache/2.2.8 (Ubuntu) DAV/2
- PHP: PHP/5.2.4-2ubuntu5.10
The HTTP response headers disclosed specific server and PHP version information, which could provide useful information to an attacker during reconnaissance.
Web Content Discovery
The Nmap http-enum NSE script was used to identify exposed web applications, files, and directories.

nmap -p 80 --script http-enum 10.10.10.2 -oN web-content-enumeration.txt
The following resources were discovered:
| Resource | Observation |
|----------|-------------|
| `/tikiwiki/` | TikiWiki application detected |
| `/test/` | Test page exposed |
| `/phpinfo.php` | PHP information page detected |
| `/phpMyAdmin/` | phpMyAdmin interface detected |
| `/doc/` | Directory listing detected |
| `/icons/` | Directory listing detected |
| `/index/` | Potentially interesting directory |

Manual Verification
Selected findings were manually verified through the browser and with curl.
Exposed PHP Information Page
The /phpinfo.php resource was accessible from the network.

curl -I http://10.10.10.2/phpinfo.php
The server returned:

HTTP/1.1 200 OK
Server: Apache/2.2.8 (Ubuntu) DAV/2
X-Powered-By: PHP/5.2.4-2ubuntu5.10
Content-Type: text/html

The page was also successfully loaded in Firefox. An exposed PHP information page can disclose detailed information about the server's PHP configuration and environment.
Exposed phpMyAdmin Interface
The /phpMyAdmin/ path was also manually tested.

curl -I http://10.10.10.2/phpMyAdmin/

The server returned HTTP/1.1 200 OK, and the phpMyAdmin login interface was accessible through Firefox.
This demonstrates that a database administration interface was exposed over the web. No authentication bypass or unauthorized database access was attempted during this verification.

## Key Security Findings

### 1. Large Exposed Attack Surface

The full TCP scan identified 30 open ports on the target system. These included remote administration services, database services, file-sharing protocols, web services, and legacy protocols.

The number and variety of exposed services increase the attack surface of the system. Each exposed service represents a potential entry point that would require assessment, patching, configuration review, and access control in a production environment.

**Recommendation:** Disable unnecessary services and restrict access to required services using firewall rules and network segmentation.

### 2. Service and Version Information Disclosure

Service enumeration revealed specific software and version information, including Apache 2.2.8, PHP 5.2.4, OpenSSH 4.7p1, MySQL 5.0.51a, and ProFTPD 1.3.1.

Detailed version information can help an attacker research known vulnerabilities affecting the exposed software.

**Recommendation:** Keep software supported and patched, minimize unnecessary version disclosure where practical, and regularly perform vulnerability assessments.

### 3. Insecure Legacy Remote Services

Telnet and the `rsh` family of services were exposed on the target.

These legacy remote-access protocols lack modern security protections and can expose sensitive communications or provide unnecessary remote-access paths.

**Recommendation:** Disable legacy remote-access services where possible and use secure alternatives such as SSH.

### 4. Exposed PHP Information Page

The `/phpinfo.php` page was publicly accessible and returned HTTP 200.

The page disclosed information about the PHP environment and server configuration that could assist further reconnaissance.

**Recommendation:** Remove publicly accessible diagnostic pages such as `phpinfo.php` from production systems.

### 5. Exposed phpMyAdmin Interface

The `/phpMyAdmin/` administration interface was accessible from the network and displayed a login page.

Although no authentication bypass was attempted, exposing database administration interfaces increases the attack surface and may provide attackers with a target for credential attacks or vulnerability research.

**Recommendation:** Restrict administrative interfaces to trusted networks or authorized management hosts and enforce strong authentication.

### 6. Additional Services Discovered Outside the Default Scan

The default Nmap scan identified 23 open TCP ports, while the full 65,535-port scan identified 30.

Additional service enumeration confirmed services including `distccd`, UnrealIRCd, Ruby DRb, Java RMI, and RPC-related services.

This demonstrated that relying only on Nmap's default port selection can leave parts of a system's exposed attack surface undiscovered.

## What I Learned

This project helped me understand that network reconnaissance involves more than simply identifying open ports.

I learned how to:

- Perform initial and full TCP port discovery with Nmap.
- Use service/version detection to identify software behind open ports.
- Understand the difference between port-based service guesses and service detection.
- Identify services missed by Nmap's default port scan.
- Use Nmap NSE scripts for HTTP enumeration.
- Verify web findings manually using a browser and `curl`.
- Interpret reconnaissance results in terms of attack surface and security risk.
- Document findings and remediation recommendations in a structured security report.

One important observation was that the full-port scan discovered seven additional open ports that were not identified during the default scan. Service detection also provided more accurate identification of several services that initially appeared as unknown or were identified only by their common port association.

## Disclaimer

All testing in this project was performed against Metasploitable 2, an intentionally vulnerable virtual machine, inside a controlled lab environment for educational purposes.
