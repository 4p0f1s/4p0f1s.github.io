---
layout: post
title: Chill Hack
date: 2023-02-03 12:00 +0700
categories: [THM, easy]
---

### Introduction

[Chill Hack] is an easy CTF on TryHackMe that showcases scanning and enumeration, research, exploitation, and privilege escalation wich are some basics skills needed in CTF.

![chillhack](https://tryhackme-images.s3.amazonaws.com/room-icons/f28ade2b51eb7aeeac91002d41f29c47.png)



---

Let's start scaning the machine with nmap.

```sh
nmap -sSV -p- --open --min-rate 5000 <IP> -oN <outputfile.txt>
```

![Scan](/images/THM/chillhack/Captura.PNG)

 [Chill Hack]: https://tryhackme.com/room/easyctf
 [GTFO]:https://gtfobins.github.io/
 [exploit]:https://www.exploit-db.com/exploits/46635
