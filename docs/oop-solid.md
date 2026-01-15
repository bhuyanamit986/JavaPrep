# Object-Oriented Programming & SOLID Principles

This chapter covers the four pillars of OOP and the SOLID principles for writing clean, maintainable code. Understanding these concepts is essential for designing robust software and succeeding in interviews.

---

## Definitions

### Four Pillars of OOP

- **Encapsulation**: Bundling data (fields) and methods that operate on that data within a class, and restricting direct access to internal state. Uses access modifiers (private, protected, public) to control visibility.

- **Abstraction**: Hiding complex implementation details and exposing only essential features. Achieved through abstract classes and interfaces.

- **Inheritance**: Mechanism where a new class (subclass) acquires properties and behaviors from an existing class (superclass). Enables code reuse and hierarchical relationships.

- **Polymorphism**: Ability of objects to take multiple forms. Allows objects of different classes to be treated through a common interface.

### SOLID Principles

- **Single Responsibility Principle (SRP)**: A class should have only one reason to change - one responsibility.

- **Open/Closed Principle (OCP)**: Software entities should be open for extension but closed for modification.

- **Liskov Substitution Principle (LSP)**: Objects of a superclass should be replaceable with objects of its subclasses without breaking the program.

- **Interface Segregation Principle (ISP)**: Clients should not be forced to depend on interfaces they don't use.

- **Dependency Inversion Principle (DIP)**: High-level modules should not depend on low-level modules. Both should depend on abstractions.

### Other Key Terms

- **Composition over Inheritance**: Prefer composing objects (has-a) over inheriting from classes (is-a).

- **Method Overriding**: Subclass provides specific implementation of a method already defined in superclass.

- **Method Overloading**: Multiple methods with the same name but different parameters in the same class.

- **Covariant Return Type**: Overridden method can return a subtype of the original return type.

- **Marker Interface**: Interface with no methods, used to mark classes for special behavior (e.g., `Serializable`).

- **Method Hiding**: When a subclass defines a static method with same signature as superclass static method.

---

## Illustrations

### Encapsulation

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         ENCAPSULATION                                    │
│                                                                          │
│   Without Encapsulation:              With Encapsulation:               │
│   ┌─────────────────────┐             ┌─────────────────────┐           │
│   │     BankAccount     │             │     BankAccount     │           │
│   │─────────────────────│             │─────────────────────│           │
│   │ + balance: double   │  Direct     │ - balance: double   │           │
│   │                     │  Access ✗   │                     │           │
│   │                     │             │ + getBalance()      │           │
│   │                     │             │ + deposit(amount)   │           │
│   │                     │             │ + withdraw(amount)  │           │
│   └─────────────────────┘             └─────────────────────┘           │
│                                                                          │
│   Problems:                           Benefits:                          │
│   • Anyone can set balance = -1000    • Validation in methods           │
│   • No validation                     • Can add logging                 │
│   • Hard to change internal structure • Can change internals safely     │
│   • No audit trail                    • Maintain invariants             │
│                                                                          │
│   Code Example:                                                         │
│   ┌───────────────────────────────────────────────────────────────┐     │
│   │  // Without encapsulation - DANGEROUS!                        │     │
│   │  account.balance = -5000;  // Invalid state allowed           │     │
│   │                                                               │     │
│   │  // With encapsulation - SAFE!                                │     │
│   │  account.withdraw(5000);   // Can validate: balance >= 5000?  │     │
│   │                            // Can log: "Withdrawal of $5000"  │     │
│   │                            // Can throw: InsufficientFunds    │     │
│   └───────────────────────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────────────────────┘
```

### Abstraction

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          ABSTRACTION                                     │
│                                                                          │
│   Real-World Analogy: Driving a Car                                     │
│   ┌─────────────────────────────────────────────────────────────┐       │
│   │                                                              │       │
│   │   Driver sees:           Hidden complexity:                  │       │
│   │   ┌─────────────┐        ┌────────────────────────────────┐ │       │
│   │   │ • Steering  │        │ • Fuel injection system        │ │       │
│   │   │ • Pedals    │        │ • Combustion chambers          │ │       │
│   │   │ • Dashboard │  ───▶  │ • Transmission mechanics       │ │       │
│   │   │ • Gear      │        │ • Electronic control units     │ │       │
│   │   └─────────────┘        │ • Brake hydraulics             │ │       │
│   │    (Interface)           └────────────────────────────────┘ │       │
│   │                            (Implementation)                  │       │
│   └─────────────────────────────────────────────────────────────┘       │
│                                                                          │
│   In Java:                                                              │
│   ┌─────────────────────────────────────────────────────────────┐       │
│   │                                                              │       │
│   │      ┌────────────────────┐                                  │       │
│   │      │  <<interface>>     │                                  │       │
│   │      │    Database        │  ◄── Abstract (what, not how)   │       │
│   │      ├────────────────────┤                                  │       │
│   │      │ + connect()        │                                  │       │
│   │      │ + query(sql)       │                                  │       │
│   │      │ + close()          │                                  │       │
│   │      └────────────────────┘                                  │       │
│   │              △                                               │       │
│   │              │ implements                                    │       │
│   │    ┌─────────┴─────────┐                                     │       │
│   │    │                   │                                     │       │
│   │    ▼                   ▼                                     │       │
│   │ ┌─────────────┐   ┌─────────────┐                            │       │
│   │ │MySQLDatabase│   │ MongoDB     │  ◄── Concrete (how)        │       │
│   │ │             │   │             │                            │       │
│   │ │ Connection  │   │ Connection  │      Hidden implementation │       │
│   │ │ pooling,    │   │ replica     │                            │       │
│   │ │ query       │   │ sets, etc   │                            │       │
│   │ │ caching     │   │             │                            │       │
│   │ └─────────────┘   └─────────────┘                            │       │
│   │                                                              │       │
│   └─────────────────────────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────────────────────────┘
```

