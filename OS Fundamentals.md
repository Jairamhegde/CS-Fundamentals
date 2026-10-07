# Operating Systems — SDE Interview Guide

_Compiled from patterns seen across Google, Amazon, Microsoft, and other product-company interviews_

---

## 1. Process Management

**Q: What is a process, and how does the OS track it?** A process is a program in execution — code, data, stack, heap, and allocated resources (CPU registers, open files, etc.). The OS tracks each one with a **Process Control Block (PCB)** containing process state, program counter, CPU registers, memory limits, and I/O status. All PCBs are held in a process table.

**Q: What are the states of a process?** New → Ready → Running → Waiting/Blocked → Terminated. A process moves Ready→Running when scheduled, Running→Waiting when it needs I/O, and Waiting→Ready once the I/O completes.

**Q: Difference between a process and a thread?** A process is an independent, isolated unit with its own memory space; processes don't share memory directly. A thread is a lightweight unit _within_ a process that shares the process's memory and resources with sibling threads. Threads are cheaper to create/switch but need synchronization to avoid race conditions. Example: a browser is a process; each tab is a thread.

**Q: What is context switching, and why is it expensive?** Saving the CPU state (registers, program counter) of the currently running process/thread and loading another's, so the CPU can multitask. It's overhead because no useful work happens during the switch — cost comes from register save/restore, cache/TLB invalidation, and scheduler bookkeeping.

**Q: What is an inter-process communication (IPC) mechanism, and what are the common types?** Since processes are isolated, they need explicit mechanisms to exchange data: **pipes**, **message queues**, **shared memory**, **sockets**, and **signals**. Shared memory is fastest (no kernel copy) but needs manual synchronization; pipes/message queues are safer but slower.

**Q: User-level threads vs. kernel-level threads?** User-level threads are managed by a library in user space — fast to create/switch, but the kernel sees only one thread, so a blocking call blocks the whole process. Kernel-level threads are managed by the OS — can run in parallel on multiple cores and one blocking doesn't block others, but context switches are costlier.

---

## 2. CPU Scheduling

**Q: What is CPU scheduling, and what are its goals?** The OS component that decides which ready process runs next on the CPU. Goals: maximize CPU utilization and throughput, minimize waiting time, turnaround time, and response time, and ensure fairness.

**Q: Name the common CPU scheduling algorithms.**

- **FCFS (First-Come-First-Served):** simple queue, non-preemptive; causes the _convoy effect_ (short jobs stuck behind long ones).
- **SJF (Shortest Job First):** picks the shortest burst time next; optimal average waiting time but needs burst-time prediction; can starve long jobs.
- **SRTF:** preemptive version of SJF.
- **Round Robin:** each process gets a fixed time quantum in a cycle; fair, good for time-sharing systems; performance depends heavily on quantum size.
- **Priority Scheduling:** highest-priority process runs first; can cause starvation of low-priority processes (fixed via **aging**, which gradually raises the priority of processes that have waited long).
- **Multilevel Queue / Multilevel Feedback Queue:** processes are split into queues (e.g., interactive vs. batch) with different scheduling rules, and can move between queues based on behavior.

**Q: What is starvation, and how is it prevented?** A process waits indefinitely because higher-priority processes keep getting scheduled ahead of it. Prevented with **aging** — increase the priority of waiting processes over time.

**Q: What's the difference between preemptive and non-preemptive scheduling?** Preemptive scheduling can interrupt a running process to give the CPU to another (e.g., Round Robin, SRTF); non-preemptive lets a process run to completion or until it voluntarily yields (e.g., FCFS, SJF).

**Q: How does Linux's Completely Fair Scheduler (CFS) work (often asked at senior level)?** CFS tracks each task's "virtual runtime" and always picks the task with the least accumulated runtime, using a red-black tree for O(log n) selection — approximating ideal fair CPU sharing rather than using fixed time slices.

---

## 3. Process Synchronization

**Q: What is a race condition?** A bug where the outcome depends on the non-deterministic timing/interleaving of multiple threads/processes accessing shared data without synchronization, e.g., two threads incrementing the same counter and losing an update.

**Q: What is a critical section, and what conditions must a solution satisfy?** The part of code that accesses shared resources. A valid solution needs: **mutual exclusion** (only one process in the critical section at a time), **progress** (no indefinite postponement of processes not in the critical section that want in), and **bounded waiting** (a limit on how long a process waits before its request is granted).

**Q: Mutex vs. Semaphore?** A **mutex** is a locking mechanism for mutual exclusion, typically owned/released by the same thread that locked it. A **semaphore** is a signaling counter: a _binary_ semaphore behaves like a mutex, while a _counting_ semaphore allows up to N threads to access a resource pool, and can be signaled by a different thread than the one that waited.

**Q: What is a monitor?** A higher-level synchronization construct that bundles shared data with the procedures that operate on it, guaranteeing only one thread executes inside at a time, plus condition variables (`wait`/`signal`) for blocking until a condition holds.

