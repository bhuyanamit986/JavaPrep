# Design Patterns (Practical Interview Set)

Design patterns are reusable solutions to common software design problems. This guide covers patterns most frequently asked in Java interviews, with practical examples and real-world use cases.

## Definitions

- **Design pattern**: a proven template for solving a recurring design problem.
- **Creational pattern**: focuses on object creation (e.g., Singleton, Factory, Builder).
- **Structural pattern**: focuses on object composition (e.g., Adapter, Decorator).
- **Behavioral pattern**: focuses on object interaction (e.g., Strategy, Observer).
- **Singleton**: ensures one instance with global access (use carefully).
- **Strategy**: swaps behavior at runtime via composition.

## Illustrations

- **Factory**: ordering a product from a menu instead of assembling it yourself.
- **Strategy**: choosing different routes (fastest, cheapest) for the same trip.
- **Observer**: subscribing to updates and getting notified when something changes.

## Code Examples

```java
interface Formatter { String format(String s); }
class UpperCase implements Formatter {
    public String format(String s) { return s.toUpperCase(); }
}
class Printer {
    private final Formatter formatter;
    Printer(Formatter formatter) { this.formatter = formatter; }
    String print(String s) { return formatter.format(s); }
}
```

## Interview Questions

1. What problem does the Singleton pattern solve, and what are its risks?
2. When would you pick Strategy over inheritance?
3. Explain the difference between Factory and Builder.
4. How does Decorator differ from Proxy?
5. Give a real example of Observer in Java.

---

## 1) Pattern Categories

| Category | Purpose | Patterns |
|----------|---------|----------|
| **Creational** | Object creation | Singleton, Factory, Abstract Factory, Builder, Prototype |
| **Structural** | Object composition | Adapter, Decorator, Facade, Proxy, Composite, Bridge, Flyweight |
| **Behavioral** | Object communication | Strategy, Observer, Template Method, Command, Iterator, State, Chain of Responsibility |

---

## 2) Singleton Pattern

**Intent**: Ensure a class has only one instance and provide global access to it.

### Basic Implementation (NOT Thread-Safe)

```java
public class Singleton {
    private static Singleton instance;
    
    private Singleton() {}  // Private constructor
    
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();  // Race condition!
        }
        return instance;
    }
}
```

### Thread-Safe: Synchronized Method

```java
public class Singleton {
    private static Singleton instance;
    
    private Singleton() {}
    
    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
// Problem: Synchronization overhead on every call
```

### Thread-Safe: Double-Checked Locking

```java
public class Singleton {
    private static volatile Singleton instance;  // volatile is CRITICAL!
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        if (instance == null) {                  // First check (no lock)
            synchronized (Singleton.class) {
                if (instance == null) {          // Second check (with lock)
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

**Why volatile?** Without it, the compiler might reorder the initialization, causing another thread to see a partially constructed object.

### Thread-Safe: Bill Pugh Singleton (Recommended)

```java
public class Singleton {
    private Singleton() {}
    
    private static class Holder {
        private static final Singleton INSTANCE = new Singleton();
    }
    
    public static Singleton getInstance() {
        return Holder.INSTANCE;
    }
}
```

**Why this works:**
- Inner class not loaded until `getInstance()` called (lazy)
- Class loading is inherently thread-safe
- No synchronization overhead

### Thread-Safe: Enum Singleton (Best)

```java
public enum Singleton {
    INSTANCE;
    
    private final Connection connection;
    
    Singleton() {
        connection = createConnection();
    }
    
    public Connection getConnection() {
        return connection;
    }
}

// Usage
Singleton.INSTANCE.getConnection();
```

**Benefits:**
- Thread-safe by JVM guarantee
- Serialization-safe (enums handled specially)
- Reflection-safe (can't instantiate enum via reflection)

### Singleton Pitfalls

1. **Testing difficulty**: Hard to mock/substitute
2. **Hidden dependencies**: Global state is implicit
3. **Concurrency bugs**: If not properly synchronized
4. **Serialization**: Can create new instances (unless enum)
5. **Reflection**: Can bypass private constructor (unless enum)

### When to Use

- Configuration managers
- Logging
- Connection pools
- Caches

### Interview Questions

- How do you make Singleton thread-safe?
- What is double-checked locking and why does it need volatile?
- How does enum prevent reflection attacks?
- What are alternatives to Singleton (dependency injection)?

---

## 3) Factory Patterns

### Simple Factory (Not a GoF Pattern)

```java
public class ShapeFactory {
    public static Shape createShape(String type) {
        return switch (type.toLowerCase()) {
            case "circle" -> new Circle();
            case "rectangle" -> new Rectangle();
            case "triangle" -> new Triangle();
            default -> throw new IllegalArgumentException("Unknown shape: " + type);
        };
    }
}

