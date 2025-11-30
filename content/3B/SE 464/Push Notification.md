Push notification are how we handle the case where the **server** wants to start an interaction. Up to now, we have only talked about the client wanting to start an interaction.

**Email is a simple way for us to achieve this**:
- Many services still do it today. Just have to connect to an SMTP server
- These services are offered by every cloud provider and other third parties
- SMS Does the same


Solution - Push notification Services

### Push notification Services
- Client's OS makes one connection for all apps
- Services send notifications through third part (Apple or Google)

![[Pasted image 20251130155716.png]]

**Smartphones:**
- Location registration is used for iOS and Android 
- Whenever phone gets a new IP address, the OS opens a long-lived connection to the PNS which stores `<userid, address>`
- Smartphone apps also have to connect to the PNS - sending `<userid, message>`
- To deal with NAT - OS sends keepalive msgs, just one port is needed for all 

**Device Registration**: 
- Every time a decide connects, OS creates a long-lived connection
1. App registers for notifications
2. PNS returns a unique push ID
3. App sends push ID to its backend service which stores the users push ID for later
4. Backend gets push ID for that user from the DB - send notification request to PNS
5. PNS uses push ID to send notification to user


### Websockets
Websockets are how we pull data from web servers - as they are designed to only send data and polling is an inefficient method in practice. (Used long-polling before websockets)
- Long-lived bi-directional network connection
- Similar to a TCP socket, but it's available to Javascript code in a browser
- JS app creates websocket connection to a server, can send API requests through it and responses come back through the websocket
- The connection stays open and server can send messages at any time, independent of client requests

**Challenges**:
- Whether using APN, GCM, Websockets, or HTTP long-polling, the challenge is finding the **one** long-lived connection to the client
- A socket is tied to one IP address 
- Notifications can come from anywhere, and this is why they are often a separate microservice

![[Pasted image 20251130160811.png]]

Services like AWS API gateway handles requests by serverless functions (lambdas) - when connection is established, save connection id. Later use the connection id to push data to clients.
