# Day 2 — Linux Processes and Process Lifecycle

## 1. Learning Objectives

By the end of this lesson, I should be able to:

- Understand what a process is in Linux.
- Explain PID and PPID.
- Differentiate between foreground and background processes.
- Inspect running processes.
- Find the PID of a specific process.
- Understand process termination and signals.
- Start, pause, resume, and terminate test processes safely.
- Use `ps`, `top`, `sleep`, `kill`, `jobs`, `bg`, and `fg`.

---

## 2. What Is a Process?

A **process** is an instance of a program that is currently executing.

When a program starts running, the operating system creates a process to manage its execution and resources.

Examples:

- Running a shell command.
- Executing a Python script.
- Running a Docker container's main application.
- Starting a web server.
- Running a Kubernetes node agent such as `kubelet`.

A program is the code or executable. A process is a running instance of that program.

For example, executing the same Python script twice can create two separate processes.

### Process Information

A Linux process can have:

- Process ID (PID)
- Parent Process ID (PPID)
- Execution state
- User and group identity
- CPU and memory usage
- Open files and other resources

---

## 3. What Is a PID?

**PID stands for Process ID.**

The Linux kernel assigns a process ID to identify a process.

Example:

```bash
sleep 300 &
```

Possible output:

```text
[1] 24567
```

Here:

- `1` is the shell job number.
- `24567` is the process ID in this example.

Actual values will differ on your machine.

### Find a PID

```bash
ps
```

This displays processes associated with the current terminal.

```bash
ps -ef
```

This displays a broader list of processes on the system.

```bash
ps -ef | grep sleep
```

This searches the displayed process list for entries containing `sleep`. The output may also contain the `grep` command itself.

### Find the Parent Process ID

```bash
ps -o pid,ppid,stat,comm -p <PID>
```

Replace `<PID>` with the actual process ID.

Example:

```bash
ps -o pid,ppid,stat,comm -p 24567
```

The output contains:

| Column | Meaning |
|---|---|
| PID | Process ID |
| PPID | Parent Process ID |
| STAT | Process state |
| COMMAND | Command name |

---

## 4. Foreground vs Background Processes

### Foreground Process

A foreground process runs attached to the current terminal session. The shell generally waits for the command to finish before presenting the next prompt.

Example:

```bash
sleep 30
```

The command occupies the terminal for 30 seconds.

### Background Process

A background job allows the shell to accept additional commands while the job continues running.

Example:

```bash
sleep 300 &
```

The `&` symbol instructs the shell to start the command asynchronously as a background job.

### Comparison

| Feature | Foreground | Background |
|---|---|---|
| Terminal prompt | Usually unavailable until command finishes | Available after command starts |
| How to start | `sleep 30` | `sleep 300 &` |
| Shell job management | Not initially a background job | Managed through shell job control |
| Inspect process | `ps`, `top` | `ps`, `top` |
| Stop temporarily | `Ctrl+Z` | `kill -STOP <PID>` or job control |
| Resume | `fg` after suspension | `bg` for a stopped shell job |

**Important:** A background job is not necessarily independent of the terminal. Its behavior when the terminal closes depends on the shell, signals, and how the process was started.

---

## 5. Understanding the `sleep` Command

The `sleep` command pauses execution for a specified duration.

### Examples

```bash
sleep 10
```

Waits for 10 seconds.

```bash
sleep 60
```

Waits for 60 seconds.

```bash
sleep 300 &
```

Starts a 300-second sleep as a background job.

### Why Use `sleep` for Practice?

It creates a predictable, low-resource test process without changing system configuration or performing destructive operations.

It is useful for practicing process identification, job control, and termination.

---

## 6. Inspecting Processes with `ps`

The `ps` command displays information about processes.

### Common Commands

```bash
ps
```

Displays processes associated with the current terminal.

```bash
ps -ef
```

Displays processes across the system in full-format listing.

```bash
ps aux
```

Displays processes with user, CPU, memory, and other information.

```bash
ps -C sleep -o pid,ppid,stat,comm
```

Lists processes whose command name is `sleep`, with selected columns.

```bash
ps -p <PID> -o pid,ppid,stat,%cpu,%mem,comm
```

Displays selected details for a specific PID.

### Important Options

- `-e`: Select all processes.
- `-f`: Full-format listing.
- `-p`: Select a process by PID.
- `-o`: Choose output columns.
- `-C`: Select processes by command name.

---

## 7. Monitoring Processes with `top`

The `top` command provides a continuously updated view of system processes and resource usage.