**Q: Explain the Producer-Consumer (bounded buffer) problem.** Producers add items to a shared fixed-size buffer, consumers remove them; both must not overrun/underrun the buffer. Standard solution: three semaphores — `mutex` (1, for mutual exclusion on the buffer), `empty` (buffer_size, counts empty slots), `full` (0, counts filled slots). Producer waits on `empty`, signals `full`; consumer waits on `full`, signals `empty`.

**Q: What is the Dining Philosophers problem, and why does it matter?** Five philosophers alternate thinking and eating, each needing two shared forks (one left, one right). It's a classic model for deadlock (if all pick up their left fork simultaneously) and starvation, used to illustrate correct resource-ordering/allocation strategies (e.g., asymmetric ordering, or allowing at most N-1 philosophers to sit at once).

**Q: What is the Readers-Writers problem?** Multiple readers can access shared data concurrently, but a writer needs exclusive access. Solutions must balance reader throughput against writer starvation (e.g., giving writers priority once one is waiting).

---

## 4. Deadlocks

**Q: What is a deadlock?** A state where two or more processes are each waiting for a resource held by another, so none can proceed.

**Q: What are the four necessary conditions for deadlock (Coffman conditions)?**

1. **Mutual exclusion** — resource can't be shared.
2. **Hold and wait** — a process holds one resource while waiting for another.
3. **No preemption** — a resource can't be forcibly taken away.
4. **Circular wait** — a cycle of processes each waiting on the next. All four must hold simultaneously for deadlock to occur.

**Q: How can deadlocks be handled?**

- **Prevention:** break one of the four conditions (e.g., request all resources at once, enforce a global resource-ordering to prevent circular wait).
- **Avoidance:** only grant a request if the system stays in a "safe state" — the **Banker's Algorithm** is the classic example.
- **Detection & Recovery:** allow deadlocks to occur, detect them (e.g., cycle detection in a resource-allocation graph), then recover by killing/rolling back a process or preempting a resource.
- **Ignore it** (the "ostrich algorithm") — practical for systems where deadlocks are rare (used by many general-purpose OSes).

**Q: Explain the Banker's Algorithm briefly.** Before granting a resource request, the OS simulates the allocation and checks if the system would remain in a state where all processes could still finish given the maximum resources they might need — only then is the request actually granted. Prevents entering unsafe states that could lead to deadlock.

**Q: Deadlock vs. Starvation?** Deadlock: processes are permanently blocked in a circular wait — nobody proceeds. Starvation: a process is repeatedly denied a resource (often due to scheduling policy) but the system as a whole keeps making progress.

---

> ## 5. Memory Management

## Q: What is paging?
A memory management scheme that eliminates external fragmentation by dividing logical memory into fixed-size **pages** and physical memory into equal-size **frames**. The OS maintains a **page table** per process, mapping pages to frames, so a process's memory need not be contiguous. Can still cause internal fragmentation (last page partially used).

**Q: What is segmentation, and how does it differ from paging?** Segmentation divides a program into logically meaningful variable-sized units (code, stack, heap segments), matching how programmers think about memory, whereas paging uses fixed-size, logically-arbitrary blocks. Segmentation can cause external fragmentation; paging avoids it but can cause internal fragmentation. Many real systems (x86) combine both: segmentation on top of paging.

**Q: What is virtual memory, and why is it useful?** An abstraction that gives each process its own large, contiguous address space regardless of physical RAM size, backed by disk. It lets programs larger than physical memory run, isolates processes from each other (security), and simplifies memory allocation.

**Q: What is demand paging?** A lazy-loading approach where a page is loaded into physical memory only when it's actually referenced (page fault triggers the load), rather than loading the whole process upfront — saves memory and speeds up process start.

**Q: What is a page fault, and what happens when one occurs?** A page fault happens when a process accesses a page not currently in physical memory. The OS traps the fault, locates the page on disk, finds a free frame (evicting one if needed via a replacement policy), loads the page, updates the page table, and resumes the process.

**Q: What are common page replacement algorithms?**

- **FIFO:** evict the oldest-loaded page; simple but can suffer **Belady's Anomaly** (more frames → more faults, an unusual case unique to FIFO).
- **LRU (Least Recently Used):** evict the page unused for the longest time; good approximation of future behavior but costlier to implement exactly.
- **Optimal:** evict the page that won't be used for the longest time in the future — theoretical best-case benchmark, not implementable in practice (needs future knowledge).
- **Clock/Second-Chance:** a practical, cheap approximation of LRU using a reference bit.

**Q: What is thrashing, and how do you fix it?** When the system spends more time swapping pages in/out than executing actual work, because processes don't have enough frames for their **working set**, causing very high page-fault rates and CPU utilization to crash. Fixes: reduce the degree of multiprogramming, use a better replacement algorithm, increase physical memory, or use working-set-based admission control.

