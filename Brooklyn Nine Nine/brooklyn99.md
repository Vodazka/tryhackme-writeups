# Room: Brooklyn Nine Nine

## Overview
This write‑up covers the *Brooklyn Nine Nine* room on [TryHackMe](https://tryhackme.com/room/brooklynninenine), created by [Fsociety2006](https://tryhackme.com/p/Fsociety2006).
The objective is to gain root access to the target machine by capturing two flags: a user flag and a root flag.
The room can be completed in two different ways, offering multiple paths to ownership.

## Setup
- Tools used: 
- Notes: My initial focus was simply to gain access. Afterwards, I retraced my steps to follow the intended path “correctly,” and later explored the alternative method to achieve root access.

---

## Method 1
I began with enumeration to understand the target environment and gather clues for my next steps.
For this, I ran the following command:

```bash
nmap -sC -sV <Target IP> 
```

The output revealed useful information about the services running on the target.

![We can see ftp, ssh and apache running](media/nmap.png)

*We can see ftp, ssh and apache running*

I decided to go the apache route and opened the target ip in my browser.

![The page only displayed an immage with some text below it](media/webpage.png)

*The page only displayed an immage with some text below it*

Tried directory traversal without sucess, so I ran gobuster to enumerate directories and look for something useful.

![Which ended up not happening](media/gobuster.png)

*Which ended up not happening*

There had to be something on the webpage so I opened inspect mode and found a clue.

![Inspect mode revealed a clue pointing toward steganography](media/steganography.png)

*Inspect mode revealed a clue pointing toward steganography*

There was only one image on the page so I got its name using inspect mode.

![The image was named brooklyn99.jpg](media/brooklyn.png)

*The image was named brooklyn99.jpg*

I downloaded the image localy using wget.

```bash
wget http://<Target IP>/brooklyn99.jpg
```

Then, I attempted to extract hidden data using steghide.

```bash
steghide extract -sf brooklyn99.jpg
```

I was prompted for a password, which confirmed I was on the right track.
I tried a few common passwords and succeeded on my second attempt, which was surprisingly easy.

![This revealed valid credentials](media/holt.png)

*This revealed valid credentials*

Using those credentials, I logged in via SSH and obtained the user flag.

![An 'ls' of the home directory showed a file named nano.save, but it turned out to be irrelevant](media/sshuserflag.png)

*An 'ls' of the home directory showed a file named nano.save, but it turned out to be irrelevant*

With 'sudo -l', I was able to verify holt's permissions:

![Turns out we can run nano as root without a password](media/sudo-l.png)

*Turns out we can run nano as root without a password*

Since the user flag was at `/home/holt`, I decided to open `/root` directly with nano.

![And got the root flag](media/rootflag.png)

*And got the root flag*

At this point, the room was finished.
Still, I wasn't satisfied.
I knew I had to go back.

---

## Backtracking

Although I did obtain the root flag, I wasn’t satisfied with how I got there.

My first run relied on guessing, which gave me the flag but not actual root access.

This approach would never be acceptable in a real engagement.

So I went back.

I wanted to do it the right way, to actually escalate privileges.

Documenting both approaches highlights the difference between “getting lucky” and developing the skills needed to truly pwn a machine.

---

## Method 1 (Again)

I replicated the scenario in a lab to understand how limited '/bin/nano' privileges could be abused. 

It was during this process that I realized the 'sudoers' file had been available all along. 
The escalation path was clear.

I opened the sudoers file using nano and modified it to grant myself full permissions.

![Replacing '/bin/nano' with 'ALL'](media/sudoers-edited.png)

*Replacing '/bin/nano' with 'ALL'*

![Result in terminal](media/sudoers-terminal.png)

*Result in terminal*

With that change in place, I ran:

```bash
sudo su root
```
![This time I not only obtained the root flag, but full root access.](media/whoami-root.png)

*This time I not only obtained the root flag, but full root access.*

---

The seganography password can be obtained with:

```bash
stegcracker brooklyn99.jpg /usr/share/wordlists/rockyou.txt
```

![Stegcracker was not available on the THM machine, so I couldn’t brute force the password.](media/stegcracker.png)

*Stegcracker was not available on the THM machine, so I couldn’t brute force the password.*

---

## Method 2

![Nmap showing port 21 with FTP open](media/nmap.png)

*Nmap showing port 21 with FTP open*

![I logged anonymously, without a password.](media/ftp-anon-login.png)

*I logged anonymously, without a password.*

After listing the contents of the server, I found a note left by Amy. 

![FTP file transfer.](media/ftp-file-transfer.png)

*FTP file transfer.*

![Amy's note.](media/amynote.png)

*Amy's note.*

The note suggested Jake’s password was weak.  
I used Hydra with the 'rockyou.txt' wordlist to brute force his SSH login:

```bash
hydra -l jake -P /usr/share/wordlists/rockyou.txt ssh://<Target IP>
```

![There it is.](media/hydra-jake.png)

*There it is.*

I was then able to log in as jake and list his permissions.

![Jake can run the 'less' command with 'sudo'](media/jake-login-sudo-l.png)

*Jake can run the 'less' command with 'sudo'*

According to [GTFOBins](https://gtfobins.github.io/gtfobins/less/),this can be abused to escalate privileges.

![As we can see here](media/gtfobins-less.png)

*As we can see here*

And just like that.

![I got root](media/jake-privesc.png)

*I got root*