### Start `top`

```bash
top
```

Observe:

- CPU utilization.
- Memory utilization.
- Process IDs.
- Process states.
- CPU and memory usage by process.

Press `q` to quit `top`.

### Find a Specific Process

While `top` is running, inspect the process list and look for the PID of the test process.

For detailed information about a specific process, use `ps` as well.

**Difference:** `ps` gives a snapshot of process information. `top` provides a continuously refreshing view.

---

## 8. Process States

The `STAT` column in `ps` provides information about process state.

| State | Meaning |
|---|---|
| `R` | Running or runnable |
| `S` | Interruptible sleep |
| `D` | Uninterruptible sleep, often waiting for I/O |
| `T` | Stopped or suspended |
| `Z` | Zombie process |

A sleeping process is not necessarily unhealthy. For example, `sleep 300` normally waits without consuming significant CPU.

A zombie process is a process that has terminated but whose exit status has not yet been collected by its parent.

---

## 9. Understanding Process Termination

Terminating a process means requesting or causing the process to stop executing.

Linux commonly uses signals to communicate with processes.

### `kill` Command

Despite its name, `kill` does not always forcefully terminate a process. It sends a signal to the specified process.

### Graceful Termination — SIGTERM

```bash
kill <PID>
```

By default, `kill` sends `SIGTERM` (signal 15).

This requests that the process terminate. The process can handle the signal, perform cleanup, or ignore it if permitted.

### Forceful Termination — SIGKILL

```bash
kill -9 <PID>
```

This sends `SIGKILL` (signal 9).

The process cannot catch or ignore this signal, and the kernel terminates it. Cleanup handlers inside the process cannot run.

Use `SIGKILL` only when necessary, because it prevents normal application cleanup.

### Check Before and After Termination

```bash
ps -p <PID> -o pid,stat,comm
```

After termination, rerun the command. If the process no longer exists, `ps` normally displays no process entry.

**Safety rule:** Only terminate processes you own or are authorized to manage. Never experiment with `kill -9` on random system processes.

---

## 10. Shell Job Control — `jobs`, `bg`, and `fg`

These commands manage jobs associated with the current shell.

### `jobs`

Lists jobs known to the current shell.

```bash
sleep 300 &
jobs
```

Example output:

```text
[1]+ Running    sleep 300 &
```

The job number is `[1]`. It is not the process ID.

### `Ctrl+Z`

When a foreground command is running, pressing `Ctrl+Z` normally suspends it by sending a stop signal through the terminal's job-control mechanism.

Example:

```bash
sleep 300
```

Press `Ctrl+Z`, then run:

```bash
jobs
```

The job should appear as stopped.

### `bg`

Resumes a stopped job in the background.

```bash
bg %1
```

Here `%1` refers to shell job number 1.

### `fg`

Brings a shell job into the foreground.

```bash
fg %1
```

The terminal waits for that job again.

Press `Ctrl+C` to request termination of the foreground command.

### Important Distinction

- `PID`: Identifies a process.
- `%1`: Refers to job number 1 in the current shell.
- `bg` and `fg` operate on shell jobs, not arbitrary PIDs.

---

## 11. Practical Lab — Start, Inspect, and Stop a Process

### Step 1: Start a Background Process

In Terminal 1:

```bash
sleep 300 &
```

Record the PID shown by your terminal.

You can also run:

```bash
echo $!
```

`$!` contains the PID of the most recently started background process in the current shell.

### Step 2: Inspect the Process from Another Terminal

Open Terminal 2 and run:

```bash
ps -ef | grep '[s]leep 300'
```

This searches for the test process without matching the `grep` command itself.

Record the PID.

### Step 3: Inspect Process Details

Replace `<PID>` with the actual PID:

```bash
ps -p <PID> -o pid,ppid,stat,%cpu,%mem,comm
```

Observe the PID, parent PID, state, CPU usage, memory usage, and command name.

### Step 4: Monitor the Process

```bash
top
```

Find the test process in the list.

Press `q` to exit.

### Step 5: Terminate the Test Process Safely

From Terminal 1 or Terminal 2, run:

```bash
kill <PID>
```

This requests graceful termination.

Verify:

```bash
ps -p <PID> -o pid,stat,comm
```

If the process has terminated, it should no longer appear.

### Step 6: Compare Foreground and Background Execution

Run the following in Terminal 1:

```bash
sleep 30
```

Notice that the terminal prompt is unavailable while the command runs.

Wait for completion or press `Ctrl+C` to interrupt it.

Now run:

```bash
sleep 300 &
jobs
```

The shell prompt returns while the background job is running.

### Step 7: Practice Job Control

Start a foreground process:

```bash
sleep 300
```

Press `Ctrl+Z`.

Then execute:

```bash
jobs
bg
jobs
fg
```

The job resumes in the background and is then brought into the foreground. Press `Ctrl+C` to terminate the test process.

Finally, verify that no test process remains:

```bash
ps -ef | grep '[s]leep 300'
```

If the output is empty, the test process is no longer running.

---

## 12. Troubleshooting

### Problem: `kill` says "No such process"

Possible causes:

- The process already terminated.
- The PID was copied incorrectly.
- The PID belongs to a different process now.

Recheck the PID with `ps`.

### Problem: `jobs` Does Not Show the Process

`jobs` only shows jobs managed by the current shell. A process started in another terminal generally will not appear in your current shell's job list.

Use `ps` to inspect system processes.

### Problem: `kill` Says "Operation not permitted"

You may not own the process or have sufficient permissions. Do not automatically use `sudo`; first verify the PID, process owner, and reason you need to terminate it.

### Problem: The Process Is Still Present After `kill`

The process may be handling or delaying its response to `SIGTERM`, or the PID may refer to a different process than expected.

Inspect the process again before taking further action. Do not immediately force-kill an unfamiliar process.

---

## 13. Quick Command Reference

| Command | Purpose |
|---|---|
| `ps` | Inspect processes associated with the terminal |
| `ps -ef` | List processes across the system |
| `ps aux` | Show detailed process and resource information |
| `ps -p PID` | Inspect a particular PID |
| `top` | Monitor processes and resource usage |
| `sleep 30` | Run a timed foreground process |
| `sleep 300 &` | Start a timed background job |
| `echo $!` | Print the most recent background process PID |
| `kill PID` | Send SIGTERM by default |
| `kill -9 PID` | Send SIGKILL |
| `jobs` | List current shell jobs |
| `bg %1` | Resume job 1 in the background |
| `fg %1` | Bring job 1 into the foreground |
| `Ctrl+C` | Interrupt a foreground command |
| `Ctrl+Z` | Suspend a foreground job |

---

## 14. Interview Questions

### Q1. What is a process?

A process is an executing instance of a program managed by the operating system.

### Q2. What is a PID?

A PID is the unique process identifier assigned to a process at a given time.

### Q3. What is a PPID?

PPID stands for Parent Process ID. It identifies the parent process associated with a process.

### Q4. What is the difference between a foreground and a background process?

A foreground process occupies the terminal's command interaction until it finishes or is interrupted. A background job allows the shell to accept more commands while the job runs.

### Q5. What is the difference between `ps` and `top`?

`ps` provides a snapshot of process information, while `top` continuously refreshes process and resource information.

### Q6. What does `kill PID` do?

By default, it sends `SIGTERM` to request graceful termination.

### Q7. What is the difference between SIGTERM and SIGKILL?

`SIGTERM` can be handled by the process, allowing cleanup. `SIGKILL` cannot be caught or ignored and forces termination.

### Q8. What is the difference between a PID and a job number?

A PID identifies a process at the operating-system level. A job number identifies a job managed by the current shell.

### Q9. What does `Ctrl+Z` do?

It normally suspends a foreground job through the shell's job-control mechanism.

### Q10. What is the difference between `bg` and `fg`?

`bg` resumes a stopped shell job in the background. `fg` brings a shell job into the foreground.

---

## 15. Day 2 Deliverables

- [ ] Explain what a process and PID are.
- [ ] Start a `sleep` process.
- [ ] Find the process PID.
- [ ] Inspect the process with `ps`.
- [ ] Observe process information using `top`.
- [ ] Terminate the test process using `kill`.
- [ ] Compare foreground and background execution.
- [ ] Practice `jobs`, `bg`, and `fg`.
- [ ] Verify that the test process has terminated.

---

## 16. Key Takeaways

1. A process is a running instance of a program.
2. PID identifies a process; PPID identifies its parent.
3. `ps` provides process information, while `top` monitors it continuously.
4. `&` starts a shell command as a background job.
5. `kill` sends signals; it does not always force termination.
6. `SIGTERM` allows graceful termination, while `SIGKILL` forces termination.
7. `jobs`, `bg`, and `fg` manage jobs in the current shell.
8. Always verify the PID and process ownership before terminating a process.

**Day 2 Goal:** Identify a running process, find its PID, inspect its details, and safely stop it.