// Usage
Shape shape = ShapeFactory.createShape("circle");
```

### Factory Method Pattern

**Intent**: Define an interface for creating an object, but let subclasses decide which class to instantiate.

```java
// Product interface
public interface Transport {
    void deliver();
}

// Concrete products
public class Truck implements Transport {
    @Override
    public void deliver() {
        System.out.println("Delivering by land in a truck");
    }
}

public class Ship implements Transport {
    @Override
    public void deliver() {
        System.out.println("Delivering by sea in a ship");
    }
}

// Creator (factory) abstract class
public abstract class Logistics {
    // Factory method
    public abstract Transport createTransport();
    
    // Business logic using the factory method
    public void planDelivery() {
        Transport transport = createTransport();
        transport.deliver();
    }
}

// Concrete creators
public class RoadLogistics extends Logistics {
    @Override
    public Transport createTransport() {
        return new Truck();
    }
}

public class SeaLogistics extends Logistics {
    @Override
    public Transport createTransport() {
        return new Ship();
    }
}

// Usage
Logistics logistics = new RoadLogistics();
logistics.planDelivery();  // Uses Truck
```

**Real-world examples:**
- `java.util.Calendar.getInstance()`
- `java.text.NumberFormat.getInstance()`
- `java.util.ResourceBundle.getBundle()`

### Abstract Factory Pattern

**Intent**: Provide an interface for creating families of related objects without specifying concrete classes.

```java
// Abstract products
public interface Button {
    void render();
}

public interface Checkbox {
    void render();
}

// Concrete products - Windows family
public class WindowsButton implements Button {
    @Override
    public void render() {
        System.out.println("Rendering Windows button");
    }
}

public class WindowsCheckbox implements Checkbox {
    @Override
    public void render() {
        System.out.println("Rendering Windows checkbox");
    }
}

// Concrete products - Mac family
public class MacButton implements Button {
    @Override
    public void render() {
        System.out.println("Rendering Mac button");
    }
}

public class MacCheckbox implements Checkbox {
    @Override
    public void render() {
        System.out.println("Rendering Mac checkbox");
    }
}

// Abstract factory
public interface GUIFactory {
    Button createButton();
    Checkbox createCheckbox();
}

// Concrete factories
public class WindowsFactory implements GUIFactory {
    @Override
    public Button createButton() {
        return new WindowsButton();
    }
    
    @Override
    public Checkbox createCheckbox() {
        return new WindowsCheckbox();
    }
}

public class MacFactory implements GUIFactory {
    @Override
    public Button createButton() {
        return new MacButton();
    }
    
    @Override
    public Checkbox createCheckbox() {
        return new MacCheckbox();
    }
}

// Client code
public class Application {
    private final Button button;
    private final Checkbox checkbox;
    
    public Application(GUIFactory factory) {
        this.button = factory.createButton();
        this.checkbox = factory.createCheckbox();
    }
    
    public void render() {
        button.render();
        checkbox.render();
    }
}

// Usage
GUIFactory factory = detectOS().equals("Windows") 
    ? new WindowsFactory() 
    : new MacFactory();
Application app = new Application(factory);
app.render();
```

### Factory vs Abstract Factory

| Aspect | Factory Method | Abstract Factory |
|--------|----------------|------------------|
| Creates | One product | Family of products |
| Method | Single factory method | Multiple factory methods |
| Inheritance | Through subclassing | Through composition |
| Coupling | Products coupled to creator | Products coupled to factory |

---

## 4) Builder Pattern

**Intent**: Separate the construction of a complex object from its representation, allowing the same construction process to create different representations.

### Classic Builder

```java
public class User {
    private final String firstName;   // Required
    private final String lastName;    // Required
    private final int age;            // Optional
    private final String phone;       // Optional
    private final String address;     // Optional
    
    private User(Builder builder) {
        this.firstName = builder.firstName;
        this.lastName = builder.lastName;
        this.age = builder.age;
        this.phone = builder.phone;
        this.address = builder.address;
    }
    
    public static class Builder {
        // Required parameters
        private final String firstName;
        private final String lastName;
        
        // Optional parameters - initialized to defaults
        private int age = 0;
        private String phone = "";
        private String address = "";
        
        public Builder(String firstName, String lastName) {
            this.firstName = firstName;
            this.lastName = lastName;
        }
        
        public Builder age(int age) {
            this.age = age;
            return this;
        }
        
        public Builder phone(String phone) {
            this.phone = phone;
            return this;
        }
        
        public Builder address(String address) {
            this.address = address;
            return this;
        }
        
        public User build() {
            return new User(this);
        }
    }
    
    // Getters...
}

// Usage
User user = new User.Builder("John", "Doe")
    .age(30)
    .phone("123-456-7890")
    .address("123 Main St")
    .build();