### Inheritance

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          INHERITANCE                                     │
│                                                                          │
│   IS-A Relationship:                                                    │
│   ┌────────────────────────────────────────────────────────────┐        │
│   │                      Animal                                 │        │
│   │                    ┌──────────┐                              │        │
│   │                    │- name    │                              │        │
│   │                    │- age     │                              │        │
│   │                    │+ eat()   │                              │        │
│   │                    │+ sleep() │                              │        │
│   │                    └────┬─────┘                              │        │
│   │                         │ extends                            │        │
│   │           ┌─────────────┼─────────────┐                      │        │
│   │           │             │             │                      │        │
│   │           ▼             ▼             ▼                      │        │
│   │      ┌────────┐   ┌─────────┐   ┌─────────┐                  │        │
│   │      │  Dog   │   │   Cat   │   │  Bird   │                  │        │
│   │      │+ bark()│   │+ meow() │   │+ fly()  │                  │        │
│   │      │+ fetch()│   │+ purr()│   │+ sing() │                  │        │
│   │      └────────┘   └─────────┘   └─────────┘                  │        │
│   │                                                              │        │
│   │   Dog IS-A Animal                                           │        │
│   │   Dog inherits name, age, eat(), sleep() from Animal        │        │
│   │   Dog adds its own bark(), fetch() methods                  │        │
│   └────────────────────────────────────────────────────────────┘        │
│                                                                          │
│   Method Resolution (at runtime):                                       │
│   ┌────────────────────────────────────────────────────────────┐        │
│   │                                                             │        │
│   │   Animal animal = new Dog();                                │        │
│   │   animal.eat();  // Which eat() is called?                  │        │
│   │                                                             │        │
│   │   1. Check Dog class → eat() found? Use it (override)      │        │
│   │   2. If not, check Animal class → eat() found? Use it      │        │
│   │   3. If not, check Object class → (default)                 │        │
│   │                                                             │        │
│   └────────────────────────────────────────────────────────────┘        │
│                                                                          │
│   Types of Inheritance in Java:                                         │
│   ┌────────────────────────────────────────────────────────────┐        │
│   │  Single:  A → B         ✓ Supported                        │        │
│   │  Multi:   A,B → C       ✗ NOT supported (Diamond problem)   │        │
│   │  Multi-level: A → B → C ✓ Supported                        │        │
│   │  Hierarchical: A → B,C  ✓ Supported                        │        │
│   │  Interface: A,B → C     ✓ Multiple interfaces allowed      │        │
│   └────────────────────────────────────────────────────────────┘        │
└─────────────────────────────────────────────────────────────────────────┘
```

### Polymorphism

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         POLYMORPHISM                                     │
│                       "Many Forms"                                       │
│                                                                          │
│   1. COMPILE-TIME (Static) Polymorphism - Method Overloading            │
│   ┌────────────────────────────────────────────────────────────┐        │
│   │                                                             │        │
│   │   class Calculator {                                        │        │
│   │       int add(int a, int b)           { return a + b; }     │        │
│   │       double add(double a, double b)  { return a + b; }     │        │
│   │       int add(int a, int b, int c)    { return a + b + c; } │        │
│   │   }                                                         │        │
│   │                                                             │        │
│   │   Compiler decides which method to call based on arguments  │        │
│   │   calc.add(1, 2)       → int add(int, int)                 │        │
│   │   calc.add(1.0, 2.0)   → double add(double, double)        │        │
│   │   calc.add(1, 2, 3)    → int add(int, int, int)            │        │
│   │                                                             │        │
│   └────────────────────────────────────────────────────────────┘        │
│                                                                          │
│   2. RUNTIME (Dynamic) Polymorphism - Method Overriding                 │
│   ┌────────────────────────────────────────────────────────────┐        │
│   │                                                             │        │
│   │   class Shape { void draw() { println("Shape"); } }         │        │
│   │   class Circle extends Shape { void draw() { println("●"); } │        │
│   │   class Square extends Shape { void draw() { println("□"); } │        │
│   │                                                             │        │
│   │   Shape s = new Circle();  // Reference: Shape, Object: Circle       │
│   │   s.draw();                // Output: ● (Circle's draw!)    │        │
│   │                                                             │        │
│   │   JVM decides at RUNTIME based on actual object type        │        │
│   │                                                             │        │
│   │   Practical Example:                                        │        │
│   │   ┌─────────────────────────────────────────────────────┐   │        │
│   │   │  List<Shape> shapes = Arrays.asList(                │   │        │
│   │   │      new Circle(),                                  │   │        │
│   │   │      new Square(),                                  │   │        │
│   │   │      new Triangle()                                 │   │        │
│   │   │  );                                                 │   │        │
│   │   │                                                     │   │        │
│   │   │  for (Shape shape : shapes) {                       │   │        │
│   │   │      shape.draw();  // Each draws differently!      │   │        │
│   │   │  }                                                  │   │        │
│   │   │  // Output: ●  □  △                                 │   │        │
│   │   └─────────────────────────────────────────────────────┘   │        │
│   │                                                             │        │
│   └────────────────────────────────────────────────────────────┘        │
│                                                                          │
│   Why Polymorphism Matters:                                             │
│   • Write code against interfaces, not implementations                  │
│   • Add new types without changing existing code                        │
│   • Makes code extensible and maintainable                              │
└─────────────────────────────────────────────────────────────────────────┘
```

### Interface vs Abstract Class

```
┌─────────────────────────────────────────────────────────────────────────┐
│                  Interface vs Abstract Class                             │
│                                                                          │
│   ┌─────────────────────────────┬─────────────────────────────┐         │
│   │      INTERFACE              │    ABSTRACT CLASS           │         │
│   ├─────────────────────────────┼─────────────────────────────┤         │
│   │ Contract (what to do)       │ Base implementation         │         │
│   │ Multiple inheritance        │ Single inheritance only     │         │
│   │ All methods abstract*       │ Mix of abstract & concrete  │         │
│   │ Fields: public static final │ Any fields allowed          │         │
│   │ No constructor              │ Has constructor (for init)  │         │
│   │ implements keyword          │ extends keyword             │         │
│   ├─────────────────────────────┼─────────────────────────────┤         │
│   │ * default/static methods    │                             │         │
│   │   allowed since Java 8      │                             │         │
│   └─────────────────────────────┴─────────────────────────────┘         │
│                                                                          │
│   When to Use:                                                          │
│   ┌─────────────────────────────────────────────────────────────┐       │
│   │                                                              │       │
│   │   INTERFACE                      ABSTRACT CLASS             │       │
│   │   • Define a capability          • Share code among related │       │
│   │     (Comparable, Runnable)         classes                  │       │
│   │   • Unrelated classes can        • Subclasses share common  │       │
│   │     implement                      state/behavior           │       │
│   │   • Multiple inheritance         • Need constructors        │       │
│   │     needed                       • Need non-public members  │       │
│   │   • API contract definition      • Partial implementation   │       │
│   │                                                              │       │
│   └─────────────────────────────────────────────────────────────┘       │
│                                                                          │
│   Example:                                                              │
│   ┌─────────────────────────────────────────────────────────────┐       │
│   │                                                              │       │
│   │   interface Flyable { void fly(); }     // Capability       │       │
│   │   interface Swimmable { void swim(); }  // Capability       │       │
│   │                                                              │       │
│   │   abstract class Bird {                 // Base class       │       │
│   │       protected String name;                                │       │
│   │       Bird(String name) { this.name = name; }              │       │
│   │       abstract void makeSound();        // Subclass defines │       │
│   │       void eat() { /* shared code */ }  // Shared behavior │       │
│   │   }                                                         │       │
│   │                                                              │       │
│   │   class Duck extends Bird implements Flyable, Swimmable {   │       │
│   │       // Must implement: makeSound(), fly(), swim()         │       │
│   │       // Inherits: eat()                                    │       │
│   │   }                                                         │       │
│   │                                                              │       │
│   └─────────────────────────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────────────────────────┘
```

