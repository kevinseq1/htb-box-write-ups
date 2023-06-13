# Meow

- TL;DR: The `telnet` service allows root login without a password.

## Reconnaissance

- Run an nmap scan:
```
nmap -sC -sV -O -oA nmap_scans/Meow 10.129.50.8
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-12 20:40 PDT
Nmap scan report for 10.129.50.8
Host is up (0.077s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
23/tcp open  telnet  Linux telnetd
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.93%E=4%D=6/12%OT=23%CT=1%CU=41009%PV=Y%DS=2%DC=I%G=Y%TM=6487E54
OS:7%P=aarch64-unknown-linux-gnu)SEQ(SP=103%GCD=1%ISR=10A%TI=Z%CI=Z%II=I%TS
OS:=A)SEQ(SP=103%GCD=1%ISR=10A%TI=Z%CI=Z%TS=A)OPS(O1=M539ST11NW7%O2=M539ST1
OS:1NW7%O3=M539NNT11NW7%O4=M539ST11NW7%O5=M539ST11NW7%O6=M539ST11)WIN(W1=FE
OS:88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN(R=Y%DF=Y%T=40%W=FAF0%O=M5
OS:39NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4
OS:(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%
OS:F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%
OS:T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%R
OS:ID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 42.74 seconds
```
<br>

## Exploitation

- We find the following port open:
	- `tcp port 23`: Running the telnet service
- Telnet allows a user to run commands on a remote host (Sometimes without a password)
- We try the following common user names:
	- `admin`
	- `administrator`
	- `root`
- Entering the username as `root` with a blank password allows us to login as the root user on the remote machine.



