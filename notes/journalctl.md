# journalctl

[ref](https://www.youtube.com/watch?v=0dG3vUYt7Uk)

## what is it?

- Each linux system has a number of processes that run in the bkgd.
- They output info into log files, which can be inspected.
- `journalctl` is part of `systemd`, and most distros use `system` to control processes.

```bash
shad@linux:~/linux/notes$ sudo systemctl start apache2
shad@linux:~/linux/notes$ systemctl status apache2
● apache2.service - Apache is awesome
     Loaded: loaded (/usr/lib/systemd/system/apache2.service; disabled; preset: enab>
    Drop-In: /etc/systemd/system/apache2.service.d
             └─override.conf
     Active: active (running) since Sun 2025-10-26 20:00:05 UTC; 3s ago
       Docs: https://httpd.apache.org/docs/2.4/
    Process: 13441 ExecStart=/usr/sbin/apachectl start (code=exited, status=0/SUCCES>
   Main PID: 13446 (apache2)
      Tasks: 55 (limit: 9447)
     Memory: 5.2M (peak: 5.6M)
        CPU: 29ms
     CGroup: /system.slice/apache2.service
             ├─13446 /usr/sbin/apache2 -k start
             ├─13448 /usr/sbin/apache2 -k start
             └─13449 /usr/sbin/apache2 -k start

Oct 26 20:00:05 linux systemd[1]: Starting apache2.service - Apache is awesome...
Oct 26 20:00:05 linux systemd[1]: Started apache2.service - Apache is awesome.

# to experiment, i acutally ran sudo systemctl edit apache2 and changed the desc.

shad@linux:~/linux/notes$ sudo systemctl edit apache2
Successfully installed edited file '/etc/systemd/system/apache2.service.d/override.conf'.
shad@linux:~/linux/notes$ sudo systemctl reload apache2
shad@linux:~/linux/notes$ systemctl status apache2
● apache2.service - Apache is chill
     Loaded: loaded (/usr/lib/systemd/system/apache2.service; disabled; preset: enab>
    Drop-In: /etc/systemd/system/apache2.service.d
             └─override.conf
     Active: active (running) since Sun 2025-10-26 20:00:05 UTC; 1min 36s ago
       Docs: https://httpd.apache.org/docs/2.4/
    Process: 13736 ExecReload=/usr/sbin/apachectl graceful (code=exited, status=0/SU>
   Main PID: 13446 (apache2)
      Tasks: 55 (limit: 9447)
     Memory: 5.2M (peak: 7.2M)
        CPU: 80ms
     CGroup: /system.slice/apache2.service
             ├─13446 /usr/sbin/apache2 -k start
             ├─13740 /usr/sbin/apache2 -k start
             └─13741 /usr/sbin/apache2 -k start

Oct 26 20:00:05 linux systemd[1]: Starting apache2.service - Apache is awesome...
Oct 26 20:00:05 linux systemd[1]: Started apache2.service - Apache is awesome.
Oct 26 20:01:36 linux systemd[1]: Reloading apache2.service - Apache is chill...
Oct 26 20:01:36 linux systemd[1]: Reloaded apache2.service - Apache is chill.
lines 1-20/20 (END)
```

Now we'll look at the logs for apache2 using `journalctl`.

```bash
shad@linux:~/linux/notes$ journalctl -u apache2
Oct 26 16:54:34 linux systemd[1]: Starting apache2.service - The Apache HTTP Server.>
Oct 26 16:54:34 linux systemd[1]: Started apache2.service - The Apache HTTP Server.
Oct 26 17:14:52 linux systemd[1]: Stopping apache2.service - The Apache HTTP Server.>
Oct 26 17:14:52 linux systemd[1]: apache2.service: Deactivated successfully.
Oct 26 17:14:52 linux systemd[1]: Stopped apache2.service - The Apache HTTP Server.
Oct 26 17:19:11 linux systemd[1]: Starting apache2.service - The Apache HTTP Server.>
Oct 26 17:19:12 linux systemd[1]: Started apache2.service - The Apache HTTP Server.
Oct 26 17:19:19 linux systemd[1]: Stopping apache2.service - The Apache HTTP Server.>
Oct 26 17:19:19 linux systemd[1]: apache2.service: Deactivated successfully.
Oct 26 17:19:19 linux systemd[1]: Stopped apache2.service - The Apache HTTP Server.
Oct 26 20:00:05 linux systemd[1]: Starting apache2.service - Apache is awesome...
Oct 26 20:00:05 linux systemd[1]: Started apache2.service - Apache is awesome.
Oct 26 20:01:36 linux systemd[1]: Reloading apache2.service - Apache is chill...
Oct 26 20:01:36 linux systemd[1]: Reloaded apache2.service - Apache is chill.
lines 1-14/14 (END)
```

How to find systemd units? Below is how I found out Ubuntu uses `ssh` not `sshd`.

```bash
shad@linux:~/linux/notes$ systemctl list-units --type=service | grep ssh
ssh.service                                                 loaded active running OpenBSD Secure Shell server

# loads real time, and also i attempted ssh from another box.
shad@linux:~/linux/notes$ journalctl -fu ssh

Oct 26 20:18:45 linux sshd[15079]: Accepted publickey for shad from 172.56.32.114 port 25089 ssh2: RSA SHA256:<redacted>
Oct 26 20:18:45 linux sshd[15079]: pam_unix(sshd:session): session opened for user shad(uid=1000) by shad(uid=0)
Oct 26 20:18:47 linux sshd[15081]: Connection closed by authenticating user root 59.120.55.242 port 45026 [preauth]
```

Can also filter too:

```bash
shad@linux:~/linux/notes$ journalctl --since "2025-10-26T20:18:00Z" -u ssh
Oct 26 20:18:45 linux sshd[15079]: Accepted publickey for shad from 172.56.32.114 port 25089 ssh2: RSA SHA256:<redacted>
Oct 26 20:18:45 linux sshd[15079]: pam_unix(sshd:session): session opened for user shad(uid=1000) by shad(uid=0)
Oct 26 20:18:47 linux sshd[15081]: Connection closed by authenticating user root 59.120.55.242 port 45026 [preauth]
Oct 26 20:20:03 linux sshd[15221]: Accepted publickey for shad from 50.186.34.129 port 9921 ssh2: RSA SHA256:<redacted>
Oct 26 20:20:03 linux sshd[15221]: pam_unix(sshd:session): session opened for user shad(uid=1000) by shad(uid=0)
Oct 26 20:20:34 linux sshd[15310]: Connection closed by authenticating user root 59.120.55.242 port 35080 [preauth]

# you can even do 'yesterday':
shad@linux:~/linux/notes$ journalctl --since "yesterday" -u ssh
```

btw, to confirm that ur logs are utc:

```bash
shad@linux:~/linux/notes$ timedatectl
               Local time: Sun 2025-10-26 20:30:35 UTC
           Universal time: Sun 2025-10-26 20:30:35 UTC
                 RTC time: Sun 2025-10-26 20:30:35
                Time zone: Etc/UTC (UTC, +0000)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

Could also do `journalctl --since "1 hour ago"`.

## viewing user logs

First we need to find the UID of the user we're curious about.

```bash

# typical for the first user account created on the system.
shad@linux:~/linux/notes$ id -u shad
1000

shad@linux:~/linux/notes$ journalctl _UID=1000
```

btw, i couldn't find "_UID" in the journalctl man pages, but instead they are systemd journal fields, which are metadata fields for the systemd event log entries. u can view those via `man systemd.journal-fields`.

## shrinking log size with "vacuum"

```bash
sudo journalctl --vacuum-size=500M
```


