# Day 03 – Linux Commands Practice

## Goal

Build confidence using Linux commands commonly needed in DevOps for:

- Process management
- File and directory management
- Searching and inspecting files
- Permissions
- Networking troubleshooting
- Logs and system inspection

The goal is not to memorize every Linux command. The goal is to know **which command to use when troubleshooting a problem**.

---

# 1. Process Management

A **process** is a running instance of a program.

For example, when you run:

```bash
python3 app.py
```

Linux creates a process for the Python program.

Each process receives a unique **PID (Process ID)**.

---

## `ps` – View Processes

```bash
ps
```

Shows processes associated with the current terminal.

More useful:

```bash
ps aux
```

Shows processes running across the system.

Example:

```bash
ps aux | grep nginx
```

Use this when you want to check whether a process is running.

---

## `top` – Monitor Processes

```bash
top
```

Displays processes and system resource usage in real time.

Useful information includes:

- PID
- CPU usage
- Memory usage
- Running processes
- Load average

Useful during:

- High CPU incidents
- High memory usage
- Performance troubleshooting

Press:

```text
q
```

to exit.

---

## `pgrep` – Find Process IDs

```bash
pgrep nginx
```

Finds PIDs of processes matching a name.

More detailed:

```bash
pgrep -a nginx
```

Example output:

```text
1520 nginx: master process
1521 nginx: worker process
```

---

## `kill` – Send a Signal to a Process

```bash
kill PID
```

Example:

```bash
kill 1520
```

By default this sends:

```text
SIGTERM
```

which asks the process to terminate gracefully.

If the process refuses to stop:

```bash
kill -9 1520
```

This sends `SIGKILL`, which immediately terminates the process.

Prefer normal `kill` before using `kill -9`.

---

## `pkill` – Kill Process by Name

Instead of finding the PID:

```bash
pkill process_name
```

Example:

```bash
pkill nginx
```

Be careful because multiple matching processes may be affected.

---

## `jobs` – View Background Jobs

```bash
jobs
```

Shows jobs started from your current shell.

Example:

```bash
sleep 300 &
```

Then:

```bash
jobs
```

---

## `bg` – Continue Job in Background

```bash
bg
```

Example:

```bash
sleep 300
```

Press:

```text
Ctrl+Z
```

Then:

```bash
bg
```

The process continues running in the background.

---

## `fg` – Bring Job to Foreground

```bash
fg
```

Brings a background shell job back into the foreground.

---

# 2. File System Commands

---

## `pwd` – Current Directory

```bash
pwd
```

Shows your current working directory.

Example:

```text
/home/user/projects
```

---

## `ls` – List Files

```bash
ls
```

Detailed listing:

```bash
ls -l
```

Include hidden files:

```bash
ls -la
```

Human-readable sizes:

```bash
ls -lh
```

Common troubleshooting command:

```bash
ls -lah
```

---

## `cd` – Change Directory

```bash
cd /var/log
```

Go to home directory:

```bash
cd ~
```

Go one directory back:

```bash
cd ..
```

Go to previous directory:

```bash
cd -
```

---

## `mkdir` – Create Directory

```bash
mkdir logs
```

Create nested directories:

```bash
mkdir -p project/logs/archive
```

---

## `touch` – Create Empty File

```bash
touch app.log
```

Also updates an existing file's timestamp.

---

## `cp` – Copy Files

```bash
cp source.txt destination.txt
```

Copy directory recursively:

```bash
cp -r source_dir backup_dir
```

---

## `mv` – Move or Rename Files

Move:

```bash
mv app.log /tmp/
```

Rename:

```bash
mv old.txt new.txt
```

---

## `rm` – Delete Files

```bash
rm file.txt
```

Delete directory recursively:

```bash
rm -r directory
```

Force recursive deletion:

```bash
rm -rf directory
```

> Be extremely careful with `rm -rf`. Deleted files normally cannot be recovered using an undo command.

---

# 3. Viewing and Searching Files

These commands are extremely useful when troubleshooting logs.

---

## `cat` – Display File

```bash
cat file.txt
```

Useful for small files.

Example:

```bash
cat config.json
```

Avoid using `cat` for huge log files.

---

## `less` – Read Large Files

```bash
less application.log
```

Useful for inspecting large logs.

Inside `less`:

```text
/ERROR
```

searches for `ERROR`.

Press:

```text
n
```

for the next match.

Press:

```text
q
```

to exit.

---

