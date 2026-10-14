# Day 3 — Linux Users, Groups and Permissions

## 1. Learning Objectives

By the end of this lesson, I should be able to:

- Understand root and normal users.
- Identify users, UIDs, groups, and GIDs.
- Understand primary and supplementary groups.
- Interpret Linux file permissions.
- Change permissions using `chmod`.
- Inspect file ownership and understand `chown`.
- Understand `sudo` and the principle of least privilege.
- Diagnose basic permission-denied errors.
- Create and execute a shell script.

---

## 2. Root User vs Normal User

Linux uses user identities to determine which operations a process can perform.

### Root User

The root user is the superuser account in Linux.

- Username: `root`
- UID: `0`
- Has extensive administrative privileges.
- Can manage system configuration, users, permissions, and protected resources.

### Normal User

A normal user has access to resources permitted by file permissions, group memberships, and other access-control rules.

For example, a regular user can usually manage files inside their own home directory but may not be allowed to modify protected system configuration files.

### Check the Current User

```bash
whoami
```

Displays the current effective username.

```bash
id
```

Displays the effective UID, primary GID, and supplementary group memberships.

Example output:

```text
uid=1000(sagar) gid=1000(sagar) groups=1000(sagar),27(sudo)
```

The values above are illustrative. Actual IDs and groups depend on the system.

### Important Distinction

The root account and `sudo` are not the same thing.

- `root` is a user account with UID `0`.
- `sudo` allows authorized users to execute permitted commands with elevated privileges.

Example:

```bash
whoami
sudo whoami
```

The first command displays the current effective username. If authorized, the second will typically display `root`.

Using `sudo` for one command does not permanently change the current shell into a root shell.

---

## 3. Users and Groups

Linux assigns numerical identifiers to users and groups.

| Term | Meaning |
|---|---|
| UID | User ID |
| GID | Group ID |
| Primary group | The user's main group |
| Supplementary groups | Additional groups the user belongs to |

Groups simplify permission management by allowing multiple users to share access to resources.

For example, a `developers` group can be assigned access to a shared project directory.

### Commands

```bash
whoami
id
groups
```

- `whoami`: Displays the effective username.
- `id`: Displays user and group identifiers.
- `groups`: Displays group memberships.

To inspect another user's groups:

```bash
groups username
```

Replace `username` with the actual username.

### Primary and Supplementary Groups

The primary group is commonly used as the default group owner for newly created files, subject to directory settings and system configuration.

Supplementary groups provide additional group memberships that may grant access to shared resources.

**Important:** Membership in a group does not automatically grant access to every file. The applicable permissions and access-control rules must also allow access.

---

## 4. Understanding Linux File Permissions

Linux permissions control what users can do with files and directories.

The three basic permissions are:

| Permission | Symbol | For files | For directories |
|---|---|---|---|
| Read | `r` | Read file contents | List directory entries |
| Write | `w` | Modify file contents | Create, delete, or rename entries, subject to other restrictions |
| Execute | `x` | Execute a program or script | Traverse the directory |

Directory permissions have different meanings from file permissions.

For example, a user may have permission to read a file but still be unable to access it if they cannot traverse one of its parent directories.

### Inspect Permissions with `ls -l`

```bash
ls -l
```

Example output:

```text
-rwxr-xr-- 1 sagar developers 120 Oct 14 10:00 script.sh
```

The permission string is:

```text
-rwxr-xr--
```

It can be divided into four sections:

| Characters | Meaning |
|---|---|
| `-` | Regular file |
| `rwx` | Owner permissions |
| `r-x` | Group permissions |
| `r--` | Others' permissions |

The three permission groups are:

- **Owner:** The user who owns the file.
- **Group:** Members of the file's owning group, subject to applicable access rules.
- **Others:** Users who are neither the owner nor members of the owning group for this permission check.

The first character indicates the file type. For example, `d` generally indicates a directory and `-` indicates a regular file.

---

## 5. Numeric Permissions

Linux supports numeric, or octal, permission notation.

