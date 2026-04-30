# File Analysis - Scanning Result
Vulnerability Analysis Lab - W6 &amp; W7

# Question 1: Analyse packet1.pcap and find the flag. 

<img width="1012" height="469" alt="q1" src="https://github.com/user-attachments/assets/17630631-6db1-4e2b-b264-df1ddb9814c0" />

<img width="1141" height="674" alt="qq1" src="https://github.com/user-attachments/assets/71cd5ebc-167f-483c-a166-30f51f8ed815" />

---

# Question 2: Analyse packet2.pcap and find the flag. 

<img width="1906" height="994" alt="q2" src="https://github.com/user-attachments/assets/80fb7098-c1de-4fc3-bf48-5cc3bebaf85e" />

---

# Question 3: Interpret an Nmap Output

PORT     STATE SERVICE     VERSION
21/tcp   open ftp         vsftpd 2.3.4
22/tcp   open ssh         OpenSSH 5.3p1
80/tcp   open http        Apache 2.2.8
139/tcp  open netbios-ssn
445/tcp  open microsoft-ds Windows 7 Professional 7601 Service Pack 1

### 1. What can an attacker do with each port? 

**i. Port 21 (FTP - vsftpd 2.3.4)**
-	An attacker can upload, download, rename and delete files on the FTP server.
-	Perform brute force attacks to guess login credentials.
-	Exploit the known backdoor in vsftpd 2.3.4 to gain a root shell without authentication.
-	If anonymous login is enabled, access files without any credentials.

**ii. Port 22 (SSH – OpenSSH 5.3p1)**
-	Perform brute force attacks on usernames and passwords.
-	If, successful, gain remote command execution on the server.
-	Use port forwarding to access internal network resources.
-	Install backdoors or malware for persistent access.
-	Transfer files using SCP or SFTP.

**iii. Port 80 (HTTP – Apache 2.2.5)**
-	Perform web application attacks such as SQL injection, Cross-Site Scripting (XSS), Local File Inclusion (LFI), Remote File Inclusion (RFI) and Command Injection.
-	Browse directories if directory listing is misconfigured.
-	Upload a webshell to gain persistent access.
-	Deface the website.
-	Exploit known Apache vulnerabilities for denial of service or remote code execution.

**iv. Port 139 (NetBIOS – Ssn)**
-	Enumerate NetBIOS names, workgroups and shared resources
-	Gather information about users, running services and the network structure.
-	Exploit legacy NetBIOS vulnerabilities to intercept or modify traffic.
-	Perform SMB relay attacks (if SMB over NetBIOS is enabled).
-	Conduct man-in-the-midle (MITM) attacks on NetBIOS name resolution. 


**v. Port 445 (MICROSOFT-DS – Windows 7 Professional 7601 Service Pack 1)**
-	Access shared files, folders and printers on the Windows 7 machine.
-	Remote code execution via EternalBlue (MS17-010) – one of the most critical attacks.
-	Perform pass-the-hash attacks to authenticate without cracking passwords.
-	Dump credentials from memory using tools like Mimikatz.
-	Move laterally across the network to compromise other systems.
-	Deploy ransomware (e.g., WannaCry used port 445).
-	Enumerate users, groups and system information.

---

### 2. What vulnerabilities are likely present based on the version?

| Service | Version | Likely Vulnerabilities |
|---------|---------|------------------------|
| vsftpd | 2.3.4 | **CVE-2011-2523** – Backdoor command execution. Sending a username containing `:)` triggers a backdoor on port 6200, giving a root shell. |
| OpenSSH | 5.3p1 | **CVE-206-6210** – User enumeration vulnerability allows attackers to check if a username exists.<br>**CVE-2008-5161** – Plaintext recovery attack. |
| Apache | 2.2.8 | **CVE-2011-3192** – Range header denial of service (Apache Killer).<br>**CVE-2012-0052** – HTTP request smuggling vulnerability. Multiple outdated modules with known exploits. |
| Windows 7 SP1 | SMB | **MS17-010 (EternalBlue)** – Critical remote code execution via SMB port 445.<br>**MS08-067** – Older but possible if not patched.<br>SMB relay attacks.<br>ZeroLogon (**CVE-2020-1472**) if system is a domain controller. |

---

### 3. Which one is the highest risk and why?

The highest risk is port 21 which is running vsftpd version 2.3.4. This version contains a well-known backdoor vulnerability that is triggered when an attacker sends a username containing the smiley face characters ":)". When this backdoor is triggered, it opens another port which is port 6200 and provides the attacker with a root shell on the target system without requiring any password or authentication. This makes the vulnerability extremely dangerous because the attacker can gain full control of the system immediately and remotely. Public exploits for this vulnerability are readily available, including a module in the Metasploit framework, making it trivially easy for any attacker to compromise the system. While port 445 on Windows 7 with EternalBlue is also very critical, it sometimes requires additional conditions or bypassing firewalls, whereas the vsftpd backdoor works reliably without any authentication. Therefore, port 21 with vsftpd 2.3.4 is the highest risk.

---

### 4.	What attack path can be built from this?

