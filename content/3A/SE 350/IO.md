Most I/O devices are extremely slow conpared to the processor, so we need to handle them as fast as possible to ensure it doesn't block processes unnecessarily

### Structure

User Processes -> **Logical I/O** -> **Device I/O** -> **Scheduling & Control** -> Hardware

#### Logical I/O
- Treats device like a logical resource w/ no regards for control 
- Manages I/O functions for user processes
- ie. Filesystem has directories, file ops, networks have sockets, TCP stacks

#### Device I/O
 - converts requested operations into sequences of I/O instructions, commands, orders

#### Scheduling & Control
- Handles queuing and scheduling of I/O operations
- handles interrupts, checks I/O status

### Buffering
-  On a read, OS allocates a buffer in kernel memory where the I/O writes input data
	- OS copies buffer content to the requesting processes memory space
- On a write, OS copies from the user process to the buffer (kernel)

This gives protection (kernel buffer, user process) and improves performance, as after data is copied new data can be placed in the buffer

- A **Simple Buffer** is similar to producer/consumer
	- Can hold one data unit (single buffer) two (double buffer) or more (circular buffering)
	- Good solution for sequential access (keyboard)
- A **Buffer cache** divides storage into blocks (like a cache) 
	- Good solution for a random access device (like an SSD)
	- uses locality

**Replacement Policy**:
- LRU is **too expensive** for page management
	- Lots of pages in resident set + accessed every CPU cycle
- LRU is **OK** for buffer cache management
	- I/O more infrequent and can be done in software

**SSDs**
- Storage drives are the most important peripheral
- Use flash memory to store data
- reduced access time and better reliability
	- But, can only be written a limited number of times before it stops working
- Contains multiple memory chips, which can be accessed in parallel
	- each chip has many blocks, which have many pages
	- pages cannot be overwritten, have to remove a block first
		- Read whole content of block
		- Write whole content (including modification) to another location
		- Erase old block