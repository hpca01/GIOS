---
date updated: '2021-08-25T07:04:37-07:00'

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

