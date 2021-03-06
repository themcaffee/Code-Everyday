---
layout: post
title: Process Management
date:   2014-11-07 00:00:00
---

### The Process

__Process__: A program in the midst of execution

__Threads__: the objects of activity with the process. Each thread includes
a unique program counter, process stack, and set of processor registers. The
kernel schedules individual threads, not processes.

Processes provide two virtualizations : a virtualized processor and virtual
memory. A process is created in Linux with the use of fork() system call, which
creates a new process by duplicating an existing one.

- Process that calls fork() is the __parent__.
- New process is the __child__.

In contemporary Linux kernels, fork() is implemented via the clone() system call.
A program exits with the exit() system call. This function terminates the process
and frees all its resources. Linux kernel refers to processes as __tasks__.

### Process Descriptor and the Task Structure

The kernel stores the list of processes in a circular doubly linked list called
the __task list__. Each element in the task list is a __process descriptor__ of
the type struct taskstruct.

This contains all the information that the kernel has and needs about a process.

### Storing the Process Descriptor

The system identifies processes by a unique process identification value or PID.
Looking up the process descriptor of the current executing task is dependent on
the architecture.

### Process State

The state field of the process descriptor describes the current condition of
the process. Each process is in one of these file states.

- TASK_RUNNING : The process is runnable; it is either currently running or on a
run-queue to run. Only possible state of a process executing in user-space.
- TASK_INTERRUPTIBLE : The process is sleeping, waiting for some condition to exist.
- TASK_UNINTERRUPTIBLE : Identical to TASK_INTERRUPTIBLE except that it does not
wake up and become runnable if it receives a signal.
- TASK_TRACED : The process is being traced by another process, such as a debugger.
- TASK_STOPPED : Process execution has stopped; the task is not running nor is
it eligible to run.

### The Process Family Tree

All processes are descendants of the init process, whose PID is one. The kernel
starts init in the last step of the boot process, which in turn reads the 
system initscripts and executes more programs. 

### Process Creation

In Unix processes are created in 2 steps:

1. fork() : creates a child process that is a copy of the current task
2. exec() : loads a new executable into the address space and beings executing it.

### Copy-on-write

Copy-on-write (COW) is a technique to delay or altogether prevent copying data
from a parent process. Data is shared until it needs to be written to, then a duplicate
is made.

### Forking

Linux implements fork() via the done() system call. The clone() system call then
calls do_fork(). The bulk of the work is done then copy_process() is called. Then
the process starts running.

### vfork()

The vfork() has the same effect as fork(), except that the page table entries of
the parent process are not copied. Instead, the child executes as the sole thread
in the parent's address space, and the parent is blocked until the child either
calls exec() or exits.

### The Linux Implementation of threads

Threads allow multiple threads of execution in parallel. Linux doesn't differentiate
between process and threads.

### Kernel threads

Standard processes that run in kernel-space : New task is created via the clone()
system call. When started, a kernel thread continue to exist until it calls do_exit()
or another part of the kernel thread calls kthread_stop()

### Process Termination

A process can exit through a exit() system call or when the process receives a
signal or exception it cannot handle. Regardless of how a process terminates, the
bulk of the work is done by do_exit().

### Removing the process descriptor

After do_exit() completes, the process descriptor for the terminated process still
exits, but the process is a zombie and is unable to run. After the parent has
obtained information on its terminated child, or signified to the kernel
that it does not care, the child's task_struct is deallocated.

### The Dilemma of the Parentless Task

If a parent exits before its children the task's children is reparented on exit
to either another process in the current thread group, or if that fails, the init
process. 