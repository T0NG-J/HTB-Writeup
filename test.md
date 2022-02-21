nmap result
```
# Nmap 7.91 scan initiated Tue Dec 14 01:31:36 2021 as: nmap -sSVC -p- -oA nmap/timing timing.htb
Nmap scan report for timing.htb (10.129.246.91)
Host is up (0.26s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 d2:5c:40:d7:c9:fe:ff:a8:83:c3:6e:cd:60:11:d2:eb (RSA)
|   256 18:c9:f7:b9:27:36:a1:16:59:23:35:84:34:31:b3:ad (ECDSA)
|_  256 a2:2d:ee:db:4e:bf:f9:3f:8b:d4:cf:b4:12:d8:20:f2 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
|_http-server-header: Apache/2.4.29 (Ubuntu)
| http-title: Simple WebApp
|_Requested resource was ./login.php
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Dec 14 02:16:16 2021 -- 1 IP address (1 host up) scanned in 2679.86 seconds

```

timingboi.php
```php
<?php
while(true) {
  $file_hash = uniqid();
  $file_name = "cat.jpg";

  $payload = md5('$file_hash' . time()) . '_' . $file_name;
  echo date("D M j G:i:s T Y");
  echo " = ";
  echo $payload;
  echo "\n";
  sleep(1);
}
?>
```