```

### Builder with Validation

```java
public User build() {
    // Validate before building
    if (age < 0) {
        throw new IllegalStateException("Age cannot be negative");
    }
    if (phone != null && !phone.matches("\\d{3}-\\d{3}-\\d{4}")) {
        throw new IllegalStateException("Invalid phone format");
    }
    return new User(this);
}
```

### Builder with Lombok

```java
@Builder
@Getter
public class User {
    private final String firstName;
    private final String lastName;
    @Builder.Default
    private final int age = 0;
    private final String phone;
    private final String address;
}

// Usage
User user = User.builder()
    .firstName("John")
    .lastName("Doe")
    .age(30)
    .build();
```

### Real-world Examples

- `StringBuilder` / `StringBuffer`
- `Stream.Builder`
- `ProcessBuilder`
- `Locale.Builder`
- `Calendar.Builder`

### When to Use

- Many constructor parameters
- Mix of required and optional parameters
- Immutable objects with complex construction
- Objects that need validation before creation

---

## 5) Strategy Pattern

**Intent**: Define a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from clients that use it.

```java
// Strategy interface
public interface PaymentStrategy {
    void pay(int amount);
}

// Concrete strategies
public class CreditCardPayment implements PaymentStrategy {
    private final String cardNumber;
    
    public CreditCardPayment(String cardNumber) {
        this.cardNumber = cardNumber;
    }
    
    @Override
    public void pay(int amount) {
        System.out.println("Paid $" + amount + " using credit card: " + cardNumber);
    }
}

public class PayPalPayment implements PaymentStrategy {
    private final String email;
    
    public PayPalPayment(String email) {
        this.email = email;
    }
    
    @Override
    public void pay(int amount) {
        System.out.println("Paid $" + amount + " using PayPal: " + email);
    }
}

public class CryptoPayment implements PaymentStrategy {
    private final String walletAddress;
    
    public CryptoPayment(String walletAddress) {
        this.walletAddress = walletAddress;
    }
    
    @Override
    public void pay(int amount) {
        System.out.println("Paid $" + amount + " using crypto wallet: " + walletAddress);
    }
}

// Context
public class ShoppingCart {
    private final List<Item> items = new ArrayList<>();
    
    public void addItem(Item item) {
        items.add(item);
    }
    
    public int calculateTotal() {
        return items.stream().mapToInt(Item::getPrice).sum();
    }
    
    public void pay(PaymentStrategy strategy) {
        int total = calculateTotal();
        strategy.pay(total);
    }
}

// Usage
ShoppingCart cart = new ShoppingCart();
cart.addItem(new Item("Book", 50));
cart.addItem(new Item("Laptop", 1000));

// Pay with different strategies
cart.pay(new CreditCardPayment("1234-5678-9012-3456"));
cart.pay(new PayPalPayment("user@example.com"));
cart.pay(new CryptoPayment("0x1234..."));
```

### Strategy with Lambdas (Java 8+)

```java
// Strategy as functional interface
@FunctionalInterface
public interface DiscountStrategy {
    double applyDiscount(double price);
}

// Usage with lambdas
DiscountStrategy noDiscount = price -> price;
DiscountStrategy tenPercent = price -> price * 0.9;
DiscountStrategy twentyPercent = price -> price * 0.8;
DiscountStrategy memberDiscount = price -> price * 0.85;

double price = 100;
System.out.println("No discount: " + noDiscount.applyDiscount(price));
System.out.println("10% off: " + tenPercent.applyDiscount(price));
```

### Real-world Examples

- `java.util.Comparator`
- `java.util.concurrent.RejectedExecutionHandler`
- `javax.servlet.http.HttpServlet` (service strategy)
- Spring's `AuthenticationManager`

---

## 6) Observer Pattern

**Intent**: Define a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically.

```java
// Observer interface
public interface Observer {
    void update(String event);
}

// Subject interface
public interface Subject {
    void attach(Observer observer);
    void detach(Observer observer);
    void notifyObservers(String event);
}

// Concrete subject
public class NewsAgency implements Subject {
    private final List<Observer> observers = new ArrayList<>();
    private String latestNews;
    
    @Override
    public void attach(Observer observer) {
        observers.add(observer);
    }
    
    @Override
    public void detach(Observer observer) {
        observers.remove(observer);
    }
    
    @Override
    public void notifyObservers(String event) {
        for (Observer observer : observers) {
            observer.update(event);
        }
    }
    
    public void publishNews(String news) {
        this.latestNews = news;
        notifyObservers(news);
    }
}

// Concrete observers
public class NewsChannel implements Observer {
    private final String name;
    
    public NewsChannel(String name) {
        this.name = name;
    }
    
    @Override
    public void update(String news) {
        System.out.println(name + " received news: " + news);
    }
}

