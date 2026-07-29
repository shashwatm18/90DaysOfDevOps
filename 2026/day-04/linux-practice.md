# Day 04 – Linux Practice: Processes and Services

## Objective

The goal of Day 04 is to practice three important Linux troubleshooting areas:

1. **Processes** – What programs are currently running?
2. **Services** – What background services are running and managed by `systemd`?
3. **Logs** – What happened when a service started, stopped, or failed?

A common Linux troubleshooting flow is:

```text
Problem
  ↓
Check Process
  ↓
Check Service
  ↓
Check Logs
  ↓
Identify Cause
  ↓
Fix
  ↓
Verify
```

---

# 1. Linux Processes

## What is a Process?

A **process** is a running instance of a program.

For example, when you run:

```bash
python3 app.py
```

`app.py` runs as a process.

Similarly, applications such as:

* Nginx
* Docker
* SSH server
* Python applications
* Java applications

run as one or more processes.

Every process has a unique number called a **PID – Process ID**.

Example:

```text
PID     COMMAND
1234    nginx
2456    python3
```

The PID allows Linux and users to identify and manage a particular process.

---

# 2. `ps` – View Running Processes

The `ps` command displays information about running processes.

```bash
ps
```

This normally shows processes associated with your current terminal.

Example:

```text
PID     TTY          TIME CMD
4120    pts/0    00:00:00 bash
4378    pts/0    00:00:00 ps
```

For troubleshooting, a much more useful command is:

```bash
ps aux
```

### Breaking it down

```text
ps aux
│  ││
│  │└── x → include processes without a terminal
│  └── u → user-oriented detailed format
└──── a → processes from all users
```

Example:

```text
USER       PID  %CPU %MEM    VSZ   RSS TTY   STAT START TIME COMMAND
root         1   0.0  0.1  23456  9000 ?     Ss   09:00 0:02 /sbin/init
root       821   0.1  0.5 123456 20000 ?     Ssl  09:01 0:04 dockerd
user      4210   0.0  0.1  12000  5000 pts/0 S+   10:30 0:00 bash
```

Important columns:

| Column  | Meaning                     |
| ------- | --------------------------- |
| USER    | User running the process    |
| PID     | Process ID                  |
| %CPU    | CPU utilization             |
| %MEM    | Memory utilization          |
| VSZ     | Virtual memory              |
| RSS     | Physical RAM currently used |
| STAT    | Process state               |
| START   | When process started        |
| TIME    | CPU time consumed           |
| COMMAND | Command/program running     |

---

# 3. Finding a Specific Process

Instead of looking through every process manually, combine `ps` with `grep`.

Example:

```bash
ps aux | grep nginx
```

The pipe:

```text
|
```

passes the output of the first command to the second command.

So:

```bash
ps aux | grep nginx
```

means:

```text
Get all processes
       ↓
ps aux
       ↓
Filter lines containing "nginx"
       ↓
grep nginx
```

You might see:

```text
root      1024  ... nginx: master process
www-data  1025  ... nginx: worker process
www-data  1026  ... nginx: worker process
```

---

# 4. `pgrep` – Find a Process by Name

`pgrep` searches running processes by name.

```bash
pgrep nginx
```

Example:

```text
1024
1025
1026
```

To see the PID and process name:

```bash
pgrep -a nginx
```

Example:

```text
1024 nginx: master process /usr/sbin/nginx
1025 nginx: worker process
```

This is often cleaner than:

```bash
ps aux | grep nginx
```

because `grep` itself may appear in the `ps` output.

---

# 5. `top` – Monitor Processes in Real Time

Run:

```bash
top
```

Unlike `ps`, which gives you a snapshot, `top` continuously updates process information.

Example:

```text
top - 14:30:22 up 2:41, 1 user, load average: 0.52, 0.41, 0.39

%Cpu(s):  4.0 us,  1.0 sy, 95.0 id

MiB Mem :  15900 total
            8200 free
            4200 used
            3500 buff/cache

PID USER      %CPU %MEM COMMAND
821 root       2.3  1.2 dockerd
934 user       1.1  2.4 chrome
```

Important values:

```text
%CPU
%MEM
PID
COMMAND
```

Press:

```text
q
```

to exit `top`.

Useful shortcuts inside `top`:

```text
P → sort by CPU
M → sort by memory
q → quit
```

---

# 6. Process Troubleshooting Example

Suppose a Python application should be running.

Check:

```bash
pgrep -a python3
```

Or:

```bash
ps aux | grep python3
```

If you know its PID:

```bash
ps -p 1234 -f
```

You can terminate a process with:

```bash
kill 1234
```

This normally sends `SIGTERM`, asking the process to terminate gracefully.

If the process refuses to terminate, `SIGKILL` exists:

```bash
kill -9 1234
```

Use `kill -9` only when necessary because the process gets no opportunity to clean up before termination.

---

# 7. Linux Services

## What is a Service?

A **service** is usually a long-running background process that provides some functionality.

Examples:

```text
ssh
cron
docker
nginx
postgresql
```

For example:

```text
nginx
```

provides a web server.

```text
ssh
```

allows remote SSH connections.

```text
docker
```

provides the Docker daemon.

On most modern Linux distributions, services are managed using:

```text
systemd
```

and we interact with `systemd` using:

```text
systemctl
```

---

# 8. `systemctl status`

To inspect a service:

```bash
systemctl status ssh
```

or, if Docker is installed:

```bash
systemctl status docker
```

Example:

```text
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled)
     Active: active (running)
   Main PID: 821 (dockerd)
      Tasks: 21
     Memory: 98.5M
```

Important fields:

### Loaded

```text
Loaded: loaded
```

The service definition was successfully loaded.

### Enabled

```text
enabled
```

The service is configured to start automatically during boot.

### Active

```text
Active: active (running)
```

The service is currently running.

### Main PID

```text
Main PID: 821
```

The PID of the service's main process.

---

# 9. Active vs Enabled

These two concepts are different.

```text
Active  → Is the service running right now?
Enabled → Should the service automatically start during boot?
```

A service could therefore be:

```text
active + enabled
active + disabled
inactive + enabled
inactive + disabled
```

Check whether it is active:

```bash
systemctl is-active docker
```

Example:

```text
active
```

Check whether it starts automatically:

```bash
systemctl is-enabled docker
```

Example:

```text
enabled
```

---

# 10. List Running Services

Run:

```bash
systemctl list-units --type=service
```

Example:

```text
UNIT                  LOAD   ACTIVE SUB     DESCRIPTION
cron.service          loaded active running Regular background processing
docker.service        loaded active running Docker Application Container Engine
ssh.service           loaded active running OpenBSD Secure Shell server
```

To show only currently running services:

```bash
systemctl list-units --type=service --state=running
```

This is useful when you want to discover what services are available on your machine before choosing one to inspect.

---

# 11. Managing Services

Start a service:

```bash
sudo systemctl start nginx
```

Stop:

```bash
sudo systemctl stop nginx
```

Restart:

```bash
sudo systemctl restart nginx
```

Reload configuration without a full restart, if the service supports it:

```bash
sudo systemctl reload nginx
```

Enable automatic startup:

```bash
sudo systemctl enable nginx
```

Disable automatic startup:

```bash
sudo systemctl disable nginx
```

Be careful when practicing `stop` or `restart` on services your system depends on.

---

# 12. Linux Logs

Processes and services tell you **what is running**.

Logs help tell you **what happened**.

For services managed by `systemd`, logs are commonly accessed using:

```text
journalctl
```

---

# 13. `journalctl`

View system logs:

```bash
journalctl
```

This can produce a lot of output.

To inspect logs for one service:

```bash
journalctl -u docker
```

Or:

```bash
journalctl -u ssh
```

The `-u` option means:

```text
unit
```

A systemd service is represented by a unit such as:

```text
docker.service
ssh.service
cron.service
```

---

# 14. View Recent Service Logs

Instead of displaying the entire history:

```bash
journalctl -u docker -n 50
```

This displays the latest 50 log entries for the service.

To see logs from the current boot:

```bash
journalctl -u docker -b
```

Useful when troubleshooting something that happened after the latest reboot.

---

# 15. Follow Logs in Real Time

Run:

```bash
journalctl -u docker -f
```

`-f` means **follow**.

