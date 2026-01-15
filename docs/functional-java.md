# Functional Programming in Java (Interview Essentials)

Java is multi-paradigm. Functional-style code in Java relies on lambdas, method references, functional interfaces, and streams. This chapter focuses on the language features, correctness, performance pitfalls, and common interview questions.

---

## Definitions

- **Functional interface**: An interface with exactly one abstract method (SAM - Single Abstract Method). Can have any number of default and static methods. Annotated with `@FunctionalInterface` for compile-time checking.

- **Lambda expression**: An anonymous function that can be passed as a value. It provides a concise way to implement functional interfaces without creating a separate class.

- **Method reference**: A shorthand notation for a lambda that simply calls an existing method. Uses the `::` operator.

- **Effectively final**: A local variable that is never reassigned after initialization. Required for variables captured in lambdas.

- **Higher-order function**: A method that takes a function as a parameter or returns a function as its result.

- **Pure function**: A function that always produces the same output for the same input and has no side effects.

- **Side effects**: Observable changes outside a function's scope (e.g., modifying external state, I/O operations).

- **Closure**: A lambda that captures variables from its enclosing scope.

- **Currying**: Transforming a function with multiple arguments into a sequence of functions, each taking a single argument.

---

## Illustrations

### Lambda vs Traditional Approach

```
Traditional (Anonymous Inner Class):
┌─────────────────────────────────────────┐
│ Comparator<String> comp =               │
│   new Comparator<String>() {            │
│     @Override                           │
│     public int compare(String a,        │
│                        String b) {      │
│       return a.length() - b.length();   │
│     }                                   │
│   };                                    │
└─────────────────────────────────────────┘

Lambda (Concise):
┌─────────────────────────────────────────┐
│ Comparator<String> comp =               │
│   (a, b) -> a.length() - b.length();    │
└─────────────────────────────────────────┘
```

### Method Reference Types

```
┌─────────────────────────────────────────────────────────────────┐
│ Type                   │ Syntax              │ Lambda Equivalent │
├─────────────────────────────────────────────────────────────────┤
│ Static method          │ Class::staticMethod │ x -> Class.static │
│                        │                     │     Method(x)     │
├─────────────────────────────────────────────────────────────────┤
│ Instance method        │ obj::instanceMethod │ x -> obj.instance │
│ (bound)                │                     │     Method(x)     │
├─────────────────────────────────────────────────────────────────┤
│ Instance method        │ Class::instanceMeth │ (obj, x) -> obj.  │
│ (unbound)              │                     │   instanceMethod(x)│
├─────────────────────────────────────────────────────────────────┤
│ Constructor            │ Class::new          │ x -> new Class(x) │
└─────────────────────────────────────────────────────────────────┘
```

### Variable Capture in Lambdas

```
┌─────────────────────────────────────────────────────────────────┐
│                    Enclosing Method                             │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  int base = 10;  // Effectively final                     │  │
│  │  int multiplier = 2;                                      │  │
│  │  // multiplier = 3;  // ❌ Would break effectively final  │  │
│  │                                                           │  │
│  │  Lambda captures 'base' and 'multiplier' by value:        │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │  Function<Integer, Integer> calc =                  │  │  │
│  │  │    x -> x * multiplier + base;  // Reads captured   │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### this Reference: Lambda vs Anonymous Class

```
class Outer {
    String name = "Outer";
    
    void demo() {
        // Anonymous inner class: 'this' refers to anonymous class instance
        Runnable r1 = new Runnable() {
            String name = "Anonymous";
            public void run() {
                System.out.println(this.name);  // "Anonymous"
            }
        };
        
        // Lambda: 'this' refers to enclosing Outer instance
        Runnable r2 = () -> {
            System.out.println(this.name);  // "Outer"
        };
    }
}
```

---

## Code Examples

### 1) Functional Interfaces - Built-in Types

```java
import java.util.function.*;

// Predicate<T>: T -> boolean
Predicate<String> isEmpty = String::isEmpty;
Predicate<String> notEmpty = isEmpty.negate();
Predicate<String> isShort = s -> s.length() < 5;
Predicate<String> isShortAndNotEmpty = notEmpty.and(isShort);

