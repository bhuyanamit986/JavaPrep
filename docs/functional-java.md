# Functional Programming in Java

Functional programming (FP) is a programming paradigm that treats computation as the evaluation of mathematical functions. Java 8 introduced functional programming features that enable cleaner, more expressive, and often more maintainable code.

---

## 1) What is Functional Programming?

### Definition

**Functional Programming** is a declarative programming paradigm where programs are constructed by applying and composing functions. It emphasizes:
- **Immutability**: Data doesn't change after creation
- **Pure Functions**: Same input always produces same output, no side effects
- **First-Class Functions**: Functions can be passed as arguments, returned, and assigned
- **Declarative Style**: Describe *what* to compute, not *how* to compute it

### Imperative vs Functional Style

```
┌─────────────────────────────────────────────────────────────────────┐
│                     IMPERATIVE (Traditional)                         │
├─────────────────────────────────────────────────────────────────────┤
│  • Focus on HOW to do something (step-by-step instructions)         │
│  • Uses loops, conditionals, mutable variables                      │
│  • State changes throughout execution                               │
│  • Example: "Iterate through list, check each element, accumulate"  │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                     FUNCTIONAL (Declarative)                         │
├─────────────────────────────────────────────────────────────────────┤
│  • Focus on WHAT to compute (describe the transformation)           │
│  • Uses functions, compositions, immutable data                     │
│  • Minimal state changes                                            │
│  • Example: "Filter elements matching criteria, transform, collect" │
└─────────────────────────────────────────────────────────────────────┘
```

### Code Comparison

```java
// IMPERATIVE: Find names of adults and convert to uppercase
List<String> adultNames = new ArrayList<>();
for (Person person : people) {
    if (person.getAge() >= 18) {
        adultNames.add(person.getName().toUpperCase());
    }
}

// FUNCTIONAL: Same operation, declarative style
List<String> adultNames = people.stream()
    .filter(p -> p.getAge() >= 18)
    .map(p -> p.getName().toUpperCase())
    .collect(Collectors.toList());
```

### Benefits of Functional Programming

| Benefit | Description |
|---------|-------------|
| **Readability** | Code reads like a description of what it does |
| **Testability** | Pure functions are easy to test (no hidden state) |
| **Parallelism** | Immutable data enables safe parallel processing |
| **Composability** | Small functions combine into complex operations |
| **Maintainability** | Less state to track means fewer bugs |
| **Conciseness** | Express complex operations in fewer lines |

---

## 2) Functional Interfaces

### Definition

A **Functional Interface** is an interface with exactly **one abstract method** (SAM - Single Abstract Method). It can have any number of default and static methods. Functional interfaces enable lambda expressions.

### Illustration

```
┌─────────────────────────────────────────────────────────────────────┐
│                      Functional Interface                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  @FunctionalInterface                                               │
│  public interface MyFunction {                                      │
│      │                                                              │
│      │  ┌─────────────────────────────────┐                         │
│      └──│ R apply(T t);  ← ONE abstract  │ ← Target for lambda     │
│         └─────────────────────────────────┘                         │
│                                                                     │
│      default void helper() { }  ← Allowed (any number)              │
│      static void util() { }     ← Allowed (any number)              │
│  }                                                                  │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### @FunctionalInterface Annotation

```java
@FunctionalInterface  // Optional but recommended
public interface Calculator {
    int calculate(int a, int b);  // Single abstract method
    
    // Default methods are allowed
    default int addAndMultiply(int a, int b, int c) {
        return calculate(calculate(a, b), c);
    }
    
    // Static methods are allowed
    static Calculator adder() {
        return (a, b) -> a + b;
    }
}

// Usage
Calculator add = (a, b) -> a + b;
Calculator multiply = (a, b) -> a * b;
System.out.println(add.calculate(5, 3));       // 8
System.out.println(multiply.calculate(5, 3));  // 15
```

### Built-in Functional Interfaces (java.util.function)

```
┌──────────────────────────────────────────────────────────────────────────┐
│                    Core Functional Interfaces                             │
├─────────────────┬──────────────────┬──────────────────────────────────────┤
│    Interface    │     Method       │           Description                │
├─────────────────┼──────────────────┼──────────────────────────────────────┤
│ Predicate<T>    │ boolean test(T)  │ Test a condition                     │
│ Function<T,R>   │ R apply(T)       │ Transform T to R                     │
│ Consumer<T>     │ void accept(T)   │ Consume T (side effect)              │
│ Supplier<T>     │ T get()          │ Supply/produce T                     │
├─────────────────┼──────────────────┼──────────────────────────────────────┤
│ BiPredicate<T,U>│ boolean test(T,U)│ Test with two inputs                 │
│ BiFunction<T,U,R│ R apply(T,U)     │ Transform two inputs to R            │
│ BiConsumer<T,U> │ void accept(T,U) │ Consume two inputs                   │
├─────────────────┼──────────────────┼──────────────────────────────────────┤
│ UnaryOperator<T>│ T apply(T)       │ Transform T to T (same type)         │
│ BinaryOperator<T│ T apply(T,T)     │ Combine two T into one T             │
└─────────────────┴──────────────────┴──────────────────────────────────────┘
```

### Detailed Examples

```java
// Predicate - Tests a condition, returns boolean
Predicate<String> isEmpty = String::isEmpty;
Predicate<String> isNotEmpty = isEmpty.negate();
Predicate<Integer> isPositive = n -> n > 0;
Predicate<Integer> isEven = n -> n % 2 == 0;
Predicate<Integer> isPositiveEven = isPositive.and(isEven);

