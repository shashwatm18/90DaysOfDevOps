# Day 05 – Linux Troubleshooting Drill: CPU, Memory, and Logs

## Goal

Practice a **repeatable Linux troubleshooting routine** for investigating a running service.

You will check:

* System information
* Filesystem
* CPU and memory
* Disk usage and I/O
* Network/ports
* Service logs

> **Important:** Run these commands on your own Linux machine and paste the actual outputs. Don't copy example outputs.

---

# 1. Choose a Target Service

Pick **one running service** and use it throughout the exercise.

Examples:

* `ssh`
* `cron`
* `docker`
* `nginx`

For this practice, you can use:

```bash
docker
```

Check whether it is running:

```bash
systemctl status docker
```

Find its process:

```bash
ps aux | grep dockerd
```

Record the PID of `dockerd`.

```text
Target Service: docker
PID:
Status:
```

---

# 2. Environment Basics

## Command 1 – Kernel and System Information

```bash
uname -a
```

### What to look for

Shows information about the Linux kernel, architecture, and hostname.

### My Output

```text
Paste your output here.
```

### Observation

```text
Example: System is running a 64-bit Linux kernel.
```

---

## Command 2 – Linux Distribution

```bash
lsb_release -a
```

If unavailable:

```bash
cat /etc/os-release
```

### What to look for

Identifies the Linux distribution and version.

### My Output

```text
Paste your output here.
```

### Observation

```text
Example: Machine is running Ubuntu 24.04 LTS.
```

---

# 3. Filesystem Sanity Check

Before troubleshooting deeper issues, confirm basic filesystem operations work.

## Command 3 – Create Temporary Directory

```bash
mkdir /tmp/runbook-demo
```

Verify:

```bash
ls -ld /tmp/runbook-demo
```

### My Output

```text
Paste your output here.
```

### Observation

```text
Example: Directory was created successfully, so basic filesystem writes are working.
```

---

## Command 4 – Copy and Read a File

```bash
cp /etc/hosts /tmp/runbook-demo/hosts-copy
ls -l /tmp/runbook-demo
```

Optional:

```bash
cat /tmp/runbook-demo/hosts-copy
```

### My Output

```text
Paste your output here.
```

### Observation

```text
Example: File copied successfully and permissions look normal.
```

---

# 4. CPU and Memory Snapshot

## Command 5 – Check Docker CPU and Memory

First get the PID:

```bash
pgrep dockerd
```

Then:

```bash
ps -o pid,pcpu,pmem,comm -p <PID>
```

Example:

```bash
ps -o pid,pcpu,pmem,comm -p 821
```

The columns mean:

| Column    | Meaning                |
| --------- | ---------------------- |
| `PID`     | Process ID             |
| `%CPU`    | CPU utilization        |
| `%MEM`    | Percentage of RAM used |
| `COMMAND` | Process name           |

### My Output

```text
Paste your output here.
```

### Observation

```text
Example: dockerd is using very little CPU and memory, so there is no obvious resource problem.
```

For live monitoring, you can also run:

```bash
top -p <PID>
```

Press `q` to exit.

---

## Command 6 – Check System Memory

```bash
free -h
```

Focus on:

```text
total
used
free
available
swap
```

### Important

Don't judge memory pressure only from the `free` column.

Linux intentionally uses unused RAM for filesystem cache. The **available** column is generally more useful when checking how much memory applications can still use.

### My Output

```text
Paste your output here.
```

### Observation

```text
Example: Available memory is healthy and swap usage is low, so the system does not appear memory constrained.
```

---

# 5. Disk and I/O Snapshot

## Command 7 – Check Filesystem Usage

```bash
df -h
```

Focus on:

```text
Filesystem
Size
Used
Avail
Use%
Mounted on
```

Pay special attention to `/`.

### My Output

```text
Paste your output here.
```

### Observation

```text
Example: Root filesystem is 55% utilized, so disk capacity is currently healthy.
```

A filesystem approaching **90–100%** usage deserves investigation.

---

## Command 8 – Check Log Directory Size

```bash
sudo du -sh /var/log
```

### My Output

```text
Paste your output here.
```

### Observation

```text
Example: /var/log consumes 700 MB and does not appear unusually large.
```

If `/var/log` is unexpectedly large:

```bash
sudo du -h /var/log | sort -h | tail
```

This helps identify large log directories/files.

---

## Command 9 – Check System Activity

```bash
vmstat 1 5
```

This collects five samples at one-second intervals.

