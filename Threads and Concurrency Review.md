---
date updated: '2021-08-29T13:13:11-07:00'

---

![](Pasted%20image%2020210828074835.png)

![](Pasted%20image%2020210828074847.png)

Refer to [intro to programming with threads paper](https://s3.amazonaws.com/content.udacity-data.com/courses/ud923/references/ud923-birrell-paper.pdf)

## What are threads?

### Visual Metaphor

![](Pasted%20image%2020210828154544.png)

- Active Entity
  - Executing unit of work for a given toy order
- Work simultaneously work with others
  - Many workers completing toy orders, either same or different ones at the same time
- Require Coordination
  - Share tools, parts, workstations.

#### What about threads?

- Executing unit of process
- Many threads work together
- Sharing I/O, CPU and memory.

![](Pasted%20image%2020210828154734.png)

## How threads differ from processes?

- Process that is single threaded has the standard PCB structure

![](Pasted%20image%2020210828155150.png)

- Process with multiple threads has a slightly modified PCB structure that keeps track of each thread's registers and stack

![](Pasted%20image%2020210828155251.png)

- Common resources are still shared, but resources related to execution context(eg thread specific registers, program counter, and stack pointers, stack)

## What data structures are used to implement and manage threads?

### Basic Mechanisms

#### Mutex

- Essentially locks all execution of the critical section execution to just 1 thread, so only 1 thread can execute the critical section in the process until the lock is released.
  - Portion of code which executes between lock and unlock is known as critical section
- Also known as mutual exclusion principle
- Threads **without** a critical section can execute whatever they want to.
- Critical section is usually set up in place to prevent any undefined behavior from occurring, eg data races

#### Conditional Variables


#### How to support threads?

- Thread data structure
  - identify threads, keep track of resource usage
- Mechanism to create and manage threads
- Mechanisms to safely coordinate among threads running concurrently in the same address space.
  - eg. threads don't overwrite each other's inputs or results
- Coordination issues
  - ![](Pasted%20image%2020210829073946.png)
  - Isolating is easier with processes as the OS helps with this
  - Aka Data Race
- Concurrency control & coordination, also known as sync mechanisms
  - Mutual Exclusion
    - exclusive operation to only 1 thread at a time
    - eg. mutexes
  - Conditional waiting
    - specific condition to wait before proceeding
    - eg. condition variables
  - Waking up other threads from wait state

#### Thread creation

![](Pasted%20image%2020210829075029.png)

- Thread type
  - Thread data structure:
    - thread id
    - program counter
    - stack pointer
    - registers
    - stack
    - attributes
      - for scheduling, debugging or other thread management
- Fork (procedure, args) -- NOT related to unix fork
  - create a thread
- Join
  - Terminate a thread
  - retrieve its results

## Benefits of Multi-threading

- Parallelization
  - Speeds up execution(eg multiple worker nodes for 1 set of data)
- Specialization
  - Specialize specific threads for specific types of tasks, or different portions of program
  - If a thread keeps using a specific core to execute, over time its data might end up in a local cache more often, which results in better performance.
- Sharing of data
  - All the threads in a process share the same address space. (eg they can have access to global variables across all threads)
- No need for IPC
  - Communication between threads is achieved by shared memory which has virtually no overhead.

### What if num of threads > num of cpus?

- Example
  - If process is asking for disk op, which naturally takes a long time, and the cost of context switching is less than the time it takes. It is better to switch context and work on another process while waiting for disk op to complete.
    - if idle time > 2 times context switch time
  - Threads **do not** require context switching as all the data is contained within the same PCB. So in the case of multiple threads, it is better to context switch threads.
    - Helps us hide latency with IO operations

### How about OS code?

![](Pasted%20image%2020210829072918.png)

- Helps multi-threaded kernel code run concurrently
- Support multiple execution contexts

### Quiz

Do the following statements apply to processes, threads or both?

![](Pasted%20image%2020210829073038.png)

- Can share a virtual address space
  - technically both can, process can do it via ipc shared memory mechanism although not entire address spaces just parts of them
  - **quiz answer** is that it is only threads that share a virtual address space
- Take longer to context switch
  - processes
- have an execution context
  - both have their own execution context
- usually results in hotter caches when multiple exist
  - multiple threads can usually result in hotter caches
- Make use of some communication mechanisms
  - both have their own, processes have ipc, and threads have sync primitives
