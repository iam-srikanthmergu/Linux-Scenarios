## 1. Your server CPU suddenly spikes to 100%. How do you identify the root cause?

**Answer:**

First, I confirm whether the CPU is actually high and not just a temporary spike.

```bash
top
```

```bash
uptime
```

This shows load average. If it is consistently high, I check which process is consuming CPU:

```bash
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu | head
```

I may also use:

```bash
htop
```

**Explanation:**

* Identify the process first
* Check if it's expected or abnormal

**Real scenario:**
In one case, a Python script was consuming 95% CPU due to a loop issue. I restarted the service temporarily and informed the development team.

## 2. A process is consuming high memory but you don’t know which one. How do you find and fix it?

**Answer:**

Check memory usage:

```bash
free -m
```

Find top memory-consuming processes:

```bash
ps aux --sort=-%mem | head
```

```bash
top
```

Check detailed memory usage:

```bash
pmap -x <pid>
```

**Fix:**

* Restart process if safe
* Analyze logs if critical

**Real scenario:**
A Java application had high memory usage due to incorrect heap settings.

## 3. A production server becomes slow at random times. What steps will you take to troubleshoot?

**Answer:**

Check system performance:

```bash
top
vmstat 1 5
iostat -x 1 5
```

Check logs:

```bash
journalctl -xe
```

Check disk usage:

```bash
df -h
```

Check cron jobs:

```bash
crontab -l
```

**Real scenario:**
A nightly backup job was causing high disk I/O. Rescheduling it resolved the issue.

## 4. Disk space is 100% full on /. How will you handle it without downtime?

**Answer:**

Check disk usage:

```bash
df -h
```

Find large directories:

```bash
du -sh /* | sort -h
```

Find large files:

```bash
find / -type f -size +500M
```

Immediate fix:

```bash
truncate -s 0 /var/log/syslog
```

**Real scenario:**
Jenkins logs filled the root partition. Cleared logs and implemented log rotation.

## 5. A service is down but the server is running. How do you debug this?

**Answer:**

Check service status:

```bash
systemctl status nginx
```

Start service if needed:

```bash
systemctl start nginx
```

Check logs:

```bash
journalctl -u nginx
```

Check port:

```bash
ss -tulnp | grep 80
```

Check firewall:

```bash
iptables -L
```

**Real scenario:**
Nginx failed due to configuration error. Logs helped identify and fix it.