// Usage
NewsAgency agency = new NewsAgency();

Observer cnn = new NewsChannel("CNN");
Observer bbc = new NewsChannel("BBC");

agency.attach(cnn);
agency.attach(bbc);

agency.publishNews("Breaking: Java 21 released!");
// Output:
// CNN received news: Breaking: Java 21 released!
// BBC received news: Breaking: Java 21 released!

agency.detach(bbc);
agency.publishNews("Update: Java 21 adoption grows");
// Only CNN receives this
```

### Observer with Java Built-ins

```java
// Using PropertyChangeListener (java.beans)
public class ObservableBean {
    private final PropertyChangeSupport support = new PropertyChangeSupport(this);
    private String name;
    
    public void addPropertyChangeListener(PropertyChangeListener listener) {
        support.addPropertyChangeListener(listener);
    }
    
    public void removePropertyChangeListener(PropertyChangeListener listener) {
        support.removePropertyChangeListener(listener);
    }
    
    public void setName(String newName) {
        String oldName = this.name;
        this.name = newName;
        support.firePropertyChange("name", oldName, newName);
    }
}

// Usage
ObservableBean bean = new ObservableBean();
bean.addPropertyChangeListener(event -> 
    System.out.println("Property " + event.getPropertyName() + 
        " changed from " + event.getOldValue() + 
        " to " + event.getNewValue()));
bean.setName("Alice");
```

### Real-world Examples

- Java Swing event listeners
- `java.util.Observer` (deprecated in Java 9)
- JavaBeans `PropertyChangeListener`
- RxJava Observables
- Spring's `ApplicationEventPublisher`

---

## 7) Decorator Pattern

**Intent**: Attach additional responsibilities to an object dynamically. Decorators provide a flexible alternative to subclassing for extending functionality.

```java
// Component interface
public interface Coffee {
    double getCost();
    String getDescription();
}

// Concrete component
public class SimpleCoffee implements Coffee {
    @Override
    public double getCost() {
        return 2.0;
    }
    
    @Override
    public String getDescription() {
        return "Simple coffee";
    }
}

// Base decorator
public abstract class CoffeeDecorator implements Coffee {
    protected final Coffee decoratedCoffee;
    
    public CoffeeDecorator(Coffee coffee) {
        this.decoratedCoffee = coffee;
    }
    
    @Override
    public double getCost() {
        return decoratedCoffee.getCost();
    }
    
    @Override
    public String getDescription() {
        return decoratedCoffee.getDescription();
    }
}

// Concrete decorators
public class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public double getCost() {
        return super.getCost() + 0.5;
    }
    
    @Override
    public String getDescription() {
        return super.getDescription() + ", with milk";
    }
}

public class SugarDecorator extends CoffeeDecorator {
    public SugarDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public double getCost() {
        return super.getCost() + 0.2;
    }
    
    @Override
    public String getDescription() {
        return super.getDescription() + ", with sugar";
    }
}

public class WhipDecorator extends CoffeeDecorator {
    public WhipDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public double getCost() {
        return super.getCost() + 0.7;
    }
    
    @Override
    public String getDescription() {
        return super.getDescription() + ", with whipped cream";
    }
}

// Usage - decorators can be stacked!
Coffee coffee = new SimpleCoffee();
System.out.println(coffee.getDescription() + " $" + coffee.getCost());
// Simple coffee $2.0

coffee = new MilkDecorator(coffee);
System.out.println(coffee.getDescription() + " $" + coffee.getCost());
// Simple coffee, with milk $2.5

coffee = new SugarDecorator(coffee);
System.out.println(coffee.getDescription() + " $" + coffee.getCost());
// Simple coffee, with milk, with sugar $2.7

coffee = new WhipDecorator(coffee);
System.out.println(coffee.getDescription() + " $" + coffee.getCost());
// Simple coffee, with milk, with sugar, with whipped cream $3.4
```

### Real-world Examples (Java I/O)

```java
// Classic decorator chain in Java I/O
InputStream is = new BufferedInputStream(
    new FileInputStream("file.txt"));

// Even more decorators
DataInputStream dis = new DataInputStream(
    new BufferedInputStream(
        new GZIPInputStream(
            new FileInputStream("data.gz"))));
```

### Decorator vs Inheritance

| Aspect | Decorator | Inheritance |
|--------|-----------|-------------|
| Flexibility | Runtime composition | Compile-time |
| Combinations | Mix and match | Class explosion |
| Single Responsibility | Each decorator one feature | Features in one class |
| Complexity | Multiple small classes | Large class hierarchies |

---

## 8) Adapter Pattern

**Intent**: Convert the interface of a class into another interface clients expect. Adapter lets classes work together that couldn't otherwise because of incompatible interfaces.

### Object Adapter (Composition)

```java
// Target interface (what client expects)
public interface MediaPlayer {
    void play(String filename);
}

