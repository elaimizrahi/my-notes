Sometimes waiting for a response takes a long time. So we realize that sometimes it's ok to just acknowledge that the request was received, and just finish the work later. 
- Allows client to quickly move on
- But, client does not learn whether the request succceeded

How do we support this?

### Request Record
The server can store a request record in a DB and return the unique id. When it's done, the server updates the request record in the DB. The client can later check using the request id. 


### Callback to client
The client provides a callback function (webhook) where it expects to receive a response.
- This only works if the client can listen for responses (always running, not NATed, etc.)

### Side Channel for Feedback
We often let clients send "fire and forget" requests when failure is rare. When a failure does occur, the server sends it to another system. ie. if the email fails log an error for devops or customer support staff to review


## Things that would benefit
- Transfer file over network
- Fetch file from tape storage
- Create virtual machine
- Purchase Shopping cart
- Book a flight, concert, ticket
- Send a text

## Message Queues
If the client doesn't care about the response status, it can just put the request on a queue. Adding a message to a queue is fast because it's just a data copy, and prevents the slowdown of upstream service due to downstream congestion.
- Putting a message on a queue is like making an API request
- Content of the message defines the request

**Producer:** pushes/creates messages and adds to queue
**Consumer**: pulls/does something with messages and removes from queue

**Example: Twitter**
1. Client posts request and API receives it
2. Sends request to auth service to check user is valid
3. Puts tweet creation job on a queue to be processed later
4. Sends success response to user
5. later...
6. Publication service handles the request by fetching job from queue
7. Gets list of followers to receive the tweet
8. Adds tweet to followers feeds
9. Queue next task
10. Notification service alerts users

This can be faster - but errors usually get ignored until much later than normal

**This decoupling with message queues helps with scaling, as many producers and consumers can connect at once**

**Active Queues:**
 - Queue knows where to send messages
 - Actively pushes messages out to subscribers
 - Subscribers listen for messages
 
**Passive queues:**
- Queue accepts and stores messages until they are requested
- Queue is a specialized DB
- Consumers must request (poll)
- Producer pushes and consumer pulls

![[Pasted image 20251201122943.png]]

**Full Queues:**
 - Don't want to reject a message because queue is full
 - Devs should keep an eye on queue sizes and be sure to scale if needed
 - Ordering
	 - If messages need to be ordered, combine them into one message

**Note:** The frontend should **NEVER** be able to access the backend queues