Each basic permission has a numeric value:

| Permission | Value |
|---|---:|
| Read (`r`) | 4 |
| Write (`w`) | 2 |
| Execute (`x`) | 1 |

The values are added together for each permission group.

| Numeric value | Permission | Meaning |
|---|---|---|
| `7` | `rwx` | Read, write, execute |
| `6` | `rw-` | Read and write |
| `5` | `r-x` | Read and execute |
| `4` | `r--` | Read only |
| `0` | `---` | No permissions |

### Example: `chmod 754`

```bash
chmod 754 script.sh
```

This sets the following permissions:

- Owner: `7` = `rwx`
- Group: `5` = `r-x`
- Others: `4` = `r--`

The resulting permission string is:

```text
-rwxr-xr--
```

### Common Permission Modes

| Mode | Permissions | Typical use |
|---|---|---|
| `600` | `rw-------` | Private files |
| `644` | `rw-r--r--` | Common non-sensitive text files |
| `700` | `rwx------` | Private executable scripts or directories |
| `755` | `rwxr-xr-x` | Executable scripts and commonly accessible directories |
| `750` | `rwxr-x---` | Owner and group access |
| `640` | `rw-r-----` | Owner write access and group read access |

Choose permissions according to the resource's purpose and security requirements. These modes are examples, not universal defaults.

---

## 6. Changing Permissions with `chmod`

`chmod` stands for change mode. It changes the permissions of files and directories.

### Numeric Mode

Create a test file:

```bash
touch demo.txt
```

Set permissions:

```bash
chmod 600 demo.txt
ls -l demo.txt
```

Expected permission string:

```text
-rw-------
```

Now change the permissions:

```bash
chmod 644 demo.txt
ls -l demo.txt
```

Expected permission string:

```text
-rw-r--r--
```

### Symbolic Mode

Symbolic mode uses letters to specify the target and permission changes.

| Symbol | Meaning |
|---|---|
| `u` | Owner/user |
| `g` | Group |
| `o` | Others |
| `a` | All permission classes |
| `+` | Add permissions |
| `-` | Remove permissions |
| `=` | Set the specified permissions exactly |

Examples:

```bash
chmod u+x script.sh
```

Adds execute permission for the owner.

```bash
chmod g-w demo.txt
```

Removes write permission from the group.

```bash
chmod o-r demo.txt
```

Removes read permission from others.

```bash
chmod u=rw,g=r,o= demo.txt
```

Sets the owner to read/write, the group to read-only, and others to no permissions.

### Verify Changes

```bash
ls -l demo.txt
```

Always inspect the result after modifying permissions.

---

## 7. File Ownership and `chown`

Every file and directory has an owner and an owning group.

Example:

```text
-rw-r--r-- 1 sagar developers 120 Oct 14 10:00 notes.txt
```

Here:

- `sagar` is the owner.
- `developers` is the owning group.

Inspect ownership:

```bash
ls -l notes.txt
```

### Change Ownership

The `chown` command changes file ownership.

```bash
sudo chown username:groupname notes.txt
```

Replace `username` and `groupname` with valid account and group names.

To change only the owner:

```bash
sudo chown username notes.txt
```

To change only the group, use:

```bash
chgrp groupname notes.txt
```

Changing ownership often requires elevated privileges.

**Safety note:** Do not change ownership of system files or recursively change ownership of large directory trees without understanding the consequences. For today's lab, inspecting ownership is sufficient.

### `chmod` vs `chown`

| Command | Purpose |
|---|---|
| `chmod` | Changes permissions |
| `chown` | Changes the owner and optionally the group |
| `chgrp` | Changes the owning group |

---

## 8. Understanding `sudo`

`sudo` allows an authorized user to execute commands with elevated privileges according to the system's configuration.

Example:

```bash
sudo whoami
```

On a typical system, if the user is authorized, the output is:

```text
root
```

Check the current effective username separately:

```bash
whoami
```

### Best Practices