System.out.println(isPositiveEven.test(4));   // true
System.out.println(isPositiveEven.test(-4));  // false
System.out.println(isPositiveEven.test(3));   // false

// Function - Transforms input to output
Function<String, Integer> length = String::length;
Function<String, String> toUpper = String::toUpperCase;
Function<String, String> trim = String::trim;
Function<String, String> trimThenUpper = trim.andThen(toUpper);

System.out.println(length.apply("hello"));              // 5
System.out.println(trimThenUpper.apply("  hello  "));   // "HELLO"

// Consumer - Performs action, returns nothing
Consumer<String> print = System.out::println;
Consumer<String> log = s -> logger.info("Value: {}", s);
Consumer<String> printAndLog = print.andThen(log);

printAndLog.accept("test");  // Prints and logs "test"

// Supplier - Provides a value
Supplier<Double> randomDouble = Math::random;
Supplier<LocalDateTime> now = LocalDateTime::now;
Supplier<List<String>> emptyList = ArrayList::new;

System.out.println(randomDouble.get());  // Random number
System.out.println(now.get());           // Current datetime

// BiFunction - Two inputs, one output
BiFunction<String, String, String> concat = String::concat;
BiFunction<Integer, Integer, Integer> max = Math::max;

System.out.println(concat.apply("Hello, ", "World"));  // "Hello, World"
System.out.println(max.apply(5, 3));                   // 5

// UnaryOperator - Same input and output type
UnaryOperator<String> exclaim = s -> s + "!";
UnaryOperator<Integer> doubleIt = n -> n * 2;

System.out.println(exclaim.apply("Hello"));   // "Hello!"
System.out.println(doubleIt.apply(5));        // 10

// BinaryOperator - Two same-type inputs, same-type output
BinaryOperator<Integer> add = Integer::sum;
BinaryOperator<String> longer = (a, b) -> a.length() >= b.length() ? a : b;

System.out.println(add.apply(5, 3));              // 8
System.out.println(longer.apply("hi", "hello"));  // "hello"
```

### Primitive Specializations (Avoid Boxing)

```java
// Primitive versions avoid boxing/unboxing overhead
IntPredicate isPositive = n -> n > 0;
IntFunction<String> intToString = Integer::toString;
IntConsumer printInt = System.out::println;
IntSupplier randomInt = () -> (int)(Math.random() * 100);
IntUnaryOperator square = n -> n * n;
IntBinaryOperator sum = Integer::sum;

// Also: LongPredicate, DoubleFunction, etc.
LongSupplier currentTime = System::currentTimeMillis;
DoubleUnaryOperator sqrt = Math::sqrt;
ToIntFunction<String> stringLength = String::length;
```

---

## 3) Lambda Expressions

### Definition

A **Lambda Expression** is a concise way to represent an anonymous function (a function without a name). Lambdas enable you to treat functionality as a method argument or code as data.

### Syntax

```
┌───────────────────────────────────────────────────────────────────────┐
│                        Lambda Syntax                                   │
├───────────────────────────────────────────────────────────────────────┤
│                                                                       │
│    (parameters) -> expression                                         │
│                                                                       │
│    (parameters) -> { statements; }                                    │
│                                                                       │
├───────────────────────────────────────────────────────────────────────┤
│  Examples:                                                            │
│                                                                       │
│    () -> 42                        // No params, returns 42           │
│    x -> x * 2                      // One param, implicit return      │
│    (x) -> x * 2                    // Parentheses optional for one    │
│    (x, y) -> x + y                 // Two params, implicit return     │
│    (int x, int y) -> x + y         // Explicit types                  │
│    (x, y) -> { return x + y; }     // Block body, explicit return     │
│    (String s) -> {                 // Multi-statement block           │
│        String upper = s.toUpperCase();                                │
│        return upper + "!";                                            │
│    }                                                                  │
│                                                                       │
└───────────────────────────────────────────────────────────────────────┘
```

### Lambda vs Anonymous Class

```java
// BEFORE Java 8: Anonymous inner class
Comparator<String> comparator1 = new Comparator<String>() {
    @Override
    public int compare(String s1, String s2) {
        return s1.length() - s2.length();
    }
};

// JAVA 8+: Lambda expression
Comparator<String> comparator2 = (s1, s2) -> s1.length() - s2.length();

// Even shorter with method reference
Comparator<String> comparator3 = Comparator.comparingInt(String::length);
```

### Important Differences: Lambda vs Anonymous Class

```
┌────────────────────────────────────────────────────────────────────────┐
│              Lambda Expression vs Anonymous Inner Class                 │
├────────────────────┬───────────────────────────────────────────────────┤
│      Aspect        │              Difference                           │
├────────────────────┼───────────────────────────────────────────────────┤
│ `this` keyword     │ Lambda: refers to enclosing class                 │
│                    │ Anonymous: refers to the anonymous class itself   │
├────────────────────┼───────────────────────────────────────────────────┤
│ Compilation        │ Lambda: uses invokedynamic (more efficient)       │
│                    │ Anonymous: creates a separate .class file         │
├────────────────────┼───────────────────────────────────────────────────┤
│ State              │ Lambda: stateless (no instance fields)            │
│                    │ Anonymous: can have state                         │
├────────────────────┼───────────────────────────────────────────────────┤
│ Target type        │ Lambda: must be functional interface              │
│                    │ Anonymous: any interface or abstract class        │
└────────────────────┴───────────────────────────────────────────────────┘
```

### `this` Behavior Example

```java
public class ThisExample {
    private String name = "Outer";
    
