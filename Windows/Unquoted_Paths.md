# Unquoted paths

If we find a service running with an unquoted path and spaces in the path we can hijack the path and use it to elevate privileges.

Find vulnerable services using the command:

```
wmic service get name,displayname,pathname,startmode |findstr /i "Auto" |findstr /i /v "C:\Windows\\" |findstr /i /v """
```

For example:

```
C:\Program Files\something\xampp.exe
```

We could place our payload with any of the following paths:

```
C:\Program.exe

C:\Program Files.exe

```

## Permissions / Acsses check

In some case, we might able to overwrite the binary itself. First we need to check out permissions:

```
icacls "C:\Program Files\something\xampp.exe"
```

If wmic is not available we can use sc instend:

```
sc qc xampp
```

If wmic and sc is not available, you can use accesschk. Download from here:
https://web.archive.org/web/20080530012252/http://live.sysinternals.com/accesschk.exe

```
accesschk.exe -uwcqv "Authenticated Users" * /accepteula
accesschk.exe -qdws "Authenticated Users" C:\Windows\ /accepteula
accesschk.exe -qdws Users C:\Windows\
```

We need to see permissions with (F) / (C) / (M) for our group or user.
