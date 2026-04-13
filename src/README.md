# Design Patterns — A Theoretical Guide

This document is a language-agnostic reference explaining **what design patterns are**, **why they exist**, and **when each family should (and should not) be applied**. The code examples under `java/`, `python/`, `js/`, `swift/`, `cpp/`, `csharp/`, and `go/` are concrete translations of these ideas — but the ideas themselves belong to software design, not to any one language.

---

## What Is a Design Pattern?

A **design pattern** is a *named, reusable solution to a recurring problem in software design*. Patterns are not algorithms, libraries, or snippets you copy. They are **shapes of solutions** — recognizable arrangements of classes, objects, responsibilities, and collaborations that have been repeatedly proven to handle a certain kind of design tension.

The concept was popularized in 1994 by the "Gang of Four" (Gamma, Helm, Johnson, Vlissides) in *Design Patterns: Elements of Reusable Object-Oriented Software*, which cataloged 23 patterns grouped into three families: **Creational**, **Structural**, and **Behavioral**. The SOLID principles, formulated later by Robert C. Martin, are a complementary foundation — they describe the *properties* good object-oriented designs tend to have, while patterns describe *recurring strategies* for achieving those properties.

A pattern is useful when:

- You can name the force it resolves (e.g., "I need to decouple the code that *creates* an object from the code that *uses* it").
- The problem actually exists in your current design, not a hypothetical future one.
- The added indirection pays for itself in flexibility, testability, or clarity.

A pattern is harmful when applied speculatively. Over-patterned code is as hard to read as unstructured code — every extra layer of abstraction has a cognitive cost.

---

## SOLID — The Principles Behind the Patterns

Before the patterns themselves, SOLID defines *why* many of them exist. Patterns are often the mechanical expression of one or more SOLID principles.

| Principle | Meaning | Typical Violation Smell |
|---|---|---|
| **S — Single Responsibility** | A module should have one, and only one, reason to change. | A class that both parses input *and* writes to a database. |
| **O — Open/Closed** | Software entities should be open for extension, closed for modification. | Adding a new case requires editing a long `switch` in core logic. |
| **L — Liskov Substitution** | Subtypes must be usable wherever their base types are expected, without surprising callers. | A `Square` subclass of `Rectangle` that breaks width/height invariants. |
| **I — Interface Segregation** | Clients should not be forced to depend on methods they do not use. | A "fat" interface with 20 methods where most implementers throw `NotSupported`. |
| **D — Dependency Inversion** | Depend on abstractions, not concretions; high-level policy must not depend on low-level detail. | Business logic that directly instantiates a specific database driver. |

Most Gang-of-Four patterns are, at their core, tools for applying **O** and **D**: they let you add new behavior by *adding* code instead of *editing* existing code, and they let high-level modules stay ignorant of concrete implementations.

---

## The Three Families

Patterns are traditionally grouped by the *kind of problem* they solve.

### 1. Creational Patterns — *How objects come into existence*

**Purpose.** Decouple *what* is being created from *how, when,* and *by whom*. They exist because the `new` keyword is a hidden dependency: every time a class names a concrete type, it welds itself to that type forever. Creational patterns give you a seam between "I need a thing that does X" and "here is the specific implementation of X."

**When to reach for them:**
- The construction logic is non-trivial (validation, wiring, pooling, caching).
- The exact type to instantiate depends on configuration, environment, or runtime state.
- You want to swap implementations in tests without touching production code paths.
- You need to guarantee a single instance, a family of related instances, or a controlled assembly order.

**When *not* to:**
- Construction is a single line and the type is never going to change. A factory around `new Point(x, y)` is noise.

**Core patterns and the tension each resolves:**

