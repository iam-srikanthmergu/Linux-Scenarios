# Linux DevOps Interview Questions

---

## 1. Production & Troubleshooting

1. Your server CPU suddenly spikes to 100%. How do you identify the root cause?
2. A process is consuming high memory but you don’t know which one. How do you find and fix it?
3. A production server becomes slow at random times. What steps will you take to troubleshoot?
4. Disk space is 100% full on `/`. How will you handle it without downtime?
5. A service is down but the server is running. How do you debug this?

---

## 2. Processes & System Behavior

6. A process is stuck and not getting killed even with `kill -9`. What will you do?
7. How do you find which process is using a specific port (e.g., 8080)?
8. You accidentally started a process in the foreground. How do you move it to background without stopping it?
9. What happens internally when you run a command in Linux?
10. What is the difference between zombie and orphan processes? How do you handle them in real-time?

---

## 3. File System & Storage

11. How do you find large files in a server quickly?
12. What will you do if inode usage is 100% but disk space is available?
13. Explain a real-time scenario where symbolic links are useful.
14. How do you recover a deleted file in Linux?
15. What is the difference between soft link and hard link with a real use case?

---

## 4. Networking

16. A server is not reachable. How do you troubleshoot step by step?
17. How do you check if a port is open or not on a remote server?
18. DNS is not resolving on your server. What will you check?
19. What is the difference between `netstat`, `ss`, and `lsof` in real-time usage?
20. Your application cannot connect to a database. How will you debug it?

---

## 5. Permissions & Security

21. A user cannot access a file even though permissions look correct. What could be wrong?
22. What is the difference between `chmod 777` and `chmod 755` in real-world impact?
23. How do you give sudo access to a user safely?
24. What are sticky bit, SUID, and SGID? Explain with real-time usage.
25. A script runs manually but fails in cron. Why?

---

## 6. Logs & Monitoring

26. Where do you check logs for system issues?
27. How do you monitor logs in real-time for errors?
28. Your application logs are growing rapidly. How do you manage them?
29. What is log rotation and why is it important?
30. How do you debug a failed systemd service?

---

## 7. Package & System Management

31. How do you check installed packages in Linux?
32. A package installation failed due to dependencies. How do you resolve it?
33. What is the difference between `apt` and `yum` in real-world usage?
34. How do you update a system without breaking running services?
35. How do you check OS version and kernel details?

---

## 8. Cron & Automation

36. A cron job is not running. How do you debug it?
37. How do you schedule a job to run every 5 minutes?
38. What is the difference between `cron` and `at`?
39. Where are cron logs stored?
40. How do you prevent overlapping cron jobs?

---

## 9. DevOps-Oriented Linux

41. How does Linux support containers like Docker?
42. What are namespaces and cgroups in Linux?
43. How do you debug a Docker container that is not starting?
44. How do you check resource usage of containers from Linux?
45. How does Linux handle networking for containers?

---

## 10. Advanced / Real Interview Level

46. What happens when a system runs out of memory (OOM)?
47. What is OOM killer and how do you debug it?
48. How do you tune Linux performance for high traffic applications?
49. What is load average and how do you interpret it?
50. How do you troubleshoot high I/O wait?

---