### SOLID Principles Visualized

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    S - Single Responsibility Principle                   │
│                                                                          │
│   ❌ BAD: Multiple responsibilities                                      │
│   ┌────────────────────────────────────────────────┐                    │
│   │            UserService                          │                    │
│   │  ┌────────────────────────────────────────┐    │                    │
│   │  │ + createUser()      // Business logic  │    │                    │
│   │  │ + validateEmail()   // Validation      │    │                    │
│   │  │ + sendEmail()       // Notification    │    │                    │
│   │  │ + generateReport()  // Reporting       │    │                    │
│   │  │ + saveToDatabase()  // Persistence     │    │                    │
│   │  └────────────────────────────────────────┘    │                    │
│   │  5 reasons to change = VIOLATION!              │                    │
│   └────────────────────────────────────────────────┘                    │
│                                                                          │
│   ✅ GOOD: Each class has one responsibility                             │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                  │
│   │ UserService  │  │ Validator    │  │ EmailService │                  │
│   │ + create()   │  │ + validate() │  │ + send()     │                  │
│   └──────────────┘  └──────────────┘  └──────────────┘                  │
│   ┌──────────────┐  ┌──────────────┐                                    │
│   │ ReportGen    │  │ UserRepo     │                                    │
│   │ + generate() │  │ + save()     │                                    │
│   └──────────────┘  └──────────────┘                                    │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│                    O - Open/Closed Principle                             │
│                                                                          │
│   ❌ BAD: Must modify class to add new shape                             │
│   ┌─────────────────────────────────────────────────────────────┐       │
│   │  class AreaCalculator {                                      │       │
│   │      double calculate(Object shape) {                        │       │
│   │          if (shape instanceof Circle)                        │       │
│   │              return π * r * r;                               │       │
│   │          else if (shape instanceof Rectangle)                │       │
│   │              return w * h;                                   │       │
│   │          // Must ADD code here for each new shape!          │       │
│   │          else if (shape instanceof Triangle) ...             │       │
│   │      }                                                       │       │
│   │  }                                                           │       │
│   └─────────────────────────────────────────────────────────────┘       │
│                                                                          │
│   ✅ GOOD: Open for extension, closed for modification                   │
│   ┌─────────────────────────────────────────────────────────────┐       │
│   │                                                              │       │
│   │     ┌───────────────────────┐                                │       │
│   │     │   <<interface>>       │                                │       │
│   │     │      Shape            │                                │       │
│   │     │   + area(): double    │                                │       │
│   │     └───────────┬───────────┘                                │       │
│   │                 │                                            │       │
│   │     ┌───────────┼───────────────────┐                        │       │
│   │     │           │                   │                        │       │
│   │     ▼           ▼                   ▼                        │       │
│   │  ┌──────┐   ┌──────────┐       ┌──────────┐                  │       │
│   │  │Circle│   │Rectangle │       │ Triangle │   Add new        │       │
│   │  │area()│   │ area()   │       │ area()   │   shapes         │       │
│   │  └──────┘   └──────────┘       └──────────┘   without        │       │
│   │                                               modifying      │       │
│   │  class AreaCalculator {                       existing       │       │
│   │      double calculate(Shape s) {              code!          │       │
│   │          return s.area();  // NEVER changes                  │       │
│   │      }                                                       │       │
│   │  }                                                           │       │
│   │                                                              │       │
│   └─────────────────────────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│                    L - Liskov Substitution Principle                     │
│                                                                          │
│   "Subtypes must be substitutable for their base types"                 │
│                                                                          │
│   ❌ CLASSIC VIOLATION: Rectangle/Square                                 │
│   ┌─────────────────────────────────────────────────────────────┐       │
│   │                                                              │       │
│   │   class Rectangle {                                          │       │
│   │       void setWidth(int w)  { this.width = w; }             │       │
│   │       void setHeight(int h) { this.height = h; }            │       │
│   │   }                                                          │       │
│   │                                                              │       │
│   │   class Square extends Rectangle {                           │       │
│   │       void setWidth(int w)  { width = height = w; }  // !!  │       │
│   │       void setHeight(int h) { width = height = h; }  // !!  │       │
│   │   }                                                          │       │
│   │                                                              │       │
│   │   // This code breaks with Square!                           │       │
│   │   void resize(Rectangle r) {                                 │       │
│   │       r.setWidth(5);                                         │       │
│   │       r.setHeight(10);                                       │       │
│   │       assert r.area() == 50;  // FAILS for Square (100)!    │       │
│   │   }                                                          │       │
│   │                                                              │       │
│   └─────────────────────────────────────────────────────────────┘       │
│                                                                          │
│   ✅ GOOD: Design for substitutability                                   │
│   ┌─────────────────────────────────────────────────────────────┐       │
│   │                                                              │       │
│   │   interface Shape { double area(); }                         │       │
│   │                                                              │       │
│   │   class Rectangle implements Shape {                         │       │
│   │       Rectangle(int width, int height) { ... }               │       │
│   │       double area() { return width * height; }               │       │
│   │   }                                                          │       │
│   │                                                              │       │
│   │   class Square implements Shape {                            │       │
│   │       Square(int side) { ... }                               │       │
│   │       double area() { return side * side; }                  │       │
│   │   }                                                          │       │
│   │                                                              │       │
│   │   // Both work correctly as Shape                            │       │
│   │   void printArea(Shape s) {                                  │       │
│   │       System.out.println(s.area());  // Works for any Shape │       │
│   │   }                                                          │       │
│   │                                                              │       │
│   └─────────────────────────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│                    I - Interface Segregation Principle                   │
│                                                                          │
│   "Many specific interfaces are better than one general interface"      │
│                                                                          │
│   ❌ BAD: Fat interface forces unwanted implementations                  │
│   ┌─────────────────────────────────────────────────────────────┐       │
│   │                                                              │       │
│   │   interface Worker {                                         │       │
│   │       void work();                                           │       │
│   │       void eat();                                            │       │
│   │       void sleep();                                          │       │
│   │   }                                                          │       │
│   │                                                              │       │
│   │   class Robot implements Worker {                            │       │
│   │       void work() { /* OK */ }                               │       │
│   │       void eat()  { /* EMPTY - robots don't eat! */ }       │       │
│   │       void sleep() { /* EMPTY - robots don't sleep! */ }    │       │
│   │   }                                                          │       │
│   │                                                              │       │
│   └─────────────────────────────────────────────────────────────┘       │
│                                                                          │
│   ✅ GOOD: Segregated interfaces                                         │
│   ┌─────────────────────────────────────────────────────────────┐       │
│   │                                                              │       │
│   │   interface Workable { void work(); }                        │       │
│   │   interface Eatable  { void eat(); }                         │       │
│   │   interface Sleepable { void sleep(); }                      │       │
│   │                                                              │       │
│   │   class Human implements Workable, Eatable, Sleepable {      │       │
│   │       // Implements all three                                │       │
│   │   }                                                          │       │
│   │                                                              │       │
│   │   class Robot implements Workable {                          │       │
│   │       // Only implements what it needs!                      │       │
│   │   }                                                          │       │
│   │                                                              │       │
│   └─────────────────────────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│                    D - Dependency Inversion Principle                    │
│                                                                          │
│   "Depend on abstractions, not concretions"                             │
│                                                                          │
│   ❌ BAD: High-level depends on low-level                                │
│   ┌─────────────────────────────────────────────────────────────┐       │
│   │                                                              │       │
│   │   HIGH-LEVEL                                                 │       │
│   │   ┌──────────────────────────┐                               │       │
│   │   │    OrderService          │                               │       │
│   │   │    ─────────────────     │                               │       │
│   │   │    MySQLDatabase db      │──────┐                        │       │
│   │   └──────────────────────────┘      │ Direct dependency      │       │
│   │                                     │ (concrete class)       │       │
│   │                                     ▼                        │       │
│   │   LOW-LEVEL                         │                        │       │
│   │   ┌──────────────────────────┐      │                        │       │
│   │   │    MySQLDatabase         │◀─────┘                        │       │
│   │   └──────────────────────────┘                               │       │
│   │                                                              │       │
│   │   Problem: Can't easily switch to PostgreSQL!                │       │
│   │                                                              │       │
│   └─────────────────────────────────────────────────────────────┘       │
│                                                                          │
│   ✅ GOOD: Both depend on abstraction                                    │
│   ┌─────────────────────────────────────────────────────────────┐       │
│   │                                                              │       │
│   │   HIGH-LEVEL                   ABSTRACTION                   │       │
│   │   ┌──────────────────┐        ┌──────────────────┐           │       │
│   │   │  OrderService    │───────▶│  <<interface>>   │           │       │
│   │   │  ──────────────  │        │    Database      │           │       │
│   │   │  Database db     │        │  + query()       │           │       │
│   │   └──────────────────┘        │  + save()        │           │       │
│   │                               └────────┬─────────┘           │       │
│   │                                        △                     │       │
│   │                                        │ implements          │       │
│   │                          ┌─────────────┼─────────────┐       │       │
│   │                          │             │             │       │       │
│   │   LOW-LEVEL              ▼             ▼             ▼       │       │
│   │                    ┌──────────┐  ┌──────────┐  ┌──────────┐  │       │
│   │                    │  MySQL   │  │ Postgres │  │  Mongo   │  │       │
│   │                    └──────────┘  └──────────┘  └──────────┘  │       │
│   │                                                              │       │
│   │   Now we can swap implementations easily!                    │       │
│   │                                                              │       │
│   └─────────────────────────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────────────────────────┘
```

### Composition vs Inheritance

```
┌─────────────────────────────────────────────────────────────────────────┐
│                 Composition vs Inheritance                               │
│                                                                          │
│   Inheritance (IS-A)                 Composition (HAS-A)                │
│   ┌────────────────────────┐         ┌────────────────────────┐         │
│   │                        │         │                        │         │
│   │   ┌──────────┐         │         │   ┌──────────┐         │         │
│   │   │  Engine  │         │         │   │  Engine  │         │         │
│   │   └────┬─────┘         │         │   └──────────┘         │         │
│   │        │ extends       │         │         │              │         │
│   │        ▼               │         │         │ has-a        │         │
│   │   ┌──────────┐         │         │         ▼              │         │
│   │   │   Car    │         │         │   ┌──────────┐         │         │
│   │   │ (IS-A    │         │         │   │   Car    │──┐      │         │
│   │   │  Engine?)│ ✗       │         │   │ - engine │  │has-a │         │
│   │   └──────────┘         │         │   │ - wheels │──┘      │         │
│   │                        │         │   └──────────┘         │         │
│   │   A Car IS NOT         │         │   ┌──────────┐         │         │
│   │   an Engine!           │         │   │  Wheels  │         │         │
│   │                        │         │   └──────────┘         │         │
│   └────────────────────────┘         └────────────────────────┘         │
│                                                                          │
│   When to Use:                                                          │
│   ┌─────────────────────────────────────────────────────────────┐       │
│   │                                                              │       │
│   │   Inheritance:                    Composition:               │       │
│   │   • True IS-A relationship        • HAS-A relationship      │       │
│   │   • Share behavior & state        • Combine behaviors       │       │
│   │   • Subclass is a "kind of"       • More flexible           │       │
│   │     parent                        • Easier to change        │       │
│   │   • Example: Dog IS-A Animal      • Example: Car HAS-A      │       │
│   │                                     Engine                  │       │
│   │                                                              │       │
│   │   Prefer Composition when:                                   │       │
│   │   • You need to use functionality, not specialize it        │       │
│   │   • The relationship may change                             │       │
│   │   • You want to combine multiple behaviors                  │       │
│   │                                                              │       │
│   └─────────────────────────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Code Examples

