# Exceptions, Error Handling & Logging

Exception handling is a critical aspect of writing robust Java applications. This guide covers the exception hierarchy, best practices, custom exceptions, try-with-resources, and logging fundamentals commonly asked in interviews.

---

## Definitions

- **Throwable**: The root class of all errors and exceptions in Java. Has two direct subclasses: `Error` and `Exception`.

- **Error**: Represents serious problems that applications typically should not try to catch (e.g., `OutOfMemoryError`, `StackOverflowError`). Usually indicates JVM-level issues.

- **Exception**: Represents conditions that an application might want to catch and handle. The base class for all recoverable problems.

- **Checked exception**: An exception that must be declared in the method signature (`throws` clause) or caught in a `try-catch` block. Checked at compile time.

- **Unchecked exception**: A `RuntimeException` or its subclasses. Not required to be caught or declared. Typically represents programming errors.

- **Try-with-resources**: A `try` statement that automatically closes resources implementing `AutoCloseable` when the block exits, even if an exception occurs.

- **Suppressed exception**: An exception that occurs while closing a resource in try-with-resources. It's attached to the primary exception rather than lost.

- **Exception chaining**: Wrapping one exception in another to preserve the original cause while adding context.

- **Stack trace**: A list of method calls that shows where an exception occurred and how the program got there.

- **Logging level**: A severity indicator (TRACE, DEBUG, INFO, WARN, ERROR) that helps filter log output.

---

## Illustrations

### Exception Hierarchy

```
                            java.lang.Throwable
                                    │
                 ┌──────────────────┴──────────────────┐
                 │                                      │
          java.lang.Error                      java.lang.Exception
          (Don't catch!)                              │
                 │                     ┌──────────────┴──────────────┐
    ┌────────────┴────────────┐        │                              │
    │                         │   RuntimeException              Checked Exceptions
OutOfMemoryError    StackOverflowError  (Unchecked)             (Must handle)
VirtualMachineError    LinkageError        │                         │
AssertionError                   ┌─────────┴─────────┐        ┌──────┴──────┐
                                 │                   │        │             │
                    NullPointerException    IllegalArgumentException  IOException  SQLException
                    IndexOutOfBoundsException  IllegalStateException  FileNotFoundException
                    ClassCastException         UnsupportedOperationException  InterruptedException
                    ArithmeticException                                ParseException
```

### Exception Flow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         Exception Propagation                            │
│                                                                          │
│   main()        │        │         │                                     │
│     │           │        │         │                                     │
│     ▼           │        │         │                                     │
│   method1()     │        │         │                                     │
│     │           │        │         │                                     │
│     ▼           │        │         │                                     │
│   method2()     │        │         │                                     │
│     │           │        │         │                                     │
│     ▼           │  catch │         │                                     │
│   method3() ──throw──────┼─────────┼─▶ If not caught, bubbles up         │
│                          │         │   until caught or terminates JVM    │
│                          │         │                                     │
│                   method2 catches  │                                     │
│                   and handles      │                                     │
└─────────────────────────────────────────────────────────────────────────┘
```

### Try-Catch-Finally Flow

```
try {
    // 1. Execute try block
    // If exception thrown → jump to matching catch
    // If no exception → skip catch blocks
}
catch (SpecificException e) {
    // 2. Handle specific exception
}
catch (GeneralException e) {
    // 3. Handle more general exception
    // Order matters: specific before general
}
finally {
    // 4. ALWAYS executed (unless System.exit() or JVM crash)
    // Cleanup code goes here
    // Runs after try or catch completes
}
// 5. Continue execution (if no uncaught exception)
```

### Try-With-Resources

```
┌─────────────────────────────────────────────────────────────────────────┐
│  try (Resource r1 = new Resource1();  // Created first                   │
│       Resource r2 = new Resource2()) { // Created second                 │
│      // Use resources                                                    │
│      // If exception thrown here → "primary exception"                   │
│  }                                                                       │
│  // Resources closed in REVERSE order:                                   │
│  // r2.close() first, then r1.close()                                    │
│  // If close() throws → "suppressed exception"                           │
│                                                                          │
│  Primary exception wins, suppressed attached via getSuppressed()         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Logging Levels

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         Logging Level Pyramid                           │
│                                                                          │
│                            ▲                                             │
│                           /│\  ERROR   ← Always see (critical failures) │
│                          / │ \                                           │
│                         /  │  \  WARN  ← Usually see (potential issues) │
│                        /   │   \                                         │
│                       /    │    \ INFO  ← Default level (key events)    │
│                      /     │     \                                       │
│                     /      │      \ DEBUG ← Development (detailed flow) │
│                    /       │       \                                     │
│                   /        │        \ TRACE ← Verbose (method entry/exit)│
│                  ─────────────────────                                   │
│                                                                          │
│  When level = INFO, you see: ERROR, WARN, INFO                          │
│  When level = DEBUG, you see: ERROR, WARN, INFO, DEBUG                  │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Code Examples