boolean result = isShortAndNotEmpty.test("Hi");  // true

// Function<T, R>: T -> R
Function<String, Integer> length = String::length;
Function<String, String> trim = String::trim;
Function<String, Integer> trimThenLength = trim.andThen(length);

int len = trimThenLength.apply("  Hello  ");  // 5

// Consumer<T>: T -> void
Consumer<String> printer = System.out::println;
Consumer<String> logger = s -> System.err.println("LOG: " + s);
Consumer<String> printAndLog = printer.andThen(logger);

printAndLog.accept("Message");

// Supplier<T>: () -> T
Supplier<LocalDateTime> now = LocalDateTime::now;
Supplier<List<String>> listFactory = ArrayList::new;

LocalDateTime timestamp = now.get();
List<String> newList = listFactory.get();

// UnaryOperator<T>: T -> T (same type)
UnaryOperator<String> toUpper = String::toUpperCase;
UnaryOperator<String> addPrefix = s -> "PREFIX_" + s;
UnaryOperator<String> combined = toUpper.andThen(addPrefix);

String result2 = combined.apply("hello");  // "PREFIX_HELLO"

// BinaryOperator<T>: (T, T) -> T
BinaryOperator<Integer> sum = Integer::sum;
BinaryOperator<Integer> max = Integer::max;

int total = sum.apply(5, 3);  // 8

// BiFunction<T, U, R>: (T, U) -> R
BiFunction<String, Integer, String> repeat = String::repeat;
String repeated = repeat.apply("ab", 3);  // "ababab"

// BiPredicate<T, U>: (T, U) -> boolean
BiPredicate<String, String> startsWith = String::startsWith;
boolean starts = startsWith.test("Hello", "He");  // true

// BiConsumer<T, U>: (T, U) -> void
BiConsumer<Map<String, Integer>, String> incrementer = 
    (map, key) -> map.merge(key, 1, Integer::sum);
```

### 2) Primitive Specializations (Avoid Boxing Overhead)

```java
// IntFunction, LongFunction, DoubleFunction
IntFunction<String> intToString = Integer::toString;
String s = intToString.apply(42);  // "42"

// ToIntFunction, ToLongFunction, ToDoubleFunction
ToIntFunction<String> stringLength = String::length;
int len = stringLength.applyAsInt("Hello");  // 5

// IntPredicate, LongPredicate, DoublePredicate
IntPredicate isEven = n -> n % 2 == 0;
boolean even = isEven.test(4);  // true

// IntConsumer, LongConsumer, DoubleConsumer
IntConsumer printInt = System.out::println;
printInt.accept(42);

// IntSupplier, LongSupplier, DoubleSupplier
IntSupplier randomInt = () -> (int) (Math.random() * 100);
int rand = randomInt.getAsInt();

// IntUnaryOperator, LongUnaryOperator, DoubleUnaryOperator
IntUnaryOperator square = n -> n * n;
int squared = square.applyAsInt(5);  // 25

// IntBinaryOperator, LongBinaryOperator, DoubleBinaryOperator
IntBinaryOperator multiply = (a, b) -> a * b;
int product = multiply.applyAsInt(3, 4);  // 12

// ObjIntConsumer, ObjLongConsumer, ObjDoubleConsumer
ObjIntConsumer<StringBuilder> appendInt = StringBuilder::append;
StringBuilder sb = new StringBuilder("Value: ");
appendInt.accept(sb, 42);  // "Value: 42"
```

### 3) Creating Custom Functional Interfaces

```java
// Simple custom interface
@FunctionalInterface
public interface Transformer<T> {
    T transform(T input);
    
    // Default method for chaining
    default Transformer<T> andThen(Transformer<T> after) {
        return input -> after.transform(this.transform(input));
    }
    
    // Static factory method
    static <T> Transformer<T> identity() {
        return input -> input;
    }
}

// Usage
Transformer<String> upper = String::toUpperCase;
Transformer<String> trim = String::trim;
Transformer<String> pipeline = trim.andThen(upper);
String result = pipeline.transform("  hello  ");  // "HELLO"

// Functional interface with multiple type parameters
@FunctionalInterface
public interface TriFunction<A, B, C, R> {
    R apply(A a, B b, C c);
}

