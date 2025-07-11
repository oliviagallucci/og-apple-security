# Syscalls
*Pwncollege* 

- **System call (syscall)**  
  A mechanism by which user-mode code requests services from the kernel. The ABI specifies:
  - A unique syscall number (e.g. `60` for `exit` on x86_64 Linux)  
  - Which registers carry arguments and return values  
  - How user → kernel transitions occur  


## Are system calls part of computer architecture?**
No. System calls belong to the system-level software architecture—specifically the Application Binary Interface (ABI)—not to the hardware ISA or microarchitecture. They define how user-mode programs request services from the OS kernel.

## Analogy

Calling a restaurant: you don't cook in the kitchen yourself; you place an order and the restaurant (a separate system) fulfills it. Likewise, when a program needs to perform an operation outside its own privileges (e.g., file I/O, networking, memory allocation), it issues a system call to the OS.

## Invocation

On Linux/x86-64, each syscall is identified by a unique number. To invoke one, move its number into **RAX** and execute the `syscall` instruction. Parameters follow the Linux convention:

1. **RDI** – first argument
2. **RSI** – second argument
3. **RDX** – third argument
4. ... and so on.

## Example: `write`

```c
// write(fd, buffer, count);
write(1, 0x1337000, 10);
```

Here, RAX = syscall number for `write`,
RDI = 1 (stdout),
RSI = 0x1337000 (buffer address),
RDX = 10 (bytes to write).

## File descriptors

Each process begins with three standard descriptors:

* **0 (stdin)**
* **1 (stdout)**
* **2 (stderr)**

To write to stderr, set RDI = 2.

## Performance 

System calls incur overhead due to context switches between user and kernel mode and hardware interactions. To minimize cost, batch operations—e.g., write larger buffers in a single call.

## Example: `read`

```c
// read(fd, buffer, count);
read(0, 0x1337000, 5);
```

Reading 5 bytes from stdin into memory at 0x1337000 might yield the ASCII bytes for "HELLO" (0x48, 0x45, 0x4C, 0x4C, 0x4F).
