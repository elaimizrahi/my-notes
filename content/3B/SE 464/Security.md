*“The protection afforded to an automated information system in order to attain the applicable objectives of preserving the integrity, availability and confidentiality of information system resources (includes hardware, software, firmware, information/data, and telecommunications).”*  
-National Institute of Standards and Technology

## NFP
- **Confidentiality:** Preventing unauthorized parties from accessing the info
	- Is the agent’s claimed identity authentic?
- **Integrity**: Only authorized parties can manipulate the information and only in authorized ways
	- Is this request actually the one the agent made?
- **Availability:** Resources are available if they are accessible by authorized parties on all appropriate occasions
	- Has a proper authority granted permission to this agent to perform this action?

![[Pasted image 20251130170242.png]]
**This is the guard model**
 - Can have **Adversarial Attacks** which are targeted, rarely random, and rarely independent
 - Securing a system starts by saying our goals (policy) and assumptions (threat model) 
 - Guard model provides **complete mediation**

## Threats 
- **Unauthorized information release**
	- Unauthorized person can read and take advantage of information in the network
- **Unauthorized information modification**
	- An unauthorized person can make changes to the stored information in the network - They do not need to see the data to do this
- **Unauthorized denial of use**
	- Can prevent an authorized user from reading or modifying information, even though the adversary may not be able to read of modify the information


**Having a narrow view of security is dangerous because the objective of a secure system is to prevent *all* unauthorized actions**

## Design Principles for Security 
- **Least Privilege**:
	- Give each component only the privilege it requires
- **Fail-safe Defaults**
	- Deny access if explicit permission is absent
- **Economy of mechanism** 
	- Adopt simple security mechanisms (e.g. don't assume the inputs from the client)
- **Complete Mediation**
	- Ensure every access is permitted
- **Open Design**
	- do not rely on secrecy for security, it is not realistic and will never happen 
- **Separation of Privilege**
	- introduce multiple parties to avoid exploitation of privileges
- **Least Common Mechanism**
	- Limit critical resource sharing to only a few mechanisms
- **Psychological Acceptability**
	- Make security mechanisms usable
- **Defence in Depth**
	- Have multiple layers of countermeasures


## Authentication
1. Authentication: Is the principal who they claim to be?  
2. Authorization: does principal have access to performance request on resource?

To establish authentication, we need to ensure two facts:
1. Who is making the request?
2. Is this request actually the one that said person made, or has an adversary modified the message?

**Steps to authenticate**:
1. Rendezvous step: a real-world person configures the guard  - agreeing on a method by which the guard can later identify the principal identifier for the person
2. Verification of identity, which happens later using the agreed method (fingerprint, id, password, etc.)

**One notable example is HTTP -> HTTPS (TLS), HTTPS encrypts the data in transit and the data cannot be intercepted in transit**


#### Approaches
**Password in request:**

Send
```
{
	name: "Elai",
	password: "abc123"
}
```
This is **SUPER UNSAFE** as adversaries can pick up the message in transit, learn your password as it's in plaintext, then log into your account like it's you

Usually to avoid this we hash the password upon sending, but this can still cause issues:
- adversaries can set up "rainbow tables" to compute your password. In practice though it's almost impossible to find the input just from the hash without knowing the hash function
- **Dictionary Attack** - adversary compiles a list of likely passwords, then computes the hash of these strings and compares the result with the value stored in the system, or attempts to log in with these strings

**Avoiding Multiple Password Transmissions**:
Once the client is authenticated, the server sends it a "cookie" which it can use to keep authenticating itself for some period of time

But, adversaries can make their own cookies:
- So we use the same hashing based on some value (server key, username, expiration) to ensure that it's hard to replicate

These cookies are then sent with every request to tell the server who is making the request without having to authorize them every time. 

This is also done with **session keys** which are stored in the user account DB, and gets returned to the client to send with each request. 


![[Pasted image 20251201103110.png]]


**API Keys**:
API Keys are like authentication tokens that are valid for a long time. Usually it's to give a 3rd party access to your system. 

#### Preventing Phishing
We want to avoid where an adversary tricks a user into revealing their password. 

**Challenge-response protocol:**
- Before the user sends their password to the server, the server gives the user a random number and a hash function
- The user hashes their password with the hash and random number
- Then sends that hash back to the server, so the password is never actually sent
- If the adversary-owned server tries to replicate this, it only learns for one random number, and can't recover the password from that

**2FA:**
Sometimes we send another code to the user's email or SMS that verifies it is the actual person that owns the account. 


## Authorization
With authorization we need to decide whether access to a protected resource should be granted or denied. Usually this is based on the identity of the requestor, the resource, and whether the requestor has permission to access. This is called **Access Control**

**Ticket System**:
- Each guard holds a ticket for each object it is guarding 
- A principal holds a separate ticket for each different object the principal is authorized to use
- To authorize a principal to have access, authority gives the principal a matching ticket for the object
- principal can simple pass the ticket to other principals
- To revoke, either remove from principal or reissue tickets

**List System:**
- Each principal has a token identifying the principal (ie. its name) and the guard holds a list of tokens that correspond to the set of principals that the authority has authorized
- To revoke, the authority removes the principal's token from the guard's list

![[Pasted image 20251201104043.png]]

**Mandatory Access Control:** If you have access to "top secret" you have access to everything below that security level too

**Impersonation**: Malicious person tried to be another to get into a system

**Known-key attacks:** Malicious person takes authorized person's key while it's in transit

**Replay Attacks:** Malicious person intercepts the session while it's in transit and replays the same session 

**Misrepresentation**: Malicious person tells system that the authorized person is actually bad

**Collusion**: Backing up your misrepresentation with another malicious person

**Fraudulent Actions:** Not doing what you say you will - like not sending an item after a customer paid for it

**Addition of Unknowns:** Adding people we know nothing about causes others to lose some trust and reduces interaction. They could also be malicious

**Trust Management**
- Trust is the particular level of the subjective probability with which an agent assesses that another agent will perform a particular action in a context that affects his actions
- Reputation is the expectation about their behaviour based on past behaviour
- Two types of trust systems:
	- Credential and policy based
	- Reputation based

#### **Example: Chrome**
Lessons from Google: http://queue.acm.org/detail.cfm?id=1556050
Security architecture of the chromium browser: http://seclab.stanford.edu/websec/chromium/chromium-security-architecture.pdf

- Chrome relies on least privilege, separation of privilege, and defence in depth to securely parse and render insecure content

![[Pasted image 20251201105005.png]]

After a vulnerability is discovered: 
- on-duty security sheriff triages the severity of it and assigns an engineer to resolve the issue
- The engineer diagnoses and writes a patch to fix the bug
- The patched binary goes through a QA process to make sure it's fixed and the patch hasn't broken anything else

**Google is also well known for warning users before rendering malicious content**