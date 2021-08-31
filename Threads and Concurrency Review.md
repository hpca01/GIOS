---
date updated: '2021-08-31T05:48:50-07:00'

---

![](./img/Pasted%20image%2020210828074835.png)

![](./img/Pasted%20image%2020210828074847.png)

Refer to [intro to programming with threads paper](https://s3.amazonaws.com/content.udacity-data.com/courses/ud923/references/ud923-birrell-paper.pdf)

## What are threads?

### Visual Metaphor

![](./img/Pasted%20image%2020210828154544.png)

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

![](./img/Pasted%20image%2020210828154734.png)

## How threads differ from processes?

- Process that is single threaded has the standard PCB structure

![](./img/Pasted%20image%2020210828155150.png)

- Process with multiple threads has a slightly modified PCB structure that keeps track of each thread's registers and stack

![](./img/Pasted%20image%2020210828155251.png)

- Common resources are still shared, but resources related to execution context(eg thread specific registers, program counter, and stack pointers, stack)

## What data structures are used to implement and manage threads?

### Basic Mechanisms

#### Mutex

- Essentially locks all execution of the critical section execution to just 1 thread, so only 1 thread can execute the critical section in the process until the lock is released.
  - Portion of code which executes between lock and unlock is known as critical section
- Also known as mutual exclusion principle
- Threads **without** a critical section can execute whatever they want to.
- Critical section is usually set up in place to prevent any undefined behavior from occurring, eg data races
- It is important to note that the lock does not guarantee a deterministic behavior to picking which of the waiting threads gets to lock next.

#### Conditional Variables

- Should be used in conjunction with mutex
- Flow
  - Section of code acquires lock, and then initiates a wait on a conditional variable, going to sleep
    - When waiting, it releases the lock
  - A different thread with a different section of code acquires lock and then **signals** the condition variable.
    - Upon signaling, the thread is awake again and tries to reacquire the lock
    - Once lock is obtained it continues its execution path.
- It is important to note that the condition variable does not guarantee which of the woken up threads will get to lock first.
- It is also possible to use **broadcast** vs **signal**, but it might not always be useful as after waking up all threads, only 1 of them can actually get a lock.

#### How to support threads?

- Thread data structure
  - identify threads, keep track of resource usage
- Mechanism to create and manage threads
- Mechanisms to safely coordinate among threads running concurrently in the same address space.
  - eg. threads don't overwrite each other's inputs or results
- Coordination issues
  - ![](./img/Pasted%20image%2020210829073946.png)
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

![](./img/Pasted%20image%2020210829075029.png)

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

![](./img/Pasted%20image%2020210829072918.png)

- Helps multi-threaded kernel code run concurrently
- Support multiple execution contexts

### Quiz

Do the following statements apply to processes, threads or both?

![](./img/Pasted%20image%2020210829073038.png)

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

## Readers/Writer Problem

Let's look at a scenario where there is some subset of threads that want to read from shared state, and one thread that wants to write to shared state. This is commonly known as the **readers/writer** problem.

At any given point in time, 0 or more readers can access the shared state at a given time, and 0 or 1 writers can access the shared state at a given time. The readers and writer cannot access the shared state at the same time.

One naive approach would be to wrap access to the shared state itself in the mutex. However, this approach is too restrictive. Since mutexes only allow access to shared state one thread at time, we would not be able to let multiple readers access state concurrently.

Let's enumerate the conditions in which reading is allowed, writing is allowed, and neither is allowed. We will use a `read_counter` and a `write_counter` to express the number of readers/writers at a given time.

If `read_counter == 0` and `write_counter == 0`, then both writing and reading is allowed. If `read_counter > 0`, then only reading is allowed. If `write_counter == 1`, neither reading nor writing is allowed.

We can condense our two counters into one variable, `resource_counter`. If the `resource_counter` is zero, we will say that resource is free; that is, available for reads and writes. If the resource is being accessed for reading, the `resource_counter` will be greater than zero. We can encode the case where the resource is being accessed for writing by encoding `resource_counter` as a negative number.

Our `resource_counter` is a **proxy variable** that reflects the state that the current resource is in. Instead of controlling updates to the shared state, we can instead control access to this proxy variable. As long as any update to the shared state is first reflected in an update to the proxy variable, we can ensure that our state is accessed via the policies we wish to enforce.

![](./img/Pasted%20image%2020210829133519.png)

## Readers/Writer Problem Example

[Link to problem being solved w/ a semaphore mutex](https://www.geeksforgeeks.org/readers-writers-problem-set-1-introduction-and-readers-preference-solution/)

In this example, we can see that our reading and writing operations exist outside of a locked mutex, but are preceded and followed by a mutex enforced update to the shared variable, `resource_counter`.

Our program will require four things:

- `resource_counter` - a proxy variable for the state of the shared resource
- `counter_mutex` - a mutex which controls access to `resource_counter`
- `read_phase`  - a condition variable signifying that the resource is ready for reading
- `write_phase` - a condition variable signifying that the resource is ready for writing

Readers can begin reading by first locking the mutex and incrementing the value of the `resource_counter`. If multiple readers try to lock the mutex at once, all but one will block and each will unblock as the mutex becomes available.

If a writer tries to enter the fray while multiple readers are accessing the state, the writer will have to `wait`, since our application is such that writes cannot proceed while reads are ongoing. The writer will wait on the condition variable `write_phase`.

Once the reader decrements `resource_counter`, it will check to see if `resource_counter` is zero. If so, that means our application is in a state where writes can proceed, so this final reader will `signal` on the `write_phase` variable to wake up a waiting writer. Given that only one writer can proceed at a time, it does not make sense to use a `broadcast` here.

The pending writer will be removed from the wait queue that is associated with the `write_phase` variable, and the counter mutex will be reacquired before coming out of the wait operation. We recheck that our predicate (`resource_counter != 0`) is still true before we adjust `resource_counter`. We can then proceed with our write.

**NB**: We need to recheck our condition in our while loop because the act of removing the thread from the wait queue and the act of reacquiring the mutex occur as two separate procedures. This means that the mutex could be reacquired in between these two steps by a _different thread_, with `resource_counter` being updated by that thread. Thus, we must check that our condition holds one last time after we acquire the mutex.

If a new thread wants to write while another thread is currently writing, this new thread will have to `wait` on `write_phase`. If a new thread wants to read while another thread is currently writing, this new thread will also have to `wait`, but on `read_phase` instead.

Once the current writer completes, it resets `resource_counter` to 0, and then `broadcast`s to the `read_phase` and `signals` to `write_phase`. We `signal` to `write_phase` because only one writer can access the shared state at a given time, and we `broadcast` to `read_phase` because multiple threads are allowed to read from the shared state at a given time.

**NB**: Note that multiple threads cannot access the mutex concurrently, as this defeats the point of the mutex. But, after each thread accesses the mutex in turn, all readers can read from the shared state concurrently. This is why a mutex over the shared state itself was not sufficient: concurrent reads would not have been possible.

Even though we call `broadcast` before `signal` we don't really have control over whether a waiting reader or writer will be woken up first. That decision is left up to the thread scheduler, into which we do not usually have insight.

![](./img/Pasted%20image%2020210829135127.png)

**Note: when it says 'readers' in code, it really means resource_counter**

## Critical Section structure

![](./img/Pasted%20image%2020210829161113.png)

- Colored blocks are enter/exit critical section codes
  - What we really want to protect is the file
  - Limit entry phase and exit phase
- Inside each of the colored blocks, they also have critical sections
- Flow
  - ![](./img/Pasted%20image%2020210829161459.png)
    - Lock mutex
    - Check predicate before performing action
      - If not okay, wait with a mutex and a condition variable
    - Check predicate again after wait and and if okay exit out of while loop to do the update on a shared state
    - Notify other threads via signal/broadcast through the appropriate conditional variables
    - Unlock mutex

![](./img/Pasted%20image%2020210829161516.png)

![](./img/Pasted%20image%2020210829161531.png)

![](./img/Pasted%20image%2020210829161605.png)

## Critical section structure with proxy

![](./img/Pasted%20image%2020210829161757.png)

- This kind of structure allows some level of deterministic behavior when multiple threads are reading and writing to a shared state
  - Mutex default behavior makes it so that only 1 thread can execute critical section of code.
- In this way of sandwiching sets of mutex and wait operations around code that needs to be executed concurrently, data races are avoided.

## Common pitfalls

![](./img/Pasted%20image%2020210829162552.png)
![](./img/Pasted%20image%2020210829163123.png)

- Keep track of mutex/condition variables used with a resource
  - make sure you keep track of which shared resource, which operation, or shared state you want to the variable to be used with
- Check that you are **always** and **correctly** protect the variable/resource using lock and unlock with the same mutex
  - did you forget to lock/unlock?
  - sometimes compilers will generate warnings
- Use a single mutex to access a single resource
  - Do not use multiple mutexes for the same resource you are trying to protect
- Check that you are signaling the correct condition variable
- Check that you are not using signal when broadcast is needed
  - Signal is only for 1 thread to proceed, remaining threads will continue to wait
- Do you need priority guarantees?
  - Execution of threads is not controlled by order in which signals are sent to a conditional variable

### Other kind of pitfalls

![](./img/Pasted%20image%2020210829163216.png)
![](./img/Pasted%20image%2020210830052404.png)

#### Spurious wake-ups

- Unnecessary wake-ups
- Doesn't effect correctness, but affects performance
  - wastes cycles
- Can we unlock mutex before broadcast/signal?
  - it is possible in some cases but in other cases it is not, especially if you are trying to access a shared resource outside of the lock.
    - Eg. For the readers/writer example, it checks to see if the resource counter is 0 before signaling the write phase. If the logic for checking the counter and signaling the writer was put outside, it would not work as you are now accessing a shared resource outside of a synchronized context.
  - ![](./img/Pasted%20image%2020210830052657.png)

#### Deadlocks

- Two or more competing threads are waiting on each other to complete but none of them ever do.

![](./img/Pasted%20image%2020210830053049.png)
![](./img/Pasted%20image%2020210830053147.png)
![](./img/Pasted%20image%2020210830053459.png)

- How to address
  - Fine grained control
  - Get all locks upfront
  - Maintain lock order

- Summary
  - Maintain a lock order is best way
  - A cycle in the wait graph of a thread is necessary and sufficient for a deadlock to occur
  - Prevention
    - Check if a lock operation will cause a cycle
      - Delay the operation
    - Detection and recovery
      - Analysis of wait graph
        - Requires a rollback of execution to recover
        - Maintain enough state
    - Apply the ostrich algorithm
      - Do NOTHING.
      - Sometimes when code is too complex and input is coming from multiple sources, it is really hard to guarantee that a mutex will have a correct order of locking.

## Kernel Level Threads

![](./img/Pasted%20image%2020210830061446.png)

- Threads can exist both kernel and user level
- Are visible to kernel and managed by kernel level scheduler
  - OS scheduler decides how they are mapped to physical cpu and which one can execute at any given point of time
- Some are existent to simply run the user level threads
- Some are there to run OS level services

### One to One Model

![](./img/Pasted%20image%2020210830062011.png)

- 1 kernel level thread to 1 user level thread
- Kernel manages the treads
- All threads gain implicit OS level mechanisms for synchronization and blocking
  - This is a double-edged sword, because if the OS doesn't implement a specific kind of construct or sync primitive related to threading. Your program will not be able to be ported.
  - Limits of the OS when it comes to threading apply to your program
- Expensive gap between traversing user-level and kernel-level

### Many to One

![](./img/Pasted%20image%2020210830062944.png)

- All user level threads are mapped to 1 kernel level thread.
- User will manage the threads
- Pros
  - Portable
  - Not limited by OS limits and policies
- Cons
  - OS has no insights into application as threading is managed within the user process
  - OS may block entire process if one user level thread blocks on I/O

### Many to Many

![](./img/Pasted%20image%2020210830062930.png)

- Hybrid of both approaches mentioned above
- Pros
  - Kernel knows about the inside of the process as it will assign some threads on a 1:1 basis
- Cons
  - Requires coordination between user/kernel level thread managers
    - usually for performance opportunities

## Scope of Multi-threading

- Different levels where multi-threading is supported
  - Levels reflect the scope of multi-threading in relation to what the OS sees
    - E.g. If OS is aware of all threads in a process it might be able to allocate resources more fairly vs if the threads in an process are simply managed by user level thread library
- Kernel
  - System wide scope
    - OS will look at the entire platform before making decisions on allocating resources
- User level
  - Process scope
    - User level library manages threads within a single process

## Multi-threading Patterns

- Boss workers
- Pipeline
- Layer

Use case:
![](./img/Pasted%20image%2020210830063826.png)

### Boss workers

- Boss
  - assigns work to workers

- Workers
  - perform entire task

- Throughput is limited by the boss thread.

- Measured as 1/boss_time_per_order

- Boss needs to be efficient if the system is to perform well.

- How does work get passed?
  - Keep track of which workers are available
    - Signal to worker to do work
    - Cons: Boss has to do extra work here, it has to keep track of workers and wait for them to accept work
    - Pros: Workers do not have to synchronize with each other.

- Establish queue between workers and boss
  - Place work in queue, when worker is free it can pick work from the queue
  - Boss and workers have to do extra work to make sure they're not stepping on each other's toes
    - Still performs better

#### How many workers?

- If work queue is full, boss cannot add any more to the queue which causes the boss to get backed up.
  - probability of queue filling up is based on the # of workers

- How to solve?
  - Create workers on demand?
    - Cost of creating workers on the fly might be too high.
  - Create a pool of workers at the beginning?
    - Less overhead vs creating 1 thread on demand everytime
  - Create a pool **and** increase the size of it via demand
    - best of both worlds

- Benefit
  - Simple, you only need 1 boss and many workers

- Downfall
  - Thead-pool management overhead
  - Locality
    - Boss doesn't know anything about threads completing a specific task, it is possible to assign a type of task to a thread that was working on a similar one. For example if a task had a subtask of square rooting 25, and a thread already has it in a local thread storage, it is better for that thread to process that task.

#### Worker Variants

- All workers created equal
- Specialized workers
  - Boss has to do more work per order if this is the case
  - Can be more efficient, because it exploits locality
    - hotter cache
  - Quality of service management
    - can increase thread count in specific tasks that require more resources
  - Adds complexity of load balancing to the equation
    - how many threads per task?

### Pipeline

![](./img/Pasted%20image%2020210831053256.png)
![](./img/Pasted%20image%2020210831053612.png)

- Threads assigned one subtask in the system
- entire task is pipeline of threads
- multiple tasks in the system can be in different stages in the pipeline
- throughput depends on the weakest link in the system
  - can be mitigated with multiple threads assigned to the weakest link
- how to pass work between each stage?
  - use a shared-buffer communication between stages
- Pros
  - specialization
    - results locality
- Cons
  - Complex to maintain balancing and synchronization
  - More synchronization
  - Extra overhead

### Layer

![](./img/Pasted%20image%2020210831054848.png)

- Each layer is assigned a group of related tasks
  - A thread in the layer can perform _any_ subtasks in the layer
- Task must pass up and down all layers
- Pros
  - Specialization
    - locality
  - Less fine grained than pipeline
    - easy to decide how many threads to assign per layer
- Cons
  - not suitable for all applications
  - heavy synchronization needed

### Quiz

![](./img/Pasted%20image%2020210831055523.png)

**Boss-Worker Formula:**

-   time_to_finish_1_order * ceiling (num_orders / num_concurrent_threads)

**Pipeline Formula:**

-   time_to_finish_first_order + (remaining_orders * time_to_finish_last_stage)


## Summary

- Threads vs Processes
- What are threads?
- How and why we use threads?
- Thread mechanisms for sync
- Design challenges and solutions related to multi-threading.