## `head` – First Lines of File

```bash
head application.log
```

Default:

```text
10 lines
```

Custom number:

```bash
head -n 20 application.log
```

---

## `tail` – Last Lines of File

```bash
tail application.log
```

Last 50 lines:

```bash
tail -n 50 application.log
```

Follow logs in real time:

```bash
tail -f application.log
```

This is one of the most useful DevOps commands.

---

## `grep` – Search Text

```bash
grep "ERROR" application.log
```

Case-insensitive:

```bash
grep -i "error" application.log
```

Show line numbers:

```bash
grep -n "ERROR" application.log
```

Recursive search:

```bash
grep -r "database" /etc/
```

Example troubleshooting:

```bash
grep -i "timeout" application.log
```

---

## `find` – Find Files

Find by filename:

```bash
find /var/log -name "*.log"
```

Find directory:

```bash
find / -type d -name "nginx" 2>/dev/null
```

Find files:

```bash
find /tmp -type f
```

---

# 4. File Permissions

Linux permissions are important because services often fail because they cannot read, write, or execute something.

Check permissions:

```bash
ls -l
```

Example:

```text
-rwxr-xr-- 1 user devops 1200 Jul 29 script.sh
```

Permissions represent:

```text
Owner | Group | Others
```

---

## `chmod` – Change Permissions

Make script executable:

```bash
chmod +x script.sh
```

Example numeric permissions:

```bash
chmod 755 script.sh
```

Meaning:

```text
Owner  = rwx
Group  = r-x
Others = r-x
```

---

## `chown` – Change Ownership

```bash
sudo chown user file.txt
```

Change owner and group:

```bash
sudo chown user:group file.txt
```

Recursive:

```bash
sudo chown -R user:group directory/
```

---

# 5. Disk and Storage Commands

---

## `df` – Check Disk Space

```bash
df -h
```

Example:

```text
Filesystem      Size  Used  Avail  Use%
/dev/sda1       100G   92G     8G   92%
```

Useful when investigating:

```text
No space left on device
```

---

## `du` – Check Directory Size

```bash
du -sh /var/log
```

Example:

```text
4.2G    /var/log
```

Check directories in current location:

```bash
du -sh *
```

Very useful when finding what is consuming disk space.

---

# 6. Networking Commands

Networking troubleshooting is one of the most important DevOps skills.

Think about network debugging in layers:

```text
Network interface
      ↓
IP connectivity
      ↓
DNS
      ↓
Port
      ↓
HTTP/Application
```

---

## `ip addr` – Check IP Address

```bash
ip addr
```

Short form:

```bash
ip a
```

Shows:

- Network interfaces
- IP addresses
- Interface status

Example:

```text
eth0
inet 192.168.1.10/24
```

Use this first when checking network configuration.

---

## `ping` – Test Connectivity

```bash
ping google.com
```

Send four packets:

```bash
ping -c 4 google.com
```

Example:

```text
64 bytes from 142.x.x.x: time=20 ms
```

Tests basic IP connectivity and latency.

Some servers intentionally block ICMP, so failed `ping` does not always mean the server is unavailable.

---

## `dig` – DNS Troubleshooting

```bash
dig google.com
```

Short result:

```bash
dig +short google.com
```

Example:

```text
142.250.x.x
```

Use when investigating DNS resolution.

---

## `curl` – Test HTTP/HTTPS

```bash
curl https://example.com
```

Headers only:

```bash
curl -I https://example.com
```

Verbose:

```bash
curl -v https://example.com
```

Test local application:

```bash
curl http://localhost:8000
```

Test API:

```bash
curl http://localhost:8000/health
```

`curl` is one of the most important DevOps troubleshooting tools.

---

## `ss` – Inspect Network Connections

Show listening ports:

```bash
ss -tuln
```

Useful options:

```text
-t = TCP
-u = UDP
-l = listening
-n = numeric ports
-p = process information
```

Show listening ports and processes:

```bash
sudo ss -tulpn
```

Example:

```text
LISTEN 0 128 0.0.0.0:8000
```

This tells you something is listening on port `8000`.

---

## `ip route` – Check Routing

```bash
ip route
```

Example:

```text
default via 192.168.1.1 dev eth0
```

Useful when a machine cannot reach another network or the internet.

---

# 7. Logs and System Troubleshooting

---

## `journalctl` – View systemd Logs

View logs:

```bash
journalctl
```

Logs for a service:

```bash
journalctl -u nginx
```

