# Reconnisance 
We start off with a full nmap port scan
```bash
$ nmap -sC -sV -p- atom.htb
```
The scan reveals a few open ports. The most interesting ones in our case are :
- 80/443  - Apache HTTPS Server
- 135/445 - SMB Server
- 6379    - Redis 
We also find out that the host operating system is Windows 10 Pro 19042

Let's see if we have access on any SMB shares:
```bash
$ smbclient -N -L atom.htb
```

Running this command reveals the following shares:
ADMIN$ - Remote Admin Disk
C$     - Default disk share
IPC$   - Remote IPC
Software_Updates

The first 3 shares are non-accessible, however we do have access to the Software_Updates share:
```bash
$ smbclient -N 10.10.10.237
smb > cd Software_Updates
smb > ls
```
We find a curious UAT_Testing_Procedures.pdf file.
... to be continued