### 1) Exception Hierarchy and Basics

```java
// Checked exception - MUST be handled or declared
public void readFile(String path) throws IOException {
    BufferedReader reader = new BufferedReader(new FileReader(path));
    String line = reader.readLine();
    reader.close();
}

// Unchecked exception - no forced handling
public void validateAge(int age) {
    if (age < 0) {
        throw new IllegalArgumentException("Age cannot be negative: " + age);
    }
}

// Error - typically should NOT catch
public void riskyOperation() {
    // This might cause OutOfMemoryError
    List<byte[]> list = new ArrayList<>();
    while (true) {
        list.add(new byte[1024 * 1024]);  // 1MB each
    }
}
```

### 2) Try-Catch-Finally Patterns

```java
// Basic try-catch
public void basicExample() {
    try {
        int result = 10 / 0;  // ArithmeticException
    } catch (ArithmeticException e) {
        System.err.println("Division by zero: " + e.getMessage());
    }
}

// Multiple catch blocks (specific to general)
public void multiCatch() {
    try {
        FileReader file = new FileReader("test.txt");
        int data = file.read();
    } catch (FileNotFoundException e) {
        System.err.println("File not found: " + e.getMessage());
    } catch (IOException e) {
        System.err.println("IO error: " + e.getMessage());
    } catch (Exception e) {
        System.err.println("Unexpected error: " + e.getMessage());
    }
}

// Multi-catch (Java 7+) - handle multiple exceptions the same way
public void multiCatchSingleBlock() {
    try {
        // risky code
    } catch (IllegalArgumentException | IllegalStateException e) {
        // Handle both the same way
        // Note: 'e' is implicitly final in multi-catch
        System.err.println("State or argument error: " + e.getMessage());
    }
}

// Try-finally without catch
public void tryFinallyOnly() {
    Lock lock = new ReentrantLock();
    lock.lock();
    try {
        // Critical section
        // Exception propagates up after finally
    } finally {
        lock.unlock();  // Always release lock
    }
}

// Finally block quirks
public int finallyReturnExample() {
    try {
        return 1;
    } finally {
        // This runs before method returns
        System.out.println("Finally executed");
        // return 2;  // ❌ Don't return from finally - overwrites try's return!
    }
    // Returns 1 (from try block)
}

// Finally with exception
public void finallyWithException() {
    try {
        throw new RuntimeException("From try");
    } finally {
        System.out.println("Finally runs");
        // throw new RuntimeException("From finally");  // ❌ Would mask try's exception!
    }
}
```

### 3) Try-With-Resources (Preferred for Closeable Resources)