// Adaptee (existing interface, incompatible)
public class VLCPlayer {
    public void playVLC(String filename) {
        System.out.println("Playing VLC file: " + filename);
    }
}

public class MP4Player {
    public void playMP4(String filename) {
        System.out.println("Playing MP4 file: " + filename);
    }
}

// Adapter
public class MediaAdapter implements MediaPlayer {
    private final VLCPlayer vlcPlayer;
    private final MP4Player mp4Player;
    
    public MediaAdapter() {
        this.vlcPlayer = new VLCPlayer();
        this.mp4Player = new MP4Player();
    }
    
    @Override
    public void play(String filename) {
        if (filename.endsWith(".vlc")) {
            vlcPlayer.playVLC(filename);
        } else if (filename.endsWith(".mp4")) {
            mp4Player.playMP4(filename);
        } else {
            throw new UnsupportedOperationException("Format not supported");
        }
    }
}

// Client code
MediaPlayer player = new MediaAdapter();
player.play("movie.mp4");
player.play("video.vlc");
```

### Class Adapter (Inheritance)

```java
// Only works when adapting single class
public class VLCAdapter extends VLCPlayer implements MediaPlayer {
    @Override
    public void play(String filename) {
        playVLC(filename);
    }
}
```

### Real-world Examples

- `java.util.Arrays#asList()` - Array to List
- `java.io.InputStreamReader` - InputStream to Reader
- `java.io.OutputStreamWriter` - OutputStream to Writer
- JDBC drivers - Adapt different databases to common interface

---

## 9) Template Method Pattern

**Intent**: Define the skeleton of an algorithm in an operation, deferring some steps to subclasses. Template Method lets subclasses redefine certain steps of an algorithm without changing the algorithm's structure.

```java
// Abstract class with template method
public abstract class DataProcessor {
    
    // Template method - defines algorithm structure
    public final void process() {
        readData();
        processData();
        writeData();
        cleanup();  // Hook method (optional override)
    }
    
    // Abstract methods - must be implemented
    protected abstract void readData();
    protected abstract void processData();
    protected abstract void writeData();
    
    // Hook method - default implementation, can be overridden
    protected void cleanup() {
        // Default: do nothing
    }
}

// Concrete implementation for CSV
public class CSVProcessor extends DataProcessor {
    private List<String[]> data;
    
    @Override
    protected void readData() {
        System.out.println("Reading CSV file...");
        // Read CSV logic
    }
    
    @Override
    protected void processData() {
        System.out.println("Processing CSV data...");
        // Process CSV logic
    }
    
    @Override
    protected void writeData() {
        System.out.println("Writing processed CSV...");
        // Write logic
    }
}

// Concrete implementation for JSON
public class JSONProcessor extends DataProcessor {
    private JsonNode data;
    
    @Override
    protected void readData() {
        System.out.println("Reading JSON file...");
    }
    
    @Override
    protected void processData() {
        System.out.println("Processing JSON data...");
    }
    
    @Override
    protected void writeData() {
        System.out.println("Writing processed JSON...");
    }
    
    @Override
    protected void cleanup() {
        System.out.println("Cleaning up JSON resources...");
    }
}

// Usage
DataProcessor csvProcessor = new CSVProcessor();
csvProcessor.process();

DataProcessor jsonProcessor = new JSONProcessor();
jsonProcessor.process();
```

### Real-world Examples

- `java.io.InputStream#read()` - read() calls read(byte[], int, int)
- `java.util.AbstractList#indexOf()` - uses get() which subclass implements
- `javax.servlet.http.HttpServlet#service()` - calls doGet(), doPost(), etc.
- JUnit test lifecycle - @Before, @Test, @After

---

## 10) Proxy Pattern

**Intent**: Provide a surrogate or placeholder for another object to control access to it.

### Types of Proxies

1. **Virtual Proxy**: Lazy initialization
2. **Protection Proxy**: Access control
3. **Remote Proxy**: Local representative for remote object
4. **Logging Proxy**: Add logging

### Virtual Proxy (Lazy Loading)

```java
// Subject interface
public interface Image {
    void display();
}

// Real subject (expensive to create)
public class RealImage implements Image {
    private final String filename;
    
    public RealImage(String filename) {
        this.filename = filename;
        loadFromDisk();  // Expensive operation
    }
    
    private void loadFromDisk() {
        System.out.println("Loading image: " + filename);
        // Simulate expensive loading
    }
    
    @Override
    public void display() {
        System.out.println("Displaying image: " + filename);
    }
}

// Proxy
public class ImageProxy implements Image {
    private final String filename;
    private RealImage realImage;  // Lazy initialized
    
    public ImageProxy(String filename) {
        this.filename = filename;
    }
    
    @Override
    public void display() {
        if (realImage == null) {
            realImage = new RealImage(filename);  // Load only when needed
        }
        realImage.display();
    }
}

// Usage
Image image = new ImageProxy("photo.jpg");
// Image not loaded yet

image.display();  // Now loads and displays
image.display();  // Uses cached RealImage
```

