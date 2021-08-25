---
date updated: '2021-08-24T06:44:50-07:00'

---

# OS

![simple_os_def](../mg/Pasted%20image%2020210823071224.png)

**Abstract** - simply what hardware looks

**Arbitrate** - control hardware use

## Visual Metaphor

A visual metaphor to help illustrate an OS:
![visual_metaphor](../mg/Pasted%20image%2020210823071458.png)

![os_fitting_vs](../mg/Pasted%20image%2020210823071644.png)

## What is an OS?

- Computing system consist of CPU, Memory, Network Connections, GPU(optional), Storage, USB devices
- Layer that sits between complex hardware and other applications(eg. Skype, MS Excel, etc)

![layer_os](../mg/Pasted%20image%2020210823071927.png)

![features_of_os](../mg/Pasted%20image%2020210823072134.png)

- Job
  - hiding hardware complexity
  - resource management
  - provide isolation and protection

### OS definition

Layer of systems software that:

- Directly has privileged access to the underlying hardware
- hides hardware complexity
- manages hardware on behalf of one or more applications, according to **pre-defined policies**
- Ensures isolation and protection for apps.

![official_os_def](../mg/Pasted%20image%2020210823072621.png)

- Parts of OS
  - File system
  - Device Driver
  - Scheduler

Why not CACHE MEMORY? OS does not DIRECTLY manage the cache, it is usually the apps that run on the OS that do it such as loading the stack and heap on and off.

- Abstraction(B)
  - makes it easy to use a type of device regardless of its specs
    - supporting different types of speakers
    - interchangeable access of hard disk or SSD

- Arbitration(R)
  - Does the actual work of managing resources
    - distributing memory between process

### Examples

- Desktop
  - MS Windows
  - Unix-based
    - Mac OSX
    - Linux
      - many variations

- Embedded
  - Android
  - iOS
  - Symbian

### Elements

- Abstractions
  - Process, threads
  - File, socket, memory page

- Mechanisms
  - create process, schedule process
  - open file/socket, write to file/socket, allocate memory page

- Policies
  - HOW the managing happens
  - Control the number of sockets an app has access to
  - LRU, earliest deadline first scheduling

#### Memory Management Example

- Abstraction - Memory Page
- Mechanism - allocate, map to a process
- Policies - LRU used to move the DRAM to disk

![os_elements_mem_mng_example](../mg/Pasted%20image%2020210824053853.png)

## Design Principles

- Separation of mechanisms and policy
  - Implement flexible mechanisms to support many policies
    - LRU, LFU, random
  - You don't want one size fits all solution, some settings may warrant the use of LRU over LFU and vice versa.
- Optimizing for the **common** case
  - Where will the OS be used?
    - Specs, eg ram, cpu, ssd/hdd, other devices attached.
  - What will the user want to execute on that machine?
    - will it be running a web browser only, or other things as well
  - What are the workload requirements?
    - what does that workload look like, are the apps being used very resource intensive or not

## User/Kernel Protection Boundary

- User Level
  - Applications ran by the user live here
  - CAN access hardware using **user-kernel switch**
    - "privilege bit" is set in CPU in Kernel mode
      - If access to hardware is requested while bit is **not** set then application will be trapped for trying to gain access without "privilege bit", then OS will decide what to do after interrupting the app.
    - System call interface
      - Set of operations the user level program can ask the OS to do on its behalf, this is usually performing privileged access.
        - open(file)
        - send(socket)
        - mmap(memory)
- Kernel(privileged) Level
  - OS kernel
  - Hardware access
    - RAM
    - CPU

### System Calls

![sys_call_flow_chart](../mg/Pasted%20image%2020210824060855.png)

- Not a cheap operation
  - Changing contexts
  - Passing arguments
  - Jumping around in memory
- To make a sys call an app needs to
  - Write arguments
  - Save relevant data at a well defined location
  - Make system call via its number
- Modes
  - Synchronous
    - Waits until the sys call completes
  - Asynchronous
    - Picks up when the sys call returns (eg the select system call)

![sys_call_arg_passing](../mg/Pasted%20image%2020210824060935.png)

### Crossing the OS Boundary

- User/Kernel transitions
  - hardware supported
    - eg. user instruction is trapped when cpu privilege bit is not set, and executed when it is set
      - application cannot change cpu register contents or give itself more processing time or memory
  - involves a number of to do the user-kernel transition
    - eg. takes ~50-100 nanoseconds on a 2ghz machine running linux
  - switches locality
    - affects hardware cache
      - in order to do the task requested by the application, the OS might have to eject application data from hardware cache(100x speed vs RAM).
      - Hot(cache) - all app's needs are in the cache
      - Cold(cache) - all app's needs are NOT in cache, it needs to go to RAM
  - NOT Cheap
- Necessary during app executions

## OS Services

- Services
  - Directly linked to components of hardware
    - Scheduling
      - processes
    - Memory manager
      - RAM management
    - Block device driver
      - Disk access
    - File system abstraction
  - Basic services OS needs to provide
    - Process management
    - File Management
    - Device management
    - Memory Management
    - Storage Management
    - Security

![system_calls_windows_vs_linux](../mg/Pasted%20image%2020210824063554.png)

![sys_calls_quiz](../mg/Pasted%20image%2020210824064147.png)

## Monolithic OS

![monolithic_os_img](../mg/Pasted%20image%2020210824064439.png)

- Pros
	- Everything included
	- Inlining and compile-time optimizations
- Cons
	- No customizations
	- Not portable/manageable
	- Too much code
	- Large footprint that impacts performance
- 
