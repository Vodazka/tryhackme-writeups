## Commands Used

---

#### gobuster
`gobuster - Directory/file & DNS busting tool written in Go`
```
gobuster dir -u https://futurevera.thm -k -w /usr/share/wordlists/dirb/common.txt
```

```
gobuster vhost -u https://futureeva.thm --append-domain -k -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
```

> dir - the classic directory brute-forcing mode

>vhost - virtual host brute-forcing mode - not the same as DNS

>-u, --url string  

>-w, --wordlist string

>-k
>> Skip TLS certificate verification

>--append-domain, --ad
>> Append main domain from URL to words from wordlist. Otherwise the fully qualified domains need to be specified in the wordlist. (default : false)

---

#### nmap

`nmap - Network exploration tool and security / port scanner`
```
nmap futurevera.thm
```

---

