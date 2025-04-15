We need a method of storing and organizing collections of data with these properties:
1. Long term storage
2. Sharable data between processes
3. Structured data organization

And it should have the usual file functions:
- create
- delete
- open
- close
- read
- write

Two types of files:
1. **Unstructured**: file is a stream of bytes, supports random access, general purpose systems
2. **Structured**: file is a collection of records, composed of multiple fields with fixed/variable length, usually used in databases

Record operations:
- retrieve_all
- retrieve_one
- retrieve_next
- retrieve_previous
- insert_one
- delete_one
- update_one


**Indexing** - providing quick lookup capability to reach the vicinity of the desired record
- contains key field and pointer to the file
- finds highest key value >= input, then search continues in main file from pointer location

**B-Tree** - data structure to implement large indexes
- build balanced tree with each node having many children, fit each node in one disk block
- deletion much more complex than insertion

### Directories
File directories store information about files (name, type, location is secondary mem, access control, usage info)
- organizes files, maps files to names

**Operations**:
- search
- create
- delete
- list files in directory
- update directory

**Approaches:**
1. Single list of entries for each file
2. Tree structure where each directory may have subdirectories


### Block Management
Must divide file into fixed-size logical blocks and allocate these blocks in secondary memory - ie. associate logical and physical blocks
- For unstructured files, use paging
- Fixed blocking: integral number of fixed-length records - may cause internal fragmentation
- Variable-length unspanned blocking: v-length records packed into blocks with each record in a single block - still internal fragmentation
- Variable-length spanned blocking: Same as above, records can span two blocks - no space wasted but reading might need two blocks

**Secondary Memory**: 
- How much space to allocate?
	- **Preallocation** - alloc max size right away
	- **Dynamic Allocation** - grow as needed
- Size of alloc units (portions)
	- Variable + large - similar to segmentation with external framentation
	- Fixed + small - Better flexibility and reduced fragmentation - possibly performance issues
- Allocation method
	- **File allocation table** (like page table)

#### Allocation methods
**Contiguous Allocation** - single preallocated partion is assigned to file
- similar to dynamic partitioning, need to perform compaction
**Chained Allocation** - Each block contains pointer to next block in chain
- bad for random access, requires defragmentation for performance
**Indexed Allocation** - File Index (B-Tree) points to fixed/variable size portions

**Free Space Management** - how we keep track of free blocks
- Bit table
	- One bit for each block - 1 TB disk w/ 512 byte blocks -> 265Mbytes table (expensive)
- Free Block List
	- Manage in FIFO or LIFO, only keep head or tail on disk
- Chained free portion
	- Use pointers in each block to chain them
	- No space overhead but expensive to walk the list
- Indexing
	- keep index of free portions


### Erasing
To erase a file:
- Remove file from directory
- Remove file from file allocation table
- Mark alloced blocks as free

**Note:** SSDs do not support overwriting pages - so the file system must explicitly erase first

### Reliability
If the system crashes during an erasure, we will permanently lose the data
- Solution: **Journaling** 
	- Log modifications in buffer before performing them
	- After failure, use logs to restore state before crash

### Security
Typically implemented with Discretionary Access Control 
- **Subjects** - users 
- **Objects** - files
We can represent the access control with an access matrix![[Pasted image 20250409104237.png]]

**Access Rights**
1. None - user may not know file exists 
2. Knowledge - User can see it exists and who the owner is
3. Execution - The user can load and execute a program but not copy
4. Reading - Can read for any purpose, including copying
5. Appending - Can add data but not modify or delete
6. Updating - Can add, modify, and delete data
7. Changing Protection - Can change access rights of other users
8. Deletion - Can delete file

These can be stored with: 
- Permission bits
- Access control list
- Capability tickets

**Note:** when files are shared, we must enforce reader/writer protocol of locking and unlocking files - usually easier with unstructured files

### Unix File Management
Uses device drivers to convert from file operations to I/O device operations
- **Character Devices** used for streaming devices - transfer bytes
- **Block Devices** used for random access devices - transfer blocks

**Inodes** - Index nodes are file control structures to store all attributes of files
- mutliple files names can be the same inode
- Modifying file modifies the corresponding inode
- Deleting only deletes inode if it's the node associated file

**Access Control**
- Each user belongs to 1+ groups
- Each file belongs to a user and a group
- 12 permission bits per file
- Read, Write, Execute permissions
- Set user ID and group ID
	- If set and file used, resulting process gets same access rights as user/group
- Sticky bit for directories - only file owner can modify directory for file
- Superuser is exempt from access control

**Linux page cache** - unified page buffer (virtual memory) and disk cache
- Each entry can be a virtual memory page, file system block, or both

