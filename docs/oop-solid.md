# OOP + SOLID (Interview-Focused)

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

## 7) Interview questions

- Give an example where inheritance is the wrong choice and composition is better.
- Explain LSP with a real example (not just definitions).
- When would you prefer interface over abstract class?
- How do you design an immutable class? Why does it help concurrency?

