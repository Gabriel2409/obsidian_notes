#sd 

3 main types of design pattern: 

## Creational Patterns

**Purpose:** Handle object creation in a flexible and controlled way

**When to use:** When you need to control how objects are created, initialized, or configured. These patterns help when direct object instantiation would be too rigid or complex.

**Common scenarios:**

- You need only one instance of something: [[Design patterns - Singleton]]
- Object creation is complex and should be centralized [[Design patterns - Factory]]
- You want to build objects step-by-step  [[Design patterns - Builder]]
- You need to create families of related objects [[Design patterns - Abstract Factory]]

## Structural Patterns

**Purpose:** Deal with object composition and relationships between objects

**When to use:** When you need to organize objects and classes into larger structures while keeping these structures flexible and efficient.

**Common scenarios:**

- Making incompatible interfaces work together [[Design patterns - Adapter]]
- Adding functionality without changing existing code [[Design patterns - Decorator]]
- Simplifying complex subsystems [[Design patterns - Facade]]
- Representing hierarchical structures [[Design patterns - Composite]]
- Controlling access to objects [[Design patterns - Proxy]]

## 3. Behavioral Patterns

**Purpose:** Focus on communication between objects and how responsibilities are distributed

**When to use:** When you need to define how objects interact and communicate, or when you want to make algorithms and object responsibilities more flexible.

**Common scenarios:**

- Notifying multiple objects about changes [[Design patterns - Observer]]
- Switching between different algorithms [[Design patterns - Strategy]]
- Processing requests in a chain [[Design patterns - Chain of Responsibility]]
- Encapsulating operations as objects [[Design patterns - Command]]
- Managing object states and transitions [[Design patterns - State]]
- Performing operations on objects without modifying their classes [[Design patterns - Visitor]]

## How to Choose the Right Pattern

**Ask yourself:**

1. **What's the core problem?** Creation, structure, or behavior?
2. **What needs to be flexible?** Object creation, object relationships, or object interactions?
3. **What might change in the future?** This often points to where you need abstraction

**General guidelines:**

- Use **Creational** when object instantiation is the challenge
- Use **Structural** when organizing existing objects is the issue
- Use **Behavioral** when object communication or responsibility assignment needs work