### 1) Encapsulation

```java
// ❌ Without Encapsulation - DANGEROUS
class BankAccountBad {
    public double balance;  // Anyone can modify directly!
}

BankAccountBad bad = new BankAccountBad();
bad.balance = -1000;  // Invalid state! No validation.

// ✅ With Encapsulation - SAFE
class BankAccount {
    private double balance;
    private final String accountId;
    private final List<String> transactionHistory;
    
    public BankAccount(String accountId, double initialDeposit) {
        if (initialDeposit < 0) {
            throw new IllegalArgumentException("Initial deposit cannot be negative");
        }
        this.accountId = accountId;
        this.balance = initialDeposit;
        this.transactionHistory = new ArrayList<>();
        log("Account opened with $" + initialDeposit);
    }
    
    // Getter - read access
    public double getBalance() {
        return balance;
    }
    
    // Getter - defensive copy for mutable collections
    public List<String> getTransactionHistory() {
        return List.copyOf(transactionHistory);  // Immutable copy
    }
    
    // Controlled modification with validation
    public void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Deposit must be positive");
        }
        balance += amount;
        log("Deposited $" + amount);
    }
    
    public void withdraw(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Withdrawal must be positive");
        }
        if (amount > balance) {
            throw new InsufficientFundsException("Balance: " + balance);
        }
        balance -= amount;
        log("Withdrew $" + amount);
    }
    
    private void log(String message) {
        transactionHistory.add(LocalDateTime.now() + ": " + message);
    }
}

// Usage
BankAccount account = new BankAccount("ACC-001", 1000);
account.deposit(500);     // Validated, logged
account.withdraw(200);    // Validated, logged
// account.balance = -1000;  // Compile error! Cannot access private field
```

### 2) Abstraction

