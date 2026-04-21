## 1. Your server CPU suddenly spikes to 100%. How do you identify the root cause?

**Answer:**

First, I confirm whether it’s a real CPU issue or just a temporary spike.

```bash
top
```

* Shows real-time CPU usage
* Look at `%us` (user), `%sy` (system), `%id` (idle)
* If idle is very low → CPU is actually busy

```bash
uptime
```

* Gives load average (1, 5, 15 mins)
* If load is higher than CPU cores → system is overloaded

Next, identify the top CPU-consuming processes:

```bash
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu | head
```

* Lists processes sorted by CPU usage
* Helps quickly identify the culprit

Optional (more interactive view):

```bash
htop
```

**What I analyze:**

* Is it application-related (Java, Python, Node)?
* Is it expected load (traffic spike) or abnormal?

**Real scenario:**
A Python script was consuming ~95% CPU due to an infinite loop. I restarted the service to stabilize the system and worked with the dev team to fix the logic.

**Key point:**
I always identify the exact process before taking action instead of blindly restarting services.

## 2. A process is consuming high memory but you don’t know which one. How do you find and fix it?

**Answer:**

First, check overall memory usage:

```bash
free -m
```

* Shows total, used, free, and available memory
* Important to focus on "available" memory, not just "used"

Then identify top memory-consuming processes:

```bash
ps aux --sort=-%mem | head
```

or

```bash
top
```

* Press `M` in top to sort by memory

To analyze a specific process:

```bash
pmap -x <pid>
```

* Shows memory breakdown (heap, stack, etc.)

**Fix approach:**

* If it's safe → restart the process
* If critical → check logs before restarting
* Long-term → fix memory leak or adjust configuration

**Real scenario:**
A Java application was consuming high memory due to incorrect heap size. I temporarily increased memory allocation and later tuned JVM settings.

**Key point:**
I try to understand whether it’s a memory leak or expected behavior before restarting.

## 3. A production server becomes slow at random times. What steps will you take to troubleshoot?

**Answer:**

Since the issue is intermittent, I try to capture system behavior during the slowdown.

Check CPU and system stats:

```bash
top
```

```bash
vmstat 1 5
```

* Shows CPU, memory, and process stats every second

```bash
iostat -x 1 5
```

* Helps identify disk I/O bottlenecks (high %util means disk is busy)

Check logs:

```bash
journalctl -xe
```

* Shows recent system errors

Check disk usage:

```bash
df -h
```

Check scheduled jobs:

```bash
crontab -l
```

**What I look for:**

* Any spikes in CPU, memory, or disk usage
* Background jobs running at that time
* Errors in logs

**Real scenario:**
Server slowdown was happening every night. After checking cron jobs, I found a backup job causing high disk I/O. Rescheduling it fixed the issue.

**Key point:**
Random slowness is usually due to scheduled tasks, traffic spikes, or resource contention.

## 4. Disk space is 100% full on /. How will you handle it without downtime?

**Answer:**

First, confirm disk usage:

```bash
df -h
```

* Shows which partition is full

Find large directories:

```bash
du -sh /* | sort -h
```

* Helps identify which top-level directory is consuming space

Drill down further into large directories if needed.

Find large files:

```bash
find / -type f -size +500M
```

* Quickly locates big files

Immediate fix (safe log cleanup):

```bash
truncate -s 0 /var/log/syslog
```

* Clears file content without deleting file

**Other quick actions:**

* Remove old backups
* Clean temporary files (`/tmp`)
* Clear unused Docker images (if applicable)

**Real scenario:**
Jenkins logs filled up the root partition. I cleared logs immediately and then implemented log rotation.

**Key point:**
First restore space quickly, then implement a permanent fix like logrotate.

## 5. A service is down but the server is running. How do you debug this?

**Answer:**

Start with checking service status:

```bash
systemctl status nginx
```

* Shows whether service is active, failed, or stopped
* Also shows recent error logs

If stopped, try starting:

```bash
systemctl start nginx
```

If it fails, check detailed logs:

```bash
journalctl -u nginx
```

Check if service is listening on expected port:

```bash
ss -tulnp | grep 80
```

* Confirms whether port is open and which process is using it

Check firewall rules:

```bash
iptables -L
```

**What I verify:**

* Service status
* Configuration errors
* Port binding
* Network/firewall issues

**Real scenario:**
Nginx failed due to a syntax error in configuration. Logs clearly showed the issue. After fixing config, service started successfully.

**Key point:**
I follow a structured approach: service → logs → port → network instead of random checks.
