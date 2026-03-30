# Day 02

---

# Linux Architecture, Processes, and systemd

--- 

### The core components of Linux (kernel, user space, init/systemd)

   -  **Kernal** : The Linux kernel is the core of the operating system, acting as a bridge between the physical hardware and the software running on the system. It runs in a privileged "kernel mode" and has full access to all system resources. 

   - **User Space** : The user space is the environment where user applications, system libraries, and utilities run. Code in the user space runs in an unprivileged "user mode", meaning it cannot access hardware directly or interfere with the kernel or other processes without explicit permission via system calls. A crash in a user-space application typically does not bring down the entire system, as the kernel's memory remains protected. 

   - **Init System / Systemd** : The init (initialization) system is the very first user-mode process started by the kernel after the system boots, and it is assigned the special Process ID (PID) of 1. It is responsible for bringing the system to a usable state and managing the system's processes and services throughout its operation, until shutdown.

--- 

### How processes are created and managed

In Linux, processes are created using a combination of the fork() and exec() system calls and managed by the kernel through various tools and mechanisms, including unique Process IDs (PIDs), scheduling algorithms, and signals. 

- fork(): A running process (the parent) uses the fork() system call to create an almost exact duplicate of itself, known as the child process. 

- exec(): After the fork(), the child process typically uses an exec() system call (e.g., execve(), execlp()) to load a new program into its memory space. 

#### Process States

- Running (R): The process is currently executing on the CPU or is ready to run.
- Sleeping/Waiting (S or D): The process is waiting for an event, such as I/O completion or a resource to become available.
- Stopped (T): The process has been paused, typically by a user signal (e.g., Ctrl+Z).
- Zombie (Z): The process has finished execution, but its parent has not yet read its exit status. It is "dead" but still occupies a process table entry.
- Terminated: The process is fully finished and its resources are cleaned up. 
- Idle (I): The process is a kernel thread that is not currently using the CPU and is waiting for an event or work to do.

---

### What systemd does and why it matters

systemd (system daemon) is the default system and service manager for most modern Linux distributions (e.g., Ubuntu, Fedora, Debian, Red Hat). It serves as the init system, acting as the very first program (PID 1) to run at boot and the last to shut down. 

It is designed to coordinate all parts of the Linux system to work together, acting as a "conductor" that brings the system from powering on to a fully operational state. 

- Systemd is crucial in modern Linux and hence it matters , the reasons are as following : 
    - Faster boot perfomance
    - Reliability and Stability
    - Standarization
    - Simplified Troubleshooting
    - Modern Features

## *This is the foundation for all troubleshooting you will do as a DevOps engineer.*

--- 

- List **5 commands** you would use daily

    - ls
    - cd
    - clear
    - mkdir
    - touch
    - cat
    - man
    - grep
    - sudo 
    - cp
    - mv
