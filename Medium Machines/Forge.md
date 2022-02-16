# HTB - Forge

![cover](https://user-images.githubusercontent.com/58801547/154256461-3054c5c5-f852-49c3-be21-42c117617fb8.png)

## Linux - Medium

IP = 10.10.11.111

## Enumeration
Same as always I run `nmap` first.

**Nmap:** `nmap -sSVC -p- -v -T4 -oA nmap/forge forge.htb`

| PORT         | SERVICE & VERSION |
| -------------|:-------------:|
| 21/tcp     | ftp |
| 22/tcp     | OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 |
| 80/tcp     | Apache httpd 2.4.41 |

![nmap](https://user-images.githubusercontent.com/58801547/154244127-d7c029ff-86e2-41d3-9b27-da507c18ca0c.png)
> `ftp` is interesting but I need to find the credentials first.


**Gobuster (dir):** `gobuster dir -u http://forge.htb/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -z -o gobuster/forge-dir`

![gobuster_dir](https://user-images.githubusercontent.com/58801547/154246653-d17e6ca4-a2d1-4d87-b943-d2df42204163.png)
> There's the upload route.


**Gobuster (vhost):** `gobuster vhost -u http://forge.htb -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -z -o subdomain/sub.txt`

![gobuster_vhost](https://user-images.githubusercontent.com/58801547/154246976-ea9e2c3a-3a58-4b6f-9dcd-386ac8208c03.png)
> Found `admin.forge.htb` subdomain but can't directly access it.

![web1](https://user-images.githubusercontent.com/58801547/154247049-9362cf47-1ad8-4a44-bdd7-5d6ec498b9ef.png)
> Look like this web have SSRF vulnerability, So I try to inject with 127.0.0.1, localhost or admin.forge.htb but it's all getting filtered.

![found_vul](https://user-images.githubusercontent.com/58801547/154247634-086c0d01-ae5c-464d-8f17-9e34f564caa9.png)
> I bypass the filter with Uppercase character and It just works.

![meme](https://c.tenor.com/rkI1a8s2Z6QAAAAC/todd-howard-it-just-works.gif)

![admin](https://user-images.githubusercontent.com/58801547/154248479-23093ee3-b85d-47c9-8826-101c4d37869a.png)
> Now I got the `admin.forge.htb` page and It has another interesting route.

`user:heightofsecurity123!`
![announcement](https://user-images.githubusercontent.com/58801547/154248708-a05f643b-b70f-4b04-917e-8cbe650dc9a1.png)
> The announcements page give the information of `ftp` credential and how to upload the file with `ftp` through URL, So I can construct the payload to retrieve some essential file like `.ssh/id_rsa` the ssh private key.

![try_ssh](https://user-images.githubusercontent.com/58801547/154250396-95085771-4594-45e2-b38d-114e0e28c8fa.png)
> I try `ssh` with the `ftp` credential but It not work and It only uses the **Private key** for authentication.

## Exploitation

With the information on the announcements page, I can retrieve the file with the parameter `u`
The payload will be:
`http://admin.Forge.htb/upload?u=ftp://user:heightofsecurity123!@Forge.htb/.ssh/id_rsa`


Now I can login with `user`and got the user part as well.
![user](https://user-images.githubusercontent.com/58801547/154251209-796b7d61-34e1-4ae6-955e-80f929bce2b4.png)


## Privilege Escalation

1st things TODO after got user try `sudo -l`.

![sudo -l](https://user-images.githubusercontent.com/58801547/154251225-1c16f6db-0a8e-478e-94a9-d4d6fab246ff.png)
> The `remote-manage.py` file can lead to the root part.

![remote-manage.py](https://user-images.githubusercontent.com/58801547/154254262-f7ba2e73-5ab8-43a9-998a-c8af6eaa9a46.png)

![remote-manage.py2](https://user-images.githubusercontent.com/58801547/154251961-91dce625-5240-405b-8e62-ceb526f832ed.png)
> After reading the source code of `remote-manage.py` look like It'll create a socket server waiting for connection on a random port.
> And It checking the secret password that obvious is in the program.
> The actually interesting part is `pdb`.

![meme2](https://c.tenor.com/zHOmFEhWax8AAAAC/sarcasm-obviously.gif)

`pdb` is The module pdb defines an interactive source code debugger for Python programs
Then I just run the `remote-manage.py` with `sudo` permission and use `nc` to connect back to the socket server.

![error](https://user-images.githubusercontent.com/58801547/154255310-08961d9e-6aae-49ea-8883-99d00875d08b.png)
> And then I'm trying to cause an error to trigger the `pdb` with string input.

![image](https://user-images.githubusercontent.com/58801547/154255456-0e23e183-e08a-44f8-a987-868b9575d203.png)
> The exception handler triggers the pdb and then just import os module and boom! got the root shell.

This machine should be easy in my opinion lul. 

## Ref:

https://docs.python.org/3/library/pdb.html