- **Singleton** — *"There must be exactly one."* Resolves shared-state access. Widely misused; often a global variable in disguise. Legitimate uses: hardware device handles, OS-level resource managers. Dangerous with concurrency and with testing (global state resists isolation).
- **Factory Method** — *"A superclass defines the interface for creation; subclasses decide the concrete class."* Lets a framework create objects it doesn't know the type of at compile time.
- **Abstract Factory** — *"Create families of related objects without specifying their concrete classes."* Used when products come in matched sets (e.g., `WindowsButton` + `WindowsCheckbox` vs `MacButton` + `MacCheckbox`) and mixing families would break invariants.
- **Builder** — *"Separate construction of a complex object from its representation."* Right answer when a constructor would need 8+ parameters, many optional, with ordering or validation constraints. Produces fluent, readable call sites.
- **Prototype** — *"Create new objects by cloning an existing one."* Useful when construction is expensive and most of the state is shared, or when the type is only known at runtime (you hold an instance, not a class).

### 2. Structural Patterns — *How objects are composed into larger structures*

**Purpose.** Describe how classes and objects are combined to form larger wholes while keeping those wholes flexible and efficient. Where creational patterns answer *"who makes it?"*, structural patterns answer *"how do the pieces fit together?"*

**When to reach for them:**
- You need to reconcile incompatible interfaces (legacy code, third-party libraries).
- You want to add responsibilities to objects dynamically, without subclass explosion.
- You need a simpler face over a complex subsystem.
- You are optimizing memory by sharing fine-grained objects.

**When *not* to:**
- The interfaces already match and the composition is obvious. A `Decorator` wrapping a class nobody will ever extend is ceremony.

**Core patterns and the tension each resolves:**

- **Adapter** — *"Convert the interface of a class into another interface clients expect."* The classic integration pattern: your code wants shape A, the library offers shape B, the adapter is the translation layer. Crucial for isolating third-party dependencies behind your own abstraction (a direct application of Dependency Inversion).
- **Bridge** — *"Decouple an abstraction from its implementation so the two can vary independently."* Used when a hierarchy threatens to explode along two independent dimensions (e.g., `Shape` × `RenderingAPI`). Instead of `N × M` subclasses, you compose an abstraction with an implementor.
- **Composite** — *"Compose objects into tree structures to represent part-whole hierarchies."* The key insight: *leaves and composites share the same interface*, so client code treats a single item and a group identically. Natural fit for file systems, UI widget trees, and expression trees.
- **Decorator** — *"Attach additional responsibilities to an object dynamically."* A flexible alternative to subclassing. Each decorator wraps an object of the same interface and adds behavior before/after delegating. Classic example: I/O stream wrappers (`Buffered(Gzip(File(...)))`).
- **Facade** — *"Provide a unified, higher-level interface to a subsystem."* Not a restriction — the underlying subsystem remains accessible — but a convenience for the common case. Facades are one of the cheapest, most honest patterns: they simply admit that most callers want the 20% path.
- **Flyweight** — *"Use sharing to support large numbers of fine-grained objects efficiently."* An optimization pattern. Separates *intrinsic* state (shared, immutable) from *extrinsic* state (passed in by context). Only justified once profiling shows the memory cost is real.
- **Proxy** — *"Provide a surrogate or placeholder for another object to control access."* Variants include *virtual proxies* (lazy loading), *remote proxies* (RPC stubs), *protection proxies* (access control), and *smart references* (ref counting, caching). Same shape, different motivations.

### 3. Behavioral Patterns — *How objects interact and distribute responsibility*

**Purpose.** Describe not the *structure* of objects but the *flow of control and data* between them. Behavioral patterns are almost always about answering: *"who decides what, and how does the decision travel?"*

**When to reach for them:**
- You have logic that varies on an axis you want to keep open (algorithm family, request type, state).
- You need loose coupling between a sender and a receiver, or between a subject and its observers.
- You need to encapsulate a request as data (for undo, queuing, logging, or replay).
- You need to traverse or iterate a structure without exposing its internals.

**When *not* to:**
- The variation is genuinely fixed and a simple `if/else` or method call is the whole story.

**Core patterns and the tension each resolves:**

