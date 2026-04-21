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

## 6. A process is stuck and not getting killed even with kill -9. What will you do?

**Answer:**

First, I try normal termination:

```bash
kill -15 <pid>
```

* Sends SIGTERM (graceful shutdown)

If that doesn’t work:

```bash
kill -9 <pid>
```

* Forces termination (SIGKILL)

If the process is still not killed, I check its state:

```bash
ps -o pid,stat,cmd -p <pid>
```

* If state shows `D` → uninterruptible sleep (usually waiting on disk I/O)

**What it means:**

* Process is stuck in kernel-level operation (disk, NFS, etc.)
* Even kill -9 cannot terminate it

**Next steps:**

* Check disk/NFS issues
* As last resort → reboot the server

**Real scenario:**
A process was stuck due to NFS mount issue. It stayed in D state and only got cleared after fixing mount and reboot.

**Key point:**
If a process is in D state, killing won’t help — need to fix underlying I/O issue.

## 7. How do you find which process is using a specific port (e.g., 8080)?

**Answer:**

Use:

```bash
lsof -i :8080
```

* Shows process using the port with PID

Alternative:

```bash
ss -tulnp | grep 8080
```

* Faster and more modern than netstat
* Shows listening ports and associated processes

**Explanation:**

* Helps identify which service is bound to a port
* Useful during deployment conflicts

**Real scenario:**
While deploying a new app, port 8080 was already in use by an old process. I identified and killed it before restarting the new service.

**Key point:**
Always check port usage before starting services to avoid conflicts.

## 8. You accidentally started a process in the foreground. How do you move it to background without stopping it?

**Answer:**

First, suspend the process:

Press:

```
Ctrl + Z
```

Then move it to background:

```bash
bg
```

To detach it completely:

```bash
disown
```

**Alternative (better approach):**

Start process with:

```bash
nohup <command> &
```

* Runs process in background even after logout

**Explanation:**

* Foreground → blocks terminal
* Background → runs independently

**Real scenario:**
While running a long script, I forgot to use nohup. I moved it to background using Ctrl+Z and bg without stopping execution.

**Key point:**
Knowing job control avoids restarting long-running tasks.

## 9. What happens internally when you run a command in Linux?

**Answer:**

When a command is executed:

1. Shell receives the command (bash)
2. Shell searches command in PATH
3. Forks a new process
4. Executes command using `exec`
5. Waits for process to complete

**You can verify PATH:**

```bash
echo $PATH
```

**Check command location:**

```bash
which ls
```

**Explanation:**

* `fork()` creates a child process
* `exec()` replaces it with actual command

**Real understanding:**
Every command you run is a process created by the shell.

**Key point:**
Understanding this helps in debugging process-related issues and scripting.

## 10. Difference between zombie and orphan processes? How do you handle them in real-time?

**Answer:**

**Zombie Process:**

* Process finished execution but still in process table
* Parent has not read exit status

Check zombies:

```bash
ps aux | grep Z
```

**Orphan Process:**

* Parent process is terminated
* Child is adopted by init/systemd

**Handling zombie:**

* Restart parent process
* Or kill parent process so init takes over

**Handling orphan:**

* Usually no issue (handled by system)

**Real scenario:**
An application was creating zombie processes due to improper child handling. Restarting parent process cleared them.

**Key point:**
Zombies don’t consume CPU but can exhaust process table if not handled.

## 11. How do you find large files in a server quickly?

**Answer:**

Use:

```bash
find / -type f -size +500M
```

* Finds files larger than 500MB

Sort directories by size:

```bash
du -sh /* | sort -h
```

* Helps identify which directories are consuming space

**Explanation:**

* `find` → file-level search
* `du` → directory-level analysis

**Real scenario:**
A backup file accidentally stored in `/tmp` was consuming several GB. Identified and removed using find.

**Key point:**
Quick identification helps prevent downtime due to disk issues.

## 12. What will you do if inode usage is 100% but disk space is available?

**Answer:**

Check inode usage:

```bash
df -i
```

**Explanation:**

* Inodes track number of files, not size
* Too many small files → inode exhaustion

Find directories with many files:

```bash
find / -xdev -type f | wc -l
```

or:

```bash
du --inodes -d 2 / | sort -n
```

**Fix:**

* Remove unnecessary small files
* Clean logs or temp directories

**Real scenario:**
Application logs created millions of tiny files. Disk had space but inode limit was hit. Cleaning files resolved it.

**Key point:**
“No space left” is not always disk — sometimes inode limit.

## 13. Explain a real-time scenario where symbolic links are useful.

**Answer:**

Create symbolic link:

```bash
ln -s /var/log/app.log /home/user/app.log
```

