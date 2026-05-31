```
Unlike in Windows, Linux logs most events into plain text files.
This means you can read the logs via any text editor without the need for specialized tools like Event Viewer.
On the other hand, default Linux logs are less structured as there are no event codes and strict log formatting rules. 
Use commands such as grep,tail,head to make your life easier
e.g $ cat /var/log/syslog | grep CRON
$ cat /var/log/auth.log | grep -E 'session opened|session closed'
$ cat /var/log/auth.log | grep "sshd" | grep -E 'Accepted|Failed'
$ cat /var/log/auth.log | grep -E '(passwd|useradd|usermod|userdel)\['
$ cat /var/log/auth.log | grep -E 'COMMAND='

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

(By default, commands are first stored in memory during your session, and then written to the per-user ~/.bash_history file when you log out.
You can open the ~/.bash_history file to review commands from previous sessions or use the history command to view commands from both your current and past sessions)
# Attackers can simply add a leading space to the command to avoid being logged
ubuntu@thm-vm:~$  echo "huehuehue"
# Attackers can paste their commands in a script to hide them from Bash history
ubuntu@thm-vm:~$ nano legit.sh && ./legit.sh
# Attackers can use other shells like /bin/sh that don't save the history like Bash
ubuntu@thm-vm:~$ sh
$ echo "I am no longer tracked by Bash!"

SSH keys — ~/.ssh/
systemd unit files — /etc/systemd/system/
crontab — /etc/crontab, /var/spool/cron/
tmp directories — /tmp, /var/tmp
user home dirs — /home/*
```

```

Linux doesn't log process creation, file changes, or network-related events, collectively known as runtime events. Interestingly, Windows faces the same limitation, which is why in Wibdows we had to use an additional tool: Sysmon.
However in Linux we have system calls which can assist. In short, whenever you need to open a file, create a process, access the camera, or request any other OS service, you make a specific system call. There are over 300 (https://man7.org/linux/man-pages/man2/syscalls.2.html)  system calls in Linux, like execve to execute a program.instructions located in /etc/audit/rules.d/ that define which system calls to monitor and which filters to apply:

<img width="1160" height="268" alt="image" src="https://github.com/user-attachments/assets/53467dbc-3904-4704-bc18-43d96b10f55d" />


Audit Daemon
Auditd (Audit Daemon) is a built-in auditing solution often used by the SOC team for runtime monitoring.
You can view the generated logs in real time in /var/log/audit/audit.log, but it is easier to use the ausearch command, as it formats the output for better readability and supports filtering options. Let's see an example based on the rules from the example above by searching events matching the "proc_wget" key:

root@thm-vm:~$ ausearch -i -k proc_wget
----
type=PROCTITLE msg=audit(08/12/25 12:48:19.093:2219) : proctitle=wget https://files.tryhackme.thm/report.zip
type=CWD msg=audit(08/12/25 12:48:19.093:2219) : cwd=/root
type=EXECVE msg=audit(08/12/25 12:48:19.093:2219) : argc=2 a0=wget a1=https://files.tryhackme.thm/report.zip
type=SYSCALL msg=audit(08/12/25 12:48:19.093:2219) : arch=x86_64 syscall=execve [...] ppid=3752 pid=3888 auid=ubuntu uid=root tty=pts1 exe=/usr/bin/wget key=proc_wget

The terminal above shows a log of a single "wget" command. Here, auditd splits the event into four lines: the PROCTITLE shows the process command line, CWD reports the current working directory, and the remaining two lines show the system call details, like:

pid=3888, ppid=3752: Process ID and Parent Process ID. Helpful in linking events and building a process tree
auid=ubuntu: Audit user. The account originally used to log in, whether locally (keyboard) or remotely (SSH)
uid=root: The user who ran the command. The field can differ from auid if you switched users with sudo or su
tty=pts1: Session identifier. Helps distinguish events when multiple people work on the same Linux server
exe=/usr/bin/wget: Absolute path to the executed binary, often used to build SOC detection rules
key=proc_wget: Optional tag specified by engineers in auditd rules that is useful to filter the events

```

<img width="1156" height="274" alt="image" src="https://github.com/user-attachments/assets/1837f75b-5433-4021-a457-7cd09f673f54" />
