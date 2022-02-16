# HTB - Forge

![cover](https://user-images.githubusercontent.com/58801547/154243455-c2fc2f32-b101-4966-a1de-c79114b96feb.png)

## Linux - Medium

## Enumeration
Same as always run `nmap` first

** Nmap: ** `nmap -sSVC -p- -v -T4 -oA nmap/forge forge.htb`

| PORT         | SERVICE & VERSION |
| -------------|:-------------:|
| 21/tcp     | ftp |
| 22/tcp     | OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 |
| 80/tcp     | Apache httpd 2.4.41 |

![nmap](https://user-images.githubusercontent.com/58801547/154244127-d7c029ff-86e2-41d3-9b27-da507c18ca0c.png)
> FTP is interesting but I need to find the credentials first

** Gobuster (dir): **
![gobuster_dir](https://user-images.githubusercontent.com/58801547/154246653-d17e6ca4-a2d1-4d87-b943-d2df42204163.png)
> There's the upload route

** Gobuster (vhost): **
![gobuster_vhost](https://user-images.githubusercontent.com/58801547/154246976-ea9e2c3a-3a58-4b6f-9dcd-386ac8208c03.png)
> Found `admin.forge.htb` subdomain but can't directly access it

![web1](https://user-images.githubusercontent.com/58801547/154247049-9362cf47-1ad8-4a44-bdd7-5d6ec498b9ef.png)
> Look Like This web have SSRF vulnerability, so I try to inject with 127.0.0.1, localhost or admin.forge.htb but it's all get filterd

![image](https://user-images.githubusercontent.com/58801547/154247634-086c0d01-ae5c-464d-8f17-9e34f564caa9.png)
> I bypass the filtered with Uppercase character and It just work

![image](https://user-images.githubusercontent.com/58801547/154247936-5979f665-42cd-4a21-ac20-9c85a97c36e5.png)


## Exploitation


## Privilege Escalation