```java
// Interface defines WHAT (abstraction)
interface PaymentProcessor {
    boolean processPayment(double amount);
    boolean refund(String transactionId);
    PaymentStatus getStatus(String transactionId);
}

// Implementations define HOW (concrete)
class StripeProcessor implements PaymentProcessor {
    private final StripeClient client;
    
    public StripeProcessor(String apiKey) {
        this.client = new StripeClient(apiKey);
    }
    
    @Override
    public boolean processPayment(double amount) {
        // Stripe-specific implementation
        try {
            PaymentIntent intent = client.paymentIntents().create(
                PaymentIntentCreateParams.builder()
                    .setAmount((long)(amount * 100))
                    .setCurrency("usd")
                    .build()
            );
            return intent.getStatus().equals("succeeded");
        } catch (StripeException e) {
            return false;
        }
    }
    
    @Override
    public boolean refund(String transactionId) {
        // Stripe-specific refund logic
        // ...
    }
    
    @Override
    public PaymentStatus getStatus(String transactionId) {
        // Stripe-specific status check
        // ...
    }
}

class PayPalProcessor implements PaymentProcessor {
    private final PayPalClient client;
    
    public PayPalProcessor(String clientId, String secret) {
        this.client = new PayPalClient(clientId, secret);
    }
    
    @Override
    public boolean processPayment(double amount) {
        // PayPal-specific implementation (completely different!)
        // OAuth flow, different API, different error handling
    }
    
    // ... other methods
}

// Client code uses abstraction - doesn't care about implementation
class OrderService {
    private final PaymentProcessor paymentProcessor;
    
    // Dependency Injection - can swap implementations
    public OrderService(PaymentProcessor paymentProcessor) {
        this.paymentProcessor = paymentProcessor;
    }
    
    public Order checkout(Cart cart) {
        double total = cart.getTotal();
        
        // Same code works for Stripe, PayPal, or any future processor!
        if (paymentProcessor.processPayment(total)) {
            return new Order(cart, OrderStatus.PAID);
        }
        throw new PaymentException("Payment failed");
    }
}

// Usage - easily swap implementations
PaymentProcessor stripe = new StripeProcessor("sk_live_xxx");
PaymentProcessor paypal = new PayPalProcessor("client_id", "secret");

OrderService stripeOrders = new OrderService(stripe);
OrderService paypalOrders = new OrderService(paypal);
```

### 3) Inheritance and Polymorphism

```java
// Base class with shared behavior
abstract class Employee {
    protected String name;
    protected String id;
    protected double baseSalary;
    
    public Employee(String name, String id, double baseSalary) {
        this.name = name;
        this.id = id;
        this.baseSalary = baseSalary;
    }
    
    // Common behavior (concrete method)
    public String getDetails() {
        return String.format("ID: %s, Name: %s", id, name);
    }
    
    // Force subclasses to implement (abstract method)
    public abstract double calculatePay();
    
    // Can be overridden (template method pattern)
    public void performDuties() {
        System.out.println(name + " is working...");
    }
}

// Subclass with specific implementation
class FullTimeEmployee extends Employee {
    private double bonus;
    
    public FullTimeEmployee(String name, String id, double salary, double bonus) {
        super(name, id, salary);  // Call parent constructor
        this.bonus = bonus;
    }
    
    @Override
    public double calculatePay() {
        return baseSalary + bonus;  // Monthly salary + bonus
    }
    
    @Override
    public void performDuties() {
        super.performDuties();  // Call parent method
        System.out.println("Attending meetings, managing projects...");
    }
}

class Contractor extends Employee {
    private double hourlyRate;
    private int hoursWorked;
    
    public Contractor(String name, String id, double hourlyRate) {
        super(name, id, 0);  // No base salary
        this.hourlyRate = hourlyRate;
    }
    
    public void logHours(int hours) {
        this.hoursWorked += hours;
    }
    
    @Override
    public double calculatePay() {
        return hourlyRate * hoursWorked;  // Hourly calculation
    }
}

class Intern extends Employee {
    private double stipend;
    
    public Intern(String name, String id, double stipend) {
        super(name, id, 0);
        this.stipend = stipend;
    }
    
    @Override
    public double calculatePay() {
        return stipend;  // Fixed stipend
    }
}

// Polymorphism in action
class Payroll {
    public void processPayroll(List<Employee> employees) {
        for (Employee emp : employees) {
            // Same code handles ALL employee types!
            double pay = emp.calculatePay();  // Runtime polymorphism
            System.out.println(emp.getDetails() + " - Pay: $" + pay);
        }
    }
    
    public double calculateTotalPayroll(List<Employee> employees) {
        return employees.stream()
            .mapToDouble(Employee::calculatePay)  // Method reference
            .sum();
    }
}

// Usage
List<Employee> employees = Arrays.asList(
    new FullTimeEmployee("Alice", "FT-001", 5000, 1000),
    new Contractor("Bob", "CT-001", 75),
    new Intern("Charlie", "IN-001", 1500)
);

((Contractor) employees.get(1)).logHours(160);

Payroll payroll = new Payroll();
payroll.processPayroll(employees);
// Output:
// ID: FT-001, Name: Alice - Pay: $6000.0
// ID: CT-001, Name: Bob - Pay: $12000.0
// ID: IN-001, Name: Charlie - Pay: $1500.0
```

### 4) Method Overriding Rules

```java
class Parent {
    // Method to be overridden
    protected Number getValue() {
        return 42;
    }
    
    public void process(String input) {
        System.out.println("Parent processing: " + input);
    }
    
    // final methods CANNOT be overridden
    public final void cannotOverride() {
        System.out.println("This method is final");
    }
    
    // static methods are HIDDEN, not overridden
    public static void staticMethod() {
        System.out.println("Parent static");
    }
}

class Child extends Parent {
    // Covariant return type - can return subtype
    @Override
    protected Integer getValue() {  // Integer is subtype of Number
        return 100;
    }
    
    // Access can be SAME or MORE permissive (not less!)
    @Override
    public void process(String input) {  // protected → public: OK
        super.process(input);  // Call parent version
        System.out.println("Child processing: " + input);
    }
    
    // This HIDES parent's static method, doesn't override
    public static void staticMethod() {
        System.out.println("Child static");
    }
    
    // ❌ Cannot override final method
    // public void cannotOverride() { }  // Compile error!
}

// Overriding rules:
// 1. Same method name
// 2. Same parameters (or covariant for return type)
// 3. Access: same or more permissive (private → protected → public)
// 4. Exceptions: can throw same, narrower, or none (not broader)
// 5. @Override annotation recommended (catches errors)

// Static binding vs Dynamic binding
Parent p = new Child();
p.process("test");     // Dynamic: calls Child.process() 
Parent.staticMethod(); // Static: calls Parent.staticMethod()
Child.staticMethod();  // Static: calls Child.staticMethod()

// Type determines static methods, object determines instance methods
Parent ref = new Child();
ref.staticMethod();    // Calls Parent.staticMethod()! (static binding)
```

### 5) SOLID - Single Responsibility Principle

