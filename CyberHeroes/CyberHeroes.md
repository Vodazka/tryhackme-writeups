# Room: [CyberHeroes](https://tryhackme.com/room/cyberheroes)

## Overview
This write‑up covers the *Cyber Heroes* room on [TryHackMe](https://tryhackme.com), created by [tryhackme](https://tryhackme.com/p/tryhackme), [cmnatic](https://tryhackme.com/p/cmnatic) and [THMDan](https://tryhackme.com/p/THMDan).

The objective is to log in to the CyberHeroes website.

## Setup
- Tools used: gobuster was used for enumeration but it is not required to solve the room.
- Notes: The challenge demonstrates that sometimes the simplest approach is enough.
---

## Method 

I started by inserting the target IP address into my browser as instructed, which led me to the index page of the CyberHeroes website.

![](media/website.png)

While browsing the website, I eventually reached the login page, which appeared to be the main entry point.

I needed a username and password to log in, and my attempts with default credentials were unsuccessful.

![](media/login.png)

I decided to run a gobuster scan to look for hidden directories that might provide aditional clues:

```bash
gobuster dir -u http://<Target IP> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

And discovered the `/assets` directory.

![](media/gobuster.png)

Inside `/assets` I found several subdirectories, but none contained useful content.

![](media/assets.png)

At this point, I decided to inspect the source code of the login page and discovered hardcoded credentials.

The username and password I needed were stored as variables in plaintext.

![](media/source-code.png)

Entering these credentials into the login form yielded the flag.

![](media/flag.png)

---

## Final Notes

#### This room highlights the following misconfigurations and/or weaknesses:

- Hardcoded credentials in source code.
- Exposed directories.