- Use elevated privileges only when required.
- Verify a command before running it with `sudo`.
- Avoid running applications as root unnecessarily.
- Never use `sudo` merely to bypass an unfamiliar permission error.
- Follow your organization's access-control policies.

Not every Linux installation configures `sudo` in the same way. Some systems use other administrative access mechanisms.

---

## 9. Permission Denied Errors

A `Permission denied` error means an operation is not permitted under the applicable access-control rules.

Common causes include:

1. The current user lacks the required file permission.
2. The user cannot traverse a parent directory.
3. The file is not executable when direct execution is attempted.
4. The file or directory has restrictive ownership or group permissions.
5. Additional access-control mechanisms restrict access.

### Example: Restrict File Access

```bash
touch secret.txt
chmod 600 secret.txt
ls -l secret.txt
```

Expected permissions:

```text
-rw-------
```

Only the owner receives read and write permissions through these basic permission bits.

A different user may receive `Permission denied` when trying to read the file.

### Restore Read Access for the Owner and Others

```bash
chmod 644 secret.txt
ls -l secret.txt
```

Expected permissions:

```text
-rw-r--r--
```

**Important:** Privileged access, ACLs, capabilities, security policies, and other mechanisms may affect the final result. Basic permission bits are only one part of Linux access control.

---

## 10. The Principle of Least Privilege

The principle of least privilege means granting a user, process, or service only the permissions necessary to perform its intended tasks.

### Examples in DevOps

- Run applications as non-root users whenever practical.
- Restrict access to configuration files and secrets.
- Avoid granting unnecessary administrative privileges to CI/CD runners.
- Limit Kubernetes service account permissions.
- Give deployment automation access only to the resources it needs.
- Avoid broad, unnecessary permission settings such as `chmod 777`.

### Why It Matters

Excessive privileges increase the impact of compromised accounts, vulnerable applications, and accidental commands.

Least privilege reduces the potential damage from these events.

---

## 11. Practical Lab — Users, Groups, and Permissions

Perform the following exercises inside your own home directory. Do not modify system files or files belonging to other users.

### Step 1: Inspect the Current User

```bash
whoami
id
groups
```

Identify your effective username, UID, primary GID, and supplementary groups.

### Step 2: Create the Lab Directory

```bash
mkdir -p ~/linux-devops-labs/day-03
cd ~/linux-devops-labs/day-03
```

### Step 3: Create and Inspect a File

```bash
touch demo.txt
ls -l demo.txt
```

Observe the default permissions, owner, and group.

Default permissions depend on the system's configuration and `umask`.

### Step 4: Change Permissions

```bash
chmod 600 demo.txt
ls -l demo.txt
```

Expected permissions:

```text
-rw-------
```

Now run:

```bash
chmod 644 demo.txt
ls -l demo.txt
```

Expected permissions:

```text
-rw-r--r--
```

### Step 5: Observe a Permission Error

```bash
chmod 000 demo.txt
ls -l demo.txt
cat demo.txt
```

The file has no read, write, or execute permissions in its basic permission bits.

A normal, unprivileged user will generally be unable to read it.

Restore the owner's permissions immediately:

```bash
chmod 644 demo.txt
cat demo.txt
```

The file may be empty, but the read operation should no longer fail because of its basic file permission bits.

### Step 6: Create an Executable Script

Create the script:

```bash
cat > hello.sh <<'EOF'
#!/bin/sh
echo "Hello from Linux Day 3"
EOF
```

Inspect its permissions:

```bash
ls -l hello.sh
```

Add execute permission for the owner:

```bash
chmod u+x hello.sh
```

Execute the script:

```bash
./hello.sh
```

Expected output:

```text
Hello from Linux Day 3
```

The `./` prefix specifies a path relative to the current directory.

### Step 7: Inspect Ownership

```bash
ls -l demo.txt hello.sh
id
```

Identify the owner and owning group of each file.

### Step 8: Verify the Final State

```bash
ls -l
```

Confirm that the test files have the expected permissions and that the script executes successfully.

---

## 12. Troubleshooting

