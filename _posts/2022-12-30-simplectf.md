---
layout: post
title: Simple CTF
date: 2022-12-30 12:00 +0700
categories: [THM, easy]
---

### Introduction

[Simple CTF] is an easy CTF on TryHackMe that showcases scanning and enumeration, research, exploitation, and privilege escalation wich are some basics skills needed in CTF.

![simple](https://tryhackme-images.s3.amazonaws.com/room-icons/f28ade2b51eb7aeeac91002d41f29c47.png)



---

Let's start scaning the machine with nmap.

```sh
nmap -sSV -p- --open --min-rate 5000 <IP> -oN <outputfile.txt>
```

![Scan](/images/THM/simplectf/Captura.PNG)

We can see there are 3 ports open, 21(ftp), 80(http), 2222(ssh).

- How many services are running under port 1000?

>Answer: 2

- What is running on the higher port?

>Answer: ssh

If we move to the web page, we can see the Apache2 Default page.

![Web](/images/THM/simplectf/Captura2.PNG)

Let's scan the page with **gobuster**.

```sh
gobuster dir -u <URL> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -q -t <threads> -x <extensions>
```

![WebScan](/images/THM/simplectf/Captura3.PNG)

We can see there are 3 results

Let's move on to every page


![robots](/images/THM/simplectf/Captura4.PNG)

In **robots.txt**  there are nothing interesting but in the **simple** directory we can see a web page with the CMS **Made simple**.


![CMS](/images/THM/simplectf/Captura5.PNG)

If we look the version in the footer, we can see that the page is in the version 2.2.8.

![Version](/images/THM/simplectf/Captura6.PNG)

So making a little search about the cms and the version, we can see there is an CVE and one [exploit] for it.


![cve](/images/THM/simplectf/Captura7.PNG)

- What's the CVE you're using against the application?

>Answer: CVE-2019-9053

The CVE explains there is an issue and is it possible with the News module, through a crafted URL, to achieve unauthenticated blind time-based **SQL injection** via the m1_idlist parameter.

- To what kind of vulnerability is the application vulnerable?

>Answer: sqli

```sh
python2 46635.py
```

![exploit](/images/THM/simplectf/Captura8.PNG)

```sh
python2 46635.py -u <URL> --crack -w /usr/share/wordlists/rockyou.txt
```

![exploit2](/images/THM/simplectf/Captura9.PNG)

Let's run it and see what we get.

![results](/images/THM/simplectf/Captura10.PNG)

Et voila! We already have the credentials!

- What's the password?

>Answer: secret

Time to get into the machine with the credentials we've found.

```sh
ssh mitch@<IP> -p 2222

python -c 'import pty;pty.spawn("<shell>")'
```

![ssh](/images/THM/simplectf/Captura11.PNG)

- Where can you login with the details obtained?

>Answer: ssh

If we show the actual directory, we can see there is a file called user.txt. In it, we can found the user flag.

```sh
ls
cat user.txt
```

![userflag](/images/THM/simplectf/Captura12.PNG)

- What's the user flag?

>Answer: G00d j0b, keep up!

For see if there is other user, we list the /home/ directory, and we found the user sunbath.

```sh
ls /home/
```

![user](/images/THM/simplectf/Captura13.PNG)

- Is there any other user in the home directory? What's its name?

>Answer: sunbath

To finish the machine, we need to do the privileges escalation, and if we run **sudo -l** we can see that the **vim** binary can be executed with root privileges without password.

Knowing this, we can go to [GTFO] and search how to make this escalation with vim binary.

```sh
sudo -l
sudo vim -c ':!<shell>'
```

![sudo](/images/THM/simplectf/Captura14.PNG)

- What can you leverage to spawn a privileged shell?

>Answer: vim

```sh
cd /root
ls
cat root.txt
```

![rootflag](/images/THM/simplectf/Captura15.PNG)

- What's the root flag?

>Answer: W3ll d0n3. You made it!

 [Simple CTF]: https://tryhackme.com/room/easyctf
 [GTFO]:https://gtfobins.github.io/
 [exploit]:https://www.exploit-db.com/exploits/46635