```java
// ❌ BAD: One class doing too many things
class UserServiceBad {
    public void createUser(User user) {
        // Validate user
        if (user.getEmail() == null) throw new IllegalArgumentException();
        
        // Save to database
        String sql = "INSERT INTO users ...";
        jdbcTemplate.update(sql, user.getName(), user.getEmail());
        
        // Send welcome email
        String body = "Welcome " + user.getName();
        emailClient.send(user.getEmail(), "Welcome", body);
        
        // Log action
        logger.info("User created: " + user.getId());
    }
    // This class has 4 reasons to change: validation, persistence, 
    // email, logging
}

// ✅ GOOD: Each class has single responsibility
class UserValidator {
    public void validate(User user) {
        if (user == null) throw new IllegalArgumentException("User cannot be null");
        if (user.getEmail() == null || !user.getEmail().contains("@")) {
            throw new ValidationException("Invalid email");
        }
        if (user.getName() == null || user.getName().isBlank()) {
            throw new ValidationException("Name is required");
        }
    }
}

class UserRepository {
    private final JdbcTemplate jdbcTemplate;
    
    public void save(User user) {
        String sql = "INSERT INTO users (name, email) VALUES (?, ?)";
        jdbcTemplate.update(sql, user.getName(), user.getEmail());
    }
    
    public Optional<User> findById(Long id) {
        // ...
    }
}

class EmailService {
    private final EmailClient emailClient;
    
    public void sendWelcomeEmail(User user) {
        String body = buildWelcomeMessage(user);
        emailClient.send(user.getEmail(), "Welcome!", body);
    }
    
    private String buildWelcomeMessage(User user) {
        return "Welcome to our platform, " + user.getName() + "!";
    }
}

class UserService {
    private final UserValidator validator;
    private final UserRepository repository;
    private final EmailService emailService;
    
    // Dependencies injected
    public UserService(UserValidator validator, UserRepository repository, 
                       EmailService emailService) {
        this.validator = validator;
        this.repository = repository;
        this.emailService = emailService;
    }
    
    public void createUser(User user) {
        validator.validate(user);      // Validation responsibility
        repository.save(user);         // Persistence responsibility
        emailService.sendWelcomeEmail(user);  // Email responsibility
    }
}
```

### 6) SOLID - Open/Closed Principle

```java
// ❌ BAD: Must modify to add new discount types
class DiscountCalculatorBad {
    public double calculate(Order order, String discountType) {
        if ("PERCENTAGE".equals(discountType)) {
            return order.getTotal() * 0.1;
        } else if ("FLAT".equals(discountType)) {
            return 10.0;
        } else if ("BUY_ONE_GET_ONE".equals(discountType)) {
            // Complex logic here
            return order.getTotal() / 2;
        }
        // Must add more if-else for new discount types!
        return 0;
    }
}

// ✅ GOOD: Open for extension, closed for modification
interface DiscountStrategy {
    double calculate(Order order);
    boolean isApplicable(Order order);
}

class PercentageDiscount implements DiscountStrategy {
    private final double percentage;
    
    public PercentageDiscount(double percentage) {
        this.percentage = percentage;
    }
    
    @Override
    public double calculate(Order order) {
        return order.getTotal() * (percentage / 100);
    }
    
    @Override
    public boolean isApplicable(Order order) {
        return true;  // Always applicable
    }
}

class FlatDiscount implements DiscountStrategy {
    private final double amount;
    private final double minimumOrder;
    
    public FlatDiscount(double amount, double minimumOrder) {
        this.amount = amount;
        this.minimumOrder = minimumOrder;
    }
    
    @Override
    public double calculate(Order order) {
        return amount;
    }
    
    @Override
    public boolean isApplicable(Order order) {
        return order.getTotal() >= minimumOrder;
    }
}

// Can add new discounts without modifying existing code!
class SeasonalDiscount implements DiscountStrategy {
    private final double percentage;
    private final LocalDate startDate;
    private final LocalDate endDate;
    
    @Override
    public double calculate(Order order) {
        return order.getTotal() * (percentage / 100);
    }
    
    @Override
    public boolean isApplicable(Order order) {
        LocalDate now = LocalDate.now();
        return !now.isBefore(startDate) && !now.isAfter(endDate);
    }
}

class DiscountCalculator {
    private final List<DiscountStrategy> strategies;
    
    public DiscountCalculator(List<DiscountStrategy> strategies) {
        this.strategies = strategies;
    }
    
    // This method NEVER changes regardless of new discount types
    public double calculate(Order order) {
        return strategies.stream()
            .filter(s -> s.isApplicable(order))
            .mapToDouble(s -> s.calculate(order))
            .max()
            .orElse(0);
    }
}
```

### 7) SOLID - Liskov Substitution Principle

```java
// ❌ VIOLATION: Square breaks Rectangle behavior
class RectangleBad {
    protected int width;
    protected int height;
    
    public void setWidth(int width) { this.width = width; }
    public void setHeight(int height) { this.height = height; }
    public int area() { return width * height; }
}

class SquareBad extends RectangleBad {
    @Override
    public void setWidth(int width) {
        this.width = this.height = width;  // Unexpected behavior!
    }
    
    @Override
    public void setHeight(int height) {
        this.width = this.height = height;  // Unexpected behavior!
    }
}

// This code breaks with SquareBad!
void testRectangle(RectangleBad rect) {
    rect.setWidth(5);
    rect.setHeight(10);
    assert rect.area() == 50;  // FAILS with SquareBad (100)!
}

// ✅ GOOD: Design for substitutability
interface Shape {
    double area();
    double perimeter();
}

// Immutable rectangle - always valid state
class Rectangle implements Shape {
    private final double width;
    private final double height;
    
    public Rectangle(double width, double height) {
        if (width <= 0 || height <= 0) {
            throw new IllegalArgumentException("Dimensions must be positive");
        }
        this.width = width;
        this.height = height;
    }
    
    @Override
    public double area() { return width * height; }
    
    @Override
    public double perimeter() { return 2 * (width + height); }
    
    // Factory method for "resizing"
    public Rectangle withWidth(double newWidth) {
        return new Rectangle(newWidth, this.height);
    }
}

// Square is NOT a Rectangle - different construction
class Square implements Shape {
    private final double side;
    
    public Square(double side) {
        if (side <= 0) throw new IllegalArgumentException();
        this.side = side;
    }
    
    @Override
    public double area() { return side * side; }
    
    @Override
    public double perimeter() { return 4 * side; }
}

// Now any Shape works correctly
void printArea(Shape shape) {
    System.out.println("Area: " + shape.area());
}
```

### 8) SOLID - Interface Segregation Principle