    public void demonstrate() {
        // Anonymous class: 'this' refers to the anonymous class
        Runnable anonymous = new Runnable() {
            private String name = "Anonymous";
            
            @Override
            public void run() {
                System.out.println(this.name);  // "Anonymous"
            }
        };
        
        // Lambda: 'this' refers to the enclosing class (ThisExample)
        Runnable lambda = () -> {
            System.out.println(this.name);  // "Outer"
        };
        
        anonymous.run();  // "Anonymous"
        lambda.run();     // "Outer"
    }
}
```

### Common Lambda Patterns

```java
// Event handling
button.addActionListener(e -> handleClick(e));

// Collection operations
list.forEach(item -> System.out.println(item));
list.removeIf(item -> item == null);
list.replaceAll(s -> s.toUpperCase());

// Comparators
list.sort((a, b) -> a.getName().compareTo(b.getName()));
list.sort(Comparator.comparing(Person::getName));

// Threading
new Thread(() -> {
    System.out.println("Running in thread");
}).start();

// Optional handling
optional.ifPresent(value -> process(value));
optional.orElseGet(() -> computeDefault());

// Map operations
map.computeIfAbsent(key, k -> new ArrayList<>());
map.merge(key, 1, Integer::sum);
```

---

## 4) Method References

### Definition

A **Method Reference** is a shorthand notation for a lambda expression that only calls an existing method. It provides a more readable alternative when a lambda's body just invokes a method.

### Four Types of Method References

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      Method Reference Types                              │
├──────────────────────────┬──────────────────────────────────────────────┤
│         Type             │              Syntax                          │
├──────────────────────────┼──────────────────────────────────────────────┤
│ Static method            │  ClassName::staticMethod                     │
│ Instance method (bound)  │  instance::instanceMethod                    │
│ Instance method (unbound)│  ClassName::instanceMethod                   │
│ Constructor              │  ClassName::new                              │
└──────────────────────────┴──────────────────────────────────────────────┘
```

### Detailed Examples

```java
// 1. STATIC METHOD REFERENCE: ClassName::staticMethod
// Lambda equivalent: (args) -> ClassName.staticMethod(args)

Function<String, Integer> parseInt1 = s -> Integer.parseInt(s);  // Lambda
Function<String, Integer> parseInt2 = Integer::parseInt;         // Method ref

BiFunction<Integer, Integer, Integer> max1 = (a, b) -> Math.max(a, b);
BiFunction<Integer, Integer, Integer> max2 = Math::max;

// 2. BOUND INSTANCE METHOD: instance::instanceMethod
// Lambda equivalent: (args) -> instance.method(args)

String prefix = "Hello, ";
Function<String, String> greeter1 = s -> prefix.concat(s);  // Lambda
Function<String, String> greeter2 = prefix::concat;         // Method ref

List<String> list = new ArrayList<>();
Consumer<String> adder1 = s -> list.add(s);  // Lambda
Consumer<String> adder2 = list::add;         // Method ref

// 3. UNBOUND INSTANCE METHOD: ClassName::instanceMethod
// Lambda equivalent: (obj, args) -> obj.method(args)
// First parameter becomes the receiver

Function<String, Integer> length1 = s -> s.length();       // Lambda
Function<String, Integer> length2 = String::length;        // Method ref

BiPredicate<String, String> equals1 = (s1, s2) -> s1.equals(s2);
BiPredicate<String, String> equals2 = String::equals;

BiFunction<String, String, String> concat1 = (s1, s2) -> s1.concat(s2);
BiFunction<String, String, String> concat2 = String::concat;

// 4. CONSTRUCTOR REFERENCE: ClassName::new
// Lambda equivalent: (args) -> new ClassName(args)

Supplier<ArrayList<String>> listFactory1 = () -> new ArrayList<>();
Supplier<ArrayList<String>> listFactory2 = ArrayList::new;

Function<Integer, int[]> arrayFactory1 = size -> new int[size];
Function<Integer, int[]> arrayFactory2 = int[]::new;

Function<String, StringBuilder> sbFactory1 = s -> new StringBuilder(s);
Function<String, StringBuilder> sbFactory2 = StringBuilder::new;
```

### Visual Comparison

```
┌────────────────────────────────────────────────────────────────────────┐
│                   Lambda → Method Reference Conversion                  │
├────────────────────────────────────────────────────────────────────────┤
│                                                                        │
│  Lambda:  s -> System.out.println(s)                                   │
│  RefType: bound instance method                                        │
│  MethodRef: System.out::println                                        │
│                                                                        │
│  Lambda:  (s1, s2) -> s1.compareToIgnoreCase(s2)                       │
│  RefType: unbound instance method                                      │
│  MethodRef: String::compareToIgnoreCase                                │
│                                                                        │
│  Lambda:  list -> Collections.sort(list)                               │
│  RefType: static method                                                │
│  MethodRef: Collections::sort                                          │
│                                                                        │
│  Lambda:  () -> new HashMap<>()                                        │
│  RefType: constructor                                                  │
│  MethodRef: HashMap::new                                               │
│                                                                        │
└────────────────────────────────────────────────────────────────────────┘
```