```java
// Basic try-with-resources
public String readFirstLine(String path) throws IOException {
    try (BufferedReader reader = new BufferedReader(new FileReader(path))) {
        return reader.readLine();
    }  // reader.close() called automatically
}

// Multiple resources (closed in reverse order)
public void copyFile(String source, String dest) throws IOException {
    try (InputStream in = new FileInputStream(source);
         OutputStream out = new FileOutputStream(dest)) {
        byte[] buffer = new byte[1024];
        int length;
        while ((length = in.read(buffer)) > 0) {
            out.write(buffer, 0, length);
        }
    }  // out.close() then in.close()
}

// With existing resource (Java 9+)
public void processResource(BufferedReader reader) throws IOException {
    try (reader) {  // Reference to existing effectively final variable
        String line;
        while ((line = reader.readLine()) != null) {
            process(line);
        }
    }  // reader closed here
}

// Custom AutoCloseable
public class DatabaseConnection implements AutoCloseable {
    private Connection conn;
    
    public DatabaseConnection(String url) throws SQLException {
        this.conn = DriverManager.getConnection(url);
    }
    
    public void executeQuery(String sql) throws SQLException {
        // Execute query
    }
    
    @Override
    public void close() throws SQLException {
        if (conn != null && !conn.isClosed()) {
            conn.close();
            System.out.println("Connection closed");
        }
    }
}

// Usage of custom AutoCloseable
public void useDatabase() {
    try (DatabaseConnection db = new DatabaseConnection("jdbc:mysql://localhost/test")) {
        db.executeQuery("SELECT * FROM users");
    } catch (SQLException e) {
        // Handle both query and close exceptions
        System.err.println("Database error: " + e.getMessage());
        
        // Check for suppressed exceptions
        for (Throwable suppressed : e.getSuppressed()) {
            System.err.println("Suppressed: " + suppressed.getMessage());
        }
    }
}
```

### 4) Suppressed Exceptions

```java
public class SuppressedExceptionDemo implements AutoCloseable {
    private final String name;
    
    public SuppressedExceptionDemo(String name) {
        this.name = name;
    }
    
    public void doWork() {
        throw new RuntimeException("Exception in doWork from " + name);
    }
    
    @Override
    public void close() {
        throw new RuntimeException("Exception in close from " + name);
    }
    
    public static void main(String[] args) {
        try (SuppressedExceptionDemo r1 = new SuppressedExceptionDemo("R1");
             SuppressedExceptionDemo r2 = new SuppressedExceptionDemo("R2")) {
            r2.doWork();  // Primary exception
        } catch (Exception e) {
            System.out.println("Primary: " + e.getMessage());
            // Primary: Exception in doWork from R2
            
            for (Throwable suppressed : e.getSuppressed()) {
                System.out.println("Suppressed: " + suppressed.getMessage());
            }
            // Suppressed: Exception in close from R2
            // Suppressed: Exception in close from R1
        }
    }
}

// Manually adding suppressed exceptions
public void manualSuppression() {
    Exception primary = new Exception("Primary");
    Exception suppressed1 = new Exception("Suppressed 1");
    Exception suppressed2 = new Exception("Suppressed 2");
    
    primary.addSuppressed(suppressed1);
    primary.addSuppressed(suppressed2);
    
    throw primary;
}
```

### 5) Exception Chaining (Preserving Cause)

```java
// ❌ BAD: Losing the original cause
public void badWrapping() {
    try {
        // Some operation
    } catch (SQLException e) {
        throw new RuntimeException("Database error");  // Original cause lost!
    }
}

// ✅ GOOD: Preserving the original cause
public void goodWrapping() {
    try {
        // Some operation
    } catch (SQLException e) {
        throw new RuntimeException("Database error", e);  // Cause preserved
    }
}

// Getting the cause chain
public void printCauseChain(Throwable t) {
    Throwable current = t;
    while (current != null) {
        System.out.println("Exception: " + current.getClass().getName());
        System.out.println("Message: " + current.getMessage());
        current = current.getCause();
    }
}

// Custom exception with cause
public class ServiceException extends Exception {
    public ServiceException(String message) {
        super(message);
    }
    
    public ServiceException(String message, Throwable cause) {
        super(message, cause);
    }
    
    public ServiceException(Throwable cause) {
        super(cause);
    }
}

// Usage: wrapping and re-throwing with context
public User findUser(long id) throws ServiceException {
    try {
        return userRepository.findById(id);
    } catch (SQLException e) {
        throw new ServiceException("Failed to find user with ID: " + id, e);
    }
}
```