New log entries will appear as they are generated.

This behaves similarly to:

```bash
tail -f
```

Press:

```text
Ctrl + C
```

to stop following the logs.

---

# 16. `tail` – Inspect Log Files

Some applications write directly to log files.

For example:

```bash
tail -n 50 /var/log/syslog
```

This displays the last 50 lines.

To follow the file:

```bash
tail -f /var/log/syslog
```

On some distributions `/var/log/syslog` may not exist. In that case, use `journalctl` or inspect files available under:

```bash
ls /var/log
```

---

# 17. Mini Troubleshooting Flow

Imagine someone reports:

> "Docker is not working."

Don't immediately restart Docker.

First investigate.

## Step 1 – Is the process running?

```bash
pgrep -a dockerd
```

If you see something like:

```text
821 /usr/bin/dockerd
```

the Docker daemon process exists.

---

## Step 2 – Check service status

```bash
systemctl status docker
```

Look for:

```text
Active: active (running)
```

or:

```text
Active: failed
```

---

## Step 3 – Check recent logs

```bash
journalctl -u docker -n 50
```

Look for messages containing words such as:

```text
error
failed
permission denied
timeout
connection refused
```

---

## Step 4 – Inspect the process

```bash
ps aux | grep dockerd
```

Check:

```text
PID
CPU
Memory
Command
```

---

## Step 5 – Check whether Docker is responding

If Docker is installed:

```bash
docker ps
```

If the daemon is healthy, Docker should respond and show the container list.

---

## Step 6 – Restart only if appropriate

After investigating the problem:

```bash
sudo systemctl restart docker
```

Then verify:

```bash
systemctl status docker
```

And:

```bash
docker ps
```

### Troubleshooting principle

```text
Observe → Diagnose → Fix → Verify
```

Avoid:

```text
Problem → Restart → Hope
```

---

# Practice Work

For this exercise, choose **one service that actually exists on your machine**.

Recommended:

```text
docker
ssh
cron
```

First discover running services:

```bash
systemctl list-units --type=service --state=running
```

Choose one service and use the same service throughout the exercise.

For the examples below, assume:

```text
SERVICE=docker
```

Replace `docker` if you choose another service.

---

# Practice 1 – Check Running Processes

## Command 1

Run:

```bash
ps aux
```

Record a few processes you recognize.

### My Observation

```text
Command:
ps aux

Processes noticed:

1.
2.
3.

What I learned:

```

---

## Command 2

Run:

```bash
top
```

Find a process consuming noticeable CPU or memory.

### My Observation

```text
Command:
top

Process:
PID:
CPU:
Memory:

What I learned:

```

---

## Command 3 – Search for Your Service Process

For Docker:

```bash
pgrep -a dockerd
```

For SSH:

```bash
pgrep -a sshd
```

For cron:

```bash
pgrep -a cron
```

### My Observation

```text
Command:

PID:

Process:

What I learned:

```

---

# Practice 2 – Inspect a Service

## Command 4

List running services:

```bash
systemctl list-units --type=service --state=running
```

Record three services:

```text
1.
2.
3.
```

---

## Command 5

Inspect your selected service.

Example:

```bash
systemctl status docker
```

Record:

```text
Service:
Loaded:
Active:
Main PID:
Memory:
```

---

## Command 6

Check whether the service is enabled:

```bash
systemctl is-enabled docker
```

Record:

```text
Command:

Output:

Does it start automatically during boot?

```

---

# Practice 3 – Inspect Logs

## Command 7

Check the latest 20 log entries:

```bash
journalctl -u docker -n 20
```

Record:

```text
Command:

Anything interesting in the logs?

Errors found:

Warnings found:

```

---

## Command 8

Check logs from the current boot:

```bash
journalctl -u docker -b
```

Record:

```text
Command:

What did you notice?

```

---

## Command 9 – Follow Logs

Run:

```bash
journalctl -u docker -f
```

Keep it running briefly and then press:

```text
Ctrl + C
```

Record:

```text
Did new logs appear?

What kind of events were logged?

```

---

# Practice 4 – Process and Service Connection

One important DevOps skill is connecting a **systemd service** to the **actual process** running behind it.

