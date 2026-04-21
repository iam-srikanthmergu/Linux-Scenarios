## 1. Your server CPU suddenly spikes to 100%. How do you identify the root cause?

Answer:

First, I confirm whether the CPU is actually high and not just a temporary spike.

top

or

uptime

This shows load average. If it is consistently high, I check which process is consuming CPU:

ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu | head

I may also use:

htop

Explanation:

Identify the process first
Check if it's expected (application load) or abnormal

Real scenario:
In one case, a Python script was consuming 95% CPU due to a loop issue. I restarted the service temporarily and informed the development team.

## 2. A process is consuming high memory but you don’t know which one. How do you find and fix it?

Answer:

Check memory usage:

free -m

Find top memory-consuming processes:

ps aux --sort=-%mem | head

or

top

Check detailed memory usage:

pmap -x <pid>

Fix:

Restart process if safe
Analyze logs if critical

Real scenario:
A Java application had high memory usage due to incorrect heap settings. I increased heap size temporarily and later fixed configuration.

## 3. A production server becomes slow at random times. What steps will you take to troubleshoot?

Answer:

Check system performance:

top
vmstat 1 5
iostat -x 1 5

Check logs:

journalctl -xe

Check disk usage:

df -h

Check cron jobs:

crontab -l

Real scenario:
A nightly backup job was causing high disk I/O. Rescheduling it resolved the issue.

## 4. Disk space is 100% full on /. How will you handle it without downtime?

Answer:

Check disk usage:

df -h

Find large directories:

du -sh /* | sort -h

Find large files:

find / -type f -size +500M

Immediate fix:

truncate -s 0 /var/log/syslog

Real scenario:
Jenkins logs filled the root partition. Cleared logs and implemented log rotation.

## 5. A service is down but the server is running. How do you debug this?

Answer:

Check service status:

systemctl status nginx

Start service if needed:

systemctl start nginx

Check logs:

journalctl -u nginx

Check port:

ss -tulnp | grep 80

Check firewall:

iptables -L

Real scenario:
Nginx failed due to configuration error. Logs helped identify and fix it.
