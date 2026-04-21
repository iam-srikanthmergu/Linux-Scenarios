1. Your server CPU suddenly spikes to 100%. How do you identify the root cause?

Answer (human way):

First, I try to confirm whether the CPU is actually saturated and not just a temporary spike.

top

or

uptime

This gives me load average. If it’s consistently high, I dig deeper.

Then I identify which process is consuming CPU:

ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu | head

If needed, I also use:

htop

What I check next:

Is it a known application (Java, Python, Nginx)?
Is it expected traffic or abnormal?

Real-time explanation:
In one case, I saw a Python script consuming 95% CPU. It turned out to be a loop issue in the code. I temporarily restarted the service to stabilize the system and informed the dev team to fix the logic.

Key point to say in interview:
I always identify the process first before taking action instead of blindly restarting services.

2. A process is consuming high memory but you don’t know which one. How do you find and fix it?

Answer:

First, I check overall memory usage:

free -m

Then I identify top memory-consuming processes:

ps aux --sort=-%mem | head

or use:

top

If I find the process:

I check its details:
pmap -x <pid>

Fix approach:

If it’s safe → restart the process
If it’s critical → analyze logs before restart

Real scenario:
A Java application was consuming too much memory due to heap misconfiguration. I increased heap size temporarily and later fixed JVM settings.

Important line:
I try to understand whether it’s a memory leak or expected usage before restarting.

3. A production server becomes slow at random times. What steps will you take to troubleshoot?

Answer:

Since the issue is intermittent, I focus on capturing data when the issue happens.

Step 1: Check system performance

top
vmstat 1 5
iostat -x 1 5

Step 2: Check logs

journalctl -xe

Step 3: Check disk usage

df -h

Step 4: Check background jobs or cron

crontab -l

Real scenario:
We had a slowdown every night. After checking cron jobs, we found a backup script running at that time consuming disk I/O. We rescheduled it.

Key explanation:
Random slowness usually comes from background jobs, spikes in traffic, or resource contention.

4. Disk space is 100% full on /. How will you handle it without downtime?

Answer:

First, I confirm disk usage:

df -h

Then I find which directory is consuming space:

du -sh /* | sort -h

Then drill down further into large directories.

Find large files:

find / -type f -size +500M

Immediate fix:

Clear unnecessary logs:
truncate -s 0 /var/log/syslog
Remove old files or backups

Real scenario:
Jenkins logs once filled the root partition. I cleared logs immediately to bring the system back and then configured log rotation.

Important line:
I always take quick action to free space first, then implement a permanent fix like logrotate.

5. A service is down but the server is running. How do you debug this?

Answer:

First, I check service status:

systemctl status nginx

If it’s stopped:

systemctl start nginx

If it fails, I check logs:

journalctl -u nginx

Then I verify if the port is listening:

ss -tulnp | grep 80

Also check:

Configuration issues
Dependency services
Firewall rules
iptables -L

Real scenario:
Once Nginx failed because of a syntax error in config. Logs clearly showed the issue. After fixing config, service started.

Key explanation:
I follow a structured approach: service → logs → port → network.