### Protection Proxy (Access Control)

```java
public interface Document {
    void read();
    void write(String content);
}

public class RealDocument implements Document {
    private String content;
    
    @Override
    public void read() {
        System.out.println("Reading: " + content);
    }
    
    @Override
    public void write(String content) {
        this.content = content;
        System.out.println("Writing: " + content);
    }
}

public class SecureDocumentProxy implements Document {
    private final RealDocument document;
    private final User user;
    
    public SecureDocumentProxy(RealDocument document, User user) {
        this.document = document;
        this.user = user;
    }
    
    @Override
    public void read() {
        if (user.hasPermission("READ")) {
            document.read();
        } else {
            throw new SecurityException("No read permission");
        }
    }
    
    @Override
    public void write(String content) {
        if (user.hasPermission("WRITE")) {
            document.write(content);
        } else {
            throw new SecurityException("No write permission");
        }
    }
}
```

### Dynamic Proxy (Java Reflection)

```java
public interface UserService {
    void createUser(String name);
    User getUser(int id);
}

// Logging invocation handler
public class LoggingHandler implements InvocationHandler {
    private final Object target;
    
    public LoggingHandler(Object target) {
        this.target = target;
    }
    
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("Calling: " + method.getName() + " with args: " + Arrays.toString(args));
        long start = System.currentTimeMillis();
        
        try {
            Object result = method.invoke(target, args);
            System.out.println("Result: " + result);
            return result;
        } finally {
            long elapsed = System.currentTimeMillis() - start;
            System.out.println("Method " + method.getName() + " took " + elapsed + "ms");
        }
    }
}

// Creating dynamic proxy
UserService realService = new UserServiceImpl();
UserService proxy = (UserService) Proxy.newProxyInstance(
    UserService.class.getClassLoader(),
    new Class<?>[] { UserService.class },
    new LoggingHandler(realService)
);

proxy.createUser("John");  // Logged automatically
```

### Real-world Examples

- Spring AOP proxies
- Hibernate lazy loading
- RMI (Remote Method Invocation)
- `java.lang.reflect.Proxy`

---

## 11) Facade Pattern

**Intent**: Provide a unified interface to a set of interfaces in a subsystem. Facade defines a higher-level interface that makes the subsystem easier to use.

```java
// Complex subsystem classes
public class CPU {
    public void freeze() { System.out.println("CPU freezing"); }
    public void jump(long position) { System.out.println("CPU jumping to " + position); }
    public void execute() { System.out.println("CPU executing"); }
}

public class Memory {
    public void load(long position, byte[] data) {
        System.out.println("Loading data at " + position);
    }
}

public class HardDrive {
    public byte[] read(long lba, int size) {
        System.out.println("Reading " + size + " bytes from sector " + lba);
        return new byte[size];
    }
}

// Facade - simplifies complex subsystem
public class ComputerFacade {
    private final CPU cpu;
    private final Memory memory;
    private final HardDrive hardDrive;
    
    public ComputerFacade() {
        this.cpu = new CPU();
        this.memory = new Memory();
        this.hardDrive = new HardDrive();
    }
    
    public void start() {
        cpu.freeze();
        memory.load(0, hardDrive.read(0, 1024));
        cpu.jump(0);
        cpu.execute();
    }
}

// Client - simple interface
ComputerFacade computer = new ComputerFacade();
computer.start();  // One call instead of many
```

### Real-world Examples

- JDBC `DriverManager` - Facade for database connectivity
- SLF4J - Facade for logging frameworks
- Spring's `JdbcTemplate` - Simplifies JDBC operations

---

## 12) Command Pattern

**Intent**: Encapsulate a request as an object, thereby letting you parameterize clients with different requests, queue or log requests, and support undoable operations.