### When to Use Method References

```java
// ✅ USE method reference when lambda just delegates to a method
list.forEach(System.out::println);              // Clear and concise
list.stream().map(String::toUpperCase);         // Readable
employees.sort(Comparator.comparing(Employee::getSalary));

// ❌ KEEP lambda when there's transformation or multiple operations
list.stream().map(s -> s.substring(0, 3));      // Needs args
list.stream().filter(s -> s.length() > 5);      // Needs comparison
list.stream().map(s -> prefix + s);             // Needs concatenation
```

---

## 5) Effectively Final and Variable Capture

### Definition

Variables used in lambda expressions must be **final or effectively final**. A variable is **effectively final** if it's never reassigned after initialization (even without the `final` keyword).

### Why This Restriction?

```
┌─────────────────────────────────────────────────────────────────────────┐
│                   Why Effectively Final?                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. THREAD SAFETY: Lambdas may execute in different threads             │
│     - Mutable shared state leads to race conditions                     │
│                                                                         │
│  2. CLOSURE SEMANTICS: Lambda captures a snapshot of the value          │
│     - Changes after capture would be confusing                          │
│                                                                         │
│  3. PREDICTABILITY: Code behaves consistently                           │
│     - No surprising changes to captured values                          │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Code Examples

```java
// ✅ VALID: final variable
final int multiplier = 10;
Function<Integer, Integer> multiplyByTen = x -> x * multiplier;

// ✅ VALID: effectively final (never reassigned)
int base = 100;  // No 'final' keyword, but never reassigned
Function<Integer, Integer> addBase = x -> x + base;

// ❌ INVALID: variable is reassigned
int counter = 0;
// counter++;  // This would make it not effectively final
// Consumer<String> count = s -> counter++;  // Compile error!

// ❌ INVALID: can't modify captured variable
int total = 0;
// list.forEach(n -> total += n);  // Compile error: can't modify!

// ✅ WORKAROUND: Use mutable container
int[] totalArr = {0};  // Array is effectively final
list.forEach(n -> totalArr[0] += n);  // Contents can change

// ✅ BETTER WORKAROUND: Use AtomicInteger
AtomicInteger atomicTotal = new AtomicInteger(0);
list.forEach(n -> atomicTotal.addAndGet(n));

// ✅ BEST: Use stream reduction (functional approach)
int total = list.stream().mapToInt(Integer::intValue).sum();
```

### What Can Be Captured

```java
public class CaptureExample {
    
    private int instanceField = 100;         // Instance field - OK
    private static int staticField = 200;    // Static field - OK
    
    public void demonstrateCapture() {
        final int finalLocal = 10;           // Final local - OK
        int effectivelyFinal = 20;           // Effectively final - OK
        int mutableLocal = 30;
        
        // Lambda can capture:
        Function<Integer, Integer> lambda = x -> {
            int sum = x;
            sum += instanceField;     // ✅ Instance fields - OK
            sum += staticField;       // ✅ Static fields - OK
            sum += finalLocal;        // ✅ Final locals - OK
            sum += effectivelyFinal;  // ✅ Effectively final - OK
            // sum += mutableLocal;   // ❌ Would fail if mutableLocal is modified
            return sum;
        };
        
        // mutableLocal = 40;  // This would break the lambda above!
    }
}
```

### Common Pitfalls

```java
// ❌ PITFALL: Capturing mutable object state
List<String> items = new ArrayList<>();
Runnable addItem = () -> items.add("item");  // Compiles but risky!
// items is effectively final (reference doesn't change)
// But contents can change - potential race condition!

// ❌ PITFALL: Modification in forEach
List<String> original = Arrays.asList("a", "b", "c");
List<String> results = new ArrayList<>();
original.forEach(s -> results.add(s.toUpperCase()));  // Works but not ideal

// ✅ BETTER: Use collect (thread-safe, immutable)
List<String> results = original.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

---

## 6) Functional Composition

### Definition

**Function Composition** is combining simple functions to build more complex ones. It's a fundamental concept in functional programming that enables building pipelines of transformations.

### Composition Methods

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Function Composition Methods                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Function<T,R>:                                                         │
│    andThen(after)  →  this.apply(x), then after.apply(result)          │
│    compose(before) →  before.apply(x), then this.apply(result)         │
│                                                                         │
│  Predicate<T>:                                                          │
│    and(other)      →  this.test(x) && other.test(x)                    │
│    or(other)       →  this.test(x) || other.test(x)                    │
│    negate()        →  !this.test(x)                                    │
│                                                                         │
│  Consumer<T>:                                                           │
│    andThen(after)  →  this.accept(x), then after.accept(x)             │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Visualizing andThen vs compose

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        f.andThen(g)                                      │
│                                                                         │
│         Input ──► [ f ] ──► intermediate ──► [ g ] ──► Output           │
│                                                                         │
│         "Apply f first, then g"                                         │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│                        f.compose(g)                                      │
│                                                                         │
│         Input ──► [ g ] ──► intermediate ──► [ f ] ──► Output           │
│                                                                         │
│         "Apply g first, then f"                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Code Examples

