# Day 09 – Linux User & Group Management Notes

## Objective

Learn how Linux manages users, groups, ownership, and permissions by performing hands-on administration tasks.

---

# Why User Management is Important

In Linux, every process runs as a user.

Users are used to:
- Login to the system
- Own files and directories
- Run applications
- Control permissions

Groups are used to:
- Share access among multiple users
- Simplify permission management
- Control access to shared resources

---

# Understanding Important Files

## /etc/passwd

Stores user account information.

```bash
cat /etc/passwd
```

Example:

```
tokyo:x:1001:1001::/home/tokyo:/bin/bash
```

Fields:

```
username
password placeholder
UID
GID
Comment
Home Directory
Login Shell
```

Example:

```
tokyo:x:1001:1001::/home/tokyo:/bin/bash
```

Meaning

- Username → tokyo
- Password stored in shadow file
- UID → 1001
- Primary Group ID → 1001
- Home → /home/tokyo
- Default shell → /bin/bash

---

## /etc/shadow

Stores encrypted passwords.

```bash
sudo cat /etc/shadow
```

Example

```
tokyo:$6$adfjk.......
```

Normal users cannot read this file.

---

## /etc/group

Stores group information.

```bash
cat /etc/group
```

Example

```
developers:x:1005:tokyo,berlin
```

Fields

```
Group Name
Password Placeholder
GID
Members
```

---

# User Management Commands

---

## Create User

```bash
sudo useradd -m tokyo
```

Meaning

- useradd → create user
- -m → create home directory

Verify

```bash
ls /home
```

Output

```
tokyo
```

---

## Set Password

```bash
sudo passwd tokyo
```

Output

```
New password:
Retype password:
passwd: password updated successfully
```

---

## Check User Exists

Method 1

```bash
id tokyo
```

Example

```
uid=1001(tokyo)
gid=1001(tokyo)
groups=1001(tokyo)
```

Method 2

```bash
grep tokyo /etc/passwd
```

---

## Delete User

Keep home directory

```bash
sudo userdel tokyo
```

Delete with home directory

```bash
sudo userdel -r tokyo
```

---

# Group Management

---

## Create Group

```bash
sudo groupadd developers
```

Verify

```bash
grep developers /etc/group
```

---

## Add User to Group

```bash
sudo usermod -aG developers tokyo
```

Meaning

```
-a
Append

-G
Supplementary Group
```

Never forget **-a**

Wrong

```bash
sudo usermod -G developers tokyo
```

This removes all previous supplementary groups.

Correct

```bash
sudo usermod -aG developers tokyo
```

---

## Check User Groups

```bash
groups tokyo
```

Example

```
tokyo : tokyo developers
```

Or

```bash
id tokyo
```

---

## Remove User from Group

```bash
sudo gpasswd -d tokyo developers
```

---

# File Ownership

Every file has

- Owner
- Group
- Permissions

Check

```bash
ls -l
```

Example

```
-rw-r--r-- 1 tokyo developers file.txt
```

Meaning

Owner

```
tokyo
```

Group

```
developers
```

---

# Change Owner

```bash
sudo chown tokyo file.txt
```

Owner + Group

```bash
sudo chown tokyo:developers file.txt
```

---

# Change Group

```bash
sudo chgrp developers file.txt
```

---

# Directory Permissions

Check

```bash
ls -ld /opt/dev-project
```

Example

```
drwxrwxr-x
```

Breakdown

```
d

Directory

rwx

Owner

rwx

Group

r-x

Others
```

Numeric

```
775
```

Meaning

Owner

```
7
=
rwx
```

Group

```
7
=
rwx
```

Others

```
5
=
r-x
```

---

# Set Permissions

```bash
sudo chmod 775 /opt/dev-project
```

---

# Testing as Another User

Run command as another user

```bash
sudo -u tokyo touch /opt/dev-project/file1
```

Another example

```bash
sudo -u berlin touch /opt/dev-project/file2
```

List files

```bash
ls -l /opt/dev-project
```

---

# Useful Commands

Check current user

```bash
whoami
```

Current UID

```bash
id
```

Current groups

```bash
groups
```

List users

```bash
cut -d: -f1 /etc/passwd
```

List groups

```bash
cut -d: -f1 /etc/group
```

Home directories

```bash
ls /home
```

---

# Complete Challenge Solution

## Task 1

```bash
sudo useradd -m tokyo
sudo passwd tokyo

sudo useradd -m berlin
sudo passwd berlin

sudo useradd -m professor
sudo passwd professor
```

Verify

```bash
grep "tokyo\|berlin\|professor" /etc/passwd

ls /home
```

---

## Task 2

```bash
sudo groupadd developers

sudo groupadd admins
```

Verify

```bash
grep "developers\|admins" /etc/group
```

---

## Task 3

