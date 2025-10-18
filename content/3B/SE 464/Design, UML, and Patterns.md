
- **Definition:** A design pattern is a *proven solution to a recurring design problem* in a particular context.  
- **Purpose:**  
  - Shared vocabulary among developers  
  - Capture and reuse existing design knowledge  
  - Increase flexibility for future change  
  - Improve reusability of code  

---

## Origins

- **Christopher Alexander (1977):** Identified common patterns in building architecture.  
- **Gang of Four (1994):** Applied these ideas to software in *Design Patterns: Elements of Reusable Object-Oriented Software*.  
- **Key Points:**  
  - Patterns are **language-independent**.  
  - Patterns are **micro-architectures**, smaller than full system designs.  
  - Patterns must be **adapted to context**, not blindly applied.

---

## GoF Design Pattern Categories

### 1. Creational Patterns  
*Focus: object creation and instantiation*

- **Singleton:** Ensure only one instance of a class exists (e.g., config manager).  
- **Factory Method:** Define an interface for creating objects but let subclasses decide which class to instantiate.  
- **Abstract Factory:** Provide an interface for creating families of related objects (e.g., GUI toolkit for Windows vs. macOS).  
- **Builder:** Separate construction of a complex object from its representation (e.g., building different meal combos).  
- **Prototype:** Create new objects by copying an existing one (cloning).  

---

### 2. Structural Patterns  
*Focus: composition and relationships between objects*

- **Adapter:** Convert one interface into another clients expect (e.g., wrapping legacy code).  
- **Bridge:** Separate abstraction from implementation to vary independently.  
- **Composite:** Treat individual objects and groups of objects uniformly (e.g., file system tree).  
- **Decorator:** Attach responsibilities dynamically without modifying the class (e.g., adding scrollbars to a window).  
- **Facade:** Provide a simplified interface to a complex subsystem.  
- **Flyweight:** Share objects to support large numbers efficiently (e.g., characters in a text editor).  
- **Proxy:** Provide a placeholder object to control access to another (e.g., remote proxy, lazy loading).  

---

### 3. Behavioral Patterns  
*Focus: interaction and responsibility among objects*

- **Chain of Responsibility:** Pass requests along a chain until handled (e.g., logging levels).  
- **Command:** Encapsulate a request as an object (e.g., undo/redo functionality).  
- **Interpreter:** Define a grammar and interpreter for a language (e.g., regex engines).  
- **Iterator:** Provide a way to sequentially access elements of a collection.  
- **Mediator:** Define an object to centralize communication between many objects.  
- **Memento:** Capture and restore an object’s state (e.g., checkpoints, undo).  
- **Observer:** Notify dependent objects automatically when state changes (e.g., event listeners).  
- **State:** Allow object behavior to change based on internal state.  
- **Strategy:** Define a family of algorithms and let them be interchangeable (e.g., different sorting strategies).  
- **Template Method:** Define the skeleton of an algorithm, letting subclasses fill in details.  
- **Visitor:** Represent operations to be performed on elements of a structure without modifying the elements.  

---

## Example Pattern: Observer

- **Problem:** Many objects need to stay updated when another object changes.  
- **Solution:** Define a subject that maintains a list of observers and notifies them automatically.  
- **Consequences:**  
  - Loose coupling between subject and observers  
  - Can lead to many notifications (performance cost)  
- **Example:** Event listeners in GUIs.  

---

## Example Pattern: Strategy

- **Problem:** Different algorithms needed at runtime, but we don’t want hard-coded conditionals.  
- **Solution:** Define a family of algorithms, encapsulate them, and make them interchangeable.  
- **Consequences:**  
  - Avoids conditional logic  
  - Clients must be aware of different strategies  
- **Example:** Sorting algorithms (`QuickSort`, `MergeSort`, `HeapSort`).  

---

## Why Patterns Matter

- Standardized **vocabulary** (e.g., “use Observer” is faster than explaining event propagation).  
- Encourage **reusable solutions** to recurring problems.  
- Enhance **communication** between engineers.  
- Provide **best practices** that improve system design quality.  

---