```java
// Function composition with andThen and compose
Function<String, String> trim = String::trim;
Function<String, String> toUpper = String::toUpperCase;
Function<String, Integer> length = String::length;

// andThen: Apply in sequence (left to right)
Function<String, String> trimThenUpper = trim.andThen(toUpper);
Function<String, Integer> trimUpperLength = trim.andThen(toUpper).andThen(length);

System.out.println(trimThenUpper.apply("  hello  "));      // "HELLO"
System.out.println(trimUpperLength.apply("  hello  "));    // 5

// compose: Apply in reverse order (right to left, like math f(g(x)))
Function<String, String> upperThenTrim = toUpper.compose(trim);
// Same as: trim.andThen(toUpper)

// Predicate composition
Predicate<Integer> isPositive = n -> n > 0;
Predicate<Integer> isEven = n -> n % 2 == 0;
Predicate<Integer> isLessThan100 = n -> n < 100;

Predicate<Integer> isPositiveAndEven = isPositive.and(isEven);
Predicate<Integer> isPositiveOrEven = isPositive.or(isEven);
Predicate<Integer> isNotPositive = isPositive.negate();
Predicate<Integer> isPositiveEvenUnder100 = isPositive.and(isEven).and(isLessThan100);

System.out.println(isPositiveAndEven.test(4));   // true
System.out.println(isPositiveAndEven.test(-4));  // false
System.out.println(isPositiveAndEven.test(3));   // false
System.out.println(isPositiveOrEven.test(-4));   // true (even)

// Consumer composition
Consumer<String> print = System.out::println;
Consumer<String> log = s -> System.err.println("[LOG] " + s);
Consumer<String> printAndLog = print.andThen(log);

printAndLog.accept("Hello");
// Prints: Hello
// Prints: [LOG] Hello

// Building a validation pipeline
Predicate<String> notNull = Objects::nonNull;
Predicate<String> notEmpty = s -> !s.isEmpty();
Predicate<String> notTooLong = s -> s.length() <= 100;
Predicate<String> validFormat = s -> s.matches("[a-zA-Z0-9]+");

Predicate<String> isValidUsername = notNull
    .and(notEmpty)
    .and(notTooLong)
    .and(validFormat);

System.out.println(isValidUsername.test("JohnDoe123"));  // true
System.out.println(isValidUsername.test(""));            // false
System.out.println(isValidUsername.test("John Doe"));    // false (space)
```

### Building Complex Transformations

```java
// Data processing pipeline
Function<Order, Order> validateOrder = order -> {
    if (order.getItems().isEmpty()) {
        throw new IllegalArgumentException("Empty order");
    }
    return order;
};

Function<Order, Order> calculateTotal = order -> {
    double total = order.getItems().stream()
        .mapToDouble(Item::getPrice)
        .sum();
    order.setTotal(total);
    return order;
};

Function<Order, Order> applyDiscount = order -> {
    if (order.getTotal() > 100) {
        order.setTotal(order.getTotal() * 0.9);
    }
    return order;
};

Function<Order, Order> addTax = order -> {
    order.setTotal(order.getTotal() * 1.1);
    return order;
};

// Compose into a single pipeline
Function<Order, Order> processOrder = validateOrder
    .andThen(calculateTotal)
    .andThen(applyDiscount)
    .andThen(addTax);

Order result = processOrder.apply(order);
```

---

## 7) Default Methods in Interfaces

### Definition

**Default Methods** (Java 8+) allow interfaces to have method implementations. They enable adding new methods to interfaces without breaking existing implementations.

### Why Default Methods?

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Purpose of Default Methods                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. BACKWARD COMPATIBILITY                                              │
│     - Add new methods to interfaces without breaking implementations    │
│     - Example: Collection.stream() added in Java 8                      │
│                                                                         │
│  2. OPTIONAL BEHAVIOR                                                   │
│     - Provide default implementation that can be overridden             │
│     - Example: Comparator.reversed()                                    │
│                                                                         │
│  3. FUNCTIONAL COMPOSITION                                              │
│     - Enable chaining methods like and(), or(), andThen()               │
│     - Example: Predicate.and(), Function.andThen()                      │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Code Examples

```java
public interface Vehicle {
    // Abstract method (must implement)
    void start();
    void stop();
    
    // Default method (has implementation)
    default void honk() {
        System.out.println("Beep beep!");
    }
    
    // Static method
    static Vehicle create(String type) {
        return type.equals("car") ? new Car() : new Bike();
    }
}

public class Car implements Vehicle {
    @Override
    public void start() {
        System.out.println("Car starting");
    }
    
    @Override
    public void stop() {
        System.out.println("Car stopping");
    }
    
    // Can override default method
    @Override
    public void honk() {
        System.out.println("Car horn: HONK!");
    }
}

public class Bike implements Vehicle {
    @Override
    public void start() {
        System.out.println("Bike starting");
    }
    
    @Override
    public void stop() {
        System.out.println("Bike stopping");
    }
    
    // Uses default honk() implementation
}
```

### Diamond Problem Resolution