```java
// ❌ BAD: Fat interface
interface MultiFunctionPrinterBad {
    void print(Document doc);
    void scan(Document doc);
    void fax(Document doc);
    void staple(Document doc);
}

// Simple printer forced to implement everything!
class SimplePrinterBad implements MultiFunctionPrinterBad {
    public void print(Document doc) { /* OK */ }
    public void scan(Document doc) { throw new UnsupportedOperationException(); }
    public void fax(Document doc) { throw new UnsupportedOperationException(); }
    public void staple(Document doc) { throw new UnsupportedOperationException(); }
}

// ✅ GOOD: Segregated interfaces
interface Printer {
    void print(Document doc);
}

interface Scanner {
    void scan(Document doc);
}

interface Fax {
    void fax(Document doc);
}

interface Stapler {
    void staple(Document doc);
}

// Simple printer only implements what it needs
class SimplePrinter implements Printer {
    public void print(Document doc) {
        System.out.println("Printing: " + doc.getName());
    }
}

// Multi-function device implements multiple interfaces
class MultiFunctionPrinter implements Printer, Scanner, Fax {
    public void print(Document doc) { /* print logic */ }
    public void scan(Document doc) { /* scan logic */ }
    public void fax(Document doc) { /* fax logic */ }
}

// Professional device with all capabilities
class ProfessionalPrinter implements Printer, Scanner, Fax, Stapler {
    public void print(Document doc) { /* ... */ }
    public void scan(Document doc) { /* ... */ }
    public void fax(Document doc) { /* ... */ }
    public void staple(Document doc) { /* ... */ }
}

// Client code depends only on what it needs
class PrintService {
    private final Printer printer;  // Only needs printing capability
    
    public PrintService(Printer printer) {
        this.printer = printer;
    }
    
    public void printDocuments(List<Document> docs) {
        docs.forEach(printer::print);
    }
}
```

### 9) SOLID - Dependency Inversion Principle

```java
// ❌ BAD: High-level depends on low-level concrete class
class NotificationServiceBad {
    private final GmailClient gmailClient = new GmailClient();  // Tight coupling!
    
    public void notify(User user, String message) {
        gmailClient.sendEmail(user.getEmail(), "Notification", message);
    }
}
// Problems:
// - Cannot test without real Gmail
// - Cannot switch to another email provider
// - High-level policy bound to low-level detail

// ✅ GOOD: Both depend on abstraction
interface NotificationSender {
    void send(String destination, String message);
    boolean supports(NotificationType type);
}

class EmailNotificationSender implements NotificationSender {
    private final EmailClient emailClient;
    
    public EmailNotificationSender(EmailClient emailClient) {
        this.emailClient = emailClient;
    }
    
    @Override
    public void send(String destination, String message) {
        emailClient.send(destination, "Notification", message);
    }
    
    @Override
    public boolean supports(NotificationType type) {
        return type == NotificationType.EMAIL;
    }
}

class SMSNotificationSender implements NotificationSender {
    private final SMSGateway smsGateway;
    
    public SMSNotificationSender(SMSGateway smsGateway) {
        this.smsGateway = smsGateway;
    }
    
    @Override
    public void send(String destination, String message) {
        smsGateway.sendSMS(destination, message);
    }
    
    @Override
    public boolean supports(NotificationType type) {
        return type == NotificationType.SMS;
    }
}

class PushNotificationSender implements NotificationSender {
    // Firebase, APNs, etc.
}

// High-level service depends on abstraction
class NotificationService {
    private final List<NotificationSender> senders;
    
    // Dependencies injected - easy to test and swap
    public NotificationService(List<NotificationSender> senders) {
        this.senders = senders;
    }
    
    public void notify(User user, String message, NotificationType type) {
        senders.stream()
            .filter(s -> s.supports(type))
            .findFirst()
            .ifPresentOrElse(
                s -> s.send(getDestination(user, type), message),
                () -> { throw new UnsupportedNotificationTypeException(type); }
            );
    }
    
    private String getDestination(User user, NotificationType type) {
        return switch(type) {
            case EMAIL -> user.getEmail();
            case SMS -> user.getPhone();
            case PUSH -> user.getDeviceToken();
        };
    }
}

// Easy to test with mock
class NotificationServiceTest {
    @Test
    void shouldSendEmail() {
        NotificationSender mockSender = mock(NotificationSender.class);
        when(mockSender.supports(NotificationType.EMAIL)).thenReturn(true);
        
        NotificationService service = new NotificationService(List.of(mockSender));
        service.notify(user, "Hello", NotificationType.EMAIL);
        
        verify(mockSender).send(user.getEmail(), "Hello");
    }
}
```

### 10) Composition Over Inheritance

```java
// ❌ Inheritance: Tight coupling, fragile base class problem
class ArrayListLogger<E> extends ArrayList<E> {
    private int addCount = 0;
    
    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    
    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);  // Problem: addAll might call add()!
    }
    
    public int getAddCount() { return addCount; }
}

// This gives WRONG count because addAll() internally calls add()
ArrayListLogger<String> list = new ArrayListLogger<>();
list.addAll(List.of("a", "b", "c"));
list.getAddCount();  // Returns 6, not 3!

// ✅ Composition: Flexible, safe
class CountingList<E> implements List<E> {
    private final List<E> delegate;  // Composition - HAS-A relationship
    private int addCount = 0;
    
    public CountingList(List<E> delegate) {
        this.delegate = delegate;
    }
    
    @Override
    public boolean add(E e) {
        addCount++;
        return delegate.add(e);
    }
    
    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return delegate.addAll(c);  // No double-counting!
    }
    
    public int getAddCount() { return addCount; }
    
    // Delegate all other List methods
    @Override
    public int size() { return delegate.size(); }
    @Override
    public E get(int index) { return delegate.get(index); }
    // ... other methods
}

// Works correctly!
CountingList<String> list = new CountingList<>(new ArrayList<>());
list.addAll(List.of("a", "b", "c"));
list.getAddCount();  // Correctly returns 3

// Real-world example: Decorator pattern
interface Coffee {
    String getDescription();
    double getCost();
}

class SimpleCoffee implements Coffee {
    public String getDescription() { return "Coffee"; }
    public double getCost() { return 2.0; }
}

// Decorator uses composition
class MilkDecorator implements Coffee {
    private final Coffee coffee;  // Wraps any Coffee
    
    public MilkDecorator(Coffee coffee) {
        this.coffee = coffee;
    }
    
    public String getDescription() {
        return coffee.getDescription() + ", Milk";
    }
    
    public double getCost() {
        return coffee.getCost() + 0.5;
    }
}

class SugarDecorator implements Coffee {
    private final Coffee coffee;
    
    public SugarDecorator(Coffee coffee) {
        this.coffee = coffee;
    }
    
    public String getDescription() {
        return coffee.getDescription() + ", Sugar";
    }
    
    public double getCost() {
        return coffee.getCost() + 0.2;
    }
}

// Stack decorators - very flexible!
Coffee order = new SugarDecorator(new MilkDecorator(new SimpleCoffee()));
order.getDescription();  // "Coffee, Milk, Sugar"
order.getCost();         // 2.7
```

### 11) Immutability

