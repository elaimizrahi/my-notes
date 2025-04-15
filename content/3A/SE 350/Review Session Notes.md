- favoured to second half
- Same amount of twists as midterm (same style)
- At least 1 concurrency question
- Some questions very difficult
	- expects most to get to 50%

**Round Robin** When scheduling, whatever comes in goes **BEFORE** the one that just got looped around

**Feedback scheduling** - once a process is done its quantum, doesn't matter if it has higher priority, it gets forced off and we must choose between other ones
- If there's other processes waiting, we should never have a process run for 2 quantas in a row
- if it's the only process it can run multiple times
- STATE ASSUMPTIONS FOR FEEDBACK IF NOT SURE!!

Steps for CLOCK replacement policy
 1. if cache not full, add item and give it a use bit
 2. After add, move pointer over one
 3. if cache full and pointer currently on item with use bit, remove use bit and move to next item
 4. once item without use bit found, replace it and give use bit
 5. If the item already exists when queried, give it a use bit

### Producer Consumer code

Look for patterns 
- Turnstiles
- why is there synchronized?
- Check problems in notes, it should follow a similar pattern

Read the list in the tutorial section for the **little book of semaphores** 😢

**LEARN THESE**
- barriers
- rendezvous
- cig smoker
- dining philosophers
- producer/consumer
- reusable barrier

- compare and swap stuff - understand how the semaphores work

Usually C1 and C2 will be mirrors of each other, just with different semaphores that it acts on

To solve:
1. what semaphores do you need? 
	1. For example maybe you need barriers to tell us P1 and P2 have arrived for **synchronization**
	2. need to store size of buffers (in sephamores) - you know if size is exceeded you can't add anything
	3. also keep track of when things are empty
	4. Need mutexes (semaphores) to keep track of when A and B are locked (act as a lock on the buffer themselves)
2. Now attempt to write code, follow similar structure to given ones
3. Producer processes will call `produce()` (P1 will produce A, P2 will produce B) 
4. Now set up your actual structure (rendezvous, dining, etc)


- Do tutorial set of semaphore problems
	- readers/writers
	- dining philosophers
	- cig smokers

**Message passing** 
- Don't think you need - can if you're comfortable
- if you want to use the light switch just call light switch pattern - same with message passing

Don't worry about anything on the midterm - nothing should have been repeated (he was pretty confident - just do a light review)

No disproportionate weighting (concurrency is a lot but not so much that you will fail if you miss it - still do it though)

Shouldn't be twists like adding hex
general math lol

some knowledge questions

Just answer every question

**Note:** exam concurrency question is easier than one in the review session
**Time** - most will be spent on concurrency by the nature of the problem, should have enough time for everything else

**Spring 2011 Q9** <- do this!!!
