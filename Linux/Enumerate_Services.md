# Weak Services

Misconfigured services may contains a privileged service running with incorrect file permissions which could lead to privilage esclation by replacing the binary or abuse its depdencies to get system.

## Enumerate all services running

Execute the following command to see what process are running under root:

```
root@kali:~# ps aux | grep root
root         1  0.0  0.2  31464  7748 ?        Ss   06:08   0:06 /sbin/init
root         2  0.0  0.0      0     0 ?        S    06:08   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        I<   06:08   0:00 [rcu_gp]
root         5  0.0  0.0      0     0 ?        I<   06:08   0:00 [kworker/0:0H]
root         7  0.0  0.0      0     0 ?        I<   06:08   0:00 [mm_percpu_wq]
root         8  0.0  0.0      0     0 ?        S    06:08   0:00 [ksoftirqd/0]
root         9  0.0  0.0
........
root       274  0.0  0.0  57168   376 ?        Ssl  06:08   0:00 /home/bob/transfersecure.sh

```

## Enumerate Cronjobs

Look for a scheduled jobs (cron jobs):

```
root@kali:~#  crontab -l; ls -alh /var/spool/cron; ls -al /etc/ | grep cron; ls -al /etc/cron*; cat /etc/cron*; cat /etc/at.allow; cat /etc/at.deny; cat /etc/cron.allow; cat /etc/cron.deny

total 12K
drwxr-xr-x 3 root root    4.0K Aug 21  2018 .
drwxr-xr-x 8 root root    4.0K Aug 21  2018 ..
drwx-wx--T 2 root crontab 4.0K Mar 12  2018 crontabs
-rw-r--r--   1 root    root       401 May 29  2017 anacrontab
drwxr-xr-x   2 root    root      4096 Aug 21  2018 cron.d
drwxr-xr-x   2 root    root      4096 Aug 21  2018 cron.daily
drwxr-xr-x   2 root    root      4096 Aug 21  2018 cron.hourly
drwxr-xr-x   2 root    root      4096 Aug 21  2018 cron.monthly
-rw-r--r--   1 root    root      1042 Mar 12  2018 crontab
drwxr-xr-x   2 root    root      4096 Aug 21  2018 cron.weekly
-rw-r--r-- 1 root root 1042 Mar 12  2018 /etc/crontab

/etc/cron.d:
total 36
drwxr-xr-x   2 root root  4096 Aug 21  2018 .
drwxr-xr-x 202 root root 12288 Jan  8 05:35 ..
-rwxr-xr-x   1 root root   285 May 29  2017 transferfilestodmz
-rw-r--r--   1 root root   607 Mar  9  2016 john
.....

```