```java
// Command interface
public interface Command {
    void execute();
    void undo();
}

// Receiver
public class Light {
    private boolean on = false;
    
    public void turnOn() {
        on = true;
        System.out.println("Light is ON");
    }
    
    public void turnOff() {
        on = false;
        System.out.println("Light is OFF");
    }
}

// Concrete commands
public class LightOnCommand implements Command {
    private final Light light;
    
    public LightOnCommand(Light light) {
        this.light = light;
    }
    
    @Override
    public void execute() {
        light.turnOn();
    }
    
    @Override
    public void undo() {
        light.turnOff();
    }
}

public class LightOffCommand implements Command {
    private final Light light;
    
    public LightOffCommand(Light light) {
        this.light = light;
    }
    
    @Override
    public void execute() {
        light.turnOff();
    }
    
    @Override
    public void undo() {
        light.turnOn();
    }
}

// Invoker
public class RemoteControl {
    private Command command;
    private final Stack<Command> history = new Stack<>();
    
    public void setCommand(Command command) {
        this.command = command;
    }
    
    public void pressButton() {
        command.execute();
        history.push(command);
    }
    
    public void pressUndo() {
        if (!history.isEmpty()) {
            Command lastCommand = history.pop();
            lastCommand.undo();
        }
    }
}

// Usage
Light light = new Light();
Command lightOn = new LightOnCommand(light);
Command lightOff = new LightOffCommand(light);

RemoteControl remote = new RemoteControl();

remote.setCommand(lightOn);
remote.pressButton();  // Light is ON

remote.setCommand(lightOff);
remote.pressButton();  // Light is OFF

remote.pressUndo();    // Light is ON (undid lightOff)
```

### Real-world Examples

- `java.lang.Runnable` - Command with execute() method
- Swing `Action` interface
- Transactions and rollbacks
- Text editor undo/redo

---

## 13) Chain of Responsibility Pattern

**Intent**: Avoid coupling the sender of a request to its receiver by giving more than one object a chance to handle the request. Chain the receiving objects and pass the request along the chain until an object handles it.

```java
// Handler interface
public abstract class Logger {
    public static final int INFO = 1;
    public static final int DEBUG = 2;
    public static final int ERROR = 3;
    
    protected int level;
    protected Logger nextLogger;
    
    public void setNextLogger(Logger nextLogger) {
        this.nextLogger = nextLogger;
    }
    
    public void logMessage(int level, String message) {
        if (this.level <= level) {
            write(message);
        }
        if (nextLogger != null) {
            nextLogger.logMessage(level, message);
        }
    }
    
    protected abstract void write(String message);
}

// Concrete handlers
public class ConsoleLogger extends Logger {
    public ConsoleLogger(int level) {
        this.level = level;
    }
    
    @Override
    protected void write(String message) {
        System.out.println("Console: " + message);
    }
}

public class FileLogger extends Logger {
    public FileLogger(int level) {
        this.level = level;
    }
    
    @Override
    protected void write(String message) {
        System.out.println("File: " + message);
    }
}

public class ErrorLogger extends Logger {
    public ErrorLogger(int level) {
        this.level = level;
    }
    
    @Override
    protected void write(String message) {
        System.err.println("Error: " + message);
    }
}

// Chain setup
Logger errorLogger = new ErrorLogger(Logger.ERROR);
Logger fileLogger = new FileLogger(Logger.DEBUG);
Logger consoleLogger = new ConsoleLogger(Logger.INFO);

consoleLogger.setNextLogger(fileLogger);
fileLogger.setNextLogger(errorLogger);

// Usage
consoleLogger.logMessage(Logger.INFO, "Info message");
// Output: Console: Info message

consoleLogger.logMessage(Logger.DEBUG, "Debug message");
// Output: Console: Debug message, File: Debug message

consoleLogger.logMessage(Logger.ERROR, "Error message");
// Output: Console: Error message, File: Error message, Error: Error message
```

### Real-world Examples

- Servlet Filters
- Spring Security filter chain
- Exception handling (try-catch chains)
- Logging frameworks

---

## 14) State Pattern

**Intent**: Allow an object to alter its behavior when its internal state changes. The object will appear to change its class.

```java
// State interface
public interface State {
    void insertCoin();
    void ejectCoin();
    void selectProduct();
    void dispense();
}

// Context
public class VendingMachine {
    private State noCoinState;
    private State hasCoinState;
    private State soldState;
    
    private State currentState;
    private int inventory = 10;
    
    public VendingMachine() {
        noCoinState = new NoCoinState(this);
        hasCoinState = new HasCoinState(this);
        soldState = new SoldState(this);
        
        currentState = noCoinState;
    }
    
    public void insertCoin() { currentState.insertCoin(); }
    public void ejectCoin() { currentState.ejectCoin(); }
    public void selectProduct() { currentState.selectProduct(); }
    public void dispense() { currentState.dispense(); }
    
    // State transitions
    public void setState(State state) { this.currentState = state; }
    public State getNoCoinState() { return noCoinState; }
    public State getHasCoinState() { return hasCoinState; }
    public State getSoldState() { return soldState; }
    
    public void releaseProduct() {
        if (inventory > 0) {
            System.out.println("Product dispensed!");
            inventory--;
        }
    }
    
    public int getInventory() { return inventory; }
}

// Concrete states
public class NoCoinState implements State {
    private final VendingMachine machine;
    
    public NoCoinState(VendingMachine machine) {
        this.machine = machine;
    }
    
    @Override
    public void insertCoin() {
        System.out.println("Coin inserted");
        machine.setState(machine.getHasCoinState());
    }
    
    @Override
    public void ejectCoin() {
        System.out.println("No coin to eject");
    }
    
    @Override
    public void selectProduct() {
        System.out.println("Insert coin first");
    }
    
    @Override
    public void dispense() {
        System.out.println("Pay first");
    }
}

public class HasCoinState implements State {
    private final VendingMachine machine;
    
    public HasCoinState(VendingMachine machine) {
        this.machine = machine;
    }
    
    @Override
    public void insertCoin() {
        System.out.println("Coin already inserted");
    }
    
    @Override
    public void ejectCoin() {
        System.out.println("Coin returned");
        machine.setState(machine.getNoCoinState());
    }
    
    @Override
    public void selectProduct() {
        System.out.println("Product selected");
        machine.setState(machine.getSoldState());
    }
    
    @Override
    public void dispense() {
        System.out.println("Select product first");
    }
}
```

