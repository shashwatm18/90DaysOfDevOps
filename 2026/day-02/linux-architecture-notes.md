# Day 02 – Linux Architecture, Processes, and systemd

## 1. Linux Architecture

Linux can be understood in layers:

```text
+-----------------------------+
|      Applications           |
|  nginx, python, docker...   |
+-----------------------------+
|         User Space          |
| shells, libraries, tools    |
+-----------------------------+
|          Kernel             |
| CPU, memory, processes, I/O |
+-----------------------------+
|          Hardware           |
| CPU, RAM, disk, network     |
+-----------------------------+
```

### Kernel

The **kernel** is the core of Linux. It sits between hardware and programs.

Main responsibilities:

* Process management
* Memory management
* Device management
* File systems
* Networking
* Security and permissions

Applications normally don't access hardware directly. They request kernel functionality through **system calls**.

Example:

```text
Python program
      ↓
System call
      ↓
Linux Kernel
      ↓
Disk / CPU / Network / Memory
```

Check the running kernel:

```bash
uname -r
```

More system information:

```bash
uname -a
```

---

## 2. User Space

**User space** is where normal applications and utilities run.

Examples:

* Bash
* Python
* Nginx
* Docker CLI
* `ps`
* `top`
* `systemctl`

User-space applications have restricted access to hardware and kernel memory.

When they need kernel functionality, they use system calls.

Example:

```text
Application
    ↓
User Space
    ↓
System Call
    ↓
Kernel
    ↓
Hardware
```

This separation helps protect the operating system from application failures.

---

## 3. Init and systemd

After Linux boots and initializes the kernel, it needs to start the rest of the operating system.

On many modern Linux distributions, this job is handled by **systemd**.

systemd normally runs as:

```text
PID 1
```

Check:

```bash
ps -p 1 -o pid,comm,args
```

You will commonly see something similar to:

```text
PID COMMAND  COMMAND
1   systemd  /sbin/init
```

systemd manages resources such as:

* Services
* Mount points
* Devices
* Timers
* Sockets
* System startup
* Service dependencies

---

# 4. Linux Processes

A **process** is a running instance of a program.

For example:

```bash
python app.py
```

`app.py` is a file/program.

When Python executes it, Linux creates a **process**.

Each process has information such as:

* PID — Process ID
* PPID — Parent Process ID
* User
* CPU usage
* Memory usage
* Process state
* Open files
* Environment variables

View processes:

```bash
ps aux
```

Or:

```bash
ps -ef
```

---

# 5. Parent and Child Processes

Linux processes form a hierarchy.

A process can create another process.

The existing process is the **parent**, and the new process is the **child**.

Example:

```text
systemd (PID 1)
   │
   ├── sshd
   │    └── bash
   │         └── python
   │
   └── nginx
        ├── nginx worker
        └── nginx worker
```

View PID and PPID:

```bash
ps -eo pid,ppid,comm
```

View the process hierarchy:

```bash
pstree -p
```

---

# 6. How Processes Are Created

A simplified Linux process creation model is:

```text
Parent Process
      ↓
    fork()
      ↓
Child Process
      ↓
    exec()
      ↓
New Program
```

### `fork()`

`fork()` creates a new process based on the calling process.

After `fork()`:

```text
Parent
   │
   └── Child
```

The child receives its own PID.

### `exec()`

The child can then use the `exec` family of system calls to replace its current program with another program.

Simplified example:

```text
bash
 │
 │ fork()
 ↓
bash child
 │
 │ exec()
 ↓
python app.py
```

So when you execute:

```bash
python app.py
```

your shell participates in starting the Python process.

---

# 7. Process States

Processes aren't always actively executing.

Common states shown by tools such as `ps` include:

### R — Running / Runnable

The process is executing or ready to execute on the CPU.

```text
R = Running/Runnable
```

### S — Interruptible Sleep

The process is waiting for an event.

Examples:

* Waiting for network data
* Waiting for user input
* Waiting on a timer

```text
S = Sleeping
```

Sleeping is normal. A process does not need CPU continuously.

### D — Uninterruptible Sleep

