# ssh

## [SSH for Beginners: The Ultimate Getting Started Guide](https://www.youtube.com/watch?v=YS5Zh7KExvE)

### What is OpenSSH?
- SSH has hops: Goes through modem (LAN) -> Internet -> business modem (LAN) -> switch -> server.
- modem, switch, server should all have port 22 open for SSH.
- server needs to have SSH server software installed and running, listening on port 22.

### Connecting to a server with OpenSSH.
- ssh client should be installed on your local machine (`sudo apt install openssh-client`).
- Command: `ssh username@server_ip_address`
- Fingerprint is added to `~/.ssh/known_hosts` on first connection. Prevents MITM attacks, as it'll see the fingerprint as changed, and gives a warning.

``` bash
shad@linux:~/linux/notes/networking$ journalctl -u ssh -f

Nov 21 02:25:29 linux sshd[4408]: Accepted publickey for shad from 167.220.148.2 port 3711 ssh2: RSA SHA256:eEV3VhwVc/Agj3t3x7zW9Gwm3fnI3G1KiZFcA/VuAbo
Nov 21 02:25:29 linux sshd[4408]: pam_unix(sshd:session): session opened for user shad(uid=1000) by shad(uid=0)
Nov 21 02:25:33 linux sshd[4408]: pam_unix(sshd:session): session closed for user shad
```

### Configuring the OpenSSH Client

``` bash
$ cat ~/.ssh/config
Host linux 20.121.121.163
  HostName 20.121.121.163
  User shad
  IdentityFile C:/Users/shad/.ssh/shad_key.pem

# Now we can do:
$ ssh linux
```

### Using public/private keys
- Generate Key Pair `ssh-keygen -t ed25519 -f ~/.ssh/server_key`. You can then copy the `server_key.pub` file to the server. Either by:
    - Copying the public key to the server and appending it to `~/.ssh/authorized_keys` on the server.
    - Using `ssh-copy-id -i ~/.ssh/server_key.pub username@server_ip_address` to copy the public key to the server.

### Managing SSH keys

- To store the passphrase for an SSH private key in memory, use the `ssh-agent`.
- Start the agent with `eval "$(ssh-agent -s)"`
- Add your private key to the agent with `ssh-add ~/.ssh/server_key` (private key file).


### SSH Server Configuration 

``` bash
shad@linux:~/linux/notes/networking$ which sshd
/usr/sbin/sshd
```

- sshd is the service that runs in the background to accept SSH connections.
- `systemctl restart/stop ssh` doesn't terminate existing connections!
- For Ubuntu, the sshd service is actually called `ssh`: `Unit sshd.service could not be found.`

``` bash
shad@linux:~/linux/notes/networking$ grep -R "Password" /etc/ssh/sshd_config.d/
grep: /etc/ssh/sshd_config.d/50-cloud-init.conf: Permission denied
/etc/ssh/sshd_config.d/60-cloudimg-settings.conf:PasswordAuthentication no
```

- `Permission denied (publickey).` means the server is not accepting password authentication, only public key authentication. And you do not have a valid public/private key pair set up.

### Troubleshooting

- Let's first check the network layer, which in itself is unrelated to OpenSSH.

``` bash
shad@linux:~/linux/notes/networking$ nc -vz 20.121.121.163 22
Connection to 20.121.121.163 22 port [tcp/ssh] succeeded!
```

- 22 needs to be allowed through any firewalls en route to the server.
- A timeout message indicates a network issue (firewall, port not open, server down, etc).

``` bash
shad@linux:~/linux/notes/networking$ ls -la ~/ | grep .ssh
drwx------  2 shad shad  4096 Oct 22 20:10 .ssh
```

- Don't change permissions on `~/.ssh/` (700), `~/.ssh/authorized_keys` (600), or the private key files (600) to be more permissive, or SSH will refuse to use them!

``` bash

# Check SSH daemon logs for troubleshooting
shad@linux:~/linux/notes/networking$ journalctl -u ssh -t sshd

# also can do 
shad@linux:~/linux/notes/networking$ tail -f /var/log/auth.log

# Check SSH service status
shad@linux:~/linux/notes/networking$ systemctl status ssh
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/usr/lib/systemd/system/ssh.service; disabled; preset: enabled)
     Active: active (running) since Fri 2025-11-21 01:12:43 UTC; 3h 15min ago
TriggeredBy: ● ssh.socket
       Docs: man:sshd(8)
             man:sshd_config(5)
    Process: 1123 ExecStartPre=/usr/sbin/sshd -t (code=exited, status=0/SUCCESS)
```

Log files:
- `/var/log/syslog`: General system log file. Use it for service errors, kernel messages, network issues, random app failures.
- `/var/log/auth.log`: Authentication log file. Use it for login attempts, sudo/su usage, SSH access, and other security-related events.
- `/var/log/kern.log`: Kernel log file. Use it for kernel messages, hardware issues, driver problems, OOM kills, filesystem problems, and low-level system events.
- `/var/log/dmesg`: Boot + kernel ring buffer. Use it for disk I/O problems, hardware-level networking issues. View with "sudo dmesg -T | tail".
- `/var/log/apt`: APT logs. Package installs, upgrades, failures.

Also, `less +G /var/log/auth.log` to go to the end of the file.