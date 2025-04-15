Allows more processes to be maintained in main memory, and also allows processes to be larger than all of main memory!

**Resident Set**: Portion of program actually in main memory

**Working Set**: set of pages the process has accessed in the near past

**Requirements**: 
- Hardware support - MMU to check if page segment is in real memory 
- OS Software support - manage replacement policies

**TLB (Translation Lookaside Buffer)**: 
 - Caches page table entries in a hardware cache
 - Provides another layer of checking if the item exists
	 - Takes in Virtual Page #, checks in buffer, then finds PT entries
 - Significant overhead

**Final Solution:** 
 - Start with segmentation mechanism, which is visible to programmer and must be places in memory
 - Use the segment to find page # with paging mechanism
 - Use paging mechanism to find frame # and offset, then grab the address data from memory
 - We can add control bits in the pages and segments to ensure protection between different users/user mode/kernel mode, or to share between them

**fork()**: Code is shared, but each process has it's own data (including stack) 
- The page isn't actually copied, only when one of the two pages tries to modify (not read) the page


#### Fetch Policy
 How many pages to fetch?
 - **Demand Paging** brings page into memory only when used
 - **Prepaging** brings in more pages than needed 
	 - Less page faults (especially at start)

#### Replacement Policy
Which page to kick out
1. Determine set of pages that are candidates (only current process or any page are options)
2. Pick from candidates
- We assume 1 is given

##### Algorithms:
1. Optimal policy 
	- Used as a comparison
2. LRU (Least recently used)
3. FIFO 
	- Page in memory the longest gets kicked out
4. Clock policy - Called a use bit
	- Runs a circular Buffer and marks as used, checking for not used ones
5. Frame Locking
	- Makes it so certain pages cannot be replaced (kernel frames) - add lock bit
6. Page Buffering
	- Implements cache for memory pages
	- replaced page is moved to a buffer (modified + non-modified ones)
	- buffer emptied at a later time

#### Resident Set policy
How many pages to keep
- **Fixed allocation** gives a process a fixed number of pages to use
	-  Local scope - high page fault rate, can't predict what the process needs
- **Variable allocation** gives a process a variable number of pages
	- Local Scope - Select page to replace from current processes pages
		- No impact on other processes page space
	- Global Scope - Select page from all processes pages
		- Could shrink size of other processes resident sets

#### Working Set
Best size for resident set?
- Use principle of locality
- $W(t, \Delta)$ is the set of pages associated with the $\Delta$ references in $[t-\Delta + 1, t]$ 
- keep the pages that were used in the last $\Delta$ time (which we pick delta)
#### Cleaning Policy
When to write back modified pages
- **Demand Cleaning** Page is written only when selected for replacement
	- Requires two transfers per fault
- **Pre-Cleaning** Writes modified pages to disk before they are selected for replacement 
	- Write may be unecessary

#### Load Control
How many process can I have at a time?
 - Too few processes - much time spent in swapping, all processes will be blocked frequently
 - Too many processes will lead to thrashing
 - Monitor working set or page fault frequency to find sweet spot

