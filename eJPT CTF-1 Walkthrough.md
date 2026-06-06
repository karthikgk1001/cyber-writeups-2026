

## Introduction

Welcome! In this walkthrough, I'll explain how I approached and solved the **eJPT CTF-1** challenge and successfully collected all the flags.

To encourage learning and hands-on practice, I will **not** be sharing the actual flag values. Instead, I'll focus on the methodology, tools, and commands used to discover them.

---

# Flag 1: Identifying Search Engine Restrictions

### Challenge Hint

> This tells search engines what to avoid and what not to crawl.

The hint directly points to the **robots.txt** file. This file is commonly used by website administrators to instruct search engines which directories or files should not be indexed.

Simply append `/robots.txt` to the target URL:

```bash
http://target.ine.local/robots.txt
```

Reviewing the contents of this file reveals the location needed to obtain the first flag.



---

# Flag 2: Identifying the Website and Version

### Challenge Hint

> What website is running on the target, and what is its version?

For this flag, basic service enumeration is sufficient. Running an Nmap scan with service detection and version enumeration provides the required information.

### Command Used

```bash
nmap -sCV -A -O target.ine.local
```

### Explanation

- `-sC` runs default NSE scripts.
    
- `-sV` detects service versions.
    
- `-A` enables OS detection, version detection, script scanning, and traceroute.
    
- `-O` attempts operating system detection.
    

The scan output reveals the web application and its version, which corresponds to the second flag.


---

# Flag 3: Discovering Hidden Directories

### Challenge Hint

> Directory browsing might reveal where files are stored.

This flag requires directory enumeration. Any directory brute-forcing tool can be used, including:

- Dirb
    
- FFUF
    
- DirBuster
    
- Gobuster
    

### Example Command

```bash
dirb http://target.ine.local
```

While reviewing the results, several directories may appear interesting. Some may lead to login pages or inaccessible locations that do not contribute to the challenge.

However, pay close attention to paths discovered by the scanner, even if direct access initially appears restricted.

One of the discovered paths is:

```text
/wp-content/uploads/
```

Navigating to:

```bash
http://target.ine.local/wp-content/uploads/
```

reveals the location containing the third flag.


---

# Flag 4: Finding an Exposed Backup File

### Challenge Hint

> An overlooked backup file in the webroot can be problematic if it reveals sensitive configuration details.

This flag is largely based on understanding common WordPress misconfigurations.

The hint suggests the presence of a backup file that has been accidentally left accessible on the web server.

Researching common WordPress backup file locations can help identify the correct path. Accessing the location downloads a file with a `.bak` extension.

After downloading the file, inspect its contents using:

```bash
cat filename.bak
```

The contents reveal the fourth flag.


---

# Flag 5: Website Mirroring and Source Analysis

### Challenge Hint

> Certain files may reveal something interesting when mirrored.

The keyword **"mirrored"** strongly suggests the use of a website mirroring tool such as **HTTrack**.

### Command Used

```bash
httrack http://target.ine.local -O INE
```

This command creates a local copy of the target website.

Once the website has been mirrored, navigate to the output directory and search through the downloaded files.

Since previous flags follow a consistent naming convention, searching recursively for the fifth flag is straightforward.

### Command Used

```bash
grep -i "FLAG5" -R target.ine.local/
```

This command recursively searches all mirrored files for occurrences of "FLAG5".

The output reveals the location of the final flag.



---

# Conclusion


By combining:

- Basic web reconnaissance
    
- Service enumeration
    
- Directory brute-forcing
    
- Backup file discovery
    
- Website mirroring
    

all five flags can be successfully identified without requiring any advanced exploitation techniques.

This challenge serves as an excellent introduction to the enumeration and reconnaissance techniques commonly used during penetration testing engagements.
