# OOP + SOLID (Interview-Focused)

## Definitions

- **Encapsulation**: hide internal state and expose safe operations that protect invariants.
- **Abstraction**: show essential behavior while hiding unnecessary detail behind interfaces or abstract classes.
- **Inheritance**: an "is-a" relationship that enables reuse and polymorphism through subtype substitution.
- **Polymorphism**: calling the same method on different types and getting type-specific behavior (dynamic dispatch).
- **Composition**: building behavior by combining smaller objects instead of inheriting from a base class.
- **Interface vs abstract class**: interfaces define capabilities; abstract classes share common state/implementation.
- **SOLID**: SRP (one reason to change), OCP (extend without modifying), LSP (subtypes must be substitutable),
  ISP (small focused interfaces), DIP (depend on abstractions, not concretions).

## Illustrations

- **Composition**: a `Car` has an `Engine` (swappable), rather than `Car` extending `Engine`.
- **Interface**: a remote control works with any TV brand because it depends on the TV "interface", not the model.
- **LSP pitfall**: a `Square` that inherits from `Rectangle` breaks expectations when width/height are set independently.
- **DIP**: a service depends on a `PaymentGateway` interface; the concrete `StripeGateway` can be swapped in tests.

## Code Examples

```java
interface DiscountStrategy {
    double apply(double total);
}

class PercentageDiscount implements DiscountStrategy {
    private final double percent;
    PercentageDiscount(double percent) { this.percent = percent; }
    public double apply(double total) { return total * (1 - percent); }
}

class Cart {
    private DiscountStrategy strategy;
    Cart(DiscountStrategy strategy) { this.strategy = strategy; }
    double checkout(double total) { return strategy.apply(total); }
}
```

## Interview Questions

1. Explain encapsulation beyond getters and setters.
2. When do you prefer composition over inheritance?
3. Interface vs abstract class - how do you choose?
4. Give a real example of violating LSP.
5. What does DIP mean in a layered architecture?
6. How do SOLID principles reduce change risk?

---

## 1) OOP pillars

### Encapsulation

- Hide internal state, expose behavior through methods.
- Favor **private fields** and controlled mutation.
- Interview pitfall: “encapsulation = getters/setters” is incomplete; it’s about **invariants** and **information hiding**.

### Abstraction

- Model essential behavior and ignore irrelevant detail.
- Achieved via interfaces/abstract classes and clean APIs.

### Inheritance

- IS-A relationship.
- Use for true subtype polymorphism, not for code reuse alone.

### Polymorphism

- Ability to treat instances of different types uniformly through a common supertype.
- Runtime (dynamic dispatch) is the typical interview focus.

## 2) Composition vs inheritance

Prefer composition when:

- You want to reuse behavior without forcing an IS-A relationship.
- You need to swap behaviors at runtime.
- You want to avoid fragile base-class problems.

Example idea: instead of `class Car extends Engine`, do `class Car { Engine engine; }`.

## 3) Interface vs abstract class

### Interface

- Defines a contract.
- Can have `default` and `static` methods (Java 8+).
- Multiple interfaces can be implemented.

### Abstract class

- Can hold shared state and partial implementation.
- Single inheritance only.

Rule of thumb:

- Use **interface** for capabilities/roles.
- Use **abstract class** when you need shared base code/state and you control the hierarchy.

## 4) SOLID principles (with Java framing)

### S — Single Responsibility Principle

- A class should have one reason to change.
- Interview signal: separate IO, business logic, and persistence.

### O — Open/Closed Principle

- Open for extension, closed for modification.
- Achieved via interfaces + composition + dependency injection.

### L — Liskov Substitution Principle (LSP)

- Subtypes must be substitutable for base types without breaking correctness.

Classic pitfall: `Square extends Rectangle` breaks LSP if setters exist:

- `Rectangle#setWidth` and `#setHeight` assumptions don’t hold for square.

### I — Interface Segregation Principle

- Prefer smaller, role-specific interfaces over “fat” interfaces.

### D — Dependency Inversion Principle

- Depend on abstractions, not concretions.
- High-level modules shouldn’t depend on low-level modules directly.

## 5) Immutability and value objects

Interview-friendly immutable pattern:

- `final` class
- `private final` fields
- No setters
- Defensive copies for mutable inputs
- Static factory methods where helpful

Benefits:

- Thread-safety by design
- Easier reasoning/testing

## 6) Common pitfalls & nuanced topics

### Overriding vs overloading

- **Overriding**: runtime dispatch (polymorphism), same signature, covariant returns allowed.
- **Overloading**: compile-time resolution based on declared parameter types.

Overloading pitfall: ambiguity with `null` and varargs.

### `Object` methods to know

- `equals`, `hashCode`, `toString`
- `getClass` vs `instanceof` considerations for equality
- `clone` (generally avoid; prefer copy constructors/factories)

