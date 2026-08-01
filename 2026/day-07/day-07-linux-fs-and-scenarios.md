# Day 07 – Linux File System Hierarchy & Scenario-Based Practice

---

# Part 1: Linux File System Hierarchy

Understanding the Linux File System Hierarchy is essential for every DevOps engineer because configuration files, logs, binaries, and applications are stored in well-defined locations.

---

## 1. `/` (Root Directory)

### Purpose
The root (`/`) is the top-most directory in Linux. Every file and directory starts from here.

### Example

```bash
ls -l /
```

Possible Output

```
bin
boot
dev
etc
home
opt
root
tmp
usr
var
```

### I would use this when...

I need to navigate the Linux filesystem or locate major system directories.

---

## 2. `/home`

### Purpose

Stores home directories of normal users.

Example:

```
/home/shashwat
/home/john
```

### Example

```bash
ls -l /home
```

Possible Output

```
shashwat
ubuntu
```

### I would use this when...

I need to access user files, scripts, documents, or SSH keys.

---

## 3. `/root`

### Purpose

Home directory of the root (administrator) user.

Unlike normal users, root's home is not inside `/home`.

### Example

```bash
ls -l /root
```

Possible Output

```
backup.sh
notes.txt
```

### I would use this when...

Working as the root user or checking administrator-specific files.

---

## 4. `/etc`

### Purpose

Contains almost all system configuration files.

Examples:

- hostname
- passwd
- ssh configuration
- network configuration

### Example

```bash
ls -l /etc
```

Possible Output

```
hostname
hosts
passwd
ssh/
systemd/
```

### I would use this when...

I need to modify system configurations or troubleshoot services.

---

## 5. `/var/log`

### Purpose

Stores system and application log files.

This is one of the most important directories for DevOps and troubleshooting.

### Example

```bash
ls -l /var/log
```

Possible Output

```
syslog
auth.log
kern.log
journal/
```

### I would use this when...

Investigating service failures, login issues, crashes, or system errors.

---

## 6. `/tmp`

### Purpose

Stores temporary files created by applications.

Usually cleared after reboot.

### Example

```bash
ls -l /tmp
```

Possible Output

```
systemd-private
tmp12345
cache.tmp
```

### I would use this when...

Applications create temporary data or while debugging.

---

## 7. `/bin`

### Purpose

Contains essential Linux commands required even in rescue mode.

Examples:

- ls
- cp
- mv
- cat
- bash

### Example

```bash
ls -l /bin
```

Possible Output

```
ls
cp
mv
bash
cat
```

### I would use this when...

Running basic Linux commands.

---

## 8. `/usr/bin`

### Purpose

Contains most user-level commands and installed applications.

Examples

- git
- python3
- curl
- vim
- ssh

### Example

```bash
ls -l /usr/bin
```

Possible Output

```
git
python3
curl
vim
ssh
```

### I would use this when...

Using installed software and development tools.

---

## 9. `/opt`

### Purpose

Contains optional or third-party applications.

Examples

```
/opt/google
/opt/tomcat
/opt/myapp
```

### Example

```bash
ls -l /opt
```

Possible Output

```
google/
tomcat/
myapp/
```

### I would use this when...

Installing or troubleshooting third-party software.

---

# Hands-on Commands

## Find Largest Log Files

```bash
du -sh /var/log/* 2>/dev/null | sort -h | tail -5
```

### What it does

- Calculates size of every log directory/file.
- Sorts them from smallest to largest.
- Displays the five largest log files/directories.

Useful when disk space is filling because of logs.

---

## View Hostname

```bash
cat /etc/hostname
```

### What it does

Displays the hostname of the Linux machine.

Example Output

```
ubuntu-server
```

Useful when identifying remote servers.

---

## View Home Directory

```bash
ls -la ~
```

### What it does

Shows all files (including hidden ones) inside your home directory.

Example Output

```
.bashrc
.profile
.ssh
Documents
Downloads
```

Useful for checking SSH keys, scripts, and configuration files.

---

# Part 2: Scenario-Based Practice

---

# Scenario 1 – Service Not Starting

### Problem

A web application service called **myapp** failed to start after reboot.