```java
interface A {
    default void method() {
        System.out.println("A");
    }
}

interface B {
    default void method() {
        System.out.println("B");
    }
}

// Class implements both interfaces with same default method
class C implements A, B {
    // MUST override to resolve conflict
    @Override
    public void method() {
        // Can call specific interface's method
        A.super.method();  // Calls A's version
        // B.super.method();  // Or call B's version
        // Or provide completely new implementation
    }
}
```

### Resolution Rules

```
┌─────────────────────────────────────────────────────────────────────────┐
│              Default Method Conflict Resolution Rules                    │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. CLASS WINS: Class method beats interface default                    │
│     class A { void m() {} }                                             │
│     interface I { default void m() {} }                                 │
│     class B extends A implements I { }  // Uses A.m()                   │
│                                                                         │
│  2. SUBTYPE WINS: More specific interface wins                          │
│     interface I { default void m() {} }                                 │
│     interface J extends I { default void m() {} }                       │
│     class A implements J { }  // Uses J.m()                             │
│                                                                         │
│  3. EXPLICIT CHOICE: If ambiguous, class must override                  │
│     interface I { default void m() {} }                                 │
│     interface J { default void m() {} }                                 │
│     class A implements I, J {                                           │
│         @Override void m() { I.super.m(); }  // Must choose             │
│     }                                                                   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 8) Pure Functions and Referential Transparency

### Definition

A **Pure Function** is a function that:
1. Always returns the same output for the same input
2. Has no side effects (doesn't modify external state)

**Referential Transparency** means an expression can be replaced with its value without changing the program's behavior.

### Pure vs Impure Functions

```java
// ✅ PURE FUNCTION: Same input → same output, no side effects
public int add(int a, int b) {
    return a + b;  // Only depends on inputs
}

public String toUpperCase(String s) {
    return s.toUpperCase();  // Only depends on input
}

// ❌ IMPURE FUNCTION: Depends on/modifies external state
private int counter = 0;

public int incrementAndGet() {
    return ++counter;  // Modifies external state
}

public int addWithLogging(int a, int b) {
    System.out.println("Adding " + a + " and " + b);  // Side effect
    return a + b;
}

public int getCurrentHour() {
    return LocalDateTime.now().getHour();  // Depends on external state (time)
}
```

### Benefits of Pure Functions

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Benefits of Pure Functions                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ✓ TESTABLE: Easy to test (no mocking, no setup)                        │
│  ✓ CACHEABLE: Results can be memoized                                   │
│  ✓ PARALLELIZABLE: Safe to execute in parallel                          │
│  ✓ PREDICTABLE: Easier to reason about                                  │
│  ✓ COMPOSABLE: Can be combined freely                                   │
│  ✓ DEBUGGABLE: Same inputs always reproduce same behavior               │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Making Functions Pure

```java
// ❌ IMPURE: Uses system time
public boolean isExpired(Document doc) {
    return doc.getExpiryDate().isBefore(LocalDate.now());
}

// ✅ PURE: Time is passed as parameter
public boolean isExpired(Document doc, LocalDate currentDate) {
    return doc.getExpiryDate().isBefore(currentDate);
}

// ❌ IMPURE: Modifies input list
public void addTax(List<Item> items) {
    for (Item item : items) {
        item.setPrice(item.getPrice() * 1.1);  // Mutates!
    }
}

// ✅ PURE: Returns new list, doesn't modify input
public List<Item> withTax(List<Item> items) {
    return items.stream()
        .map(item -> new Item(item.getName(), item.getPrice() * 1.1))
        .collect(Collectors.toList());
}
```

---

## 9) Side Effects and Functional Best Practices

### What are Side Effects?

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         Types of Side Effects                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  • Modifying a global variable or static field                          │
│  • Modifying a mutable argument passed by reference                     │
│  • Writing to a file or database                                        │
│  • Printing to console or logging                                       │
│  • Making network calls                                                 │
│  • Throwing exceptions                                                  │
│  • Reading from external sources (time, random, user input)             │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Side Effects in Stream Operations

```java
// ❌ BAD: Side effects in stream operations
List<String> results = new ArrayList<>();
list.stream()
    .filter(s -> s.length() > 3)
    .forEach(s -> results.add(s));  // Side effect!

// ✅ GOOD: Use collectors
List<String> results = list.stream()
    .filter(s -> s.length() > 3)
    .collect(Collectors.toList());

// ❌ BAD: Modifying external state in map
int[] count = {0};
list.stream()
    .map(s -> {
        count[0]++;  // Side effect!
        return s.toUpperCase();
    })
    .collect(Collectors.toList());

// ✅ GOOD: Use separate operations
long count = list.stream().count();
List<String> upper = list.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());

// ❌ ESPECIALLY BAD with parallel streams: Race condition!
List<String> results = Collections.synchronizedList(new ArrayList<>());
list.parallelStream()
    .filter(s -> s.length() > 3)
    .forEach(results::add);  // Non-deterministic order, potential race!
```

### Best Practices

```java
// 1. Prefer stream collect over forEach with side effects
// ❌ 
List<String> filtered = new ArrayList<>();
list.forEach(s -> { if (s.length() > 3) filtered.add(s); });

// ✅
List<String> filtered = list.stream()
    .filter(s -> s.length() > 3)
    .collect(Collectors.toList());

// 2. Prefer transformation over mutation
// ❌
public void processOrder(Order order) {
    order.setStatus("PROCESSED");
    order.setProcessedAt(LocalDateTime.now());
}