### 6) Custom Exceptions

```java
// Simple custom exception (checked)
public class OrderNotFoundException extends Exception {
    private final long orderId;
    
    public OrderNotFoundException(long orderId) {
        super("Order not found: " + orderId);
        this.orderId = orderId;
    }
    
    public OrderNotFoundException(long orderId, Throwable cause) {
        super("Order not found: " + orderId, cause);
        this.orderId = orderId;
    }
    
    public long getOrderId() {
        return orderId;
    }
}

// Custom runtime exception (unchecked)
public class InvalidOrderStateException extends RuntimeException {
    private final String currentState;
    private final String requiredState;
    
    public InvalidOrderStateException(String currentState, String requiredState) {
        super(String.format(
            "Invalid order state: expected '%s', but was '%s'",
            requiredState, currentState
        ));
        this.currentState = currentState;
        this.requiredState = requiredState;
    }
    
    public String getCurrentState() { return currentState; }
    public String getRequiredState() { return requiredState; }
}

// Exception hierarchy for a domain
public abstract class DomainException extends RuntimeException {
    protected DomainException(String message) {
        super(message);
    }
    
    protected DomainException(String message, Throwable cause) {
        super(message, cause);
    }
}

public class EntityNotFoundException extends DomainException {
    private final String entityType;
    private final Object entityId;
    
    public EntityNotFoundException(String entityType, Object entityId) {
        super(String.format("%s not found with ID: %s", entityType, entityId));
        this.entityType = entityType;
        this.entityId = entityId;
    }
    
    public String getEntityType() { return entityType; }
    public Object getEntityId() { return entityId; }
}

public class DuplicateEntityException extends DomainException {
    private final String entityType;
    private final String field;
    private final Object value;
    
    public DuplicateEntityException(String entityType, String field, Object value) {
        super(String.format("%s with %s '%s' already exists", entityType, field, value));
        this.entityType = entityType;
        this.field = field;
        this.value = value;
    }
}

// Usage
public class OrderService {
    public Order getOrder(long id) {
        return orderRepository.findById(id)
            .orElseThrow(() -> new EntityNotFoundException("Order", id));
    }
    
    public void cancelOrder(long id) {
        Order order = getOrder(id);
        if (!order.getStatus().equals("PENDING")) {
            throw new InvalidOrderStateException(order.getStatus(), "PENDING");
        }
        order.cancel();
    }
}
```

### 7) Best Practices

```java
// ✅ Be specific about exceptions
public void specificExceptions() throws FileNotFoundException, ParseException {
    // Declare specific exceptions, not generic Exception
}

// ❌ Don't catch generic Exception/Throwable (except at boundaries)
try {
    // risky code
} catch (Exception e) {  // Too broad!
    // May catch unexpected exceptions
}

// ✅ Catch specific exceptions
try {
    // risky code
} catch (FileNotFoundException e) {
    // Handle file not found
} catch (IOException e) {
    // Handle other IO issues
}

// ❌ Don't swallow exceptions silently
try {
    // risky code
} catch (Exception e) {
    // Empty catch block - exception lost!
}

// ✅ At minimum, log the exception
try {
    // risky code
} catch (Exception e) {
    log.error("Operation failed", e);
    throw e;  // Or handle appropriately
}

// ❌ Don't use exceptions for control flow
public boolean containsElement(List<?> list, Object element) {
    try {
        list.indexOf(element);  // Don't throw if not found
        return true;
    } catch (Exception e) {
        return false;
    }
}

// ✅ Use conditional logic instead
public boolean containsElementCorrect(List<?> list, Object element) {
    return list.contains(element);
}

// ✅ Fail fast - validate inputs early
public void processOrder(Order order) {
    Objects.requireNonNull(order, "Order cannot be null");
    if (order.getItems().isEmpty()) {
        throw new IllegalArgumentException("Order must have at least one item");
    }
    // Continue processing...
}

// ✅ Use specific exception types
public User findUser(String email) {
    if (email == null || email.isBlank()) {
        throw new IllegalArgumentException("Email cannot be null or blank");
    }
    return userRepository.findByEmail(email)
        .orElseThrow(() -> new EntityNotFoundException("User", email));
}

// ✅ Document exceptions in Javadoc
/**
 * Retrieves an order by its ID.
 *
 * @param id the order ID
 * @return the order
 * @throws EntityNotFoundException if no order exists with the given ID
 * @throws IllegalArgumentException if id is negative
 */
public Order getOrder(long id) {
    if (id < 0) {
        throw new IllegalArgumentException("ID must be non-negative");
    }
    return orderRepository.findById(id)
        .orElseThrow(() -> new EntityNotFoundException("Order", id));
}

// ✅ Clean up resources in finally or use try-with-resources
Connection conn = null;
try {
    conn = dataSource.getConnection();
    // Use connection
} finally {
    if (conn != null) {
        try {
            conn.close();
        } catch (SQLException e) {
            log.warn("Failed to close connection", e);
        }
    }
}

// Better: use try-with-resources
try (Connection conn = dataSource.getConnection()) {
    // Use connection
}
```