Run:

```bash
systemctl status docker
```

Find:

```text
Main PID
```

Suppose it is:

```text
Main PID: 821
```

Now inspect that process:

```bash
ps -p 821 -f
```

Your task is to verify that:

```text
systemd service
      ↓
Main PID
      ↓
Linux process
```

Record:

```text
Service:

Main PID:

Process command:

```

---

# Practice 5 – Mini Troubleshooting Exercise

Pretend your selected service is reported as unavailable.

Perform these checks in order.

### 1. Check process

```bash
pgrep -a dockerd
```

Question:

```text
Is the process running?

Answer:
```

### 2. Check service

```bash
systemctl status docker
```

Question:

```text
Is the service active?

Answer:
```

### 3. Check recent logs

```bash
journalctl -u docker -n 50
```

Question:

```text
Are there errors or warnings?

Answer:
```

### 4. Check boot configuration

```bash
systemctl is-enabled docker
```

Question:

```text
Will the service start automatically after reboot?

Answer:
```

### 5. Verify application functionality

For Docker:

```bash
docker ps
```

For SSH:

```bash
ss -tlnp | grep ':22'
```

Question:

```text
Is the service actually functioning?

Answer:
```

---

# Challenge Exercise – Create Your Own Service Problem

Only do this with a non-critical service on a practice machine.

Check the service first:

```bash
systemctl status <service>
```

Stop it:

```bash
sudo systemctl stop <service>
```

Now investigate:

```bash
pgrep -a <process-name>
```

```bash
systemctl status <service>
```

```bash
journalctl -u <service> -n 20
```

Start it again:

```bash
sudo systemctl start <service>
```

Verify:

```bash
systemctl status <service>
```

Then inspect the logs again:

```bash
journalctl -u <service> -n 20
```

This gives you experience seeing the difference between:

```text
running
inactive
starting
running again
```

---

# Command Cheat Sheet

| Command                               | Purpose                              |
| ------------------------------------- | ------------------------------------ |
| `ps aux`                              | Show running processes               |
| `top`                                 | Monitor processes in real time       |
| `pgrep -a <name>`                     | Find PID and command by process name |
| `ps -p <PID> -f`                      | Inspect a specific process           |
| `systemctl status <service>`          | Check service status                 |
| `systemctl is-active <service>`       | Check whether service is running     |
| `systemctl is-enabled <service>`      | Check whether service starts at boot |
| `systemctl list-units --type=service` | List service units                   |
| `systemctl restart <service>`         | Restart service                      |
| `journalctl -u <service>`             | View service logs                    |
| `journalctl -u <service> -n 50`       | Last 50 service log entries          |
| `journalctl -u <service> -b`          | Logs from current boot               |
| `journalctl -u <service> -f`          | Follow service logs                  |
| `tail -n 50 <file>`                   | Last 50 lines of a file              |
| `tail -f <file>`                      | Follow a log file                    |

---

# Day 04 Final Task

Create:

```text
linux-practice.md
```

Your final practice log should contain output from **at least 6 commands**.

Minimum requirements:

* 2 process commands
* 2 service commands
* 2 log commands
* 1 service investigated
* 1 mini troubleshooting flow

A good command set would be:

```bash
ps aux
pgrep -a dockerd

systemctl status docker
systemctl is-enabled docker

journalctl -u docker -n 20
journalctl -u docker -b
```

Don't copy example outputs into your final practice log. Run the commands on your machine and record your actual results.

---

# Key Takeaways

After Day 04, you should understand:

**Process**

```text
A running instance of a program.
```

**PID**

```text
Unique Process ID assigned to a running process.
```

**systemd**

```text
System and service manager used by many Linux distributions.
```

**systemctl**

```text
Command used to inspect and control systemd services.
```

**journalctl**

```text
Command used to inspect logs stored by the systemd journal.
```

Most importantly, remember this troubleshooting flow:

```text
Application problem
        ↓
Is the process running?
        ↓
Is the service healthy?
        ↓
What do the logs say?
        ↓
Identify root cause
        ↓
Apply fix
        ↓
Verify service/application
```

The goal is not to memorize every Linux command. The goal is to understand **which command answers which troubleshooting question**.