An attacker can build a multi-stage attack path starting from the vulnerable FTP service. First, the attacker performs reconnaissance using Nmap and discovers port 21 running vsftpd 2.3.4, port 445 running SMB on Windows 7, and port 80 running Apache 2.2.8. The attacker then initiates the attack by connecting to port 21 and sending a username containing the smiley face symbol :) to trigger the backdoor in vsftpd 2.3.4. This immediately opens port 6200, and the attacker connects to it to gain a root shell on the FTP server without needing any password. With root access on the FTP server, the attacker can now use this compromised system as a pivot point to scan the internal network. The attacker discovers the Windows 7 machine with port 445 open and uses the EternalBlue exploit (MS17-010) to attack the SMB service. This gives the attacker SYSTEM level access on the Windows 7 machine. From there, the attacker can dump credentials using Mimikatz, move laterally to other systems in the network, install backdoors for persistent access, and exfiltrate sensitive data. An alternative attack path is through port 80, where the attacker could exploit vulnerabilities in Apache 2.2.8 or the web application to gain a web shell, then escalate privileges to root and pivot to the Windows 7 machine via SMB.

---

### 5.	What should be the remediation?

**i. Port 21 (FTP - vsftpd 2.3.4)**
-	Upgrade vsftpd to the latest version that does not contain the backdoor or disable FTP completely and use SFTP (SSH File Transfer Protocol) instead.
-	Disable anonymous login to prevent unauthorized file access.
-	Restrict FTP access using firewall rules to allow only trusted IP addresses.
-	 Regularly audit FTP logs for any suspicious activity such as usernames containing :).

**ii.	Port 22 (SSH – OpenSSH 5.3p1)**
-	Upgrade OpenSSH to the latest version (9.x or newer).
-	Disable root login by setting PermitRootLogin no in sshd_confid.
-	Use key-based authentication only disable password authentication.
-	Implement fail2ban to automatically block IP addresses that perform brute force attacks.
-	Change the default SSH port from 22 to a non-standard port as an additional layer of security.
-	Enable two-factor authentication (2FA) if possible.

**iii.	Port 80 (HTTP – Apachhe 2.2.8)**
-	Upgrade Apache to version 2.4.x which is the latest stable version.
-	Remove unnecessary modules to reduce the attack surface.
-	Implement a Web Application Firewall (WAF) such as ModSecurity.
-	Use HTTPS/LTS and redirect all HTTP traffic to HTTPS.
-	Perform input validation and output encoding to prevent SQL injection, XSS and other injection attacks.
-	Disable directory listing to prevent information disclosure.
-	Regularly update all web applications and content management systems.
-	Harden PHP configuration by disabling dangerous functions like exes and system.

**iv.	Port 139 (NetBIOS – Ssn)**
-	Disable NetBIOS over TCP/IP if is not needed in the network.
-	Block port 139 at the firewall/
-	Use modern SMB (port 445) instead of legacy NetBIOS-based SMB.

**v.	Port 445 (MICROSOFT-DS – Windows 7 Professional 7601 Service Pack 1)**
-	Upgrade Windows 7 to Windows 10, Windows 11 or Windows Server 2022 immediately because Windows 7 is End of Life (EOL) and no longer receives security patches.
-	If upgrade is impossible, apply the MS17-010 (EternalBlue) patch immediately.
-	Disable SMv1 completely and use only SMBv3.
-	Block port 445 at the firewall unless it is absolutely necessary for business operations.
-	Isolate legacy systems like Windows 7 to a separate VLAN with no direct internet access.
-	Deploy Endpoint Detection and Response (EDR) on all Windows systems.
-	Use network segmentation to limit lateral movement in case of a breach.

---

# Question 4: Identify the OS (OS Fingerprinting) - TTL

<img width="1150" height="405" alt="image 1" src="https://github.com/user-attachments/assets/c372881f-36ef-4030-9cc6-dda70a0de7ad" />

Answer: **Linux (Unix-based OS)**

<img width="303" height="207" alt="image 2" src="https://github.com/user-attachments/assets/5e71689b-9069-4137-a69d-ac2a891ecf15" />

Answer: **Windows OS – based on initial TTL value of 128 (observed TTL = 125 after hops)**

<img width="847" height="195" alt="image 3" src="https://github.com/user-attachments/assets/1c6087c4-5006-4f27-bef7-0f6d4c2ded53" />

Answer: **Windows  OS – based on default TTL value of 128**

---

# Question 5: Analyse the Nessus file

Upload to your nessus (Network_Scan.nessus) and analyse the files. Focus on critical or high findings that was identifies in analysis named “Ghostcat”.

<img width="970" height="634" alt="5" src="https://github.com/user-attachments/assets/15bd6cbe-8395-49c7-b8ad-8163b57b4058" />

<img width="784" height="336" alt="55" src="https://github.com/user-attachments/assets/f07f2141-b90e-4fff-aa9a-ed01ee8e9cbc" />

<img width="342" height="477" alt="555" src="https://github.com/user-attachments/assets/92d2bcbd-7acc-4c74-9a71-4096f227cc9c" />

<img width="339" height="691" alt="5555" src="https://github.com/user-attachments/assets/21d74a0a-7052-4322-87d1-6837111252ff" />


**1. What is the affected Port number.**
- 8009

**2. What is the Affected protocol.**
- AJP (Apache JServ Protocol)

**3. What is the CVSS Score of vulnerability found.**
- 9.8 (Critical)

**4. Can you find any exploit related to this vulnerability?**
- Yes – Exploit code is publicly available and Nessus was able to exploit the issue

**5.	Find CVE for this vulnerability.**
- The primary CVE for Ghostcat is CVE-2020-1938, while CVE-2020-1745 is also listed as a related CVE in the Nessus scan



