## Refresh of Design, UML, and patterns

## UML
- Unified Modelling Language (UML) is a general purpose modelling language for describing system models
- Provides a unified language for developers on different systems, code, and more
	- Eases communication by using shared vocabulary

## SOLID Principles

#### 1. Single Responsibility 
A class should have one and only one reason to change.
- Enables **High Cohesion** between members of a class
	- Methods should access common fields
	- Methods should call each other to achieve functionality
- Better **Segregation of Concerns** - promotes readability and maintainability
	- Class can't expose unnecessary public methods to client
	- SRP Compliance minimizes the exposure of unnecessary methods to clients

*Famous example includes java.util.Date, as it does both parsing and formatting of date times*

#### 2. Open Closed Principle
Software Entities (Classes, Methods, etc.) should be open for extension but closed for modification
- **Low coupling** between program, clients, or other programs
- **Flexible, extensible** development is promoted (lower cost, higher speed)

Once a program is in production, it should no longer be open to any further changes, and any changes in requirements should be incorporated only by extending a class through inheritance


#### 3. Liskov Substitution Principle
References to the Base class must be completely substitytable by references of any of the Derived Classes
```c
SuperType reference = new SubType();
```
- the **Invariants** of the SuperType must not be violated or broken by the subtype
- the **Preconditions** on an overriden method for the SubType must be a subset of the preconditions on the SuperType 
	- Preconditions must not be strengthened
- The **PostConditions** on an overridden method must be a superset of the postconditions of the SuperType method
	- PostConditions must not be weakened

*Famous Violation includes java.util.Collections.unmodifiableList, where exception is thrown from doing functionality of superType (modify list)*
- Can be fixed by having a modifiable and unmodifiable list class

#### 4. Interface Segregation Principle
Client must not be forced to use Interfaces that it does not need
- Makes code **easy to read, refactor, and manage**
- Promotes a **High Cohesion for the classes** implementing the interfaces

Advocates splitting large interfaces with many methods into smaller interfaces with less methods
- Smaller Interfaces are "role interfaces"
- Large Interface is the "fat interface"

Each method implemented must certainly define a concrete behavior for the class - if they are not required and are implemented with empty body or return null or throw

*Java.util.List is also a famous violation, as all methods modifying the elements of the list are documented as **optional operations***

This is bad because all subclasses that require an unmodifiable list behaviour, need to implement all methods modifying  the elements of the list by throwing `UnsupportedOperationException`


#### Dependency Inversion Principle
High level modules should not depends on Low Level modules. Both should depend upon abstractions. Abstraction should not depend upon details, the details should depends on the abstraction.

![[Pasted image 20251124123641.png]]



![[Pasted image 20251124123820.png]]


