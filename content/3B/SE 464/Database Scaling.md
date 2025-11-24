**We want to ensure that our databases are scalable, fast, and robust as the amount of data + requests increases over time**

### Review
- **Computers have a hierarchy of storage** 
	- Registers -> CPU Cache -> RAM -> SSD (flash storage) -> Magnetic Disk 
	- Disk is about **ten billion** times larger than registers, but has about **ten million** times longer delay
	- Ideally we only want to work with the fastest levels
- **Storage has limited bandwidth**
	- All types of storage can only read/write a small fraction of the data at once

### **Redundant Array of Independent Disks (RAID)**
- Disks have limited capacity, throughput, and are more likely to fail
- RAID combines many disks to create one **superior virtual disk**
- Provides same interface as regular disk
- We use this to setup database servers
![[Pasted image 20250922101016.png]]

**Note:** programming languages rarely deal with storage directly (except SQL)

**Databases** let the programmer use persistent storage w/o worrying about file-level transfers, with advanced performance optimizations
- We usually interact with the DB using a special query language (eg SQL)

### Relational DBs
- Store data in multiple **tables** with each table having a predefined schema
- Rows can refer to other rows through **foreign keys**
![[Pasted image 20250922102119.png]]

- **Advantages**
	- **Scalability** – work with data larger than computer’s RAM.
	- **Persistence** – keep data around after your program finishes.
    - **Indexing** – efficiently sort & search along various dimensions.
    - **Concurrency** – multiple users or applications can read/write.
    - **Analysis** – SQL query language is concise yet powerful.
    - **Integrity** – restrict data type, disallow duplicate entries, transactions.
    - **Deduplication** – save space, keep common data consistent.
    - **Security** – different users can have access to specific data.
- **A filesystem doesn't have analysis, integrity, and deduplication. Also it only has some indexing and concurrency**

##### Indexing 
We want to be able to search through and find items in the DB without checking each out
- Sort and use binary search (index the data)
	- But, this means we can't sort in multiple dimensions (name + price)
	- Can't insert without shifting other data (about half on average)
- Tree/Hashtables
	- Self balanced binary trees give the log(N) speed of a binary search while also allowing entries to be quickly added and deleted
- Indexes are defined when the table is created (primary key is indexed, must be unique)
- Need two indexes to quickly get results for both criteria
- Since the DBMS must translate the query into a series of table lookups:
	- A complex query could have many ways of being performed, but the presence of indexes make some choices more efficient that others
- Indices consume storage space (overhead), so we need to be mindful when using them

When do we index a column?
- When the query is **slow**
- More specifically:
	- When the column is used in `WHERE`, `JOIN.. ON` 
	- When a foreign key refers to it
	- If the column is in a `MIN` or `MAX` aggregation function


## Database Scaling
Usually scalable service design relies on a single shared database for **coordination**. This causes a bottleneck within the system at the database

#### Read Replicas
- Often, >95% of DB traffic is reads. Since reads are stateless, we create replica servers that each have a **full copy** of the data and they only handle read requests
- All writes must go to the primary server - which are then pushed to read replicas
- this means the read replicas might be slightly behind the primary, so sensitive reads should use the primary DB
	- ![[Pasted image 20250922105213.png]]
- **Issues**:
	- Not super scalable, the primary DB is a bottleneck and single point of failure. 
	- If there are $N$ replicas, primary must send $N$ copies of each write
	- If there are $R$ times as many reads as writes, and we want to equalize load on Primary and replicas, we get $N = R$ 

##### Multi-Level Replication
- Add middle layer read-replicas to ensure horizontal scalability
- don't read from the middle replicas - they only push writes down to the next level
- Each level adds more **delay** between write at primary and data availability at replicas

**To use read-replicas, have a load balancer that serves and distributes the read requests through the read-replicas. Writes go only to the primary DB.**

![[Pasted image 20250922105804.png]]

**Notes:**
- Writes are not scalable
- Capacity is not scalable
- Primary DB is a **single point of failure**

We can prevent failures by having a standby primary DB that can take over if the main primary fails (backup)

### Sharding
We want horizontal scaling of writes and capacity
- some kind of partitioning is needed
	- **Functional Partitioning** creates multiple databases for each type/category of data
		- This limits queries joining rows in tables in different DBs (divide by tables)
	- **Data Partitioning** is a more general approach (divide by rows)
- Divide the data into disjoint subsets called **shards**
	- for example in Facebook, we can split by Illinois data, Wisconsin Data, NY data, etc.
	- **Sharding key** determines assignment of rows to shards
- Sharding is usually not supported in relational databases, so it must be hacked at the application
![[Pasted image 20250922110332.png]]
![[Pasted image 20250922110345.png]]


### Normalized Data
A normalized relation database has no duplication of data. References (foreign keys) point to shared data
- To optimally partition the rows into shards, we could solve a balanced graph partitioning problem

![[Pasted image 20251124144353.png | 300]]

- Nodes represent database rows
- Edges represent references (FKs)

**We want to assign the rows to shards (partition) so that total edges between partitions are minimized and nodes per partition is balanced**