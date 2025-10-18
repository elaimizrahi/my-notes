
## What Are NFRs?
- NFRs are often called **“quality attributes”**.  
- They specify **how well** the system performs its functions rather than **what** the system does.  
- Examples of questions NFRs answer:
  - How fast must it respond?
  - How easy must it be to use?
  - How secure does it have to be against attacks?
  - How easy should it be to maintain?

---

## Functional vs. Non-Functional Requirements

- **Functional Requirements (FRs):**
  - Like **verbs** — describe the features and functions of a system.
  - Example: *The system should have a secure login.*

- **Non-Functional Requirements (NFRs):**
  - Like **attributes of the verbs** — describe the qualities of the features.
  - Example: *The system should provide a **highly secure** login.*

- Key differences:
  - FRs **must always be met** (mandatory).
  - NFRs can be:
    - **Mandatory** (e.g., response time for a valve to close → critical to usability).
    - **Non-mandatory** (e.g., UI response time → system is still usable, just non-optimal).

- Market maturity:
  - Early markets: users care most about **functional requirements**.  
  - Mature markets: once FRs are satisfied, **NFRs differentiate products**.

---

## Expressing NFRs

- FRs are often expressed in **use-case form**.  
- NFRs **cannot** be expressed in use-case form since they do not exhibit externally visible behavior.  
- NFRs are:
  - **Critical** — often 20% of requirements but hardest to elicit and specify.  
  - Must be **clear, concise, and measurable**.  
  - Require **both customer and developer input** to define properly:
    - Example trade-offs:
      - Ease of maintenance (lower cost) vs. ease of adaptability.
      - Realistic vs. unrealistic performance targets.

---

## Examples of NFRs

- **Performance:** 80% of searches return results in <2 seconds.  
- **Accuracy:** Predictions are within 90% of actual cost.  
- **Portability:** System must not depend on tech preventing migration to Linux.  
- **Reusability:** Database code reusable and exported into a library.  
- **Maintainability:** Automated tests exist for all components; nightly tests complete in <24 hours.  
- **Interoperability:** Config data stored in XML, data in SQL DB, no DB triggers, must run with Java.  
- **Capacity:** Must handle 20 million users while maintaining performance.  
- **Manageability:** Must support sysadmins in troubleshooting problems.

---

## NFRs in High-Level Design

- NFRs require **special consideration during architecture**.  
- They influence **subsystems** but often **don’t map neatly** to one module.  
- Example: portability may require an OS abstraction layer.  
- Once architecture is set, **NFRs are very hard to change**:  
  - e.g., retrofitting security, reliability, or performance.

---

## Example: NFRs and Performance Bottlenecks

Three components (C1, C2, T1) share a CPU (max 12 req/sec).  

### Scenario 1: Balanced but Underutilized
- C1 = 3 req/sec  
- C2 = 3 req/sec  
- T1 = 3 req/sec  
- CPU = 9/12 = 75%  
- Throughput = 3 req/sec (T1 is bottleneck)

### Scenario 2: All Optimized Equally
- C1 = 4, C2 = 4, T1 = 4  
- CPU = 12/12 = 100%  
- Throughput = 4 req/sec (T1 still bottleneck)

### Scenario 3: Optimize T1 Only
- C1 = 3, C2 = 3, T1 = 6  
- CPU = 12/12 = 100%  
- Throughput = 6 req/sec  
- ✅ No bottleneck

### Scenario 4: Partial Optimization
- C1 = 2, C2 = 3, T1 = 5  
- CPU = 10/12 = 83%  
- Throughput = 5 req/sec  
- Still better than scenario 2.  

**Lesson:**  
Optimizing NFRs (like performance) requires **system-wide thinking**, not just local optimization.

---