### 8) Logging Fundamentals

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class OrderService {
    
    // One logger per class (private static final)
    private static final Logger log = LoggerFactory.getLogger(OrderService.class);
    
    public Order createOrder(OrderRequest request) {
        // DEBUG: Detailed information for debugging
        log.debug("Creating order for customer: {}", request.getCustomerId());
        
        try {
            // INFO: Key business events
            Order order = processOrder(request);
            log.info("Order created successfully: orderId={}, customerId={}, total={}",
                order.getId(), order.getCustomerId(), order.getTotal());
            
            return order;
            
        } catch (InsufficientInventoryException e) {
            // WARN: Potential issues that should be monitored
            log.warn("Insufficient inventory for order: customerId={}, item={}",
                request.getCustomerId(), e.getItemId());
            throw e;
            
        } catch (Exception e) {
            // ERROR: Failures that need attention
            log.error("Failed to create order for customer: {}",
                request.getCustomerId(), e);
            throw new OrderProcessingException("Order creation failed", e);
        }
    }
    
    public void debuggingExample(String userId, List<Item> items) {
        // TRACE: Very detailed debugging (method entry/exit)
        log.trace("Entering calculateTotal: userId={}, itemCount={}", userId, items.size());
        
        // Use parameterized logging (efficient - string not built if level disabled)
        log.debug("Processing items for user: {}", userId);
        
        // ❌ Don't concatenate strings (always evaluated)
        // log.debug("Processing items for user: " + userId);
        
        // ✅ Use placeholders
        log.debug("Processing {} items for user {}", items.size(), userId);
        
        // Conditional logging for expensive operations
        if (log.isDebugEnabled()) {
            String itemDetails = items.stream()
                .map(Item::toString)
                .collect(Collectors.joining(", "));
            log.debug("Item details: {}", itemDetails);
        }
        
        log.trace("Exiting calculateTotal");
    }
}
```

### 9) Logging Best Practices

```java
public class LoggingBestPractices {
    private static final Logger log = LoggerFactory.getLogger(LoggingBestPractices.class);
    
    // ✅ Include relevant context
    public void processPayment(String orderId, String customerId, BigDecimal amount) {
        log.info("Processing payment: orderId={}, customerId={}, amount={}",
            orderId, customerId, amount);
    }
    
    // ❌ Don't log sensitive data
    public void badLogging(String creditCardNumber, String password) {
        // Never log passwords, credit cards, SSN, tokens, etc.
        log.info("User credentials: card={}, password={}", creditCardNumber, password);
    }
    
    // ✅ Mask or exclude sensitive data
    public void goodLogging(String creditCardNumber) {
        String masked = "**** **** **** " + creditCardNumber.substring(creditCardNumber.length() - 4);
        log.info("Processing payment with card: {}", masked);
    }
    