```java
// Mutable class (thread-unsafe, hard to reason about)
class MutablePerson {
    private String name;
    private int age;
    private List<String> hobbies;
    
    public void setName(String name) { this.name = name; }
    public void setAge(int age) { this.age = age; }
    public List<String> getHobbies() { return hobbies; }  // Leaks internal state!
}

// ✅ Immutable class (thread-safe, predictable)
public final class ImmutablePerson {  // final prevents subclassing
    private final String name;
    private final int age;
    private final List<String> hobbies;
    
    public ImmutablePerson(String name, int age, List<String> hobbies) {
        this.name = name;
        this.age = age;
        this.hobbies = List.copyOf(hobbies);  // Defensive copy
    }
    
    // Getters only, no setters
    public String getName() { return name; }
    public int getAge() { return age; }
    public List<String> getHobbies() { return hobbies; }  // Already immutable
    
    // "Setters" return new instances
    public ImmutablePerson withName(String newName) {
        return new ImmutablePerson(newName, this.age, this.hobbies);
    }
    
    public ImmutablePerson withAge(int newAge) {
        return new ImmutablePerson(this.name, newAge, this.hobbies);
    }
    
    public ImmutablePerson withHobby(String hobby) {
        List<String> newHobbies = new ArrayList<>(this.hobbies);
        newHobbies.add(hobby);
        return new ImmutablePerson(this.name, this.age, newHobbies);
    }
}

// Usage
ImmutablePerson alice = new ImmutablePerson("Alice", 30, List.of("Reading"));
ImmutablePerson olderAlice = alice.withAge(31);  // Original unchanged!

// With Java Records (Java 14+) - immutable by design
public record Person(String name, int age, List<String> hobbies) {
    public Person {  // Compact constructor for validation/defensive copy
        hobbies = List.copyOf(hobbies);
    }
    
    public Person withName(String newName) {
        return new Person(newName, age, hobbies);
    }
}

// Benefits of Immutability:
// 1. Thread-safe - no synchronization needed
// 2. Simple - no state changes to track
// 3. Safe to share - can pass around without copying
// 4. Good map keys - hashCode never changes
// 5. Failure atomicity - operations succeed or fail, no partial state
```

---

## Interview Questions

### Basic Questions

1. **What are the four pillars of OOP?**
   - **Encapsulation**: Bundling data and methods, hiding internal state
   - **Abstraction**: Hiding complexity, showing only essential features
   - **Inheritance**: Code reuse through class hierarchy (is-a relationship)
   - **Polymorphism**: Objects taking multiple forms through interfaces

2. **What is the difference between abstraction and encapsulation?**
   - **Abstraction**: Hides complexity (WHAT vs HOW) - design level
   - **Encapsulation**: Hides data (access control) - implementation level
   - Abstraction: Using interface `Database` without knowing if it's MySQL or MongoDB
   - Encapsulation: Private fields with public getters/setters

3. **What is the difference between method overloading and overriding?**
   - **Overloading**: Same name, different parameters, same class, compile-time
   - **Overriding**: Same signature, different class (inheritance), runtime
   - Overloading: `add(int, int)` and `add(double, double)`
   - Overriding: Subclass redefines parent's method

4. **Can you override a private or static method?**
   - **Private**: No - not visible to subclasses
   - **Static**: No - static methods are hidden, not overridden
   - Static binding for static methods, dynamic binding for instance methods

5. **What is the difference between interface and abstract class?**
   - Interface: Contract, multiple inheritance, no state, `implements`
   - Abstract: Partial implementation, single inheritance, can have state, `extends`
   - Use interface for capabilities, abstract class for shared base code

### SOLID Questions

6. **Explain the Single Responsibility Principle.**
   - A class should have only one reason to change
   - Each class handles one concern (validation, persistence, logging)
   - Benefits: Easier to test, maintain, and understand

7. **Explain the Open/Closed Principle.**
   - Open for extension, closed for modification
   - Add new behavior without changing existing code
   - Achieved through: Interfaces, inheritance, strategy pattern

8. **What is the Liskov Substitution Principle?**
   - Subtypes must be substitutable for their base types
   - Programs should work correctly with any subclass
   - Classic violation: Square extending Rectangle

9. **Explain the Interface Segregation Principle.**
   - Prefer small, specific interfaces over large, general ones
   - Clients shouldn't depend on methods they don't use
   - Split fat interfaces into focused ones

10. **Explain the Dependency Inversion Principle.**
    - High-level modules shouldn't depend on low-level modules
    - Both should depend on abstractions
    - Enables: Testing, flexibility, loose coupling

### Advanced Questions

11. **Why prefer composition over inheritance?**
    - More flexible - can change behavior at runtime
    - Avoids fragile base class problem
    - Better encapsulation - only expose needed methods
    - Enables multiple behaviors (decorator pattern)

12. **What is the diamond problem and how does Java solve it?**
    - Multiple inheritance ambiguity: A extends B, C - which method?
    - Java: No multiple class inheritance (prevents problem)
    - Interfaces: Default methods - must override if conflict

13. **What are covariant return types?**
    - Overriding method can return subtype of original return type
    - Example: Parent returns `Animal`, child can return `Dog`
    - Allows more specific return types while maintaining substitutability

14. **How do you create an immutable class?**
    - Declare class as `final` (prevent subclassing)
    - Make all fields `private final`
    - No setters, only getters
    - Defensive copy mutable objects in constructor and getters
    - Or use Java Records

15. **What is a marker interface?**
    - Interface with no methods (e.g., `Serializable`, `Cloneable`)
    - Marks classes for special treatment
    - Modern alternative: Annotations
    - Example: `if (obj instanceof Serializable)`

---

## Quick Reference

### Overloading vs Overriding

| Aspect | Overloading | Overriding |
|--------|-------------|------------|
| Where | Same class | Parent-child classes |
| What changes | Parameters | Implementation |
| Binding | Compile-time | Runtime |
| Return type | Can differ | Same or covariant |
| Access modifier | Any | Same or more permissive |
| Exceptions | Any | Same or narrower |

### SOLID Summary

| Principle | One-liner |
|-----------|-----------|
| **S**RP | One class = one responsibility |
| **O**CP | Extend, don't modify |
| **L**SP | Subtypes must be substitutable |
| **I**SP | Small interfaces > large interfaces |
| **D**IP | Depend on abstractions |

### Access Modifiers

```
┌────────────┬───────┬─────────┬──────────┬───────┐
│  Modifier  │ Class │ Package │ Subclass │ World │
├────────────┼───────┼─────────┼──────────┼───────┤
│  public    │   ✓   │    ✓    │    ✓     │   ✓   │
│  protected │   ✓   │    ✓    │    ✓     │   ✗   │
│  (default) │   ✓   │    ✓    │    ✗     │   ✗   │
│  private   │   ✓   │    ✗    │    ✗     │   ✗   │
└────────────┴───────┴─────────┴──────────┴───────┘
```
