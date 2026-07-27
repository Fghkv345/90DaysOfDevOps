# Linux Architecture Notes

## 1. Core Components of Linux

### Kernel
- The core of the operating system.
- Manages:
  - CPU scheduling
  - Memory management
  - Device drivers
  - Filesystems
  - Networking
- Acts as a bridge between hardware and applications.

### User Space
- Where user applications run.
- Examples:
  - Bash shell
  - Web servers
  - Databases
  - Text editors
- Applications interact with the kernel using system calls.

### Init / systemd
- The first process started by the kernel.
- Runs as PID 1.
- Responsible for starting and managing system services.

---

## 2. Process Creation and Management

### What is a Process?
- A running instance of a program.
- Every process has a unique Process ID (PID).

### Process Lifecycle
1. Program is started.
2. Kernel creates a process.
3. Process runs and uses system resources.
4. Process exits or is terminated.

### Common Process States
- **Running (R)** – Currently executing on CPU.
- **Sleeping (S)** – Waiting for an event or resource.
- **Stopped (T)** – Paused by a signal.
- **Zombie (Z)** – Finished execution but waiting for parent process to collect exit status.
- **Uninterruptible Sleep (D)** – Waiting for I/O operations.

---

## 3. What systemd Does

### Purpose
- Modern init system used by most Linux distributions.

### Responsibilities
- Starts services during boot.
- Manages service lifecycles.
- Handles logging through `journald`.
- Tracks system state and dependencies.
- Enables automatic service restart.

### Why It Matters
- Faster boot process.
- Centralized service management.
- Easier troubleshooting and monitoring.

---

## 4. Daily Linux Commands

Linux commands fall into six main categories:

1.Filesystem navigation commands that move between directories and paths
2.File and directory management commands that create, modify, and organize files
3.User and permission commands that control access and ownership
4.Process and system monitoring commands that track performance and running services
5.System operation commands that manage shutdowns, reboots, and configurations
6.Network commands that configure connections and diagnose connectivity

#Basic Commands:

ls-List directory contents
pwd-Show current directory path
cd-Change directory
mkdir-Create a directory
rmdir-Remove an empty directory
rm-Delete files or directories
cp-Copy files or directories
mv-Move or rename files
touch-Create an empty file
grep-Search text patterns in files
cat-Display file content
systemctl-Manage system services
sudo-Run command as administrator
clear-clears page



## Key Takeaway

Linux consists of the **Kernel**, **User Space**, and **systemd (PID 1)**. Processes are created by the kernel, move through different states during execution, and are managed by systemd and the kernel. 
Understanding these components is essential for Linux troubleshooting and DevOps work.