    // ✅ Log exceptions properly
    public void exceptionLogging() {
        try {
            // risky operation
        } catch (Exception e) {
            // ❌ Don't use toString() or getMessage() - loses stack trace
            // log.error("Error: " + e.getMessage());
            // log.error("Error: {}", e.toString());
            
            // ✅ Pass exception as last argument
            log.error("Operation failed", e);
            log.error("Operation failed for userId={}", userId, e);
        }
    }
    
    // ✅ Use appropriate log levels
    public void logLevels() {
        // TRACE: Finest granularity, method entry/exit, variable values
        log.trace("Entering method X with param={}", param);
        
        // DEBUG: Detailed technical information for debugging
        log.debug("Query returned {} results", count);
        
        // INFO: Key business events, normal operations
        log.info("User {} logged in successfully", userId);
        
        // WARN: Potential problems, degraded service, recoverable issues
        log.warn("Rate limit approaching for user {}", userId);
        log.warn("Retry attempt {} for operation", retryCount);
        
        // ERROR: Failures, exceptions, things that need attention
        log.error("Failed to process payment for order {}", orderId, exception);
    }
    
    // ✅ Structured logging for better parsing
    public void structuredLogging(Order order) {
        // Key-value pairs for log aggregation tools
        log.info("Order processed: orderId={} customerId={} status={} amount={} currency={}",
            order.getId(),
            order.getCustomerId(),
            order.getStatus(),
            order.getAmount(),
            order.getCurrency());
    }
    
    // ❌ Don't log and throw - causes duplicate logs
    public void dontLogAndThrow() {
        try {
            // operation
        } catch (Exception e) {
            log.error("Error occurred", e);  // Logged here
            throw e;  // Will be logged again when caught higher up
        }
    }
    
    // ✅ Either log or throw, not both
    public void logOrThrow() {
        try {
            // operation
        } catch (Exception e) {
            // Option 1: Log and handle
            log.error("Error occurred, using fallback", e);
            return getDefaultValue();
            
            // Option 2: Wrap and throw (logged at boundary)
            // throw new ServiceException("Operation failed", e);
        }
    }
}
```

### 10) Exception Handling in Different Contexts

```java
// API/Web Layer - translate to appropriate response
@RestController
public class OrderController {
    
    @GetMapping("/orders/{id}")
    public ResponseEntity<Order> getOrder(@PathVariable long id) {
        try {
            Order order = orderService.getOrder(id);
            return ResponseEntity.ok(order);
        } catch (EntityNotFoundException e) {
            return ResponseEntity.notFound().build();
        } catch (Exception e) {
            log.error("Unexpected error getting order {}", id, e);
            return ResponseEntity.internalServerError().build();
        }
    }
    
