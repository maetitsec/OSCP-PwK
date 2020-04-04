# SMB Enumeration

Tip to remember: If you have used nmap (-sV) against newer Samba servers, nmap will report the version (>3.x). Conversely, if nmap doesn't report the version, then you are looking at a Samba (<= 2.2.x).)

## External

### SMBMapping

First, use the following command to map any exposed SMB shares:

```
root@kali:/# smbmap -H 192.168.11.10
[+] Finding open SMB ports....
[+] User SMB session establishd on [ip]...
[+] IP: [ip]:445        Name: [ip]                                      
        Disk                                                    Permissions
        ----                                                    -----------
        ADMIN$                                                  NO ACCESS
        C$                                                      NO ACCESS
        IPC$                                                    NO ACCESS
        Replication                                             READ ONLY
        SYSVOL                                                  NO ACCESS
....
```

Or using the following Nmap script:

```
root@kali:~# nmap --script smb-enum-shares -p 139,445 192.168.11.10
Starting Nmap 7.70 ( https://nmap.org ) at 2019-12-27 16:25 EDT
Nmap scan report for [ip]
Host is up (0.037s latency).

PORT    STATE SERVICE
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
MAC Address: 00:50:56:XX:XX:XX (VMware)

Host script results:
| smb-enum-shares:
|   account_used: guest
|   \\[ip]\ADMIN$:
|     Type: STYPE_DISKTREE_HIDDEN
|     Comment: Remote Admin
|     Anonymous access: <none>
|     Current user access: <none>
|   \\[ip]\C$:
|     Type: STYPE_DISKTREE_HIDDEN
|     Comment: Default share
|     Anonymous access: <none>
|     Current user access: <none>
....
``

### NUll Session


Use smbclient utility to connect share with null session:

```
root@kali:~# smbclient -L \\192.168.11.10
Enter WORKGROUP\root's password:

        Sharename       Type      Comment
        ---------       ----      -------
        IPC$            IPC       Remote IPC
        Backups         Disk
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
....
``

For example, if we found Backups share folder which allow null session we can just use the following syntax for smbclient:

```
smbclient \\\\[ip]\\[share name]
```

```
root@kali:~# smbclient \\\\192.168.11.10\\Backups
Enter WORKGROUP\root's password:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Thu Sep 27 16:26:00 2019
  ..                                  D        0  Thu Sep 27 16:26:00 2019
  FiftyShadesOfG                      D        0  Sun Dec 13 05:26:59 2019
  Accounting                          D        0  Sun Dec 13 06:55:42 2019
  Shortcut.lnk                        A      420  Sun Dec 13 05:24:51 2019

                1690825 blocks of size 2048. 794699 blocks available
```

### Automate All Those scans

Use enum4linux script (built in inside Kali) to prefrom an overall scan of those checks:

```
root@kali:~# enum4linux -a 192.168.11.10
```

### Scan for vulnerabilities

Using NMAP we could try to locate diffrent exploits for SMB versions:

```
root@kali:~# nmap --script smb-vuln* -p 139,445 192.168.11.10
Starting Nmap 7.70 ( https://nmap.org ) at 2019-12-07 16:37 EDT
Nmap scan report for [ip]
Host is up (0.030s latency).

PORT    STATE SERVICE
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
MAC Address: 00:50:56:XX:XX:XX (VMware)

Host script results:
| smb-vuln-ms06-025:
|   VULNERABLE:
|   RRAS Memory Corruption vulnerability (MS06-025)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2006-2370
|           A buffer overflow vulnerability in the Routing and Remote Access service (RRAS) in Microsoft Windows 2000 SP4, XP SP1
|           and SP2, and Server 2003 SP1 and earlier allows remote unauthenticated or authenticated attackers to
|           execute arbitrary code via certain crafted "RPC related requests" aka the "RRAS Memory Corruption Vulnerability."
|
|     Disclosure date: 2006-6-27
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2006-2370
|_      https://technet.microsoft.com/en-us/library/security/ms06-025.aspx
|_smb-vuln-ms10-054: false
|_smb-vuln-ms10-061: false
......
```

### Scan for SMB version

In newer smbclient version, the support of older protocls has been deprected so in some tools you may not getting the SMB version. A member of PWK forums publish the following script to help with that:

```
#!/bin/sh
#Author: rewardone
#Description:
# Requires root or enough permissions to use tcpdump
# Will listen for the first 7 packets of a null login
# and grab the SMB Version
#Notes:
# Will sometimes not capture or will print multiple
# lines. May need to run a second time for success.
if [ -z $1 ]; then echo "Usage: ./smbver.sh RHOST {RPORT}" && exit; else rhost=$1; fi
if [ ! -z $2 ]; then rport=$2; else rport=139; fi
tcpdump -s0 -n -i tap0 src $rhost and port $rport -A -c 7 2>/dev/null | grep -i "samba\|s.a.m" | tr -d '.' | grep -oP 'UnixSamba.*[0-9a-z]' | tr -d '\n' & echo -n "$rhost: " &
echo "exit" | smbclient -L $rhost 1>/dev/null 2>/dev/null
sleep 0.5 && echo ""
```

When you're running this you will see the following output:

```
root@kali:~/pwk/lab/public# ./smbver.sh 192.168.11.10
192.168.11.10: UnixSamba 227a
```

*TIP*: A worth check exploit for SMB 2.2.x version could be found here: https://www.exploit-db.com/exploits/7