- **Chain of Responsibility** — *"Pass a request along a chain of handlers until one handles it."* Decouples sender from the specific handler. Classic uses: middleware pipelines, event bubbling, logging filters, approval workflows.
- **Command** — *"Encapsulate a request as an object."* The enabling pattern for *undo/redo*, *macro recording*, *transactional queues*, and *deferred execution*. If you ever need to treat "a thing to do" as a first-class value, this is the shape.
- **Interpreter** — *"Define a representation for a language's grammar along with an interpreter."* Rare in application code, common in DSLs, query engines, and rule systems. Usually replaced today by parser generators or AST libraries, but the conceptual shape still matters.
- **Iterator** — *"Provide a way to access elements of a collection sequentially without exposing its underlying representation."* So foundational that most modern languages bake it into the language itself (`for ... in`, generators, `IEnumerable`). Still worth understanding because *what the iterator hides* is a design decision — lazy vs eager, forward-only vs random access.
- **Mediator** — *"Define an object that encapsulates how a set of objects interact."* Prevents the `N²` coupling problem: instead of every component talking directly to every other, all communication routes through the mediator. Used heavily in GUI dialogs and complex form logic.
- **Memento** — *"Capture and externalize an object's internal state without violating encapsulation so that it can be restored later."* The theoretical backbone of undo systems and snapshot/rollback features.
- **Observer** — *"Define a one-to-many dependency so that when one object changes state, all its dependents are notified."* The foundation of event systems, reactive programming, MVC, and pub/sub. Watch for: memory leaks from unregistered listeners, ordering surprises, and cascading updates.
- **State** — *"Allow an object to alter its behavior when its internal state changes."* Replaces a mass of `switch (this.state)` statements with polymorphism: each state is an object that knows how to handle its own transitions. Right answer whenever a type is really a small state machine in disguise.
- **Strategy** — *"Define a family of algorithms, encapsulate each, and make them interchangeable."* The workhorse of the Open/Closed Principle — you add a new algorithm by writing a new class, not by editing existing ones. Often the simplest pattern: in languages with first-class functions, a strategy is just a function passed as an argument.
- **Template Method** — *"Define the skeleton of an algorithm in a base class, deferring some steps to subclasses."* Inversion of control via inheritance: the base class calls *you*. Tradeoff: strong coupling between parent and child; favor Strategy (composition) unless the inheritance relationship is genuinely intrinsic.
- **Visitor** — *"Represent an operation to be performed on the elements of an object structure."* Lets you add new operations over a closed hierarchy of types without modifying them. The price is the reverse problem: adding a new *type* to the hierarchy becomes expensive. Choose Visitor when types are stable but operations grow; avoid it when the opposite is true.

---

## How to Choose a Pattern

A reliable mental sequence:

1. **Name the pain, not the solution.** Is the problem about *creation*, *composition*, or *interaction*? That identifies the family.
2. **Describe what needs to vary.** Patterns exist to isolate the axis of change. If nothing is varying, you probably don't need a pattern.
3. **Check if a simpler tool suffices.** A function, a record type, a lookup table, or a plain polymorphic method often beats a named pattern.
4. **Apply the smallest pattern that resolves the tension.** Prefer composition over inheritance; prefer values over machinery.
5. **Revisit later.** Patterns are not decorations — if the axis of change you anticipated never materializes, remove the indirection.

---

## Patterns Across Paradigms

Although the Gang of Four cataloged patterns in an object-oriented idiom, the underlying *forces* are universal:

- In **functional languages**, Strategy and Command collapse into "pass a function." Observer becomes a stream or signal. Visitor becomes pattern matching over a sum type.
- In **dynamic languages**, many creational patterns shrink — duck typing removes the need for explicit Abstract Factories, and Prototype is often built in.
- In **concurrent and distributed systems**, Proxy, Mediator, and Observer take on new meanings (remote calls, message brokers, event buses), but the shapes are the same.

A design pattern, in the end, is not a rule — it is a piece of shared vocabulary. Knowing the patterns gives you names for the shapes you were going to build anyway, so you and your collaborators can discuss a design before writing it.

---

## Further Reading

- Gamma, Helm, Johnson, Vlissides — *Design Patterns: Elements of Reusable Object-Oriented Software* (1994).
- Robert C. Martin — *Agile Software Development: Principles, Patterns, and Practices*.
- Eric Freeman & Elisabeth Robson — *Head First Design Patterns* (approachable, example-driven).
- Martin Fowler — *Patterns of Enterprise Application Architecture* (patterns beyond GoF, at the system level).