### `instanceof` pattern matching (modern Java)

Interview angle: safer casts and cleaner code, e.g.:

```java
if (obj instanceof String s) {
  System.out.println(s.length());
}
```

### Sealed classes (modern Java)

- Restrict inheritance to a known set of subtypes.
- Great for modeling closed hierarchies (e.g., expression trees).

## 7) Covariant Return Types

```java
class Animal {
    Animal reproduce() { return new Animal(); }
}

class Dog extends Animal {
    @Override
    Dog reproduce() {  // Covariant return type - returns subtype
        return new Dog();
    }
}
```

**Interview point**: Since Java 5, overriding methods can return a subtype (covariant return type), but parameters must match exactly (invariant).

---

## 8) Method Hiding vs Overriding

### Static Methods (Hiding)

```java
class Parent {
    static void staticMethod() { System.out.println("Parent static"); }
    void instanceMethod() { System.out.println("Parent instance"); }
}

class Child extends Parent {
    static void staticMethod() { System.out.println("Child static"); }  // Hides
    @Override
    void instanceMethod() { System.out.println("Child instance"); }    // Overrides
}

Parent obj = new Child();
obj.staticMethod();   // "Parent static" - resolved at compile time
obj.instanceMethod(); // "Child instance" - resolved at runtime
```

---

## 9) Marker Interfaces

Interfaces with no methods that "mark" a capability:
- `Serializable` - Object can be serialized
- `Cloneable` - Object can be cloned
- `RandomAccess` - List supports fast random access

```java
public class Person implements Serializable {
    // No methods to implement, but signals serialization support
}

// Modern alternative: annotations
@Immutable
public class Value { }
```

---

## 10) Code Examples

### Composition Over Inheritance

```java
// BAD: Using inheritance for code reuse
class Stack extends ArrayList {
    public void push(Object o) { add(o); }
    public Object pop() { return remove(size() - 1); }
    // Problem: inherits add(index), remove(index), etc.
}

// GOOD: Using composition
class Stack {
    private final List<Object> list = new ArrayList<>();
    
    public void push(Object o) { list.add(o); }
    public Object pop() { return list.remove(list.size() - 1); }
    public int size() { return list.size(); }
    // Only exposes stack operations
}
```

### LSP Violation Example

```java
// BAD: Square extends Rectangle violates LSP
class Rectangle {
    protected int width, height;
    
    public void setWidth(int w) { width = w; }
    public void setHeight(int h) { height = h; }
    public int area() { return width * height; }
}

class Square extends Rectangle {
    @Override
    public void setWidth(int w) { width = height = w; }  // Breaks expectations!
    @Override
    public void setHeight(int h) { width = height = h; }
}

// This code works for Rectangle but breaks for Square
void resize(Rectangle r) {
    r.setWidth(5);
    r.setHeight(4);
    assert r.area() == 20;  // Fails for Square!
}

// GOOD: Use composition or separate hierarchy
interface Shape { int area(); }
record Rectangle(int width, int height) implements Shape {
    public int area() { return width * height; }
}
record Square(int side) implements Shape {
    public int area() { return side * side; }
}
```

### Interface Segregation

```java
// BAD: Fat interface
interface Worker {
    void work();
    void eat();
    void sleep();
}

class Robot implements Worker {
    public void work() { /* ok */ }
    public void eat() { /* doesn't make sense */ }
    public void sleep() { /* doesn't make sense */ }
}

// GOOD: Segregated interfaces
interface Workable { void work(); }
interface Eatable { void eat(); }
interface Sleepable { void sleep(); }

class Human implements Workable, Eatable, Sleepable { }
class Robot implements Workable { }
```

### Dependency Inversion

```java
// BAD: High-level depends on low-level
class UserService {
    private MySQLUserRepository repo = new MySQLUserRepository();  // Concrete!
}

// GOOD: Both depend on abstraction
interface UserRepository {
    User findById(Long id);
    void save(User user);
}

class UserService {
    private final UserRepository repo;  // Abstraction
    
    public UserService(UserRepository repo) {
        this.repo = repo;  // Injected
    }
}

class MySQLUserRepository implements UserRepository { }
class MongoUserRepository implements UserRepository { }
```

---

## 11) Interview questions

- Give an example where inheritance is the wrong choice and composition is better.
- Explain LSP with a real example (not just definitions).
- When would you prefer interface over abstract class?
- How do you design an immutable class? Why does it help concurrency?
- What is the diamond problem? How does Java handle it?
- Explain the difference between method hiding and method overriding.
- What are marker interfaces? Give examples.
- How do default methods in interfaces work? What if two interfaces have the same default method?
- What is the difference between coupling and cohesion?
- Explain the Open/Closed principle with a code example.