Usually indicates the process is waiting on kernel-level I/O or another resource.

```text
D = Uninterruptible Sleep
```

A process stuck in `D` for a long time can indicate problems involving things such as storage or network filesystems.

### T — Stopped

The process has been paused.

```text
T = Stopped
```

For example, run:

```bash
sleep 300
```

Press:

```text
Ctrl + Z
```

The process becomes stopped.

Continue it with:

```bash
fg
```

### Z — Zombie

The process has finished execution, but its parent has not yet collected its exit status.

```text
Child exits
     ↓
Parent hasn't collected status
     ↓
Zombie
```

A zombie is already dead, so killing the zombie itself doesn't solve the underlying issue. The parent needs to reap it, or the parent may need investigation.

Check process states:

```bash
ps aux
```

Or:

```bash
ps -eo pid,ppid,state,comm
```

---

# 8. Signals and Process Management

Linux uses **signals** to communicate events to processes.

For example:

```bash
kill PID
```

By default, `kill` sends:

```text
SIGTERM (15)
```

SIGTERM asks the process to terminate gracefully.

Example:

```bash
kill 1234
```

Force termination:

```bash
kill -9 1234
```

This sends:

```text
SIGKILL (9)
```

SIGKILL cannot be handled or ignored by the target process.

Therefore, prefer:

```bash
kill PID
```

before using:

```bash
kill -9 PID
```

You can list signals with:

```bash
kill -l
```

---

# 9. What Is systemd?

**systemd** is the system and service manager used by many modern Linux distributions.

It is normally started during boot and becomes:

```text
PID 1
```

systemd manages services such as:

```text
nginx
ssh
docker
cron
postgresql
```

Instead of manually starting every service, systemd manages their lifecycle.

---

# 10. systemctl

`systemctl` is the primary command used to interact with systemd.

Check a service:

```bash
systemctl status ssh
```

Start:

```bash
sudo systemctl start ssh
```

Stop:

```bash
sudo systemctl stop ssh
```

Restart:

```bash
sudo systemctl restart ssh
```

Enable at boot:

```bash
sudo systemctl enable ssh
```

Disable automatic startup:

```bash
sudo systemctl disable ssh
```

Important distinction:

```text
start  → Start service now
enable → Start service automatically during future boots
```

Therefore:

```bash
sudo systemctl enable nginx
```

does not mean the same thing as:

```bash
sudo systemctl start nginx
```

To enable and start together:

```bash
sudo systemctl enable --now nginx
```

---

# 11. systemd Units

systemd manages resources using **units**.

Common unit types:

```text
.service → Services
.socket  → Sockets
.timer   → Scheduled tasks
.mount   → Filesystem mounts
.target  → Groups/synchronization points
```

Example:

```text
nginx.service
docker.service
ssh.service
```

List running services:

```bash
systemctl list-units --type=service
```

List installed service unit files:

```bash
systemctl list-unit-files --type=service
```

---

# 12. systemd Service Files

A simplified service unit could look like:

```ini
[Unit]
Description=My Python Application
After=network.target

[Service]
ExecStart=/usr/bin/python3 /opt/myapp/app.py
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

The sections have different responsibilities:

```text
[Unit]
Dependencies and metadata

[Service]
How the process should run

[Install]
How the unit participates in enablement/startup
```

Important options include:

```text
ExecStart=
Restart=
User=
WorkingDirectory=
Environment=
```

After modifying unit files, run:

```bash
sudo systemctl daemon-reload
```

---

# 13. Logs with journalctl

systemd's journal can contain logs generated by services and the system.

View logs:

```bash
journalctl
```

View logs for a specific service:

```bash
journalctl -u ssh
```

Follow logs:

```bash
journalctl -u ssh -f
```

View logs from the current boot:

```bash
journalctl -b
```

A common troubleshooting workflow is:

```text
Service not working
       ↓
systemctl status
       ↓
Check process
       ↓
ps / top
       ↓
Check logs
       ↓
journalctl
       ↓