// ✅
public Order processOrder(Order order, LocalDateTime processedAt) {
    return new Order(order.getId(), order.getItems(), "PROCESSED", processedAt);
}

// 3. Keep side effects at the boundaries
// ❌
public double calculateTotal(List<Item> items) {
    logger.info("Calculating total");  // Side effect in business logic
    return items.stream().mapToDouble(Item::getPrice).sum();
}

// ✅
public double calculateTotal(List<Item> items) {
    return items.stream().mapToDouble(Item::getPrice).sum();
}
// Let caller handle logging:
// logger.info("Calculating total");
// double total = calculateTotal(items);
// logger.info("Total: {}", total);

// 4. Use Optional instead of null
// ❌
public User findUser(Long id) {
    User user = userRepository.find(id);
    return user;  // Might return null
}

// ✅
public Optional<User> findUser(Long id) {
    return userRepository.findById(id);
}
```

---

## 10) Higher-Order Functions

### Definition

A **Higher-Order Function** is a function that either:
1. Takes one or more functions as arguments, OR
2. Returns a function as its result

### Examples

```java
// 1. Function that takes a function as argument
public <T, R> List<R> map(List<T> list, Function<T, R> mapper) {
    List<R> result = new ArrayList<>();
    for (T item : list) {
        result.add(mapper.apply(item));
    }
    return result;
}

// Usage
List<String> names = Arrays.asList("alice", "bob", "charlie");
List<String> upperNames = map(names, String::toUpperCase);

// 2. Function that returns a function
public Function<Integer, Integer> multiplier(int factor) {
    return n -> n * factor;
}

// Usage
Function<Integer, Integer> double_ = multiplier(2);
Function<Integer, Integer> triple = multiplier(3);
System.out.println(double_.apply(5));  // 10
System.out.println(triple.apply(5));   // 15

// 3. Currying: Transform multi-arg function to chain of single-arg functions
// Regular function
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
System.out.println(add.apply(3, 5));  // 8

// Curried version
Function<Integer, Function<Integer, Integer>> curriedAdd = 
    a -> b -> a + b;
    
Function<Integer, Integer> add3 = curriedAdd.apply(3);
System.out.println(add3.apply(5));  // 8
System.out.println(curriedAdd.apply(3).apply(5));  // 8

// 4. Decorator/Wrapper pattern with functions
public <T, R> Function<T, R> withLogging(Function<T, R> fn, String name) {
    return input -> {
        System.out.println("Entering " + name + " with: " + input);
        R result = fn.apply(input);
        System.out.println("Exiting " + name + " with: " + result);
        return result;
    };
}

// Usage
Function<String, Integer> length = String::length;
Function<String, Integer> loggedLength = withLogging(length, "length");
loggedLength.apply("hello");
// Output:
// Entering length with: hello
// Exiting length with: 5
```

---

## 11) Optional as a Functional Container

### Using Optional Functionally

```java
// Optional methods for functional style
Optional<User> optUser = findUser(id);

// map - Transform if present
Optional<String> name = optUser.map(User::getName);
Optional<Integer> nameLength = optUser.map(User::getName).map(String::length);

// flatMap - For nested Optionals
Optional<Address> address = optUser.flatMap(User::getAddress);
// If getAddress returns Optional<Address>

// filter - Conditionally keep value
Optional<User> adultUser = optUser.filter(u -> u.getAge() >= 18);

// Chaining operations
String result = findUser(id)
    .filter(u -> u.isActive())
    .map(User::getEmail)
    .map(String::toLowerCase)
    .orElse("no-email@default.com");

// ifPresent - Execute if present
optUser.ifPresent(user -> sendEmail(user.getEmail()));

// ifPresentOrElse (Java 9+)
optUser.ifPresentOrElse(
    user -> sendEmail(user.getEmail()),
    () -> logger.warn("User not found")
);

// or (Java 9+) - Provide alternative Optional
Optional<User> user = findUserById(id)
    .or(() -> findUserByEmail(email))
    .or(() -> createDefaultUser());

// stream (Java 9+) - Convert to stream
Stream<User> userStream = optUser.stream();
List<String> names = users.stream()
    .map(this::findUser)
    .flatMap(Optional::stream)  // Filters out empty Optionals
    .map(User::getName)
    .collect(Collectors.toList());
```

### Optional Anti-Patterns

```java
// ❌ BAD: Using Optional as method parameter
public void process(Optional<String> name) { }  // Don't do this!

// ✅ GOOD: Use overloaded methods or nullable
public void process(String name) { }
public void process() { process(null); }

// ❌ BAD: Using Optional for class fields
public class User {
    private Optional<String> middleName;  // Don't do this!
}

// ✅ GOOD: Use nullable field
public class User {
    private String middleName;  // May be null
    
    public Optional<String> getMiddleName() {
        return Optional.ofNullable(middleName);
    }
}

// ❌ BAD: Using isPresent() + get()
if (optUser.isPresent()) {
    User user = optUser.get();
    process(user);
}