**Q: Internal vs. external fragmentation?** Internal fragmentation: wasted space _inside_ an allocated block (e.g., the unused portion of the last page of a process). External fragmentation: free memory exists but is scattered in small non-contiguous chunks, so it can't satisfy a large contiguous request (common with segmentation/contiguous allocation).

**Q: What is a TLB (Translation Lookaside Buffer)?** A small, fast hardware cache of recent virtual-to-physical page-table translations, avoiding the need to walk the full page table (which can require multiple memory accesses) on every memory reference. A "TLB miss" falls back to the full page-table walk.

---

## 6. File Systems & I/O

**Q: What is the purpose of a file system?** It organizes, stores, retrieves, and manages data on persistent storage, providing an abstraction (files, directories, permissions) over raw disk blocks, plus metadata (size, timestamps, access control) via structures like inodes.

**Q: What is an inode?** A data structure that stores metadata about a file (permissions, owner, size, timestamps, and pointers to the data blocks on disk) — the filename itself is stored separately in the directory entry, which points to an inode.

**Q: Common disk scheduling algorithms?**

- **FCFS:** service requests in arrival order — simple but can cause long seek times.
- **SSTF (Shortest Seek Time First):** service the closest request next — can starve far-away requests.
- **SCAN ("elevator")**: disk arm sweeps in one direction servicing requests, reverses at the end.
- **C-SCAN:** like SCAN but only services requests in one direction, then jumps back to the start — more uniform wait times.

**Q: What is a memory-mapped file?** A technique that maps a file directly into a process's virtual address space, so file I/O becomes simple memory reads/writes handled by the paging system — used for efficient large-file access and for shared-memory IPC between processes.

**Q: What's the difference between a system call and a regular function call?** A system call requests a service from the OS kernel (e.g., file I/O, process creation) and requires a context switch to kernel mode with privilege elevation, whereas a regular function call stays in user space with no privilege change and much lower overhead.

---

## 7. Core Concepts & Miscellaneous

**Q: What is the kernel?** The core of the OS, always resident in memory, that manages hardware resources and mediates all access to them — handling process scheduling, memory management, device drivers, and interrupts. Applications interact with it via system calls.

**Q: Monolithic vs. microkernel?** A monolithic kernel runs most OS services (drivers, file system, scheduler) in kernel space as one large program — fast (no IPC overhead) but a bug can crash the whole system. A microkernel keeps only the bare minimum (IPC, basic scheduling, low-level address-space management) in kernel space and runs other services as user-space processes — more robust/modular but slower due to IPC overhead.

**Q: What is virtualization, and what's a hypervisor?** Virtualization lets multiple operating systems run concurrently on shared physical hardware, improving resource utilization and isolation. A hypervisor is the software layer that creates and manages these virtual machines — **Type 1** (bare-metal, runs directly on hardware, e.g., ESXi) vs. **Type 2** (runs atop a host OS, e.g., VirtualBox).

**Q: What is an interrupt, and how does interrupt handling work?** A signal to the CPU indicating an event needs immediate attention (hardware event like I/O completion, or a software exception). The CPU saves its current state, jumps to an **interrupt service routine (ISR)** via the interrupt vector table, handles the event, then restores state and resumes the interrupted process.
- this is the folding heading
> [!FAQ]+ Click this arrow to expand the secret text!
> This is the hidden content inside your toggle box
>you can create one more line here . if you needed iseweweaw
- .

**Q: What is a spinlock, and how does it differ from a mutex?** A spinlock makes a waiting thread busy-loop (poll) until a lock is free, instead of sleeping — avoids the cost of a context switch, so it's efficient for very short critical sections, but wastes CPU if the wait is long. A mutex instead puts the waiting thread to sleep, which is better for longer waits.

**Q: What is copy-on-write (COW)?** An optimization where, when a process is forked, the parent and child initially _share_ the same physical memory pages (marked read-only); a page is only actually copied when either process attempts to write to it — makes `fork()` cheap.

**Q: What is a zombie process vs. an orphan process?** A **zombie** is a terminated process whose exit status hasn't yet been read by its parent (via `wait()`), so its entry lingers in the process table. An **orphan** is a running process whose parent terminated before it did; it gets re-parented (typically to `init`/PID 1), which will eventually reap it.

---

## How interviewers actually use these questions

- **Entry-level / general SDE:** definitions + comparisons (process vs. thread, paging vs. segmentation, mutex vs. semaphore).
- **Mid-level:** apply concepts to scenarios ("server is thrashing under load — diagnose it," "design a producer-consumer queue with locks").
- **Senior / systems-adjacent roles (infra, cloud, kernel):** deeper internals — CFS scheduling, NUMA-aware allocation, huge pages, deadlock detection via wait-for-graph cycle detection, TLB shootdowns in multi-core systems.

**Prep tip:** Be ready to trace through the four deadlock conditions, the producer-consumer semaphore solution, and page-fault handling step-by-step on a whiteboard — these three come up across nearly every source reviewed.