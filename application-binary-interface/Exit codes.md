# Program termination and exit codes
*Pwncollege* 

Every program returns an exit status upon termination, supplied as the single argument to the `exit` system call.

Although system calls can accept multiple arguments, `exit` requires only one: the exit code.

## x86_64 Linux,
On x86_64 Linux, the first argument to any system call is placed in the `rdi` register.

An 8-bit status value passed to the kernel when a process terminates (e.g., via the `exit` syscall). The ABI dictates:
- How this status is encoded in the `wait`/`waitpid` status word  
- That only the low 8 bits are reported to the parent process or shell  

