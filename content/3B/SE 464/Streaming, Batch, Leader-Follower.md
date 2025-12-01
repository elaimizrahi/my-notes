
## Streaming vs. Batch Processing 
**Easy Problems:**
- handle independent requests from millions of customers
- Assume the data is already available
- can parallelize this easily
**Hard Problems:**
- Perform a calculation using all the data from millions of customers (i.e. recommendation system)
- Requires lots of coordination with data
- Must use MapReduce, Spark, etc. 
- Can take lots of time (That's ok)

### Big Data
- Exponential growth of data
- big data processing cycle
- Importance of Data analytics
- Data volume increase
- Cloud Computing influence

If we want to analyze 10 PB of data each day, we can theoretically do this, but it could take hours (math checks out to > 35 hours at least). To speed this up, we need to use many machines to work on the problem in parallel.

**Distributed Computing**:
 - Data and analysis are split across many machines
 - No one machine has access to all the data
 - Machines need to communicate over a network to share data and results
 - Many rounds of communication may be needed for an algorithm

Some work can use very little communication:
- Get max, min, sum, mean 
To do something like get the median though, we need to have a **leader** that picks our pivot and shares it with others to perform quickselect.

**Hadoop**
- A distributed computing platform for storing data and running analyses
- **HDFS** (Hadoop distributed file system) is used for storing the data and stores intermediate and final results
- **MapReduce** is a programming model for defining the parallel analysis steps. Usually written in Java
	- First, Map does something, producing many intermediary results
	- Later, Reduce aggregates the intermediate results
	- Programmer just needs to define these, and they can work in parallel. 
- Provides a runtime system to manage job scheduling and coordination
- Provides sorting algorithm for the "shuffling" step

![[Pasted image 20251201125703.png]]
**After the reduce, it also does a shuffle step that re-organizes the data to new nodes to perform the next map-reduce**

### Batch Processing
- Processes large data chunks, data is already stored over time
- Processed in sequential order
- Good for large jobs, and works offline to save resources
- used for financial transactions, non-time-sensitive jobs

### Stream Processing
 - Real-time data processing for large, fast-moving data streams
 - Low latency and continuous data flow
 - Immediate insights and quick decision making
 - Good for fraud detection for example

**Apache Stream**:
- Real-time computation system
- Uses spouts as data sources
- Bolts as data processing units
- Scalable and fault tolerant


## Leader-Follower Architecture
- A concurrency pattern for distributed systems
- Dynamic role assignment among threads
- Leader handles events, followers await their turn
	- Single active leader thread
	- Pool of passive follower threads
	- Event handling done by leader demulitplexing and dispatching events

This is used for high-volume transaction systems, resources constrained environments, and complex even processing
- Better resource efficiency, scalability, and flexibility
- Better throughput, simpler synchronization, and better fault tolerance
- If the leader fails, it starts an election for a new leader
This adds complexity in design and maintenance though, and adds bottlenecks and harder debugging