// Usage
TriFunction<Integer, Integer, Integer, Integer> sum3 = (a, b, c) -> a + b + c;
int total = sum3.apply(1, 2, 3);  // 6

// Checked exception functional interface
@FunctionalInterface
public interface ThrowingFunction<T, R, E extends Exception> {
    R apply(T t) throws E;
    
    // Convert to regular Function (wraps exception)
    static <T, R> Function<T, R> unchecked(ThrowingFunction<T, R, Exception> f) {
        return t -> {
            try {
                return f.apply(t);
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        };
    }
}

// Usage - handling checked exceptions in streams
List<String> lines = paths.stream()
    .map(ThrowingFunction.unchecked(Files::readString))
    .toList();
```

### 4) Lambda Syntax Variations

```java
// Full syntax
Function<Integer, Integer> f1 = (Integer x) -> { return x * 2; };

// Type inference
Function<Integer, Integer> f2 = (x) -> { return x * 2; };

// Single parameter - parentheses optional
Function<Integer, Integer> f3 = x -> { return x * 2; };

// Single expression - return and braces optional
Function<Integer, Integer> f4 = x -> x * 2;

// No parameters
Supplier<String> s1 = () -> "Hello";

// Multiple parameters
BiFunction<Integer, Integer, Integer> sum = (a, b) -> a + b;

// Multiple statements (requires braces and return)
Function<Integer, String> complex = x -> {
    int doubled = x * 2;
    String result = "Result: " + doubled;
    return result;
};

// Capturing variables
int multiplier = 3;
Function<Integer, Integer> multiply = x -> x * multiplier;
// multiplier = 4;  // Error: must be effectively final
```

### 5) Method References - All Four Types

```java
// Type 1: Static method reference
// Lambda: (args) -> ClassName.staticMethod(args)
Function<String, Integer> parseInt = Integer::parseInt;
BiFunction<String, Integer, Integer> parseIntRadix = Integer::parseInt;

List<Integer> numbers = List.of("1", "2", "3").stream()
    .map(Integer::parseInt)  // Static method reference
    .toList();

// Type 2: Instance method reference on specific object (bound)
// Lambda: (args) -> instance.method(args)
String prefix = "Hello ";
Function<String, String> addPrefix = prefix::concat;
String result = addPrefix.apply("World");  // "Hello World"

PrintStream out = System.out;
Consumer<String> print = out::println;  // Bound to 'out'

// Type 3: Instance method reference on arbitrary object (unbound)
// Lambda: (obj, args) -> obj.method(args)
// First parameter becomes the receiver
Function<String, String> toUpper = String::toUpperCase;
BiPredicate<String, String> equalsIgnoreCase = String::equalsIgnoreCase;
Comparator<String> lengthComparator = Comparator.comparingInt(String::length);

List<String> sorted = List.of("banana", "apple", "cherry").stream()
    .sorted(String::compareToIgnoreCase)  // Unbound method reference
    .toList();

// Type 4: Constructor reference
// Lambda: (args) -> new ClassName(args)
Supplier<List<String>> listFactory = ArrayList::new;
Function<Integer, StringBuilder> sbWithCapacity = StringBuilder::new;
BiFunction<String, Integer, Person> personFactory = Person::new;

// With arrays
Function<Integer, String[]> arrayFactory = String[]::new;
String[] array = arrayFactory.apply(10);  // new String[10]

// Commonly used with streams
List<Person> people = names.stream()
    .map(Person::new)  // Constructor reference
    .toList();
```

### 6) Effectively Final and Variable Capture

```java
public class VariableCaptureDemo {
    private int instanceField = 100;  // Can be captured and mutated
    private static int staticField = 200;  // Can be captured and mutated
    
