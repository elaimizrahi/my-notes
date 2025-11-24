
## What Are NFRs?
- NFRs are often called **“quality attributes”**.  
- They specify **how well** the system performs its functions rather than **what** the system does.  
- Examples of questions NFRs answer:
  - How fast must it respond?
  - How easy must it be to use?
  - How secure does it have to be against attacks?
  - How easy should it be to maintain?

![[Pasted image 20251124114621.png]]

---


## Requirements
Requirements come from users and stakeholders who have demands and needs
- Requirements also come from analysts and requirement engineers
	- Elicits demands and needs, analyzes them for consistency and feasibility, and formulates them as requirements

*Remember: Different engineers can define the requirements differently based on their personal experiences and opinions*

Example questions that might arise:
- Is this a business need or a requirement?  
- Is this a nice-to-have vs. must-have?  
- Is this the goal of the system or a contractual requirement?  
- Do we have to program in Java? Why?

## Functional vs. Non-Functional Requirements

- **Functional Requirements (FRs):**
  - Like **verbs** — describe the features and functions of a system.
  - Example: *The system should have a secure login.*

- **Non-Functional Requirements (NFRs):**
  - Like **attributes of the verbs** — describe the qualities of the features.
  - Example: *The system should provide a **highly secure** login.*
  - **Mandatory NFR's**: system is unusable without
  - **Non-mandatory NFR's:** System is usable, but not optimal


**NFR's are also known as constraints**:
- **Quality requirements** - must be fast, secure, well maintained, etc. 
- **Managerial Requirements** - Time to delivery, verify all features are present, legal responsibilities
- **Context/env requirements** - Range of conditions in which the system should operate

![[Pasted image 20251124115444.png]]

- NFR's require special consideration, as they affect various high-level designs and systems
- It's very hard to modify an NFR once you pass the architecture phase

**Example:**
![[Pasted image 20251124115947.png]]