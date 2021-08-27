---
date updated: '2021-08-27T06:33:52-07:00'

---

## What is a Process?

- Instance of an existing program, aka job or task.

### Visual Metaphor

![](./img/Pasted%20image%2020210825062926.png)

- State of execution
  - Program counter, stack
- Parts and temporary holding area
  - Data, register state, occupy state i memory
- May require special hardware
  - I/O Devices(eg. network, disk)

## How an OS represents a process?

![](./img/Pasted%20image%2020210825063933.png)

- Process includes
  - Stack
  - heap
  - data
  - text
- Each section has a starting and ending address
- Entire process has an address space V<sub>0</sub> to V<sub>max</sub>
- Types of state inside a process
  - Code(text)
    - Static
  - Heap
    - Dynamically created during execution
    - Sometimes not contiguous, although in practice it may seem like it
  - Stack
    - Grows and shrinks during program execution
    - LIFO order

### Process Address Space

- Process Address Space(Virtual Address)
  - 'In Memory' representation of a process
  - Memory addresses here are **virtual**
- Page Table
  - Mapping of virtual to physical address
  - Used to decouple virtual memory and physical memory.
  - Basically holds virtual address and physical address together.
- Physical Address
  - Locations in the **physical** memory

![](./img/Pasted%20image%2020210825065540.png)

### Address Space and Memory Management

- Parts of virtual address space may not be allocated.
- May not have enough memory to store entire process state.
  - OS decides which portion of memory for a process should be swapped in and out of the memory using disk space
    - Also must maintain a mapping of the page table
    - Also must manage isolation between processes.
    - Also must maintain mapping of which parts of memory are on disk

#### Quiz

![](./img/Pasted%20image%2020210825070809.png)

It is important to note the difference between segmentation based memory management and paging based memory management.

- Why?
  - OS is free to figure out how to **ALIGN** blocks(aka pages) of memory among the address space as it sees fit
  - If both processes take up a range of 0-64k address space, the OS is free to decide how it wants to align blocks(aka pages) of each process's address space.

## How are multiple concurrent processes managed by OS?

### Process execution state

- [PCB](#process-control-block) contains info to figure out process execution state, maintained by OS
  - Program counter
    - A way to find out where the execution state is in a given process.
    - Maintained in CPU
      - In a register
  - Stack pointer
    - Managed in Memory
    - Top of stack is defined by stack pointer
      - Why is it important?
        - LIFO nature is maintained by knowing this info

### Process Control Block

![](./img/Pasted%20image%2020210826062356.png)

- Data structure maintained by the OS
  - For each process managed by the OS
- Created when process is created
- Fields are updated when process state changes
  - Some update more than others, eg the program counter.
    - Program counter
      - Changes during execution per instruction
      - CPU has a dedicated register(PC register) for tracking current program counter
      - OS is responsible for storing where the PC is when the process **leaves** CPU workspace.
      - OS is also responsible for giving the CPU the current PC when the process **enters** CPU workspace
- How is it utilized?
  - When process is being run actively in CPU, CPU will maintain some state info in its registers
  - When process is leaving CPU, OS will copy the CPU state(PC, etc) into the process control blocks
  - When a process is entering CPU, OS will load the CPU registers with the relevant info(PC,etc), which help the CPU resume with the work in the process
  - When all of the above happens with multiple processes, it is called **context switching**

### Context Switch

- Mechanism for OS to switch the execution from context of 1 process to another.
- Expensive
  - Direct costs
    - Num of cycles for just for loading and storing values
  - Indirect
    - Data is cached in L1, L2, etc cache, when data is moved in and out of CPU during context switch, this cached data might be moved out, which means its COLD cache(cache misses).

#### Quiz

When a cache is hot...most process data is in the cache, so the process performance will be at its best. Sometimes we must context switch.(wtf?)

### Process Lifecycle

![](./img/Pasted%20image%2020210827054102.png)

- Process States
  - New
  - Ready(doesn't imply running)
  - Running
  - Waiting
  - Terminated(when process finishes or encounters an error)

#### Quiz

The CPU is able to execute a process when the process in which state?

- Running(already running)
- Ready(cpu can execute it, **it just needs to be scheduled**)

### Process Creation

![](./img/Pasted%20image%2020210827055651.png)

- Creating process is parent, created process is child
- Fork
  - Copies parent PCB into a new PCB for the child
  - Parent and Child continues execution at the instruction that is after the fork.
- Exec
  - DOES NOT behave like fork, it is not used for replicating a process
  - Replaces a PCB in a process with an image of _another_ executable program
  - This means that this new program can now be a child of the parent that called the "fork" and then the "exec" call.

#### Quiz

- Which process is regarded as the parent of all process in:
  - Unix based OS
    - Init
      - First thing to start after OS boot, all other process are spawned from there
  - Android OS
    - Zygote
      - Daemon process which is solely used to launch app process
      - OS forks this process to create a new process, probably calls exec afterwards to load the fork with whatever PCB it needs for the app it is launching.

### Role of CPU Scheduler

![](./img/Pasted%20image%2020210827060750.png)

- For a CPU to execute a process, it must be in the ready state, in the ready queue
- How do we pick the right one to work on?
  - Determined by the scheduler
    - Decides which process to dispatch to CPU from ready state
    - Decides how long it should run for
- OS must be able to preempt, schedule processes as it sees fit by dispatching the process to CPU
  - Preempt means to get it close up shop and leave the CPU
- OS **must** be efficient in the way it handles its scheduling.

### Length of a Process

![](./img/Pasted%20image%2020210827063010.png)

- Two things to consider
  - How often do we run the scheduler?
  - The longer a process runs the less frequent the scheduler executes
- A good formula for cpu utilization in regards to the _work_ it gets done
  - Process time/total elapsed time(including schedule intervals)

![](./img/Pasted%20image%2020210827063256.png)

- There are decisions and tradeoffs to be aware of
  - What are good timeslice values(timeslice is the amount of time a process gets to run)?
  - What metrics to use for choosing the next process to run?

![](./img/Pasted%20image%2020210827063349.png)
