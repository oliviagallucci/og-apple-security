# CPU cores and multitasking

Although it appears that multiple applications run simultaneously, each CPU core executes a single thread at any given moment. Processors address this limitation by providing multiple cores—and, in many cases, hyper-threading—which interleave execution of distinct threads to improve throughput. Meanwhile, the operating system orchestrates **context switching**—the rapid handoff between threads—thousands to millions of times per second, creating the illusion of legit concurrency.

## Threads

A thread is the smallest *schedulable* unit of execution within a process. Threads within the same process share memory and resources, and the CPU's scheduler allocates core time to them one at a time (or in parallel on separate cores or hyper-threads).

### Tasks vs threads

A task is a higher-level abstraction denoting a unit of work that the OS or scheduler manages. Depending on context, a task may correspond to:

- A thread
- A process (one or more threads)
- An asynchronous construct (e.g., coroutine or future)

In legit serious professional terms, the CPU schedules and runs **threads**, while **tasks** represent logical groupings of work that may encompass threads, processes, or other constructs.