**Explanation:**

* Soft link points to original file path
* Useful for shortcuts and versioning

**Real scenario:**
In deployments, we used symbolic links like:

```
current -> release_v2
```

Switching versions was as simple as updating the symlink.

**Key point:**
Symbolic links are widely used in deployments and configuration management.

## 14. How do you recover a deleted file in Linux?

**Answer:**

If file is deleted normally:

* Recovery is difficult unless backup exists

**Check open file still in use:**

```bash
lsof | grep deleted
```

* If process still holds file → data can be recovered

**Best practice:**

* Use backups (rsync, snapshots)
* Use version control where possible

**Real scenario:**
A log file was deleted but still open by a process. We copied data from `/proc/<pid>/fd/` and recovered it.

**Key point:**
Prevention (backups) is better than recovery.

## 15. Difference between soft link and hard link with real use case?

**Answer:**

**Soft Link:**

```bash
ln -s file1 softlink
```

* Points to file path
* Breaks if original file is deleted

**Hard Link:**

```bash
ln file1 hardlink
```

* Points to same inode
* Works even if original file is deleted

**Key differences:**

* Soft link → flexible, cross-filesystem
* Hard link → more robust, same filesystem only

**Real scenario:**
Soft links used for application version switching. Hard links used for backup optimization.

**Key point:**
Soft links are commonly used in DevOps deployments.

## 16. A server is not reachable. How do you troubleshoot step by step?

**Answer:**

Start with basic network connectivity:

```bash
ping <server-ip>
```

* Checks if the server is reachable over the network

If ping fails, check route:

```bash
traceroute <server-ip>
```

* Shows where the connection is failing in the network path

Check if SSH port is open:

```bash
nc -zv <server-ip> 22
```

or

```bash
telnet <server-ip> 22
```

If you have access via console/cloud:

Check network configuration:

```bash
ip a
```

Check routing:

```bash
ip route
```

Check firewall:

```bash
iptables -L
```

**Real scenario:**
A server was unreachable due to a wrong security group rule in cloud. Port 22 was blocked. After updating rules, access was restored.

**Key point:**
Always follow layered troubleshooting: network → port → firewall → server config.

## 17. How do you check if a port is open or not on a remote server?

**Answer:**

Use netcat:

```bash
nc -zv <server-ip> 80
```

* `z` → scan mode
* `v` → verbose output

Alternative:

```bash
telnet <server-ip> 80
```

If connected → port is open
If connection refused → port closed or service down

Another method:

```bash
nmap -p 80 <server-ip>
```

* Scans specific port

**Real scenario:**
Application was not accessible because port 8080 was not exposed. Verified using nc and fixed firewall rules.

**Key point:**
Always confirm port availability before debugging application.

## 18. DNS is not resolving in your server. What will you check?

**Answer:**

Check DNS resolution:

```bash
nslookup google.com
```

or

```bash
dig google.com
```

Check DNS configuration:

```bash
cat /etc/resolv.conf
```

* Shows configured DNS servers

Test connectivity to DNS server:

```bash
ping <dns-server-ip>
```

Check local hostname resolution:

```bash
cat /etc/hosts
```

**Real scenario:**
DNS was not resolving because `/etc/resolv.conf` had wrong nameserver. Updating it fixed the issue.

**Key point:**
DNS issues are often configuration-related, not network.

## 19. Difference between netstat, ss, and lsof in real-time usage?

**Answer:**

**netstat:**

```bash
netstat -tulnp
```

* Older tool
* Shows listening ports and connections

**ss:**

```bash
ss -tulnp
```

* Faster and modern replacement for netstat
* Preferred in most systems

**lsof:**

```bash
lsof -i :80
```

* Shows which process is using a port

**Explanation:**

* `ss` → best for checking ports and connections
* `lsof` → best for mapping ports to processes

**Real scenario:**
Used `ss` to check port availability and `lsof` to identify which process was holding the port.

**Key point:**
Use ss for speed, lsof for detailed process mapping.

## 20. Your application cannot connect to DB. How will you debug?

**Answer:**

Step 1: Check DB connectivity

```bash
ping <db-server-ip>
```

Step 2: Check port accessibility

```bash
nc -zv <db-server-ip> 3306
```

Step 3: Verify DB service on server

```bash
systemctl status mysql
```

Step 4: Check application configuration

* DB host
* Port
* Username/password

Step 5: Check logs

```bash
journalctl -u <app-service>
```

or application logs

**Real scenario:**
Application failed to connect because DB port was blocked in firewall. After opening port 3306, connection worked.

**Key point:**
Always verify network → port → service → credentials in order.

**Key point:**
I follow a structured approach: service → logs → port → network instead of random checks.
