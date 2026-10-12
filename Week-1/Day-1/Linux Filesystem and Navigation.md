# Day 1 — Linux Filesystem and Navigation

## 1. Learning Objectives

By the end of this lesson, I should be able to:

- Understand the Linux filesystem hierarchy.
- Explain the difference between absolute and relative paths.
- Identify the current working directory.
- Differentiate between files and directories.
- Create directories and files using Linux commands.
- Navigate between directories.
- Search for files and directories using `find`.
- Create and maintain Linux command notes.

---

## 2. Linux Filesystem Hierarchy

Linux uses a hierarchical filesystem structure that starts from the root directory `/`.

Unlike Windows, where drives such as `C:` and `D:` are commonly used, Linux organizes its filesystem under a single root directory.

### Important Directories

| Directory | Purpose |
|---|---|
| `/` | Root of the entire filesystem |
| `/home` | Contains home directories of regular users |
| `/root` | Home directory of the root user |
| `/etc` | System-wide configuration files |
| `/var` | Variable data such as logs and caches |
| `/tmp` | Temporary files |
| `/usr` | Programs, libraries, and shared resources |
| `/bin` | Essential user commands; may be linked to `/usr/bin` |
| `/sbin` | System administration commands; may be linked to `/usr/sbin` |
| `/opt` | Optional application software |
| `/dev` | Device files |
| `/proc` | Virtual filesystem exposing process and kernel information |
| `/sys` | Kernel and device information |

### Important Note

`/` and `/root` are different:

- `/` is the root of the entire filesystem.
- `/root` is the home directory of the root user.

---

## 3. Files vs Directories

### File

A file stores data, text, configuration, scripts, or other content.

Examples:

- `commands.txt`
- `app.py`
- `config.yaml`

### Directory

A directory organizes files and other directories.

Examples:

- `day-01/`
- `notes/`
- `scripts/`

Linux commands can operate on both files and directories, depending on the command and its options.

---

## 4. Current Working Directory — `pwd`

The current working directory is the directory in which the shell is currently operating.

### Command

```bash
pwd
```

Example output:

```text
/home/sagar
```

This indicates that the current working directory is `/home/sagar`.

### Use Cases

- Identify the current location in the filesystem.
- Verify whether a directory change was successful.
- Avoid executing file operations in the wrong directory.

---

## 5. Listing Directory Contents — `ls`

The `ls` command lists files and directories.

### Common Options

```bash
ls
```

Lists directory contents.

```bash
ls -l
```

Displays detailed information such as permissions, ownership, file size, and modification time.

```bash
ls -a
```

Includes hidden entries, whose names commonly start with a dot (`.`).

```bash
ls -la
```

Displays detailed information, including hidden entries.

```bash
ls /etc
```

Lists the contents of `/etc` without requiring a directory change.

---

## 6. Changing Directories — `cd`

The `cd` command changes the current working directory.

### Examples

```bash
cd /tmp
```

Changes to `/tmp`.

```bash
cd ..
```

Moves to the parent directory.

```bash
cd .
```

Refers to the current directory.

```bash
cd ~
```

Changes to the current user's home directory.

```bash
cd -
```

Returns to the previous working directory.

### Verify Directory Changes

```bash
cd /tmp
pwd
```

Always use `pwd` when you need to confirm your current location.

---

## 7. Absolute Path vs Relative Path

### Absolute Path

An absolute path starts from the root directory `/` and identifies a location independently of the current working directory.

Example:

```text
/home/sagar/linux-devops-labs/day-01/commands.txt
```

Another example:

```bash
ls /etc/hosts
```

### Relative Path

A relative path identifies a location from the current working directory.

Suppose the current directory is:

```text
/home/sagar/linux-devops-labs
```

The following is a relative path:

```bash
ls day-01/commands.txt
```

### Path Symbols

| Symbol | Meaning |
|---|---|
| `/` | Root directory when used at the beginning of a path |
| `.` | Current directory |
| `..` | Parent directory |
| `~` | Current user's home directory |
| `$HOME` | Environment variable containing the current user's home directory |

### Practical Comparison

```bash
cd ~/linux-devops-labs
ls day-01/commands.txt
ls "$HOME/linux-devops-labs/day-01/commands.txt"
```

The first `ls` uses a relative path. The second uses an absolute path resolved through `$HOME`.

Both commands refer to the same file if the example directory structure exists under the current user's home directory.

**Key difference:** An absolute path starts from `/`. A relative path starts from the current working directory.

---

## 8. Creating Directories — `mkdir`

The `mkdir` command creates directories.

### Create a Single Directory

```bash
mkdir test
```

Creates a directory named `test` in the current working directory.

### Create Multiple Directories

```bash
mkdir day-01 day-02 notes
```

Creates three directories in the current working directory.

### Create Nested Directories

```bash
mkdir -p a/b/c
```

Creates the nested directory structure, including missing parent directories.

The `-p` option also prevents an error when the requested directories already exist.

---

## 9. Creating Files — `touch`

The `touch` command can create an empty file if it does not exist.

```bash
touch commands.txt
```

Create files in specific directories:

```bash
touch day-01/commands.txt
touch day-02/practice.txt
touch notes/linux-notes.txt
```

### Important Note

If a file already exists, `touch` normally updates its access and modification timestamps. It does not erase the file's contents.

---

## 10. Searching for Files and Directories — `find`