Useful fields:

| Field | Meaning                   |
| ----- | ------------------------- |
| `r`   | Processes waiting for CPU |
| `si`  | Swap coming into RAM      |
| `so`  | Swap going out of RAM     |
| `bi`  | Blocks read from disk     |
| `bo`  | Blocks written to disk    |
| `id`  | CPU idle %                |
| `wa`  | CPU waiting for I/O %     |

### My Output

```text
Paste your output here.
```

### Observation

```text
Example: CPU idle is high, swap activity is zero, and I/O wait is low.
```

---

# 6. Network Snapshot

## Command 10 – Check Listening Ports

```bash
sudo ss -tulpn
```

To look specifically for Docker:

```bash
sudo ss -tulpn | grep -i docker
```

### What this tells us

`ss` shows sockets currently listening or communicating.

Options:

```text
-t   TCP
-u   UDP
-l   Listening sockets
-p   Process information
-n   Numeric ports/IPs
```

### My Output

```text
Paste your output here.
```

### Observation

```text
Example: No unexpected listening ports were associated with Docker.
```

> `dockerd` itself does not necessarily listen on a TCP port. It commonly communicates through a Unix socket such as `/var/run/docker.sock`.

---

## Command 11 – Test a Service Endpoint

If you have a container exposing a web service:

```bash
docker ps
```

Suppose it exposes port `8000`:

```bash
curl -I http://localhost:8000
```

### My Output

```text
Paste your output here.
```

### Observation

```text
Example: HTTP endpoint returned 200 OK, confirming the service is reachable locally.
```

If you don't have an HTTP container running, use:

```bash
ping -c 4 8.8.8.8
```

This checks basic IP connectivity.

---

# 7. Logs

## Command 12 – Docker Service Logs

```bash
sudo journalctl -u docker -n 50
```

This retrieves the latest 50 journal entries for Docker.

Look for words such as:

```text
error
failed
timeout
denied
killed
OOM
```

### My Output

```text
Paste important lines here.
```

### Observation

```text
Example: Docker started successfully and no recent critical errors were found.
```

---

## Command 13 – Follow Logs Live

```bash
sudo journalctl -u docker -f
```

Now, in another terminal, perform an action such as:

```bash
sudo systemctl restart docker
```

Observe what appears in the logs.

Stop following logs with:

```text
Ctrl+C
```

### Observation

```text
Example: Docker stopped and restarted successfully with no startup errors.
```

> Restarting Docker can interrupt running containers. Only do this if it is safe on your machine.

---

# 8. Quick Findings

After completing the checks, summarize the system state.

```text
Target service: Docker

Service Status:
Docker is running normally.

CPU:
No abnormal CPU utilization observed.

Memory:
Sufficient memory is available and no significant swap activity was observed.

Disk:
Filesystem has sufficient free space.

I/O:
No significant I/O wait or swap activity observed.

Network:
Relevant ports/endpoints are reachable as expected.

Logs:
No critical errors found in recent Docker logs.

Overall:
Docker and the host system appear healthy.
```

Replace these statements with what you actually observed.

---

# 9. If This Worsens

If the service becomes slow, unavailable, or starts consuming excessive resources:

1. **Collect more evidence before restarting**

```bash
top -p <PID>
free -h
vmstat 1 10
df -h
sudo journalctl -u docker --since "30 minutes ago"
```

Determine whether the problem is CPU, memory, disk, I/O, network, or application related.

2. **Investigate Docker/container state**

```bash
docker ps -a
docker stats
docker logs <container-name>
```

Check whether a particular container is failing or consuming excessive CPU/memory.

3. **Restart only when appropriate**

```bash
sudo systemctl restart docker
```

Then verify:

```bash
systemctl status docker
sudo journalctl -u docker -n 50
```

If restarting temporarily fixes the issue but it returns, investigate the underlying cause rather than repeatedly restarting the service.

---

# Troubleshooting Flow

A useful order during an incident:

```text
Service Problem
      |
      v
systemctl status
      |
      v
CPU / Memory
top / ps / free
      |
      v
Disk / I/O
df / du / vmstat
      |
      v
Network
ss / curl / ping
      |
      v
Logs
journalctl
      |
      v
Identify Root Cause
      |
      v
Take Action
```

## Key Principle

**Observe → Measure → Check Logs → Form Hypothesis → Take Action → Verify**

Avoid restarting a service immediately. First capture enough information to understand **why** it is unhealthy.