```bash
sudo usermod -aG developers tokyo

sudo usermod -aG developers,admins berlin

sudo usermod -aG admins professor
```

Verify

```bash
groups tokyo

groups berlin

groups professor
```

---

## Task 4

```bash
sudo mkdir /opt/dev-project

sudo chgrp developers /opt/dev-project

sudo chmod 775 /opt/dev-project
```

Verify

```bash
ls -ld /opt/dev-project
```

Test

```bash
sudo -u tokyo touch /opt/dev-project/tokyo.txt

sudo -u berlin touch /opt/dev-project/berlin.txt

ls -l /opt/dev-project
```

---

## Task 5

```bash
sudo useradd -m nairobi

sudo passwd nairobi

sudo groupadd project-team

sudo usermod -aG project-team nairobi

sudo usermod -aG project-team tokyo

sudo mkdir /opt/team-workspace

sudo chgrp project-team /opt/team-workspace

sudo chmod 775 /opt/team-workspace

sudo -u nairobi touch /opt/team-workspace/test.txt
```

Verify

```bash
groups nairobi

groups tokyo

ls -ld /opt/team-workspace

ls -l /opt/team-workspace
```

---

# Common Interview Questions

## Q1. Difference between useradd and adduser?

Answer

```
useradd

Low-level command
Minimal configuration

adduser

Friendly interactive wrapper around useradd
Automatically creates home directory and prompts for details.
```

---

## Q2. What does -m do?

Answer

Creates user's home directory.

---

## Q3. What is UID?

Answer

Unique User ID assigned to every user.

Root UID = 0

Normal users usually start from 1000.

---

## Q4. What is GID?

Answer

Unique Group ID.

Each user has one primary group and can belong to multiple supplementary groups.

---

## Q5. Difference between Primary and Supplementary Groups?

Primary Group

```
Used as default group for new files.
```

Supplementary Group

```
Extra groups used for shared access.
```

---

## Q6. What happens if you forget -a in usermod -aG?

Answer

All existing supplementary groups are removed and replaced.

---

## Q7. Which file stores users?

Answer

```
/etc/passwd
```

---

## Q8. Which file stores passwords?

Answer

```
/etc/shadow
```

---

## Q9. Which file stores groups?

Answer

```
/etc/group
```

---

## Q10. Difference between chown and chgrp?

Answer

```
chown

Changes owner

chgrp

Changes only group
```

---

## Q11. What does chmod 775 mean?

Answer

```
Owner

rwx

Group

rwx

Others

r-x
```

---

## Q12. How do you check which groups a user belongs to?

Answer

```bash
groups username
```

or

```bash
id username
```

---

## Q13. How do you verify a user was created?

Answer

```bash
id username
```

or

```bash
grep username /etc/passwd
```

---

## Q14. Why use groups instead of giving permissions to every user?

Answer

Groups simplify permission management by granting access to multiple users at once, making administration easier and more secure.

---

## Q15. How do you test permissions without logging into another account?

Answer

```bash
sudo -u username command
```

Example

```bash
sudo -u tokyo touch /opt/dev-project/test.txt
```

---

# Hands-on Practice Questions

## Practice 1

Create a user named `alice`.

### Answer

```bash
sudo useradd -m alice
sudo passwd alice
id alice
```

---

## Practice 2

Create a group called `testing` and add `alice`.

### Answer

```bash
sudo groupadd testing
sudo usermod -aG testing alice
groups alice
```

---

## Practice 3

Create `/opt/testing` owned by the `testing` group with `775` permissions.

### Answer

```bash
sudo mkdir /opt/testing
sudo chgrp testing /opt/testing
sudo chmod 775 /opt/testing
ls -ld /opt/testing
```

---

## Practice 4

Test if `alice` can create a file.

### Answer

```bash
sudo -u alice touch /opt/testing/file.txt
ls -l /opt/testing
```

---

## Practice 5

Remove `alice` from the `testing` group.

### Answer

```bash
sudo gpasswd -d alice testing
groups alice
```

---

## Practice 6

Delete `alice` and remove the home directory.

### Answer

```bash
sudo userdel -r alice
```

---

# Key Takeaways

- Every Linux user has a unique UID and a primary group (GID).
- User account information is stored in `/etc/passwd`, encrypted passwords in `/etc/shadow`, and group definitions in `/etc/group`.
- Use `useradd -m` to create users with home directories and `passwd` to set passwords.
- Always use `usermod -aG` when adding users to supplementary groups to avoid removing existing memberships.
- Ownership is managed with `chown` (owner and/or group) and `chgrp` (group only).
- Permissions such as `775` determine what the owner, group members, and others can do.
- Use `sudo -u <username>` to test permissions without logging in as another user.
- Group-based permissions are the standard way to provide shared access to directories in Linux.