### State vs Strategy

| Aspect | State | Strategy |
|--------|-------|----------|
| Purpose | Manage state transitions | Choose algorithm |
| Who changes | State changes itself | Client sets strategy |
| Awareness | States know each other | Strategies independent |
| Number | Usually fixed states | Many possible strategies |

---

## 15) Composite Pattern

**Intent**: Compose objects into tree structures to represent part-whole hierarchies. Composite lets clients treat individual objects and compositions uniformly.

```java
// Component
public interface FileSystemComponent {
    String getName();
    int getSize();
    void display(String indent);
}

// Leaf
public class File implements FileSystemComponent {
    private final String name;
    private final int size;
    
    public File(String name, int size) {
        this.name = name;
        this.size = size;
    }
    
    @Override
    public String getName() { return name; }
    
    @Override
    public int getSize() { return size; }
    
    @Override
    public void display(String indent) {
        System.out.println(indent + "📄 " + name + " (" + size + " bytes)");
    }
}

// Composite
public class Directory implements FileSystemComponent {
    private final String name;
    private final List<FileSystemComponent> children = new ArrayList<>();
    
    public Directory(String name) {
        this.name = name;
    }
    
    public void add(FileSystemComponent component) {
        children.add(component);
    }
    
    public void remove(FileSystemComponent component) {
        children.remove(component);
    }
    
    @Override
    public String getName() { return name; }
    
    @Override
    public int getSize() {
        return children.stream()
            .mapToInt(FileSystemComponent::getSize)
            .sum();
    }
    
    @Override
    public void display(String indent) {
        System.out.println(indent + "📁 " + name + " (" + getSize() + " bytes)");
        for (FileSystemComponent child : children) {
            child.display(indent + "  ");
        }
    }
}

// Usage
Directory root = new Directory("root");
Directory docs = new Directory("documents");
Directory images = new Directory("images");

docs.add(new File("resume.pdf", 1024));
docs.add(new File("report.docx", 2048));

images.add(new File("photo.jpg", 5120));
images.add(new File("logo.png", 512));

root.add(docs);
root.add(images);
root.add(new File("readme.txt", 256));

root.display("");
```

---

## 16) Interview Questions

### General
1. What is a design pattern? Why use them?
2. What are the three categories of design patterns?
3. How do you choose which pattern to use?

### Creational
4. Explain the Singleton pattern. How do you make it thread-safe?
5. What's the difference between Factory Method and Abstract Factory?
6. When would you use Builder pattern?
7. What are the downsides of Singleton?

### Structural
8. Explain the Decorator pattern with an example.
9. What is the difference between Adapter and Facade?
10. How does Proxy pattern work? What types exist?
11. When would you use Composite pattern?

### Behavioral
12. Explain Strategy pattern. How is it different from State?
13. How does Observer pattern work? Where have you used it?
14. What is Template Method pattern?
15. Explain Chain of Responsibility with an example.

### Practical
16. How is Decorator used in Java I/O?
17. Give an example of Factory pattern in JDK.
18. How does Spring use design patterns?
19. What pattern is `java.util.Collections.sort()` using?
20. Design a logging framework using patterns.

---

## 17) Quick Reference

### Pattern Selection Guide

| Problem | Pattern |
|---------|---------|
| Single instance needed | Singleton |
| Complex object creation | Builder |
| Object creation varies by subclass | Factory Method |
| Family of related objects | Abstract Factory |
| Add responsibilities dynamically | Decorator |
| Convert interface | Adapter |
| Simplify complex subsystem | Facade |
| Control access to object | Proxy |
| Interchangeable algorithms | Strategy |
| React to state changes | Observer |
| Skeleton algorithm with variable steps | Template Method |
| Request handling chain | Chain of Responsibility |
| Encapsulate request as object | Command |
| Behavior depends on state | State |
| Tree structure | Composite |