### Error: `Permission denied`

Check:

```bash
whoami
id
ls -l filename
```

Verify the current user, group memberships, file permissions, and parent directory permissions.

### Script Does Not Execute

Check the script's permissions:

```bash
ls -l hello.sh
```

If appropriate, add execute permission:

```bash
chmod u+x hello.sh
```

Then run:

```bash
./hello.sh
```

### `sudo` Is Not Available or Permission Is Denied

The system may not have `sudo` installed, or the current user may not be authorized to use it.

Do not attempt to bypass administrative restrictions. Follow the system administrator's access procedure.

### Ownership Change Fails

Inspect the existing ownership:

```bash
ls -l filename
```

Changing ownership may require administrative privileges. Confirm the intended owner and group before attempting a change.

---

## 13. Quick Command Reference

| Command | Purpose |
|---|---|
| `whoami` | Display the effective username |
| `id` | Display user and group identifiers |
| `groups` | Display group memberships |
| `ls -l` | Inspect permissions and ownership |
| `chmod 644 file` | Set permissions to `rw-r--r--` |
| `chmod 600 file` | Set permissions to `rw-------` |
| `chmod u+x file` | Add execute permission for the owner |
| `chmod g-w file` | Remove group write permission |
| `chown user file` | Change the owner |
| `chown user:group file` | Change the owner and group |
| `chgrp group file` | Change the owning group |
| `sudo command` | Run an authorized command with elevated privileges |

---

## 14. Interview Questions

### Q1. What is the difference between root and a normal user?

Root is the superuser account with UID `0`. A normal user has permissions determined by ownership, group membership, and other access-control rules.

### Q2. What is a UID?

UID stands for User ID, a numerical identifier associated with a user account.

### Q3. What is the difference between a primary group and supplementary groups?

The primary group is the user's main group. Supplementary groups provide additional group memberships that may grant access to shared resources.

### Q4. What does `rwxr-xr--` mean?

The owner has read, write, and execute permissions; the group has read and execute permissions; others have read permission.

### Q5. What does `chmod 755 script.sh` do?

It grants `rwx` to the owner and `r-x` to both the group and others.

### Q6. What is the difference between `chmod` and `chown`?

`chmod` changes permission bits, while `chown` changes ownership.

### Q7. Why might a user receive `Permission denied` even if a file is readable?

The user may lack permission to traverse a parent directory, or another applicable access-control mechanism may deny access.

### Q8. Why should `chmod 777` generally be avoided?

It grants read, write, and execute permissions to everyone through the basic permission bits, potentially allowing unauthorized modifications or execution.

### Q9. What is `sudo`?

`sudo` allows authorized users to execute permitted commands with elevated privileges.

### Q10. What is the principle of least privilege?

It is the practice of granting only the permissions required to perform a task.

---

## 15. Day 3 Deliverables

- [ ] Identify the current user using `whoami`.
- [ ] Inspect UID, GID, and groups using `id` and `groups`.
- [ ] Create a test file and inspect permissions using `ls -l`.
- [ ] Change permissions using numeric and symbolic modes.
- [ ] Observe a permission-denied scenario and restore the permissions.
- [ ] Create and execute a shell script.
- [ ] Inspect file ownership.
- [ ] Explain the difference between `chmod`, `chown`, and `sudo`.
- [ ] Explain the principle of least privilege.

---

## 16. Key Takeaways

1. Linux uses users and groups to manage access to resources.
2. Root has extensive administrative privileges; normal users have more restricted access.
3. The basic permissions are read (`r`), write (`w`), and execute (`x`).
4. Permissions are applied separately to the owner, group, and others.
5. `chmod` changes permissions; `chown` changes ownership.
6. Directory traversal permissions affect access to files inside directories.
7. `sudo` should be used only when necessary and authorized.
8. Least privilege is a fundamental security practice in Linux and DevOps.

**Day 3 Goal:** Understand Linux users, groups, ownership, and permissions; modify permissions safely; and explain why access may be denied.