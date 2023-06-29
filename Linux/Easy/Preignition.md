# Preignition
 
- TL;DR: Provide default username and password to an admin portal at `/admin.php` found on a web server using `gobuster` tool.

## Reconnaissance
- Run an nmap scan:
```
nmap -sV -oA nmap_scans/Preignition 10.129.65.156

Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-28 06:46 PDT
Nmap scan report for 10.129.65.156
Host is up (0.078s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.14.2

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.47 seconds

```

<br>

## Exploitation

- We find the following port(s) open:
    - `80/tcp` http nginx 1,14,2
- We can confrim a http service running on port 80 by navigating to the traget's IP address in the URL bar of our web browser.
- The default post-installation page for the nginx service is displayed, meaning that there is the possibility that this web application might not be adequately configured yet, or that default credentials are used to facilitate faster configuration up to the point of live deployments.
- Since there are no links or buttons on the web page we use a tool called `gobuster` for dir busting.
    - You can install `Go` if its not already installed by running `sudo apt install golang-go` because `Go` is needed to run `gobuster` tool.
    - Once `Go` is installed you can run `go install github.com/OJ/gobuster/v3@latest`. This will install `gobuster`.
    - In case the above command fails for any reason, you can always compile the tool from its' source code by running the following commands:
    ```
    sudo git clone https://github.com/OJ/gobuster.git
    cd gobuster
    go get && go build
    go install
    ```
- We now run `sudo gobuster dir -w /usr/share/wordlists/dirb/common.txt -u 10.129.65.156`
    - We get the following output:
    ```
    ===============================================================
    Gobuster v3.5
    by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
    ===============================================================
    [+] Url:                     http://10.129.65.156
    [+] Method:                  GET
    [+] Threads:                 10
    [+] Wordlist:                /usr/share/wordlists/dirb/common.txt
    [+] Negative Status codes:   404
    [+] User Agent:              gobuster/3.5
    [+] Timeout:                 10s
    ===============================================================
    2023/06/28 07:14:39 Starting gobuster in directory enumeration mode
    ===============================================================
    /admin.php            (Status: 200) [Size: 999]
    Progress: 4591 / 4615 (99.48%)
    ===============================================================
    2023/06/28 07:15:22 Finished
    ===============================================================
    ```
- We find an we page for `/admin.php`
- When we visit the page in our web browser we are presented with an admin login console.
- We would usually try to brute force this with one of the brute force tools. However in this this case we will manually just try default passwords `admin:admin` which successfully logs us in as and admin.

