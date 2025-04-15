We use control and status registers to keep track of the current process, its permissions, and more as we start switching between tasks

#### Control + Status Registers
**PC (program counter):** Contains address of next instruction
**IR (instruction register):** Contains most recent instruction
**SP (Stack Pointer):** Points to top of stack
**MAR (memory address register):** specifies address for next read + write
**MBR (memory buffer register):** contains data written or read into memory
**PSW (program status word):** Holds status information (interrupts enabled) and flags (kernel or thread mode)

#### Example of program execution
![[Pasted image 20250227192840.png]]


## Interrupts 
Processor must pause to wait for an interrupt to finish once it is called

Classes of interrupts: 
- Program (ie division by zero)
- Timer (took too long)
- I/O (must wait for I/O to return data)
- Hardware Failure (fatal error)
![[Pasted image 20250227193028.png]]

**Special Registers** exist as well to make interrupt handling easier
- **PSP (process stack pointer)** is used in thread mode to keep track of the processes stack
- **MSP (main stack pointer)** is used in thread or handler mode to keep track of the kernel stack 
- **PSR (program status register)** keeps flags for certain conditions
	- Interrupt program status (bits 8-0), if 0 in thread mode else handler mode
	- Thumb mode (bit 24)
- **Control Register** 
	- Bit 0: unprivileged (0) or privileged (1)
	- Bit 1: MSP (0) or PSP (1)

### Exception Return
![[Pasted image 20250227205805.png]]
Note: use `SVC #imm` to trigger a supervisor call (interrupt)

### Multiple Interrupts
We can either use **Sequential Interrupts** or **Nested Interrupts**

By assigning priorities nested interrupts can be handled accordingly, but they are harder to debug and use more space on the stack

Nested interrupts provide easier methods of implementing interrupts and ensure that each process can be handled before any critical errors occur

## I/O

**Interrupt Driven** polls for an interrupt in the I/O module and if found, handles it by calling the appropriate interrupt handler
- Pros: Don't have to wait for interrupt
- Cons: more complicated (more overhead) and takes more time to implement compared to programmed I/O, also consumes lots of processor time

**Direct Memory Access (DMA)** transfers a block directly to or from memory, interrupt is only sent once complete
- Pros: more efficient (less interrupts)
- Cons: Competes with processor for memory access