    // Using @ExceptionHandler
    @ExceptionHandler(EntityNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(EntityNotFoundException e) {
        return ResponseEntity
            .status(HttpStatus.NOT_FOUND)
            .body(new ErrorResponse("NOT_FOUND", e.getMessage()));
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneric(Exception e) {
        log.error("Unhandled exception", e);
        return ResponseEntity
            .status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(new ErrorResponse("INTERNAL_ERROR", "An unexpected error occurred"));
    }
}

// Service Layer - business logic exceptions
public class OrderService {
    
    public Order createOrder(CreateOrderRequest request) {
        // Validate
        validateRequest(request);
        
        // Check business rules
        if (!inventoryService.hasStock(request.getItems())) {
            throw new InsufficientInventoryException(request.getItems());
        }
        
        try {
            return orderRepository.save(createOrderEntity(request));
        } catch (DataAccessException e) {
            throw new OrderCreationException("Failed to save order", e);
        }
    }
}

// Repository Layer - wrap data access exceptions
public class JdbcOrderRepository implements OrderRepository {
    
    @Override
    public Order findById(long id) {
        try {
            return jdbcTemplate.queryForObject(
                "SELECT * FROM orders WHERE id = ?",
                orderRowMapper,
                id
            );
        } catch (EmptyResultDataAccessException e) {
            return null;  // Or throw EntityNotFoundException
        } catch (DataAccessException e) {
            throw new RepositoryException("Failed to query order: " + id, e);
        }
    }
}

// Thread/Async context - don't swallow exceptions
ExecutorService executor = Executors.newFixedThreadPool(4);

// ❌ Exception lost in Runnable
executor.submit(() -> {
    try {
        riskyOperation();
    } catch (Exception e) {
        // Exception lost! Thread dies silently
    }
});

// ✅ Use Callable and handle Future
Future<Result> future = executor.submit(() -> {
    return riskyOperation();  // Exception preserved in Future
});

try {
    Result result = future.get();
} catch (ExecutionException e) {
    Throwable cause = e.getCause();  // Get the actual exception
    log.error("Async operation failed", cause);
}

// ✅ Or use UncaughtExceptionHandler
Thread thread = new Thread(task);
thread.setUncaughtExceptionHandler((t, e) -> {
    log.error("Uncaught exception in thread {}", t.getName(), e);
});
thread.start();
```

### 11) InterruptedException Handling

```java
// InterruptedException requires special handling

// ❌ Don't ignore InterruptedException
public void badHandling() {
    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        // BAD: Swallowing interrupt - thread's interrupted status is cleared!
    }
}

// ✅ Option 1: Restore interrupted status and return
public void restoreAndReturn() {
    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();  // Restore interrupted status
        return;  // Exit the method
    }
}

// ✅ Option 2: Restore interrupted status and throw
public void restoreAndThrow() {
    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
        throw new RuntimeException("Operation interrupted", e);
    }
}

// ✅ Option 3: Propagate (if allowed by method signature)
public void propagate() throws InterruptedException {
    Thread.sleep(1000);  // Let caller handle it
}

// In a loop - check interrupted status
public void interruptibleLoop() {
    while (!Thread.currentThread().isInterrupted()) {
        try {
            // Do work
            someBlockingOperation();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            break;  // Exit the loop
        }
    }
    // Cleanup after loop
}
```

---

## Common Anti-Patterns

### 1. Swallowing Exceptions

```java
// ❌ NEVER do this
try {
    dangerousOperation();
} catch (Exception e) {
    // Empty catch block - exception completely lost
}

// ✅ At minimum, log it
try {
    dangerousOperation();
} catch (Exception e) {
    log.error("Operation failed", e);
    // Either throw or handle appropriately
}
```

### 2. Catching Throwable

```java
// ❌ Don't catch Throwable/Error
try {
    operation();
} catch (Throwable t) {  // Catches Error too!
    // May catch OutOfMemoryError, StackOverflowError
    // The JVM may be in an unrecoverable state
}

// ✅ Catch Exception only
try {
    operation();
} catch (Exception e) {
    // Handle recoverable exceptions
}

// Exception: At top-level boundaries (e.g., thread handlers)
// where you want to log everything before dying
```

### 3. Using Exceptions for Control Flow

```java
// ❌ Expensive and unclear
public boolean isNumeric(String str) {
    try {
        Integer.parseInt(str);
        return true;
    } catch (NumberFormatException e) {
        return false;
    }
}

// ✅ Use appropriate APIs
public boolean isNumericCorrect(String str) {
    if (str == null || str.isEmpty()) return false;
    for (char c : str.toCharArray()) {
        if (!Character.isDigit(c)) return false;
    }
    return true;
}
```

### 4. Throwing Generic Exceptions

```java
// ❌ Too vague
public void process() throws Exception {
    throw new Exception("Something went wrong");
}

// ✅ Throw specific exceptions
public void processCorrect() throws OrderNotFoundException, PaymentException {
    throw new OrderNotFoundException(orderId);
}
```

### 5. Losing Exception Information

```java
// ❌ Losing stack trace
try {
    operation();
} catch (SQLException e) {
    throw new RuntimeException(e.getMessage());  // Stack trace lost!
}

