# HTB - Explore

![cover](https://user-images.githubusercontent.com/58801547/141645088-c978f3d9-c220-4def-b855-46cd511358ba.png)

## Android - Easy
IP = 10.10.10.247

## Enumuration

Likes always run Nmap scan for opening port 

**Nmap:** `sudo nmap -sSVC -p- explore.htb -oA nmap/explore`

| Port         | Service      |
| -------------|:-------------:|
| 2222/tcp     | SSH-2.0-SSH Server - Banana Studio           |
| 5555/tcp     | freeciv       |
| 42135/tcp    | ES File Explorer Name Response httpd |
| 42207/tcp    | unknown       |
| 59777/tcp    | Bukkit JSONAPI httpd for Minecraft game server 3.6.0 |


![nmap1](https://user-images.githubusercontent.com/58801547/141645038-e445b2b3-8bd2-4e18-8169-cc7c7034c81d.png)

![nmap2](https://user-images.githubusercontent.com/58801547/141645389-ade36ba0-d99a-4ecf-acbb-8a08efdc88ec.png)

> Freeciv, ES File Explorer and Bukkit JSONAPI httpd is quiet interesting then I research some CVE, vulnerabilities about
> them and I found the **FreeCIV Arbitrary Code Execution** and **ES File Explorer - Arbitrary File Read** (the FreeCIV exploit can get root maybe this was privilege escalation part)

**Metasploit:** `Module ES File Explorer Open Port`

This module was create from [CVE-2019-6447](https://www.cvedetails.com/cve/CVE-2019-6447/),**ES File Explorer - Arbitrary File Read** So I use it to get more information on this machine

![met1](https://user-images.githubusercontent.com/58801547/141645965-03349cca-77f3-4e5a-bac7-b55dcff8ba86.png)

![met2](https://user-images.githubusercontent.com/58801547/141646692-b8ade327-3f15-4932-9d67-05f8b57af7d4.png)

> After list some files I found the `creds.jpg` file, looks like it contains user credentials, I access that path in browser `http://explore.htb:59777/storage/emulated/0/DCIM/creds.jpg` then I got some user credential 

![cred](https://user-images.githubusercontent.com/58801547/141646875-bfb3f5c8-6b28-4e20-a343-aaff46dcd5f9.png)

**SSH:** `ssh kristi@explore.htb -p 2222` with `Kristi:Kr1sT!5h@Rp3xPl0r3!`

![ssh](https://user-images.githubusercontent.com/58801547/141647437-4efc2217-512b-4527-bf56-e29244a98619.png)

> I try `SSH` with the Kristi user and got it now look for `user.txt` 

![user.txt](https://user-images.githubusercontent.com/58801547/141647685-4cc459b8-8fbd-40e7-9d7a-b1786bf40956.png)

> I list all files and found the link directory name `sdcard` link to primary storage of the device and then found `user.txt`

## Exploitation & Privilege Escalation

Now I try the Freeciv [CVE-2010-2445](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2010-2445) that I found early in the enumeration part I try `ADB` connect from my kali -> machine and it does not work and the remote machine does not have `ADB` too

So I use Tunneling to access the `5555` ADB port with this I can access remote machine specific port through my local machine

![tunneling](https://user-images.githubusercontent.com/58801547/141648285-4e906813-a3b7-417e-a348-d25d5f3b15a6.png)

![root](https://user-images.githubusercontent.com/58801547/141648759-dd55387e-26e4-4de6-9d5d-b5281869a5a9.png)
> Now I can use `ADB` to connect and got shell user since it doesn't require any authentication, I just `su` to login as root and got `root.txt`

## Referrences

https://www.rapid7.com/db/modules/auxiliary/scanner/http/es_file_explorer_open_port/

https://labs.f-secure.com/blog/hackin-around-the-christmas-tree/

https://packetstormsecurity.com/files/163311/Android-2.0-FreeCIV-Arbitrary-Code-Execution.html
