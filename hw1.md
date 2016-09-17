# Homework 1

### 1)

The **Process Control Block** (PCB) is a logical representation of a process within an operating system. The PCB contains information about the process such as it's state, instruction addresses, registers to use, scheduling memory-management statistical counters and other infromation that is updated as the process runs.

In Linux the PCB is represented using the **task_struct** when using the C programming language.

### 2)

User-level threads have a significant advantage over kernel threads due to:
    -  Efficiency
        +  Since the user-level threads are often managed through a thread library and not the kernel itself the user is free to create as many threads as he/she likes without having to worry about implementing scheduling for each thread execution
    -  Less overhead
        +  When creating multiple threads of the kernel thread type the user has to be cautious as each thread created this way burdens the system and causes performance to suffer far more than user-level threads.

### 3.

Threads and process are both used in executing tasks by the operating system. Processes are programs in execution by the Operating System. They are more than the program source code itself and contain information about the program including a counter which holds the registers to use in execution along with a stack and/or heap. Threads however are the units of CPU utilizations used by processes. Threads are comprised of an ID, program counter, a set of registers and a stack from which the running program can utilize; other resources such as the data and code sections along with files, signals and other resources are shared with other threads in the process.

Because of this the `fork()` and `exec()` system calls which normally create a new duplicate process and load a new program that consumes the current process and its resources respectively take on different meanings when in a threaded environment. In a multithreaded environment there are often two `fork()` methods, one for duplicating a process with all threads and another for duplicating a process with only the thread that made the system call. `exec()` performs typically the same.


### 4)

FCDS

| Process | Burst | Wait Time |
|:-------:|:-----:|:---------:|
|    p1   |   5   |     0     |
|    p2   |   5   |     3     |
|    p3   |   8   |     7     |
|    p4   |   2   |     15    |
|    p5   |   2   |     15    |
|    p6   |   2   |     12    |
|    p7   |   3   |     9     |
|    p8   |   1   |     12    |

`Avg waiting time = (0 + 3 + 7 + 15 + 15 + 12 + 9 + 12) / 8 = 9.125`


SRTF

| Process | Burst | Remainder | Wait Time |
|:-------:|:-----:|:---------:|:---------:|
|    p1   |   3   |     2     |     2     |
|    p4   |   2   |     0     |     0     |
|    p1   |   2   |     0     |    ---    |
|    p5   |   2   |     0     |     2     |
|    p2   |   1   |     4     |     10    |
|    p6   |   2   |     0     |     0     |
|    p2   |   3   |     1     |    ---    |
|    p7   |   1   |     0     |     0     |
|    p2   |   1   |     0     |    ---    |
|    p8   |   3   |     0     |     2     |
|    p3   |   8   |     0     |     17    |

`Avg waiting time = (2 + 10 + 17 + 0 + 2 + 0 + 0 + 2) / 8 = 4.125`


### 5)

``` (C)

$ strace pwd
...

open("/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3

...

write(1, "/\n", 2/
)                      = 2

...

```


``` (C)

$ strace grep
...

open("/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3

...

write(2, "Try 'grep --help' for more infor"..., 40Try 'grep --help' for more information.
) = 40

...

```

### 6)

Using the `testfork.c` file provided we can determine, even without looking at the file, that the parent process forks itself into child process that executes and upon completion the parent process resumes. This can be determined by the order of output, and when analyzing the strace output.

Firstly we wee the output as:

``` (bash)

I am the parent 9292
I am the child 0
a.out  testfork.c
Child Complete

```

This output looks as if a parent process was called, then forked to run the `ls` command which outputs the contents of the directory and exits, then the parent process is resumed and exited as well.

Taking a deeper look into the strace output we can see some times associated with the processes and their forks:

``` (C)

// First open
1474142341.810893 open("/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3

...

// Child stack created for fork (I think)
child_stack=NULL, flags=CLONE_CHILD_CLEARTID|CLONE_CHILD_SETTID|SIGCHLD, child_tidptr=0x7f919023a6d0) = 10331

...

// Child output
[pid 10331] 1474142341.811772 write(1, "I am the child 0\n", 17I am the child 0
 <unfinished ...>
...

// Child finish
[pid 10331] 1474142341.811809 execve("/bin/ls", ["ls"], [/* 47 vars */] <unfinished ...>

...

// Output
[pid 10330] 1474142341.811850 write(1, "I am the parent 10331\n", 22I am the parent 10331
) = 22


...

// Parent receive child termination
1474142341.813392 <... wait4 resumed> NULL, 0, NULL) = 10331
1474142341.813406 --- SIGCHLD {si_signo=SIGCHLD, si_code=CLD_EXITED, si_pid=10331, si_uid=1000, si_status=0, si_utime=0, si_stime=0} ---

...

// Parent finish

1474142341.813423 write(1, "Child Complete\n", 15Child Complete
) = 15
```

We can see our hypothesis was confirmed. :smile: :cat:
