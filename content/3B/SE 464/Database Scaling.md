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


#### Database Partitioning
![[Pasted image 20251124144353.png | 300]]

- Nodes represent database rows
- Edges represent references (FKs)

**We want to assign the rows to shards (partition) so that total edges between partitions are minimized and nodes per partition is balanced**

**Challenges:**
- Solving the problem is NP-complete, but this model is too simplistic
	- What if certain rows are queried more than others?
- Even with optimal partitioning, we still have data references between partitions

#### NoSQL - Denormalized data
NoSQL eliminates the edges (references) to make data partitioning trivial. Instead of following references with JOINs, we store denormalized data with copies of referenced data. This is a Distributed Hash table (DHT)

![[Pasted image 20251130090933.png]]

- Without references, iti's impossible to define a schema
	- Don't know how many columns to add 
- Just one table (even though we can sometimes allow multiple) because with dynamic rows it doesn't really matter to have more
- Just one indexed column (the key) 
- Denormalized data is duplicated

**NoSQL Databases use hashing to easily query and find documents in the DB**
- Hash table stores (key, value) pairs along with hash indexes to find the data in one step. But it does not support efficient **range** queries

**Distributed Hash table**: 
- Each cluster node is responsible for a range of hash values
- Each client gets the list of nodes and the range assignment 
- Client determines which node to query for the data based on the hash value

This is how NoSQL databases are implemented. Like one big table with just a primary key - they have a map/dictionary interface and do not support SQL queries.

We call this a **Distributed, Shared-nothing architecture** and is seen in MongoDB, DynamoDB, Cassandra, and more.
- Both throughput and capacity are directly proportional to the number of nodes. DHTs can scale to thousands of nodes

**Robust + Consistency:**
- Having many nodes means a high chance of node failure, so we replicate data to avoid data loss. 
	- To avoid this, we create some overlap in the hash ranges covered by nodes - simple and effective
- Whenever data is replicated, there is a possibility of inconsistency

**CAP Theorem**:
- **Consistency**: reads always get the most recent write (or an error) 
- **Availability**: every request received a non-error response
- **Partition Tolerance**: An arbitrary number of messages between nodes can be dropped or delayed

**We can only pick TWO of these three when building a distributed DB**


#### Client-Centric Consistency Properties
- Monotonic Reads
	- If a client reads the value of **x**, later reads of x *by that same client* will always return the same value or a more recently written value
	- This might fail when reading from two different nodes during an incomplete write
- Read your writes
	- If a client writes a value to **x**, later reads of x *by the same client* will always return the same value or a more recently written value
	- This might fail when reading from two different nodes during an incomplete write
- Monotonic Writes
	- If a client writes twice to **x**, the first write must happen before the second

These can be solved by having the client do every request to the same node and adding a timestamp/seq number on each request. We can also make the client wait until the request is updated on the whole system

**Quorums**: 
- Quorums are a set of solutions for consistence in distributed DBs. They are a minimum percentage of a committee needed to act. \
- Wait for an acknowledgement of consistent data from a certain number of replicas before considering the read/write completed

![[Pasted image 20251130093303.png]]
- We send requests to all nodes but wait for the prescribed # of responses

**Majority-read, Majority-write example**
- Client wants to write X = 1
	- Sends three write requests to three replicas
	- When an ack from two replicas is received it can proceed
	- Last write proceeds in the background
- Client reads X
	- Sends three read requests to three replicas
	- At this point one of the replicas may still have old data, but that's OK
	- Client will be satisfied when it receives two responses. If different, use the most recent one

**Singe-read, unanimous-write Example:**
- Client wants to write X = 1
	- Sends three write requests to three replicas
	- Must wait until **all three replicas** ack before proceeding
- Client reads X
	- Sends three read requests to three replicas
	- At this point all replicas have received write, so client will be satisfied when it receives any one response
Writes are slow, but reads are fast (usually we're ok with this)

**This is scalable when we partition the data**

- A distributed system is **Linearizable** if the partial ordering of distributed action is preserved
	- The distributed actors each know the order of their own action
	- This certain knowledge must never be contradicted by the distributed system
	- This creates a partial ordering of all the events in the distributed system
		- e.g. If Anita does A,B,C in that order
		- And Sam does S,T,U
		- Then no one should see B before A, nor U before T

## Choosing a Database

![[Pasted image 20251130094737.png]]

### Comparison: NoSQL Database vs Distributed Cache

#### NoSQL Database
- Items are **persistent**.
- All items are stored **on disk** (some may be cached in RAM).
- **Scalability** is the primary design goal.

#### Distributed Cache
- Items typically **expire**.
- Stored **in RAM** (may be optionally persisted).
- **Speed** is the primary goal.
- Limited by **RAM capacity**.
- When full, old or least-used items are **evicted**.
    
### CDN / Reverse Proxy Cache vs Distributed Cache

#### CDN / Reverse Proxy Cache
- Caches **HTTP responses**.
- Fully **transparent to the application**.
- Only requires configuring the **origin server**.
- Great for static assets, images, video chunks, HTML, etc.
#### Distributed Cache (e.g., Redis)
- Caches **application-level data** that contributes to responses.
- Examples:
    - Leaderboard values used across many pages.
    - User session data.
    - Frequently accessed product info.

### Redis Overview (v3.2)

#### Key Characteristics
- Extremely **fast in-memory** data store.
- Written in **C**, BSD licensed.
- Disk-backed **in-memory** database.
- Supports **master–slave replication** and automatic failover.
- Simple key–value API with **complex data structures**:
    - Lists, sets, sorted sets
    - Hashes (multi-field objects)
    - Bit/bitfield ops (used for bloom filters)
- Supports:
    - **Transactions**
    - **Lua scripting**
    - **Pub/Sub** messaging
    - **Key expiration**
    - **Geo API** (query by radius)
    - Atomic operations like `INCR` (useful for rate limiting & counters)

##### Best Uses
- Rapidly changing data that fits mostly in memory.
- Good when size is predictable.
- Examples:
    - Real-time stock prices
    - Real-time analytics
    - Leaderboards
    - Chat systems
    - Anywhere you would previously use memcached
        
### Filesystem Choices

#### Data Stores and Abstractions

|Type|Examples|Data Model|
|---|---|---|
|SQL Relational DB|MySQL, Oracle|Tables, rows, columns|
|Column-oriented DB|Snowflake, BigQuery|Tables, rows, columns|
|Search Engine|Elasticsearch|JSON, text|
|Document Store|MongoDB|Key → JSON|
|Distributed Cache|Redis|Key → values / data structures|
|NoSQL DB|Cassandra, DynamoDB|2D key-value / pseudo columns|
|Cloud Object Store|S3, Azure Blobs|Key-value or filename → contents|
|Cluster Filesystem|HDFS|Filename → contents|
|Networked Filesystem|NFS, EFS, EBS|Files with arbitrary data|

### Networked File System (NFS)
- OS-level mounting of a remote filesystem.
- Applications see it as a normal local directory.
- Not widely used by modern cloud-native apps.
- Mostly needed by legacy systems requiring local filesystem semantics.
- Recommended to migrate to **cloud-based storage** when possible.
    
### Cloud Object Store (e.g., S3)
- Highly scalable, general-purpose **remote file storage**.
- Exposed via **network API** (REST).
- Stores files like a remote database.
- Frequently provides **public HTTP GET** capabilities.
- Easily connected to **CDN**.
- Commonly used for:
    - Images
    - Videos
    - Backups
    - Static website assets

##### Example Use of S3 for Hosting Media
HTML can directly reference S3-hosted files via URLs:
- `https://s3-us-west-2.amazonaws.com/...`
- Allows browser to fetch large files directly from S3 through HTTP.
- Useful for hosting large datasets, downloads, or media assets.

### Hadoop Distributed File System (HDFS)
- Used for **distributed processing** with Hadoop/Spark.
- Data is stored on the same machines that perform computation.
- Used when datasets are too large to move around.
- Designed for big data analytics workloads.
    
### Recap: Choosing a Data Store
Choice depends on:
- **Structure of the data** (structured vs semi-structured vs blobs)
- **Patterns of access** (latency, throughput, consistency needs)
    
General mapping:
- Highly structured → SQL / Column Store
- Semi-structured JSON → Elasticsearch / MongoDB
- Key/value or rapidly changing data → Redis / DynamoDB
- Large files / media → S3 or blob storage
- Massive-scale distributed analytics → HDFS