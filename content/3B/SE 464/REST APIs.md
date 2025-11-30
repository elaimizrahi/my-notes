
## Application Programming Interfaces (APIs)
An API defines how software can be used by other software. 
- The API for a code library is the list of functions/classes it provides
- Software services provide network remote procedure call (RPC APIs)

### HTTP methods and responses
- Methods:
	- GET: request data
	- POST: post data to server to be processed
	- PUT: Create or replace a new document
	- DELETE: delete a document
	- HEAD: like GET, but just returns headers
- Responses: 
	- 200 OK: success
	- 301 Moved Permanently: redirects to another URL
	- 403 Forbidden: Lack permission
	- 404 Not found: URL is bad
	- 500 Internal Server Error: server issue
	- And many more

### Idempotence
An idempotent request can be repeated without changing the result. HTTP expects every method **except POST** to be idempotent.
-   HTTP proxies/servers may repeat your PUT or DELETE requests, and your REST API implementations should be OK with this
	- It should not make any duplicate docs
- e.g. we can't do DELETE /user/user-id/feed/posts/latest

### RESTful API Design Style
- Paths represent resources
- Want to represent arbitrary actions 
- e.g. this works, but is not RESTful: POST /inbox/createMessage
- A better alternative is: POST: /inbox/message/\[uuid\]

**We usually use JSON to describe the data format required for RESTful APIs**

#### XML
Older than JSON, but another language we can use to design query structure
- Comprised of text, tags, and attributes
- Similar looking to HTML
- `<person name="John"> <car> Ford </car> </person>`

**XML and JSON are data serialization formats. Allows us to convert data obejcts into a sequence of bytes**


### HTTP 
We use HTTP for many reasons when building new applications:
- Many common problems are already solved
	- Encryption
	- Compression
	- Libraries
	- Proxies (Squid, NginX)
- Sometimes inherits unneeded complexities, and perhaps unexpected behavioiurs
- Human-readable headers introduce overhead
- May have to rethink API to fit the URL/resource model

**Thrift and Protocol Buffers are alternative standards for networks APIs and are not build on top of HTTP**
- Messages are more space-efficient, but less readable
- Without HTTP overhead, less processing on both sides
- Specify list of functions for the API, and the tools generate libraries to easily use the API and language of choice