    public void demonstrateCapture() {
        int localVar = 50;  // Effectively final
        final int explicitFinal = 60;
        
        // ✅ Capturing effectively final local variable
        Function<Integer, Integer> addLocal = x -> x + localVar;
        
        // ❌ Cannot modify captured local variable
        // Runnable bad = () -> localVar++;  // Compile error
        
        // ✅ Capturing instance field (reads current value each time)
        Supplier<Integer> getField = () -> instanceField;
        
        // ✅ Modifying instance field inside lambda
        Runnable incrementField = () -> instanceField++;
        
        // ✅ Capturing static field
        Supplier<Integer> getStatic = () -> staticField;
        
        // ✅ Modifying static field inside lambda
        Runnable incrementStatic = () -> staticField++;
        
        // Workaround for modifying local state: use mutable container
        int[] counter = {0};  // Array is effectively final, contents are not
        Runnable increment = () -> counter[0]++;
        
        // Or use AtomicInteger
        AtomicInteger atomicCounter = new AtomicInteger(0);
        Runnable atomicIncrement = atomicCounter::incrementAndGet;
    }
}

// Why effectively final?
// 1. Lambda captures VALUE, not reference to local variable
// 2. Local variables live on stack, may be gone when lambda executes
// 3. Prevents confusing behavior with closures
// 4. Thread safety considerations
```

### 7) Functional Composition

```java
// Function composition
Function<String, String> trim = String::trim;
Function<String, String> toUpper = String::toUpperCase;
Function<String, Integer> length = String::length;

// compose: applies argument function first, then this function
Function<String, Integer> trimUpperLength = length.compose(toUpper).compose(trim);
// Execution order: trim -> toUpper -> length

// andThen: applies this function first, then argument function
Function<String, Integer> trimUpperLength2 = trim.andThen(toUpper).andThen(length);
// Same result, more readable order

int result = trimUpperLength.apply("  hello  ");  // 5

// Predicate composition
Predicate<String> notEmpty = s -> !s.isEmpty();
Predicate<String> notNull = Objects::nonNull;
Predicate<String> lengthOk = s -> s.length() < 100;

// Combine with and(), or(), negate()
Predicate<String> valid = notNull.and(notEmpty).and(lengthOk);
Predicate<String> invalid = valid.negate();

// Comparator composition
Comparator<Person> byLastName = Comparator.comparing(Person::getLastName);
Comparator<Person> byFirstName = Comparator.comparing(Person::getFirstName);
Comparator<Person> byAge = Comparator.comparingInt(Person::getAge);

Comparator<Person> fullComparator = byLastName
    .thenComparing(byFirstName)
    .thenComparing(byAge)
    .reversed();

// With null handling
Comparator<Person> nullSafe = Comparator
    .nullsFirst(Comparator.comparing(Person::getName));

// Consumer composition
Consumer<String> log = s -> System.out.println("LOG: " + s);
Consumer<String> save = s -> database.save(s);
Consumer<String> logAndSave = log.andThen(save);
```

### 8) Avoiding Side Effects

```java
// ❌ BAD: Side effects in stream operations
List<String> results = new ArrayList<>();  // External mutable state
list.stream()
    .filter(s -> s.length() > 3)
    .forEach(results::add);  // Side effect: mutating external list

// ✅ GOOD: Use collect instead
List<String> results = list.stream()
    .filter(s -> s.length() > 3)
    .collect(Collectors.toList());

// ❌ BAD: Side effect in map (logging)
list.stream()
    .map(s -> {
        System.out.println("Processing: " + s);  // Side effect!
        return s.toUpperCase();
    })
    .toList();

// ✅ OK: Use peek for debugging (remove in production)
list.stream()
    .peek(s -> System.out.println("Processing: " + s))  // Explicit debugging
    .map(String::toUpperCase)
    .toList();

// ❌ DANGEROUS: Side effects with parallel streams
List<String> unsafeResults = Collections.synchronizedList(new ArrayList<>());
list.parallelStream()
    .filter(s -> s.length() > 3)
    .forEach(unsafeResults::add);  // Race condition risk!

// ✅ SAFE: Parallel-safe collection
List<String> safeResults = list.parallelStream()
    .filter(s -> s.length() > 3)
    .collect(Collectors.toList());  // Thread-safe

// Pure functions are referentially transparent
// f(x) always returns same value for same x, with no observable side effects
Function<Integer, Integer> pure = x -> x * 2;  // ✅ Pure
Function<Integer, Integer> impure = x -> {
    System.out.println(x);  // ❌ Side effect (I/O)
    return x * 2;
};
```

### 9) Lambda vs Anonymous Class

```java
public class LambdaVsAnonymous {
    
