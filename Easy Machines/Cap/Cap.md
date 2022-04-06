# HTB - Cap 

![Cap](https://user-images.githubusercontent.com/58801547/161933091-35dc4b85-dadf-485a-981c-6e12180f649a.png)

## Linux - Easy
IP = 10.10.10.245

First Add hostname to `/etc/hosts` file, name **cap.htb**

![image](https://user-images.githubusercontent.com/58801547/134887782-75fefd0e-13eb-4d01-8aa4-deccc54761d7.png)

## Enumuration
**Nmap:** `sudo nmap -A cap.htb`

  Use Nmap scan for the open ports, 
Here is some service information and web-server banner

- 21  ftp vsfp 3.0.3
- 22  ssh
- 80  http gunicorn

(gunicorn is a python Web Server Gateway Interface (WSGI) use for UNIX system).

Somehow maybe the FTP and gunicorn have the vulnerability, after googling around I didn't find any critical or High vulnerability that can lead to exploit (trying FTP with anonymous login doesn't works too).

![image](https://user-images.githubusercontent.com/58801547/134888516-fa417f77-fe02-40b7-a93a-4395f9fc0613.png)


![image](https://user-images.githubusercontent.com/58801547/134891674-7d8e991a-5c38-4ca7-90bd-ac7902c3f365.png)

**Gobuster:** `gobuster dir -u http://cap.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 20 -z`

Enum directory of the target with gobuster dir mode, found interesting directory and now I should go to the web and testing some workflow or how this web-app works.

![image](https://user-images.githubusercontent.com/58801547/134892923-ea7602b9-a1f9-45a9-9f8c-bf704b5f3f30.png)

After discovery how this web-application works the data page contain logging of packet and store to pcap file in order and index it seem to have **broken access control** here, then I try to change index of data page by increase, decrease it and found this 0.pcap file as image below.

![image](https://user-images.githubusercontent.com/58801547/134893920-02c85de2-5fb5-4427-9475-193e9df7e1f1.png)
![image](https://user-images.githubusercontent.com/58801547/134894615-948e098f-a168-4528-8685-5820cadd1c1d.png)

**Wireshark:** 
Using wireshark to analyze the packet and found the FTP credential of **nathan**

![image](https://user-images.githubusercontent.com/58801547/134895350-7640623b-3c85-4308-89be-903b821dc321.png)

**Optional:** using tshark to extract the information of packet by filtering

**Tshark:** `tshark -r 0.pcap -Y ftp`

![image](https://user-images.githubusercontent.com/58801547/134895531-20944d28-0636-4ccb-bddd-f05b40b801da.png)

## Exploitation & Privilege Escalation

Now I got the FTP credential but it should try login on other places too such as, **SSH** and then got the user :)

![image](https://user-images.githubusercontent.com/58801547/134896003-c93b0f0f-b39f-4860-b00e-495cfcad491f.png)

Look like **nathan** doesn't have any sudo permission so I use **linpeas** to enumuration more information so I can escalate to root

![image](https://user-images.githubusercontent.com/58801547/134896243-8603cc22-0f48-44fb-85bd-4cb5b2294553.png)

**Scp:** `scp filename user@remotehost:location`

Since I know the **ssh** user credential I just use **scp** to transfer file between **local** and **remote machine**

![image](https://user-images.githubusercontent.com/58801547/134896685-f356e4fa-9177-4a0c-a5dd-a6bc2f87ddb4.png)

**Linpeas:**

Now run the **linpeas.sh** then I got some information, it obvious this machine have capability of **cap_setuid** function and can access through **python** then I can escalate to root with just basic python script

![image](https://user-images.githubusercontent.com/58801547/134897124-42c308a6-ed11-4f34-9902-580a4b1d25e8.png)

Run `python3 -c 'import os; os.setuid(0); os.system("/bin/bash -i")'` and then get rooted 🐱‍👤 quiet easy right?

![image](https://user-images.githubusercontent.com/58801547/134898083-597037e8-020e-42ca-8956-ce3b1e1f7415.png)

***