// ✅ GOOD: Use ifPresent or map
optUser.ifPresent(this::process);
optUser.map(this::transform).orElse(default);
```

---

## 12) Common Interview Questions

### Lambda Basics

**Q1: What is a lambda expression? How does it differ from an anonymous class?**

**Answer:**
- Lambda is a concise way to represent a function (anonymous function implementation)
- Differences:
  - `this` refers to enclosing class in lambda, but anonymous class instance in anonymous class
  - Lambdas use invokedynamic (more efficient), anonymous classes create separate .class files
  - Lambdas can only implement functional interfaces (single abstract method)
  - Anonymous classes can implement interfaces with multiple methods or extend classes

**Q2: What is a functional interface?**

**Answer:**
- An interface with exactly ONE abstract method (SAM - Single Abstract Method)
- Can have any number of default and static methods
- Can be annotated with @FunctionalInterface (optional but recommended)
- Examples: Runnable, Callable, Comparator, Predicate, Function, Consumer, Supplier

**Q3: What does "effectively final" mean?**

**Answer:**
- A variable that is never reassigned after initialization
- Even without the `final` keyword, if a variable is never modified, it's effectively final
- Lambdas can only capture final or effectively final local variables
- Reason: Lambdas may execute in different threads; mutable captured variables would cause race conditions

### Functional Interfaces

**Q4: What are the main functional interfaces in java.util.function?**

**Answer:**
| Interface | Method | Purpose |
|-----------|--------|---------|
| `Predicate<T>` | `boolean test(T)` | Test a condition |
| `Function<T,R>` | `R apply(T)` | Transform T to R |
| `Consumer<T>` | `void accept(T)` | Consume T (side effect) |
| `Supplier<T>` | `T get()` | Supply/produce T |
| `BiFunction<T,U,R>` | `R apply(T,U)` | Transform two inputs |
| `UnaryOperator<T>` | `T apply(T)` | Transform T to T |
| `BinaryOperator<T>` | `T apply(T,T)` | Combine two T |

**Q5: Explain the four types of method references.**

**Answer:**
1. **Static**: `ClassName::staticMethod` → `Integer::parseInt`
2. **Bound instance**: `instance::method` → `System.out::println`
3. **Unbound instance**: `ClassName::instanceMethod` → `String::length`
4. **Constructor**: `ClassName::new` → `ArrayList::new`

### Functional Concepts

**Q6: What is function composition?**

**Answer:**
- Combining simple functions to build more complex ones
- `Function` has `andThen()` and `compose()` methods
- `andThen(g)`: Apply this function, then g
- `compose(g)`: Apply g first, then this function
- `Predicate` has `and()`, `or()`, `negate()` for logical composition

**Q7: What is a pure function?**

**Answer:**
- A function that:
  1. Always returns the same output for the same input
  2. Has no side effects
- Benefits: testable, cacheable, parallelizable, predictable

**Q8: Why should we avoid side effects in stream operations?**

**Answer:**
- Parallel streams can cause race conditions
- Makes code harder to understand and debug
- Non-deterministic behavior
- Use `collect()` instead of modifying external collections

### Best Practices

**Q9: What's wrong with this code?**

```java
List<String> results = new ArrayList<>();
list.parallelStream()
    .filter(s -> s.length() > 3)
    .forEach(s -> results.add(s));
```

**Answer:**
- Race condition: Multiple threads modifying ArrayList concurrently
- Non-deterministic order of results
- Fix: Use `collect(Collectors.toList())` instead

**Q10: When would you use Optional.flatMap() vs Optional.map()?**

**Answer:**
- `map()`: When the function returns a regular value
  - `Optional<String> name = optUser.map(User::getName)`
- `flatMap()`: When the function already returns an Optional (avoids `Optional<Optional<T>>`)
  - `Optional<Address> addr = optUser.flatMap(User::getAddress)` where `getAddress()` returns `Optional<Address>`

---

## 13) Quick Reference

### Lambda Syntax

```java
() -> expression                  // No parameters
x -> expression                   // One parameter (parens optional)
(x, y) -> expression             // Multiple parameters
(x, y) -> { statements; }        // Block body
(int x, int y) -> x + y          // Explicit types
```

### Method Reference Syntax

```java
ClassName::staticMethod          // Static method
instance::instanceMethod         // Bound instance method
ClassName::instanceMethod        // Unbound instance method
ClassName::new                   // Constructor
```

### Key Functional Interfaces

```java
Predicate<T>      test(T) → boolean
Function<T,R>     apply(T) → R
Consumer<T>       accept(T) → void
Supplier<T>       get() → T
BiFunction<T,U,R> apply(T,U) → R
UnaryOperator<T>  apply(T) → T
BinaryOperator<T> apply(T,T) → T
```

### Composition Methods

```java
// Function
f.andThen(g)     // f then g
f.compose(g)     // g then f

// Predicate
p.and(q)         // p && q
p.or(q)          // p || q
p.negate()       // !p

// Consumer
c.andThen(d)     // c then d
```

### Optional Operations

```java
Optional.of(value)           // Non-null value
Optional.ofNullable(value)   // Nullable value
Optional.empty()             // Empty optional

opt.map(fn)                  // Transform if present
opt.flatMap(fn)              // Transform, flatten Optional
opt.filter(pred)             // Keep if predicate matches
opt.orElse(default)          // Get or default
opt.orElseGet(supplier)      // Get or compute default
opt.orElseThrow(supplier)    // Get or throw
opt.ifPresent(consumer)      // Execute if present
```

---

*Functional programming in Java enables cleaner, more maintainable, and more testable code!*