    public void differences() {
        // 1. Syntax: Lambda is more concise
        Runnable anonymous = new Runnable() {
            @Override
            public void run() {
                System.out.println("Anonymous");
            }
        };
        
        Runnable lambda = () -> System.out.println("Lambda");
        
        // 2. 'this' reference is different
        // Anonymous: 'this' refers to anonymous class instance
        // Lambda: 'this' refers to enclosing class instance
        
        // 3. Compilation: Different bytecode
        // Anonymous: Creates separate .class file
        // Lambda: Uses invokedynamic, more efficient
        
        // 4. State: Anonymous class can have its own state
        Runnable withState = new Runnable() {
            private int count = 0;
            @Override
            public void run() {
                count++;  // Has own state
            }
        };
        
        // Lambda cannot have its own state (would need external mutable container)
        int[] count = {0};
        Runnable lambdaWithState = () -> count[0]++;
        
        // 5. Type: Anonymous can implement any interface or extend class
        // Lambda can only implement functional interfaces
        
        // Anonymous extending abstract class
        AbstractClass instance = new AbstractClass() {
            @Override
            void doSomething() { }
        };
        
        // Lambda cannot do this - would need functional interface
    }
}
```

### 10) Default Methods and Diamond Problem

```java
interface A {
    default void greet() {
        System.out.println("Hello from A");
    }
}

interface B {
    default void greet() {
        System.out.println("Hello from B");
    }
}

// Class implementing both must resolve conflict
class C implements A, B {
    @Override
    public void greet() {
        // Option 1: Provide own implementation
        System.out.println("Hello from C");
        
        // Option 2: Call specific interface default
        A.super.greet();  // or B.super.greet();
        
        // Option 3: Call both
        // A.super.greet();
        // B.super.greet();
    }
}

// Resolution rules:
// 1. Class methods win over interface default methods
// 2. More specific interface wins (sub-interface over super-interface)
// 3. If still ambiguous, class must override and resolve

interface Parent {
    default void method() { System.out.println("Parent"); }
}

interface Child extends Parent {
    @Override
    default void method() { System.out.println("Child"); }
}

class Impl implements Parent, Child {
    // Uses Child.method() automatically (more specific)
}
```

### 11) Practical Patterns

```java
// Factory pattern with Supplier
Map<String, Supplier<Shape>> shapeFactory = Map.of(
    "circle", Circle::new,
    "rectangle", Rectangle::new,
    "triangle", Triangle::new
);

Shape shape = shapeFactory.get("circle").get();

// Strategy pattern with Function
Map<String, Function<Double, Double>> discounts = Map.of(
    "NONE", price -> price,
    "STANDARD", price -> price * 0.9,
    "VIP", price -> price * 0.8
);

double finalPrice = discounts.get(customerType).apply(originalPrice);

// Validation with Predicate chains
public class Validator<T> {
    private final Predicate<T> predicate;
    
    private Validator(Predicate<T> predicate) {
        this.predicate = predicate;
    }
    
    public static <T> Validator<T> of(Predicate<T> predicate) {
        return new Validator<>(predicate);
    }
    
    public Validator<T> and(Predicate<T> other) {
        return new Validator<>(predicate.and(other));
    }
    
    public boolean validate(T value) {
        return predicate.test(value);
    }
}

// Usage
Validator<String> emailValidator = Validator
    .of((String s) -> s != null)
    .and(s -> !s.isEmpty())
    .and(s -> s.contains("@"))
    .and(s -> s.length() <= 100);

boolean isValid = emailValidator.validate("test@example.com");

// Builder with Consumer
public class PersonBuilder {
    private String name;
    private int age;
    
    public static Person build(Consumer<PersonBuilder> config) {
        PersonBuilder builder = new PersonBuilder();
        config.accept(builder);
        return new Person(builder.name, builder.age);
    }
    
    public PersonBuilder name(String name) { this.name = name; return this; }
    public PersonBuilder age(int age) { this.age = age; return this; }
}

Person person = PersonBuilder.build(b -> b.name("John").age(30));

// Lazy evaluation with Supplier
public class Lazy<T> {
    private Supplier<T> supplier;
    private T value;
    private boolean computed = false;
    
