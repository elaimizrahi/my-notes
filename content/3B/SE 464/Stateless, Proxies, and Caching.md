
## Stateless Services
Stateless threads/processes/services remember nothing from past requests. This allows us to split load as evenly as possible across 


**Stateless**
- Behaviour is determined entirely by two things - `<input, request code>`
- Different copies of the service are running the same code, so they give exact same response for a request
- No local state
- Like a "pure function" - output is not deterministic though as there can be random behaviour for the function
- Trivially easy to scale horizontally

**Stateful**
- Changes over time, as a side effect of handling requests
- Persistent, global variables are modified by the request processing code
- Like calling a mutator on a class - can change the global variables
- Related requests (from the same client) must go to the same handler (harder to do)

Although stateless seems like the obvious choice for web, we still need to handle things like logins that are stateful
- This is done using **cookies** which are stored client-side and sent with the request to get a stateless result (`<elais-cookie, get/articles` -> `my articles`)

**Example:**
![[Pasted image 20251124133131.png]]

## Proxies + Caches
A proxy server is an intermediary router for requests. It does now know how to answer requests, but it knows who to ask. The request is relayed to another server and the response is relayed back.

**Load balancers**:
- Type of proxy that connects to many app servers
- The work done by the load balancer is very simple, so it can handle more load than an app server
- Creates a single point of contact for a large cluster of app servers

**Front-end Cache**
Used to distribute front-end requests (web) and return HTTP data
- Frequent requests are found in the cache, without re-asking MediaWiki and accessing the shared Database
- Unusual requests are not in the cache, and are relayed to MediaWiki (the DB)

#### **Cache Basics**
Caching is a general concept that applies to web servers, computer memory, filesystems, DB's, and more. It is a small data storage structure designed to improve performance when accessing a large data store.
- Stores the most recently or most frequently used data through dictionaries or maps

**Hits and Misses**
Cache is small, so when reading the data:
1. Check cache first, hopefully data is there (hit) 
2. If data was not in cache, it's a cache miss
	1. Get data from main data store
	2. Make room in the cache by evicting another element
	3. Store data in the cache and repeat
**The most common eviction policy is Least Recently Used (LRU)**

##### Types of Caches

| Managed Cache | Transparent Cache |
|---------------|-------------------|
| Client has direct access to both the small and large data stores. | Client connects to a single data store. |
| Client is responsible for implementing caching logic. | Caching is implemented inside a storage “black box.” |
| Examples: Redis, Memcached | Examples: Squid caching proxy, CDNs, database server |
*Note: A small fraction of data usually accounts for a large fraction of the traffic*

**Problem: Data writes will cause the cache to be out of date**
e.g. you cache an article, but then in the DB the article contents have now changed. How do we update the cache?
1. **Expire** cache entries after a certain time-to-live (TTL)
2. After writes, send new data or invalidation message to all caches (makes a **coherent cache** but adds overhead)
3. Don't ever change data - instead create a new filename each time data is added - this is **versioned data**

**HTTP supports caching well:**
- Since HTTP is stateless, the same response can be saved and reused fro repeats of the same request (GET article) 
- HTTP has **Cache-Control** headers for both client and server to enable/disable caching and control expiration time
- Allows web browser to skip repeated requests

![[Pasted image 20251124135601.png]]