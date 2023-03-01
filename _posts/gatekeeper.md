---
layout: post
title: Gatekeeper
date: 2023-01-02 11:00 +0700
categories: [THM, medium]
---

### Introduction

[Gatekeeper] is a medium CTF on TryHackMe.

![Gatekeeper](https://tryhackme-images.s3.amazonaws.com/room-icons/8979e58d84147f0720773889be95f4d9.jpeg)


---

Let's start scaning the machine with nmap.

We can see some ports open.

```sh
nmap -sSV -p- --open --min-rate 5000 <IP> -oN <outputfile.txt>
```

![scan](/images/THM/relevant/Captura.PNG)

[Gatekeeper]:https://tryhackme.com/room/gatekeeper
