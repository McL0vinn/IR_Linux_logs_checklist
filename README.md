```
Unlike in Windows, Linux logs most events into plain text files.
This means you can read the logs via any text editor without the need for specialized tools like Event Viewer.
On the other hand, default Linux logs are less structured as there are no event codes and strict log formatting rules. 
Use commands such as grep,tail,head to make your life easier
e.g $ cat /var/log/syslog | grep CRON
$ cat /var/log/auth.log | grep -E 'session opened|session closed'
$ cat /var/log/auth.log | grep "sshd" | grep -E 'Accepted|Failed'

let's say you hunt for all user logins, but don't know where to look for them.
Linux system logs are stored in the /var/log/ folder in plain text, so you can simply
grep for related keywords like "login", "auth", or "session" in all log files there and narrow down your next searches:
$ grep -R -E "auth|login|session" /var/log


1) Core Security‑Relevant Logs (must check first)

auth.log — /var/log/auth.log  
SSH logins, sudo usage, privilege escalation attempts, PAM failures.

secure — /var/log/secure (RHEL/CentOS equivalent of auth.log)
Same content as above on RPM‑based systems.

syslog — /var/log/syslog  
System‑wide events, service starts/stops, suspicious daemon behavior.

messages — /var/log/messages (RHEL/CentOS equivalent of syslog)
Kernel, services, hardware, boot, general system activity.

2) Authentication, Accounts & Privilege

faillog — /var/log/faillog  
Failed login attempts (binary file; use faillog -a).

lastlog — /var/log/lastlog  
Last login per user (binary; use lastlog).

wtmp — /var/log/wtmp  
All logins/logouts (last command).

btmp — /var/log/btmp  
Failed login attempts (lastb command).

sudo logs — /var/log/auth.log or /var/log/secure  
Sudo invocations, failures, escalation attempts.

3) Process, Services & Persistence

daemon.log — /var/log/daemon.log  
Background services, suspicious daemons, malware persistence.

systemd journal — /var/log/journal/ + journalctl  
Full service logs, including transient malware‑spawned units.

cron logs — /var/log/cron or inside syslog
Malicious scheduled tasks, reverse shells, persistence.

boot.log — /var/log/boot.log  
Boot‑time anomalies, injected services.

4) Network & Firewall

kern.log — /var/log/kern.log  
Kernel‑level networking, dropped packets, suspicious modules.

ufw.log — /var/log/ufw.log  
Firewall blocks, brute‑force attempts.

iptables logs — /var/log/messages or /var/log/kern.log  
Dropped packets, scanning, C2 traffic.

dmesg — kernel ring buffer
Rootkits, module loading, hardware manipulation.

5) Package, Updates & Integrity

dpkg.log — /var/log/dpkg.log  
Installed/removed packages (malicious installs).

apt logs — /var/log/apt/history.log  
Update/install history.

yum.log — /var/log/yum.log  
RPM‑based package operations.

6) Application‑Specific Logs

Apache logs — /var/log/apache2/  
Access logs, exploitation attempts, webshell indicators.

Nginx logs — /var/log/nginx/  
Reverse proxy abuse, SSRF, scanning.

MySQL logs — /var/log/mysql/  
Unauthorized DB access.

Docker logs — /var/lib/docker/containers/<id>/<id>-json.log  
Container breakout attempts, malicious containers.

7) High‑Value IR Artifacts (not logs but mandatory)

bash history — ~/.bash_history
SSH keys — ~/.ssh/
systemd unit files — /etc/systemd/system/
crontab — /etc/crontab, /var/spool/cron/

```

tmp directories — /tmp, /var/tmp

user home dirs — /home/*