Recent logs:

```bash
journalctl -u nginx -n 50
```

Follow logs:

```bash
journalctl -u nginx -f
```

---

## `systemctl` – Manage Services

Check service:

```bash
systemctl status nginx
```

Start:

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

---

# Linux Command Cheat Sheet

## Process Management

| Command | Usage |
|---|---|
| `ps aux` | Show running processes |
| `top` | Monitor processes and resources |
| `pgrep nginx` | Find PID by process name |
| `kill PID` | Gracefully terminate process |
| `kill -9 PID` | Force terminate process |
| `pkill nginx` | Terminate processes by name |
| `jobs` | Show shell background jobs |
| `bg` | Continue job in background |
| `fg` | Bring job to foreground |

## File System

| Command | Usage |
|---|---|
| `pwd` | Show current directory |
| `ls -lah` | List files with details |
| `cd DIR` | Change directory |
| `mkdir DIR` | Create directory |
| `touch FILE` | Create empty file |
| `cp SRC DEST` | Copy file |
| `mv SRC DEST` | Move or rename |
| `rm FILE` | Delete file |
| `cat FILE` | Display file |
| `less FILE` | Read large file |
| `head FILE` | Show first lines |
| `tail FILE` | Show last lines |
| `tail -f FILE` | Follow file updates |
| `grep TEXT FILE` | Search text |
| `find PATH -name NAME` | Find files |
| `chmod` | Change permissions |
| `chown` | Change ownership |
| `df -h` | Check disk usage |
| `du -sh DIR` | Check directory size |

## Networking

| Command | Usage |
|---|---|
| `ip addr` | Show IP/interface information |
| `ip route` | Show routing table |
| `ping HOST` | Test connectivity |
| `dig DOMAIN` | Test DNS resolution |
| `curl URL` | Test HTTP/API |
| `ss -tuln` | Show listening ports |

## Services and Logs

| Command | Usage |
|---|---|
| `systemctl status SERVICE` | Check service status |
| `systemctl restart SERVICE` | Restart service |
| `journalctl -u SERVICE` | View service logs |

---

# Practice Work

Do these exercises on your Linux machine instead of only reading the commands.

---

## Lab 1 – File System Practice

Create a workspace:

```bash
mkdir -p ~/linux-practice/logs
cd ~/linux-practice
```

Check your location:

```bash
pwd
```

Create files:

```bash
touch app.log
touch config.txt
touch notes.txt
```

List them:

```bash
ls -lah
```

Create backup directory:

```bash
mkdir backup
```

Copy:

```bash
cp config.txt backup/
```

Rename:

```bash
mv notes.txt linux-notes.txt
```

Verify:

```bash
find . -type f
```

### Challenge

Without leaving `~/linux-practice`, create this structure:

```text
linux-practice/
├── backup/
├── logs/
│   ├── app.log
│   └── error.log
├── config/
│   └── app.conf
└── linux-notes.txt
```

---

# Lab 2 – Log Troubleshooting

Create fake application logs:

```bash
cat > logs/app.log << EOF
INFO Application started
INFO Connecting to database
ERROR Database connection failed
INFO Retrying connection
ERROR Connection timeout
INFO Retrying connection
ERROR Database connection failed
INFO Application stopped
EOF
```

Display logs:

```bash
cat logs/app.log
```

Show first three:

```bash
head -n 3 logs/app.log
```

Show last three:

```bash
tail -n 3 logs/app.log
```

Find errors:

```bash
grep "ERROR" logs/app.log
```

Count errors:

```bash
grep -c "ERROR" logs/app.log
```

Search case-insensitively:

```bash
grep -i "timeout" logs/app.log
```

### Challenge

Find only lines containing:

```text
Database
```

Then find how many times:

```text
connection
```

appears regardless of uppercase/lowercase.

---

# Lab 3 – Process Management

Start a fake long-running process:

```bash
sleep 500 &
```

Check jobs:

```bash
jobs
```

Find it:

```bash
pgrep -a sleep
```

Or:

```bash
ps aux | grep sleep
```

Find the PID and terminate it:

```bash
kill PID
```

Verify:

```bash
pgrep -a sleep
```

### Challenge

Start three processes:

```bash
sleep 300 &
sleep 400 &
sleep 500 &
```

Use:

```bash
ps
jobs
pgrep
kill
```

to identify and terminate them.

---

# Lab 4 – CPU Troubleshooting

Run:

```bash
top
```

Observe:

```text
CPU usage
Memory usage
Load average
Processes
```

Open another terminal and generate CPU load:

```bash
yes > /dev/null &
```

Return to `top`.

Find the process consuming CPU.

Then terminate it:

```bash
kill PID
```

Verify CPU usage decreases.

> Don't forget to terminate `yes`; it intentionally consumes CPU.

---

# Lab 5 – Networking Troubleshooting

Check interfaces:

```bash
ip addr
```

Check routes:

```bash
ip route
```

Test internet connectivity:

```bash
ping -c 4 8.8.8.8
```

Test DNS:

```bash
dig google.com
```

Test HTTP:

```bash
curl -I https://example.com
```

Check listening ports:

```bash
ss -tuln
```

---

# Lab 6 – Run a Local Web Server

This combines process and networking troubleshooting.

Start:

```bash
python3 -m http.server 8000
```

Open another terminal.

Check the process:

```bash
ps aux | grep http.server
```

Check the port:

```bash
ss -tuln | grep 8000
```

Test the application:

```bash
curl http://localhost:8000
```

You now have:

```text
Python Process
      ↓
Listening on :8000
      ↓
HTTP Request
      ↓
curl
      ↓
HTTP Response
```

Stop the server with:

```text
Ctrl+C
```

---

# Troubleshooting Scenarios

## Scenario 1 – Application Is Down

Users report:

```text
Application is not accessible.
```

Investigate using commands such as:

```bash
ps aux | grep application
ss -tuln
curl http://localhost:PORT
systemctl status SERVICE
journalctl -u SERVICE
```

Think in this order:

```text
Is process running?
        ↓
Is port listening?
        ↓
Can application respond locally?
        ↓
Are there errors in logs?
```

---

## Scenario 2 – Website Cannot Be Reached

Suppose:

```text
https://example.com
```

cannot be reached.

Investigate:

```bash
ping -c 4 8.8.8.8
dig example.com
curl -v https://example.com
ip route
```

Think:

```text
Internet working?
      ↓
DNS working?
      ↓
Server reachable?
      ↓
HTTP/HTTPS working?
```

---

## Scenario 3 – Server Disk Is Full

You receive:

```text
No space left on device
```

Check:

```bash
df -h
```

Then:

```bash
du -sh /var/log
```

Inspect directories:

```bash
du -sh /var/log/*
```

Find large files:

```bash
find /var/log -type f -size +100M
```

Your objective is to determine **what filesystem is full and what is consuming the space**, not immediately delete files.

---

# Final Practice Challenge

Imagine you receive this alert:

```text
ALERT: API unavailable on port 8000
```

Your job is to investigate it entirely from the terminal.

Try:

```bash
ps aux | grep python
ss -tuln | grep 8000
curl -v http://localhost:8000
ip addr
journalctl
df -h
top
```

For every command, ask yourself:

1. What am I checking?
2. What output tells me things are healthy?
3. What output indicates a problem?
4. What command should I run next?

That troubleshooting thought process is more important than memorizing commands.

---

# Day 03 Completion Checklist

Before considering Day 03 complete, make sure you can comfortably use:

- [ ] `ps`
- [ ] `top`
- [ ] `pgrep`
- [ ] `kill`
- [ ] `pwd`
- [ ] `ls`
- [ ] `cd`
- [ ] `mkdir`
- [ ] `touch`
- [ ] `cp`
- [ ] `mv`
- [ ] `rm`
- [ ] `cat`
- [ ] `less`
- [ ] `head`
- [ ] `tail`
- [ ] `grep`
- [ ] `find`
- [ ] `chmod`
- [ ] `chown`
- [ ] `df`
- [ ] `du`
- [ ] `ip addr`
- [ ] `ip route`
- [ ] `ping`
- [ ] `dig`
- [ ] `curl`
- [ ] `ss`
- [ ] `systemctl`
- [ ] `journalctl`

## Key Principle

Don't memorize commands in isolation.

Learn them as troubleshooting chains:

```text
High CPU
→ top
→ ps
→ identify PID
→ inspect process
→ logs
→ kill/restart if appropriate

Application Down
→ systemctl status
→ ps
→ ss
→ curl
→ journalctl

Network Issue
→ ip addr
→ ip route
→ ping IP
→ dig domain
→ curl service

Disk Full
→ df
→ du
→ find
→ inspect large files
```

That is how Linux commands become a practical DevOps toolkit.