    private Lazy(Supplier<T> supplier) {
        this.supplier = supplier;
    }
    
    public static <T> Lazy<T> of(Supplier<T> supplier) {
        return new Lazy<>(supplier);
    }
    
    public T get() {
        if (!computed) {
            value = supplier.get();
            computed = true;
            supplier = null;  // Allow GC
        }
        return value;
    }
}

Lazy<ExpensiveObject> lazy = Lazy.of(() -> new ExpensiveObject());
// ExpensiveObject not created until lazy.get() is called
```

---

## Common Pitfalls

### 1. Autoboxing Overhead

```java
// ❌ BAD: Autoboxing in every iteration
int sum = list.stream()
    .map(s -> s.length())      // Returns Integer (autoboxing)
    .reduce(0, Integer::sum);  // Unboxing for each addition

// ✅ GOOD: Use primitive streams
int sum = list.stream()
    .mapToInt(String::length)  // Returns int
    .sum();                    // No boxing
```

### 2. Accidental Statefulness

```java
// ❌ BAD: Lambda depends on mutable external state
int[] counter = {0};
list.stream()
    .map(s -> {
        counter[0]++;  // Mutating external state
        return s.toUpperCase();
    })
    .toList();
// Problem: With parallel streams, counter may be incorrect

// ✅ GOOD: Keep operations stateless
long count = list.stream()
    .map(String::toUpperCase)
    .count();
```

### 3. Overusing Lambdas

```java
// ❌ BAD: Complex logic in lambda
list.stream()
    .filter(s -> {
        if (s == null) return false;
        if (s.isEmpty()) return false;
        if (s.length() > 100) return false;
        if (!s.matches("[a-zA-Z]+")) return false;
        return true;
    })
    .toList();

// ✅ GOOD: Extract to method for readability
list.stream()
    .filter(this::isValidString)
    .toList();

private boolean isValidString(String s) {
    return s != null 
        && !s.isEmpty() 
        && s.length() <= 100 
        && s.matches("[a-zA-Z]+");
}
```

### 4. Checked Exceptions in Lambdas

```java
// ❌ Problem: Checked exceptions don't work with standard functional interfaces
list.stream()
    .map(path -> Files.readString(path))  // Compile error: IOException
    .toList();

// ✅ Solution 1: Wrap in try-catch
list.stream()
    .map(path -> {
        try {
            return Files.readString(path);
        } catch (IOException e) {
            throw new UncheckedIOException(e);
        }
    })
    .toList();

// ✅ Solution 2: Utility method
public static <T, R> Function<T, R> wrap(ThrowingFunction<T, R> f) {
    return t -> {
        try {
            return f.apply(t);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    };
}

list.stream()
    .map(wrap(Files::readString))
    .toList();
```

---

## Interview Questions

### Basic Questions

1. **What is a functional interface? Give examples.**
   - An interface with exactly one abstract method (SAM)
   - May have multiple default/static methods
   - Examples: `Runnable`, `Callable`, `Comparator`, `Predicate`, `Function`
   - `@FunctionalInterface` annotation ensures compile-time checking

2. **What are the four types of method references?**
   - Static method: `Integer::parseInt`
   - Bound instance method: `System.out::println`
   - Unbound instance method: `String::toUpperCase`
   - Constructor: `ArrayList::new`

3. **Why must captured variables be effectively final?**
   - Lambdas capture values, not variables
   - Local variables exist on stack, may be gone when lambda executes
   - Prevents confusing behavior and ensures thread safety
   - Instance/static fields are different (captured by reference)

4. **How does `this` differ between lambda and anonymous class?**
   - Lambda: `this` refers to enclosing class instance
   - Anonymous class: `this` refers to anonymous class instance
   - Lambda has no `this` of its own (it's not a class)

5. **What is the difference between `andThen` and `compose`?**
   - `f.andThen(g)`: applies f first, then g → g(f(x))
   - `f.compose(g)`: applies g first, then f → f(g(x))
   - `andThen` reads left-to-right, more intuitive

### Intermediate Questions

6. **Why prefer primitive specializations like `IntPredicate`?**
   - Avoids autoboxing/unboxing overhead
   - Better performance for numeric operations
   - Example: `IntStream.sum()` vs `Stream<Integer>.reduce()`

7. **How do you handle checked exceptions in lambdas?**
   - Wrap in try-catch inside lambda
   - Create utility wrapper methods
   - Use custom functional interfaces that declare throws

8. **What are side effects? Why avoid them in functional code?**
   - Observable changes outside function (I/O, mutation)
   - Breaks referential transparency
   - Causes issues with parallel execution
   - Makes code harder to reason about

9. **What is a closure in Java?**
   - A lambda that captures variables from its enclosing scope
   - Captured variables must be effectively final
   - The lambda "closes over" the captured values

10. **How does lambda compilation differ from anonymous classes?**
    - Anonymous class: Creates separate .class file
    - Lambda: Uses invokedynamic bytecode instruction
    - Lambda creation can be optimized by JVM at runtime
    - Lambda may be more memory efficient

### Advanced Questions

11. **Explain how to implement partial application / currying in Java.**
    ```java
    // Currying: f(a,b) -> f(a)(b)
    Function<Integer, Function<Integer, Integer>> curriedAdd = 
        a -> b -> a + b;
    Function<Integer, Integer> add5 = curriedAdd.apply(5);
    int result = add5.apply(3);  // 8
    
    // Partial application
    BiFunction<Integer, Integer, Integer> add = Integer::sum;
    Function<Integer, Integer> add10 = x -> add.apply(10, x);
    ```

12. **What is the diamond problem with default methods?**
    - Two interfaces with same default method signature
    - Implementing class must override to resolve
    - Use `InterfaceName.super.methodName()` to call specific default

13. **How would you implement a memoizing function wrapper?**
    ```java
    public static <T, R> Function<T, R> memoize(Function<T, R> f) {
        Map<T, R> cache = new ConcurrentHashMap<>();
        return t -> cache.computeIfAbsent(t, f);
    }
    ```

14. **Why shouldn't you use `Optional` for fields or method parameters?**
    - `Optional` was designed for return types
    - Fields: Use null with proper documentation
    - Parameters: Use overloaded methods or @Nullable annotation
    - Adds unnecessary complexity and allocation

15. **How do parallel streams interact with effectively final?**
    - Captured variables are accessed by multiple threads
    - Mutation would cause race conditions
    - Effectively final ensures safe concurrent access to captured values

---

## Quick Reference

### Common Functional Interfaces

| Interface | Method | Signature | Use Case |
|-----------|--------|-----------|----------|
| `Predicate<T>` | `test` | `T → boolean` | Filter, validation |
| `Function<T,R>` | `apply` | `T → R` | Transform |
| `Consumer<T>` | `accept` | `T → void` | Process |
| `Supplier<T>` | `get` | `() → T` | Factory |
| `UnaryOperator<T>` | `apply` | `T → T` | Same-type transform |
| `BinaryOperator<T>` | `apply` | `(T,T) → T` | Reduce |
| `BiFunction<T,U,R>` | `apply` | `(T,U) → R` | Two-arg transform |
| `BiPredicate<T,U>` | `test` | `(T,U) → boolean` | Two-arg test |
| `BiConsumer<T,U>` | `accept` | `(T,U) → void` | Two-arg process |

### Primitive Specializations Pattern

| Generic | Int | Long | Double |
|---------|-----|------|--------|
| `Predicate<T>` | `IntPredicate` | `LongPredicate` | `DoublePredicate` |
| `Function<T,R>` | `IntFunction<R>` | `LongFunction<R>` | `DoubleFunction<R>` |
| `Consumer<T>` | `IntConsumer` | `LongConsumer` | `DoubleConsumer` |
| `Supplier<T>` | `IntSupplier` | `LongSupplier` | `DoubleSupplier` |
| `UnaryOperator<T>` | `IntUnaryOperator` | `LongUnaryOperator` | `DoubleUnaryOperator` |
| `BinaryOperator<T>` | `IntBinaryOperator` | `LongBinaryOperator` | `DoubleBinaryOperator` |