The `find` command searches for files and directories within a specified starting location.

### Search by Name

```bash
find . -name "commands.txt"
```

Searches recursively from the current directory for entries named `commands.txt`.

### Find All Directories

```bash
find . -type d
```

Lists directories under the current directory, including the starting directory itself.

### Find All Regular Files

```bash
find . -type f
```

Lists regular files under the current directory.

### Search in a Specific Directory

```bash
find ~/linux-devops-labs -name "*.txt"
```

Searches for names ending in `.txt` under the specified directory.

### Important Note

The `.` in these commands means that the search starts from the current working directory.

---

## 11. Practical Lab — Build a Linux Filesystem Structure

### Step 1: Navigate to the Home Directory

```bash
cd ~
pwd
```

Verify that the terminal is in the current user's home directory.

### Step 2: Create the Main Lab Directory

```bash
mkdir -p linux-devops-labs
cd linux-devops-labs
```

### Step 3: Create the Required Directories

```bash
mkdir -p day-01 day-02 notes
```

### Step 4: Create Files

```bash
touch day-01/commands.txt
touch day-02/practice.txt
touch notes/linux-notes.txt
```

### Step 5: Verify the Structure

```bash
pwd
ls -la
find . -type d
find . -type f
```

Expected structure:

```text
linux-devops-labs/
├── day-01/
│   └── commands.txt
├── day-02/
│   └── practice.txt
└── notes/
    └── linux-notes.txt
```

### Step 6: Practice Navigation

```bash
cd day-01
pwd
ls

cd ../day-02
pwd
ls

cd ../notes
pwd
ls

cd ..
pwd
```

Observe how the working directory changes after every `cd` command.

### Step 7: Access the Same File Using Different Paths

From the `linux-devops-labs` directory:

```bash
ls day-01/commands.txt
ls "$HOME/linux-devops-labs/day-01/commands.txt"
```

Verify that both commands access the same file.

### Step 8: Add Content to the Notes File

From the `linux-devops-labs` directory:

```bash
cat > notes/linux-notes.txt <<'EOF'
pwd - Print working directory
ls - List directory contents
cd - Change directory
mkdir - Create directories
touch - Create files or update timestamps
find - Search for files and directories
EOF
```

Display the contents:

```bash
cat notes/linux-notes.txt
```

**Warning:** The `>` operator overwrites the destination file's existing contents. Use `>>` to append instead.

---

## 12. Quick Command Reference

| Command | Purpose |
|---|---|
| `pwd` | Print current working directory |
| `ls` | List directory contents |
| `ls -l` | Show detailed listing |
| `ls -a` | Include hidden entries |
| `ls -la` | Detailed listing including hidden entries |
| `cd /path` | Change to a specified directory |
| `cd ..` | Move to the parent directory |
| `cd ~` | Move to the home directory |
| `cd -` | Return to the previous directory |
| `mkdir name` | Create a directory |
| `mkdir -p a/b` | Create nested directories |
| `touch file.txt` | Create a file or update timestamps |
| `find . -name "file.txt"` | Search by name |
| `find . -type f` | Find regular files |
| `find . -type d` | Find directories |
| `cat file.txt` | Display file contents |

---

## 13. Interview Questions

### Q1. What is the Linux root directory?

The root directory `/` is the top-level directory of the Linux filesystem hierarchy.

### Q2. What is the difference between `/` and `/root`?

`/` is the root of the filesystem, whereas `/root` is the home directory of the root user.

### Q3. What does `pwd` do?

It prints the absolute path of the current working directory.

### Q4. What is the difference between absolute and relative paths?

An absolute path starts from `/`, while a relative path is interpreted from the current working directory.

### Q5. What is the difference between `mkdir` and `touch`?

`mkdir` creates directories. `touch` creates an empty file if it does not exist or updates timestamps if it does.

### Q6. What is the purpose of `mkdir -p`?

It creates missing parent directories and avoids errors when the requested directories already exist.

### Q7. How do you find all `.yaml` files under the current directory?

```bash
find . -type f -name "*.yaml"
```

### Q8. How do you list hidden files?

```bash
ls -a
```

### Q9. What does `cd ..` do?

It changes the current working directory to its parent directory.

### Q10. How do you verify whether a file exists?

```bash
ls -l path/to/file
```

Alternatively:

```bash
test -f path/to/file && echo "File exists"
```

---

## 14. Day 1 Deliverables

- [ ] Understand the Linux filesystem hierarchy.
- [ ] Practice `pwd`, `ls`, and `cd`.
- [ ] Create the `linux-devops-labs` directory structure.
- [ ] Create files using `touch`.
- [ ] Search for files and directories using `find`.
- [ ] Access the same file using absolute and relative paths.
- [ ] Save command notes in `notes/linux-notes.txt`.
- [ ] Answer the interview questions without looking at the notes.

---

## 15. Key Takeaways

1. Linux filesystem navigation starts from a hierarchical directory structure rooted at `/`.
2. `pwd` identifies the current working directory.
3. `ls` lists directory contents.
4. `cd` changes the current working directory.
5. Absolute paths start from `/`; relative paths depend on the current working directory.
6. `mkdir` creates directories, and `touch` creates files when they do not exist.
7. `find` searches recursively from a specified starting location.
8. Always verify your location and the result of filesystem operations before moving on.

**Day 1 Goal:** Become comfortable navigating the Linux filesystem and managing basic files and directories from the terminal.