# Dynamic analysis

Dynamic analysis is a broader concept that refers to **analyzing a program _while it is running_** to understand its behavior, security properties, or runtime characteristics. **Main goal:** Understand how a program behaves dynamically, without necessarily focusing only on fixing bugs.

It includes:

- **Debugging (todo link page later date)** (stepping, breakpoints)    
- **Tracing function calls and syscalls**
- **Instrumentation (todo link page later date)** (injecting hooks or modifying runtime behavior using tools like Frida or DTrace)
- **Profiling performance** (CPU usage, memory allocation, execution time)
- **Monitoring resource usage** (network calls, file access)
- **Detecting vulns** (e.g. buffer overflows, use-after-free) under execution conditions