---

### Step 1

```bash
systemctl status myapp
```

### Why?

Checks whether the service is active, failed, or stopped. It also shows the latest error message if startup failed.

---

### Step 2

```bash
journalctl -u myapp -n 50
```

### Why?

Displays the last 50 log entries for the service. This helps identify errors such as missing files, permission issues, or configuration problems.

---

### Step 3

```bash
systemctl is-enabled myapp
```

### Why?

Checks whether the service is configured to start automatically after boot.

---

### Step 4

```bash
systemctl list-units --type=service | grep myapp
```

### Why?

Confirms whether the service exists and is recognized by systemd.

---

### What I Learned

Always check the service status first, review the logs to find the root cause, verify that the service is enabled at boot, and finally ensure the service is correctly registered with systemd.

---

# Scenario 2 – High CPU Usage

### Problem

The application server is responding slowly.

---

### Step 1

```bash
top
```

### Why?

Displays live CPU and memory usage, allowing you to identify processes consuming the most CPU.

---

### Step 2

```bash
ps aux --sort=-%cpu | head -10
```

### Why?

Lists the top 10 processes sorted by CPU usage, making it easy to identify the highest CPU consumers.

---

### Step 3

```bash
ps -fp <PID>
```

### Why?

Displays detailed information about the process using high CPU, including the user and command.

---

### Step 4

```bash
top -p <PID>
```

### Why?

Monitors only the selected process to observe its CPU usage in real time.

---

### What I Learned

Identify the process consuming CPU before taking action. Never terminate a process without understanding its role.

---

# Scenario 3 – Finding Docker Service Logs

### Problem

A developer asks where Docker service logs are located.

---

### Step 1

```bash
systemctl status docker
```

### Why?

Confirms that the Docker service is running and provides a quick summary of its current state.

---

### Step 2

```bash
journalctl -u docker -n 50
```

### Why?

Displays the last 50 log entries generated by the Docker service.

---

### Step 3

```bash
journalctl -u docker -f
```

### Why?

Follows the Docker logs in real time, similar to `tail -f`.

---

### Step 4

```bash
journalctl -u docker --since "1 hour ago"
```

### Why?

Shows only the logs generated within the last hour, making troubleshooting easier.

---

### What I Learned

Systemd-managed services store logs in the system journal, which can be accessed using `journalctl`.

---

# Scenario 4 – Permission Denied

### Problem

Running

```bash
./backup.sh
```

returns

```
Permission denied
```

---

### Step 1

```bash
ls -l /home/user/backup.sh
```

### Why?

Checks the current file permissions.

Example

```
-rw-r--r--
```

Notice that there is no **x** (execute permission).

---

### Step 2

```bash
chmod +x /home/user/backup.sh
```

### Why?

Adds execute permission to the script.

---

### Step 3

```bash
ls -l /home/user/backup.sh
```

### Why?

Verifies that execute permission has been added.

Example

```
-rwxr-xr-x
```

---

### Step 4

```bash
./backup.sh
```

### Why?

Runs the script after making it executable.

---

### What I Learned

Linux scripts require execute permission before they can be run.

---

# Quick Interview Revision

| Directory | Purpose |
|------------|---------|
| `/` | Root of the filesystem |
| `/home` | User home directories |
| `/root` | Root user's home |
| `/etc` | Configuration files |
| `/var/log` | System and application logs |
| `/tmp` | Temporary files |
| `/bin` | Essential system commands |
| `/usr/bin` | User-installed command binaries |
| `/opt` | Third-party applications |

---

# DevOps Takeaways

- Configuration files are primarily stored in **`/etc`**.
- Logs for troubleshooting are commonly found in **`/var/log`** or via **`journalctl`** for systemd services.
- User-specific data resides in **`/home`**, while the administrator's files are in **`/root`**.
- Temporary files are stored in **`/tmp`** and may be cleared on reboot.
- Essential commands are located in **`/bin`**, whereas most installed software binaries are under **`/usr/bin`**.
- Third-party applications are often installed in **`/opt`**.
- Effective troubleshooting starts with checking the service status, reviewing logs, and then verifying configuration or permissions.