// ✅ Preserve the cause
try {
    operation();
} catch (SQLException e) {
    throw new RuntimeException("Database operation failed", e);  // Cause preserved
}
```

---

## Interview Questions

### Basic Questions

1. **What is the difference between checked and unchecked exceptions?**
   - Checked: Must be declared or caught, checked at compile time, extends Exception
   - Unchecked: Not required to handle, extends RuntimeException
   - Checked for recoverable conditions, unchecked for programming errors

2. **What is the difference between Error and Exception?**
   - Error: Serious JVM-level problems (OutOfMemoryError), usually unrecoverable
   - Exception: Application-level problems, can be caught and handled
   - Don't catch Error unless at top-level boundaries

3. **What is try-with-resources?**
   - Automatic resource management for AutoCloseable resources
   - Resources closed automatically in reverse order
   - Cleaner than try-finally, handles suppressed exceptions

4. **What are suppressed exceptions?**
   - Exceptions thrown during close() in try-with-resources
   - Attached to primary exception via addSuppressed()
   - Retrieved via getSuppressed()

5. **Why should you always log the full exception, not just the message?**
   - Stack trace shows where error occurred
   - getMessage() loses context and chain
   - Use: `log.error("message", exception)`

### Intermediate Questions

6. **When should you use checked vs unchecked exceptions?**
   - Checked: Recoverable conditions, caller can reasonably handle
   - Unchecked: Programming errors, precondition violations
   - Modern trend: prefer unchecked (less boilerplate)

7. **What is exception chaining?**
   - Wrapping one exception in another
   - Preserves original cause while adding context
   - `new CustomException("message", cause)`

8. **How should you handle InterruptedException?**
   - Never swallow it silently
   - Either: restore interrupted status + return/throw
   - Or: propagate if method signature allows

9. **What is the problem with returning from finally block?**
   - Overwrites any return from try or catch
   - Can mask exceptions thrown in try
   - Never return from finally

10. **Why is catching Exception or Throwable bad?**
    - Too broad, may catch unexpected exceptions
    - Catching Throwable includes Error
    - Makes debugging harder, may mask bugs

### Advanced Questions

11. **How do you design a custom exception hierarchy?**
    - Create base domain exception
    - Subclass for specific cases
    - Include relevant context (IDs, states)
    - Provide all standard constructors

12. **Explain the log levels and when to use each.**
    - ERROR: Failures requiring attention
    - WARN: Potential problems, degraded service
    - INFO: Key business events
    - DEBUG: Technical debugging details
    - TRACE: Finest granularity

13. **What is the "log and throw" anti-pattern?**
    - Logging and rethrowing same exception
    - Causes duplicate log entries
    - Either log and handle, or wrap and throw

14. **How do you handle exceptions in async/concurrent code?**
    - Use Callable instead of Runnable
    - Handle ExecutionException from Future.get()
    - Set UncaughtExceptionHandler for threads
    - Don't let exceptions disappear silently

15. **What is the performance impact of exceptions?**
    - Creating exception is expensive (fills stack trace)
    - Don't use for control flow
    - Consider reusing exceptions for performance-critical code (advanced)

---

## Quick Reference

### Exception Handling Patterns

| Pattern | When to Use |
|---------|-------------|
| Try-catch | Handle specific exceptions |
| Try-with-resources | Managing closeable resources |
| Multi-catch | Same handling for multiple types |
| Exception chaining | Wrap lower-level exceptions |
| Custom exceptions | Domain-specific error info |

### Log Level Guidelines

| Level | When to Use | Example |
|-------|-------------|---------|
| ERROR | Failures, exceptions | Payment failed |
| WARN | Potential problems | Rate limit approaching |
| INFO | Key business events | User logged in |
| DEBUG | Technical details | Query returned 5 rows |
| TRACE | Very fine detail | Entering method X |

### Standard Exception Constructors

```java
public class CustomException extends RuntimeException {
    public CustomException() { super(); }
    public CustomException(String message) { super(message); }
    public CustomException(String message, Throwable cause) { super(message, cause); }
    public CustomException(Throwable cause) { super(cause); }
}
```