Find root cause
```

---

# 14. Five Daily Commands

### 1. `ps`

View processes.

```bash
ps aux
```

Useful when checking whether an application is running.

### 2. `top`

Monitor processes and resource usage in real time.

```bash
top
```

Useful for investigating CPU and memory usage.

### 3. `systemctl`

Manage systemd units and services.

```bash
systemctl status nginx
```

### 4. `journalctl`

Inspect system and service logs.

```bash
journalctl -u nginx
```

### 5. `kill`

Send signals to processes.

```bash
kill PID
```

---

# Practice Tasks

## Task 1 — Explore PID 1

Run:

```bash
ps -p 1 -o pid,ppid,user,comm,args
```

Answer:

* What is PID 1?
* What is its PPID?
* Which user owns it?
* Why is PID 1 important?

---

## Task 2 — Explore Processes

Run:

```bash
ps aux
```

Find:

* One process owned by your user
* One process owned by `root`
* PID of each process
* CPU usage
* Memory usage
* Process state

Then run:

```bash
ps -eo pid,ppid,state,user,comm
```

Identify the parent of one process.

---

## Task 3 — Create and Manage a Process

Run:

```bash
sleep 300
```

Open another terminal and find it:

```bash
ps aux | grep sleep
```

Identify its PID.

Terminate it gracefully:

```bash
kill <PID>
```

Verify:

```bash
ps -p <PID>
```

The process should no longer exist.

---

## Task 4 — Experiment with Process States

Run:

```bash
sleep 300
```

In another terminal:

```bash
ps -eo pid,state,comm | grep sleep
```

Observe its state.

Now press:

```text
Ctrl + Z
```

in the original terminal.

Check again:

```bash
ps -eo pid,state,comm | grep sleep
```

Notice the state change.

Resume the job:

```bash
fg
```

---

## Task 5 — Explore the Process Tree

Run:

```bash
pstree -p
```

Find your terminal shell.

Example:

```text
systemd(1)
   └── ...
       └── bash(5000)
            └── pstree(5100)
```

Understand:

```text
systemd
   ↓
...
   ↓
shell
   ↓
command
```

---

## Task 6 — Explore systemd Services

Run:

```bash
systemctl list-units --type=service
```

Pick one service, for example:

```bash
systemctl status ssh
```

Identify:

* Loaded state
* Active state
* Main PID
* Recent log messages

Do not stop important services on a work or production machine.

---

## Task 7 — Inspect a Service Unit

Choose a safe service:

```bash
systemctl cat ssh
```

Look for entries such as:

```text
ExecStart=
Restart=
After=
WantedBy=
```

Try to understand what command systemd executes to start the service.

---

## Task 8 — Investigate Logs

Run:

```bash
journalctl -b
```

Then:

```bash
journalctl -u ssh
```

Try:

```bash
journalctl -u ssh -n 20
```

Identify:

* Timestamp
* Hostname
* Process/service
* Message

---

# Mini DevOps Troubleshooting Challenge

Imagine users report:

> "The web application is down."

Assume it runs as `nginx.service`.

Start with:

```bash
systemctl status nginx
```

Then:

```bash
ps aux | grep nginx
```

Check logs:

```bash
journalctl -u nginx -n 50
```

Check live resource usage:

```bash
top
```

Your troubleshooting flow should become:

```text
Application Down
      ↓
Is service active?
      ↓
systemctl status
      ↓
Is process running?
      ↓
ps
      ↓
Any CPU/memory problem?
      ↓
top
      ↓
What happened?
      ↓
journalctl
      ↓
Fix root cause
      ↓
Restart if required
```

---

# Quick Revision

Remember this architecture:

```text
Applications
     ↓
User Space
     ↓
System Calls
     ↓
Linux Kernel
     ↓
Hardware
```

Process lifecycle:

```text
Parent
  ↓
fork()
  ↓
Child
  ↓
exec()
  ↓
Program
  ↓
exit()
  ↓
Parent collects status
```

Boot/service model:

```text
Boot
 ↓
Kernel
 ↓
systemd (PID 1)
 ↓
Services
 ↓
Application Processes
```

Most importantly, become comfortable with:

```bash
ps
top
systemctl
journalctl
kill
```

These commands form a basic toolkit for Linux process and service troubleshooting.
