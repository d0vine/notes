# Windows Internals

Note: these notes reference "Windows Internals, Part 1 (Developer Reference)" by Mark E. Russinovich, David A. Solomon and Alex Ionescu. As this isn't supposed to be a direct summary (more of a self-commented set of excerpts from the book), I do recommend byuing this book (the newer, the better).

## Services, functions and routines

| Name                  | Description    |
|:----------------------|:---------------|
| Windows API functions | Documented, callable subroutines in the Windows API (i.e. Createprocess, GetMessage, ...) |
| Native system services/system calls | Undocumented, underlying services in the OS that are callable from user mode. For example, *NtCreateUserProcess* is the internal system service the Windows CreateProcess function calls to create a new process (see: *System Service Dispatching* in **Chapter 3**) |
| Kernel support functions/routines | Subroutines inside the Windows OS that can be called only from kernel mode. For example, *ExAllocatePoolWithTag* is the routine device drivers call to allocate memory from the Windows system heaps (called *pools*) |
| Windows services | Processes started bu the Windows service control manager. For example, the Task Scheduler service runes in a user-mode process that supports the *at* command (which is similar to the UNIX commands *at* or *cron*). (Note: although the registry defines Windows device drivers as "services," they are not referred to as such in this book) |
| DLLs (dynamic-link libraries) | A set of callable subroutines linked together as a binary file that can be dynamically loaded by applications that use the subbroutines. Examples include Msvcrt.dll (the C runtime library) and Kernel32.dll (one of the Windows API subsystem libraries). Windows user-mode components and applications use DLLs extensively. The advantage DLLs provide over static libraries is that applications can share DLLs, and Windows ensures there is only one in-memory copy of DLL's code among the applications that are referencing it. Note that nonexecutable .NET assemblies are compiled as DLLs but without any exported subroutines. Instead, the CLR parses compiled metadata to access the coresponding types and members. |

Q to self: how do .so libs work (under the hood) in Linux?

## Processes, Threads and Jobs

| Name            | Definition             |
|:----------------|:-----------------------|
| Program         | A static sequence of instructions |
| Process         | A container for a set of resources used when executing the instance of the program |

Q to self: what does the process look like in Linux/MacOS/*BSD?
Q to self: what UAC bypass techniques work on Win 10?

Windows process consists of (at the highest level of abstraction):

- a private virtual address space - a set of virtual memory addresses that the process can use
- an executable program - defines initial code and data, is mapped into the process' virtual address space
- a list of open handles to various system resources - such as semaphores, communication ports and files (they're available to all threads of the process)
- a security context called an *access token* that identifies the user, security groups, privileges, User Acount Control (UAC) virtualization state, session, and limited user account state associated with the process
- A unique identifier called a *process ID* (internally part of an identifier called a *client ID*)
- at least on thread of execution (although an "empty" process is possible, it is not useful)

Each process also points to its parent or creator process. **If the parent no longer exists, this information is not updated.** Therefore, it is possible for a process to refer to a nonexistent parent. This is not a problem, because nothing relies on this information being kept current.

Experiment: **loc 728**, **loc 762**, **loc 799**

The *tlist [/t]* tool.

