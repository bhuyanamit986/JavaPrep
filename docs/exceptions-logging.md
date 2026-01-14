# Exceptions, Error Handling & Logging

Exception handling is a fundamental aspect of writing robust Java applications. This guide provides comprehensive coverage of Java's exception handling mechanism, best practices, and logging strategies.

---

## 1) What are Exceptions?

### Definition

An **exception** is an event that disrupts the normal flow of a program's execution. When an exceptional condition occurs, an exception object is created and "thrown." The runtime system then searches for code that can "catch" and handle this exception.

### Why Exceptions Matter

- **Separate error handling from regular code** - Keep business logic clean
- **Propagate errors up the call stack** - Let the appropriate layer handle the error
- **Group and differentiate error types** - Different exceptions for different problems
- **Provide context about what went wrong** - Stack traces and error messages

### How Exceptions Work

```
Normal Flow:
method1() → method2() → method3() → result returned

Exception Flow:
method1() → method2() → method3() 
                              ↓ (exception thrown)
                         search for handler
                              ↓
         method2() (no handler) ← propagate up
                              ↓
         method1() (handler found) ← catch and handle
```

---

## 2) Throwable Hierarchy

### Illustration

```
                    ┌──────────────┐
                    │  Throwable   │
                    │  (checked)   │
                    └──────┬───────┘
                           │
           ┌───────────────┴───────────────┐
           │                               │
    ┌──────┴──────┐               ┌────────┴────────┐
    │    Error    │               │    Exception    │
    │ (unchecked) │               │    (checked)    │
    └──────┬──────┘               └────────┬────────┘
           │                               │
    ┌──────┴──────┐         ┌──────────────┴──────────────┐
    │ System      │         │                             │
    │ Failures    │  ┌──────┴──────┐          ┌───────────┴───────────┐
    └─────────────┘  │   Checked   │          │   RuntimeException    │
                     │ Exceptions  │          │     (unchecked)       │
                     └─────────────┘          └───────────────────────┘
```

### Detailed Breakdown

| Category | Type | Examples | Must Handle? |
|----------|------|----------|--------------|
| **Error** | Unchecked | `OutOfMemoryError`, `StackOverflowError`, `VirtualMachineError` | No (shouldn't catch) |
| **Checked Exception** | Checked | `IOException`, `SQLException`, `ClassNotFoundException` | Yes (compile-time) |
| **RuntimeException** | Unchecked | `NullPointerException`, `IllegalArgumentException`, `IndexOutOfBoundsException` | No (but can catch) |

### Common Errors (Don't Catch These!)

```java
// OutOfMemoryError - JVM ran out of heap space
public void createHugeArray() {
    int[] massive = new int[Integer.MAX_VALUE]; // OutOfMemoryError
}

// StackOverflowError - Infinite recursion
public void infiniteRecursion() {
    infiniteRecursion(); // StackOverflowError
}

// NoClassDefFoundError - Class missing at runtime
// Usually caused by classpath issues
```

### Common Checked Exceptions

```java
// IOException - I/O operation failed
public void readFile() throws IOException {
    FileInputStream fis = new FileInputStream("file.txt");
}

// SQLException - Database operation failed
public void queryDatabase() throws SQLException {
    Connection conn = DriverManager.getConnection(url);
}

// ClassNotFoundException - Class not found during reflection
public void loadClass() throws ClassNotFoundException {
    Class<?> clazz = Class.forName("com.example.MyClass");
}
```

### Common Runtime Exceptions

```java
// NullPointerException - Operating on null reference
String s = null;
s.length(); // NullPointerException

// IllegalArgumentException - Invalid method argument
public void setAge(int age) {
    if (age < 0) {
        throw new IllegalArgumentException("Age cannot be negative: " + age);
    }
}

// IndexOutOfBoundsException - Invalid index access
int[] arr = {1, 2, 3};
arr[5]; // ArrayIndexOutOfBoundsException

// IllegalStateException - Method called at inappropriate time
public void start() {
    if (isRunning) {
        throw new IllegalStateException("Already started");
    }
}

// NumberFormatException - Failed to parse number
Integer.parseInt("abc"); // NumberFormatException
```

---

## 3) Checked vs Unchecked Exceptions

### Definition

| Type | Description | Compiler Enforces? | When to Use |
|------|-------------|-------------------|-------------|
| **Checked** | Exceptional conditions that are predictable and recoverable | Yes | I/O operations, network calls, external resources |
| **Unchecked** | Programming errors or conditions that cannot be reasonably recovered | No | Programming bugs, invalid arguments, illegal state |

### Illustration: Compile-Time Behavior

```
┌─────────────────────────────────────────────────────────────────┐
│                        CHECKED EXCEPTION                         │
│                                                                  │
│   public void readFile() {                                       │
│       FileInputStream fis = new FileInputStream("file.txt");     │
│   }                                                              │
│                          ↓                                       │
│   ❌ COMPILE ERROR: Unhandled exception: IOException             │
│                                                                  │
│   Solutions:                                                     │
│   1. throws IOException  (propagate)                             │
│   2. try-catch block     (handle)                                │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                       UNCHECKED EXCEPTION                        │
│                                                                  │
│   public void process(String s) {                                │
│       System.out.println(s.length());  // May throw NPE          │
│   }                                                              │
│                          ↓                                       │
│   ✅ COMPILES: No handling required                              │
│   ⚠️ Runtime risk if s is null                                   │
└─────────────────────────────────────────────────────────────────┘
```

### Trade-offs

**Checked Exceptions:**
```java
// PROS:
// - Forces explicit handling at compile time
// - Makes failure modes visible in API

// CONS:
// - Can lead to verbose code
// - Often wrapped into runtime exceptions in layered architectures

public void processFile(String path) throws IOException {
    // Caller MUST handle IOException
}
```

**Unchecked Exceptions:**
```java
// PROS:
// - Less boilerplate code
// - Propagate naturally up the stack

// CONS:
// - Failure paths are implicit
// - Must rely on documentation and tests

public void processData(String data) {
    if (data == null) {
        throw new IllegalArgumentException("data cannot be null");
    }
    // Caller not forced to handle, but should check documentation
}
```

### Guidelines for Choosing

```
Use CHECKED exceptions when:
├── The caller can reasonably recover from the condition
├── External resources are involved (files, network, database)
└── The exceptional condition is predictable

Use UNCHECKED exceptions when:
├── It's a programming error (bug)
├── Recovery is not possible or meaningful
├── It would pollute every method with throws declarations
└── The caller cannot do anything useful with the exception
```

---

## 4) Exception Handling Syntax

### Basic try-catch Structure

```java
try {
    // Code that might throw an exception
    riskyOperation();
} catch (SpecificException e) {
    // Handle specific exception
    handleError(e);
} catch (GeneralException e) {
    // Handle more general exception
    handleGenericError(e);
} finally {
    // Always executed (cleanup code)
    cleanup();
}
```

### Execution Flow Diagram

```
                    ┌─────────────┐
                    │ Enter try   │
                    └──────┬──────┘
                           │
               ┌───────────┴───────────┐
               │    Exception thrown?   │
               └───────────┬───────────┘
                    │             │
                   YES            NO
                    │             │
                    ▼             ▼
          ┌─────────────┐  ┌─────────────┐
          │ Match catch │  │ Skip catch  │
          │   blocks    │  │   blocks    │
          └──────┬──────┘  └──────┬──────┘
                 │                │
                 ▼                ▼
          ┌─────────────┐  ┌─────────────┐
          │Execute catch│  │             │
          │    block    │  │             │
          └──────┬──────┘  │             │
                 │         │             │
                 └────┬────┘             │
                      │                  │
                      ▼                  │
               ┌─────────────┐           │
               │  Execute    │←──────────┘
               │   finally   │
               └──────┬──────┘
                      │
                      ▼
               ┌─────────────┐
               │  Continue   │
               │  or throw   │
               └─────────────┘
```

### Multi-catch (Java 7+)

```java
// Before Java 7: Repetitive code
try {
    processFile();
} catch (IOException e) {
    logger.error("File error", e);
    throw new ProcessingException(e);
} catch (SQLException e) {
    logger.error("Database error", e);
    throw new ProcessingException(e);
}

// Java 7+: Multi-catch with pipe operator
try {
    processFile();
} catch (IOException | SQLException e) {
    logger.error("Processing error", e);
    throw new ProcessingException(e);
}

// Note: The caught exception 'e' is implicitly final in multi-catch
```

### Rethrowing Exceptions

```java
public void process() throws ProcessingException {
    try {
        riskyOperation();
    } catch (IOException e) {
        // Log and rethrow wrapped exception
        logger.error("I/O failed", e);
        throw new ProcessingException("Processing failed", e);
    }
}

// More precise rethrow (Java 7+)
public void preciseRethrow() throws IOException, SQLException {
    try {
        if (condition) throw new IOException();
        else throw new SQLException();
    } catch (Exception e) {
        // Compiler knows only IOException or SQLException possible
        throw e;  // More precise than 'throws Exception'
    }
}
```

---

## 5) try-with-resources (Must Know!)

### Definition

**try-with-resources** is a try statement that declares one or more resources. A resource is an object that must be closed after the program is finished with it. The statement ensures that each resource is closed at the end of the statement.

### Why It's Important

```
Traditional approach (error-prone):
┌────────────────────────────────────────────────────────────────┐
│  Resource leak if exception occurs before close()              │
│  Resource leak if close() throws exception                     │
│  Verbose with multiple resources                               │
│  Easy to forget to close                                       │
└────────────────────────────────────────────────────────────────┘

try-with-resources approach:
┌────────────────────────────────────────────────────────────────┐
│  ✓ Guaranteed resource closure                                 │
│  ✓ Handles exceptions in close() properly                      │
│  ✓ Concise and readable                                        │
│  ✓ Suppressed exceptions preserved                             │
└────────────────────────────────────────────────────────────────┘
```

### Code Examples

```java
// ❌ BAD: Traditional approach (error-prone)
public String readFirstLine(String path) throws IOException {
    BufferedReader br = null;
    try {
        br = new BufferedReader(new FileReader(path));
        return br.readLine();
    } finally {
        if (br != null) {
            br.close();  // What if this throws?
        }
    }
}

// ✅ GOOD: try-with-resources
public String readFirstLine(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    }  // br.close() called automatically, even if exception occurs
}

// Multiple resources (closed in reverse order)
public void copyFile(String src, String dest) throws IOException {
    try (InputStream in = new FileInputStream(src);
         OutputStream out = new FileOutputStream(dest)) {
        byte[] buffer = new byte[1024];
        int bytesRead;
        while ((bytesRead = in.read(buffer)) != -1) {
            out.write(buffer, 0, bytesRead);
        }
    }  // out closed first, then in
}

// Java 9+: Effectively final resources
public void readFile(BufferedReader reader) throws IOException {
    // reader is effectively final (not reassigned)
    try (reader) {  // Can use existing variable
        System.out.println(reader.readLine());
    }
}
```

### AutoCloseable Interface

```java
// Any class implementing AutoCloseable can be used with try-with-resources
public class DatabaseConnection implements AutoCloseable {
    
    public DatabaseConnection() {
        System.out.println("Connection opened");
    }
    
    public void query(String sql) {
        System.out.println("Executing: " + sql);
    }
    
    @Override
    public void close() {
        System.out.println("Connection closed");
    }
}

// Usage
try (DatabaseConnection conn = new DatabaseConnection()) {
    conn.query("SELECT * FROM users");
}  // close() called automatically
```

### Suppressed Exceptions

When both the try block AND close() throw exceptions, the close() exception is **suppressed** (not lost):

```java
public class SuppressedExample implements AutoCloseable {
    @Override
    public void close() {
        throw new RuntimeException("Close failed!");
    }
}

try (SuppressedExample resource = new SuppressedExample()) {
    throw new RuntimeException("Try block failed!");
} catch (RuntimeException e) {
    System.out.println("Primary: " + e.getMessage());
    // Primary: Try block failed!
    
    for (Throwable suppressed : e.getSuppressed()) {
        System.out.println("Suppressed: " + suppressed.getMessage());
        // Suppressed: Close failed!
    }
}
```

---

## 6) Custom Exceptions

### When to Create Custom Exceptions

```
Create custom exception when:
├── Built-in exceptions don't convey enough meaning
├── You need to carry additional context (IDs, codes, etc.)
├── You want to group related errors under one type
└── You want cleaner exception handling in catch blocks

Don't create custom exception when:
├── A standard exception (IllegalArgumentException, etc.) fits
├── The exception adds no additional information
└── You're creating an exception for every single error
```

### Custom Exception Design

```java
// Checked custom exception (extends Exception)
public class OrderProcessingException extends Exception {
    
    private final String orderId;
    private final ErrorCode errorCode;
    
    public enum ErrorCode {
        INVALID_ORDER,
        PAYMENT_FAILED,
        INVENTORY_UNAVAILABLE,
        SHIPPING_ERROR
    }
    
    public OrderProcessingException(String message, String orderId, ErrorCode errorCode) {
        super(message);
        this.orderId = orderId;
        this.errorCode = errorCode;
    }
    
    public OrderProcessingException(String message, String orderId, 
                                    ErrorCode errorCode, Throwable cause) {
        super(message, cause);
        this.orderId = orderId;
        this.errorCode = errorCode;
    }
    
    public String getOrderId() {
        return orderId;
    }
    
    public ErrorCode getErrorCode() {
        return errorCode;
    }
    
    @Override
    public String toString() {
        return String.format("OrderProcessingException[orderId=%s, code=%s]: %s",
            orderId, errorCode, getMessage());
    }
}
```

```java
// Unchecked custom exception (extends RuntimeException)
public class EntityNotFoundException extends RuntimeException {
    
    private final String entityType;
    private final Object entityId;
    
    public EntityNotFoundException(String entityType, Object entityId) {
        super(String.format("%s not found with id: %s", entityType, entityId));
        this.entityType = entityType;
        this.entityId = entityId;
    }
    
    public String getEntityType() {
        return entityType;
    }
    
    public Object getEntityId() {
        return entityId;
    }
}

// Usage
public User findUser(Long id) {
    return userRepository.findById(id)
        .orElseThrow(() -> new EntityNotFoundException("User", id));
}
```

### Exception Hierarchy for Applications

```java
// Base application exception
public abstract class ApplicationException extends RuntimeException {
    private final String errorCode;
    
    protected ApplicationException(String errorCode, String message) {
        super(message);
        this.errorCode = errorCode;
    }
    
    protected ApplicationException(String errorCode, String message, Throwable cause) {
        super(message, cause);
        this.errorCode = errorCode;
    }
    
    public String getErrorCode() {
        return errorCode;
    }
}

// Specific exceptions
public class ValidationException extends ApplicationException {
    private final Map<String, String> fieldErrors;
    
    public ValidationException(Map<String, String> fieldErrors) {
        super("VALIDATION_ERROR", "Validation failed");
        this.fieldErrors = fieldErrors;
    }
    
    public Map<String, String> getFieldErrors() {
        return fieldErrors;
    }
}

public class ResourceNotFoundException extends ApplicationException {
    public ResourceNotFoundException(String resource, Object id) {
        super("NOT_FOUND", String.format("%s with id %s not found", resource, id));
    }
}

public class BusinessRuleException extends ApplicationException {
    public BusinessRuleException(String rule, String message) {
        super("BUSINESS_RULE_" + rule, message);
    }
}
```

---

## 7) Exception Handling Best Practices

### Do's and Don'ts

```
┌──────────────────────────────────────────────────────────────────┐
│                          ✅ DO                                    │
├──────────────────────────────────────────────────────────────────┤
│ • Use try-with-resources for all closeable resources             │
│ • Catch specific exceptions, not generic Exception               │
│ • Include meaningful error messages with context                 │
│ • Preserve the original cause when wrapping exceptions           │
│ • Log exceptions at the appropriate level                        │
│ • Fail fast - throw exceptions as early as possible              │
│ • Document exceptions in Javadoc (@throws)                       │
│ • Clean up resources in finally or try-with-resources            │
└──────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────┐
│                          ❌ DON'T                                 │
├──────────────────────────────────────────────────────────────────┤
│ • Swallow exceptions (empty catch blocks)                        │
│ • Catch Throwable or Error (unless you rethrow)                  │
│ • Use exceptions for flow control                                │
│ • Log and rethrow the same exception (log once)                  │
│ • Lose the original stack trace when wrapping                    │
│ • Throw generic Exception                                        │
│ • Put business logic in finally blocks                           │
│ • Ignore InterruptedException without restoring interrupt status │
└──────────────────────────────────────────────────────────────────┘
```

### Code Examples

```java
// ❌ BAD: Swallowing exception
try {
    processFile();
} catch (IOException e) {
    // Silent failure - debugging nightmare!
}

// ❌ BAD: Losing stack trace
try {
    processFile();
} catch (IOException e) {
    throw new RuntimeException("Failed");  // Original cause lost!
}

// ✅ GOOD: Preserve cause
try {
    processFile();
} catch (IOException e) {
    throw new ProcessingException("Failed to process file", e);
}

// ❌ BAD: Catching too broad
try {
    processFile();
} catch (Exception e) {  // Catches NPE, array bounds, etc.
    handleFileError(e);
}

// ✅ GOOD: Catch specific exceptions
try {
    processFile();
} catch (FileNotFoundException e) {
    createNewFile();
} catch (IOException e) {
    throw new ProcessingException("I/O error", e);
}

// ❌ BAD: Using exceptions for control flow
public boolean contains(int[] arr, int target) {
    try {
        for (int i = 0; ; i++) {  // No bounds check!
            if (arr[i] == target) return true;
        }
    } catch (ArrayIndexOutOfBoundsException e) {
        return false;  // Slow and unclear!
    }
}

// ✅ GOOD: Normal control flow
public boolean contains(int[] arr, int target) {
    for (int num : arr) {
        if (num == target) return true;
    }
    return false;
}

// ❌ BAD: Log and throw (duplicate logs)
try {
    service.process();
} catch (ProcessingException e) {
    logger.error("Processing failed", e);  // Logged here
    throw e;  // And logged again in caller!
}

// ✅ GOOD: Log at boundary only
public void handleRequest(Request request) {
    try {
        processRequest(request);
    } catch (Exception e) {
        logger.error("Request failed: {}", request.getId(), e);
        throw new RequestException("Failed", e);
    }
}

// ✅ GOOD: Handle InterruptedException properly
try {
    Thread.sleep(1000);
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();  // Restore interrupt status
    throw new RuntimeException("Interrupted", e);
}
```

### Exception Documentation

```java
/**
 * Processes the order and returns the result.
 *
 * @param order the order to process
 * @return the processing result
 * @throws IllegalArgumentException if order is null
 * @throws OrderNotFoundException if order doesn't exist
 * @throws PaymentException if payment processing fails
 * @throws InventoryException if items are not available
 */
public ProcessingResult processOrder(Order order) 
        throws PaymentException, InventoryException {
    if (order == null) {
        throw new IllegalArgumentException("Order cannot be null");
    }
    // Process...
}
```

---

## 8) Exception Handling Patterns

### Pattern 1: Translate and Wrap

```java
// Layer-specific exception translation
public class UserService {
    
    private final UserRepository repository;
    
    public User findById(Long id) {
        try {
            return repository.findById(id)
                .orElseThrow(() -> new UserNotFoundException(id));
        } catch (DataAccessException e) {
            // Translate infrastructure exception to domain exception
            throw new UserServiceException("Failed to find user: " + id, e);
        }
    }
}
```

### Pattern 2: Exception Handler (Centralized Handling)

```java
// Spring-style centralized exception handling
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    private final Logger logger = LoggerFactory.getLogger(getClass());
    
    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleNotFound(ResourceNotFoundException e) {
        return new ErrorResponse(e.getErrorCode(), e.getMessage());
    }
    
    @ExceptionHandler(ValidationException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ValidationErrorResponse handleValidation(ValidationException e) {
        return new ValidationErrorResponse(
            e.getErrorCode(), 
            e.getMessage(),
            e.getFieldErrors()
        );
    }
    
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleGeneric(Exception e) {
        logger.error("Unexpected error", e);
        return new ErrorResponse("INTERNAL_ERROR", "An unexpected error occurred");
    }
}
```

### Pattern 3: Result Type (Alternative to Exceptions)

```java
// For expected failures, consider returning a Result type
public class Result<T> {
    private final T value;
    private final Exception error;
    
    private Result(T value, Exception error) {
        this.value = value;
        this.error = error;
    }
    
    public static <T> Result<T> success(T value) {
        return new Result<>(value, null);
    }
    
    public static <T> Result<T> failure(Exception error) {
        return new Result<>(null, error);
    }
    
    public boolean isSuccess() {
        return error == null;
    }
    
    public T getValue() {
        if (error != null) {
            throw new IllegalStateException("Result is failure");
        }
        return value;
    }
    
    public Exception getError() {
        return error;
    }
    
    public T getOrElse(T defaultValue) {
        return isSuccess() ? value : defaultValue;
    }
    
    public <R> Result<R> map(Function<T, R> mapper) {
        if (isSuccess()) {
            return Result.success(mapper.apply(value));
        }
        return Result.failure(error);
    }
}

// Usage
public Result<User> findUser(Long id) {
    try {
        User user = repository.findById(id);
        return user != null ? Result.success(user) : Result.failure(new UserNotFoundException(id));
    } catch (DataAccessException e) {
        return Result.failure(e);
    }
}
```

---

## 9) Logging Fundamentals

### Definition

**Logging** is the practice of recording application events to provide visibility into system behavior. Good logging enables debugging, monitoring, auditing, and troubleshooting.

### Log Levels

```
┌─────────┬─────────┬────────────────────────────────────────────────┐
│  Level  │ Priority│  When to Use                                   │
├─────────┼─────────┼────────────────────────────────────────────────┤
│  TRACE  │ Lowest  │ Very detailed diagnostic info (method entry)   │
│  DEBUG  │   Low   │ Detailed info for debugging                    │
│  INFO   │ Normal  │ General operational messages                   │
│  WARN   │  High   │ Potential issues, recoverable problems         │
│  ERROR  │ Highest │ Errors that need attention                     │
└─────────┴─────────┴────────────────────────────────────────────────┘

Production: Usually INFO and above
Development: Usually DEBUG and above
Troubleshooting: TRACE for specific classes
```

### Logging Framework Stack

```
┌─────────────────────────────────────────────────────────────────┐
│                     Your Application Code                        │
│                    logger.info("message")                        │
└─────────────────────────────────┬───────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────┐
│                        SLF4J (Facade)                            │
│               Abstracts the underlying implementation            │
└─────────────────────────────────┬───────────────────────────────┘
                                  │
            ┌─────────────────────┼─────────────────────┐
            ▼                     ▼                     ▼
   ┌─────────────┐       ┌─────────────┐       ┌─────────────┐
   │   Logback   │       │    Log4j2   │       │ JUL Bridge  │
   │  (Default)  │       │             │       │             │
   └─────────────┘       └─────────────┘       └─────────────┘
```

### Code Examples

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class UserService {
    
    // Logger per class (standard practice)
    private static final Logger logger = LoggerFactory.getLogger(UserService.class);
    
    public User createUser(CreateUserRequest request) {
        // INFO: Business-significant events
        logger.info("Creating user with email: {}", request.getEmail());
        
        // DEBUG: Detailed diagnostic information
        logger.debug("User creation request details: {}", request);
        
        try {
            User user = userRepository.save(mapToUser(request));
            
            logger.info("User created successfully: id={}, email={}", 
                user.getId(), user.getEmail());
            
            return user;
            
        } catch (DataIntegrityViolationException e) {
            // WARN: Expected issues that were handled
            logger.warn("Duplicate email attempted: {}", request.getEmail());
            throw new DuplicateEmailException(request.getEmail());
            
        } catch (DataAccessException e) {
            // ERROR: Unexpected problems needing attention
            logger.error("Failed to create user: email={}", request.getEmail(), e);
            throw new UserServiceException("User creation failed", e);
        }
    }
    
    public void processUser(Long userId) {
        // TRACE: Very detailed, usually disabled
        logger.trace("Entering processUser with userId={}", userId);
        
        // Method implementation...
        
        logger.trace("Exiting processUser");
    }
}
```

### Logging Best Practices

```java
// ✅ GOOD: Use parameterized logging (lazy evaluation)
logger.debug("Processing user: {}", userId);

// ❌ BAD: String concatenation (always evaluated)
logger.debug("Processing user: " + userId);

// ✅ GOOD: Include context
logger.info("Order processed: orderId={}, userId={}, amount={}", 
    orderId, userId, amount);

// ❌ BAD: Vague messages
logger.info("Done");

// ✅ GOOD: Log exceptions with stack trace
logger.error("Failed to process order: {}", orderId, exception);

// ❌ BAD: Losing stack trace
logger.error("Error: " + exception.getMessage());

// ✅ GOOD: Check log level for expensive operations
if (logger.isDebugEnabled()) {
    logger.debug("Complex data: {}", computeExpensiveDebugInfo());
}

// ✅ GOOD: Structured logging (key-value pairs)
logger.info("User action: userId={} action={} result={} duration={}ms",
    userId, "login", "success", duration);
```

### What NOT to Log

```java
// ❌ NEVER log sensitive information
logger.info("User login: email={}, password={}", email, password);  // NO!
logger.debug("Credit card: {}", cardNumber);  // NO!
logger.info("SSN: {}", ssn);  // NO!
logger.debug("API Key: {}", apiKey);  // NO!

// ✅ Mask or omit sensitive data
logger.info("User login: email={}", email);  // Password omitted
logger.info("Payment processed: card=****{}", lastFourDigits);  // Masked
```

---

## 10) Exception Logging Patterns

### Pattern: Log Once at Boundary

```java
// Service layer - throw, don't log
public class OrderService {
    public void processOrder(Long orderId) {
        Order order = orderRepository.findById(orderId)
            .orElseThrow(() -> new OrderNotFoundException(orderId));
        
        if (!paymentService.charge(order)) {
            throw new PaymentFailedException(orderId, order.getAmount());
        }
        // Exception propagates up without logging here
    }
}

// Controller/Handler - catch and log
@RestController
public class OrderController {
    
    private final Logger logger = LoggerFactory.getLogger(getClass());
    
    @PostMapping("/orders/{id}/process")
    public ResponseEntity<?> processOrder(@PathVariable Long id) {
        try {
            orderService.processOrder(id);
            return ResponseEntity.ok().build();
        } catch (OrderNotFoundException e) {
            logger.warn("Order not found: {}", id);
            return ResponseEntity.notFound().build();
        } catch (PaymentFailedException e) {
            logger.error("Payment failed for order: {}", id, e);
            return ResponseEntity.status(HttpStatus.PAYMENT_REQUIRED).build();
        }
    }
}
```

### Pattern: Correlation IDs

```java
// Include correlation ID for tracing requests across services
public class RequestContext {
    private static final ThreadLocal<String> correlationId = new ThreadLocal<>();
    
    public static void setCorrelationId(String id) {
        correlationId.set(id);
        MDC.put("correlationId", id);  // SLF4J MDC
    }
    
    public static String getCorrelationId() {
        return correlationId.get();
    }
    
    public static void clear() {
        correlationId.remove();
        MDC.remove("correlationId");
    }
}

// In filter/interceptor
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
    try {
        String correlationId = UUID.randomUUID().toString();
        RequestContext.setCorrelationId(correlationId);
        chain.doFilter(request, response);
    } finally {
        RequestContext.clear();
    }
}

// Log pattern in logback.xml includes correlationId
// %d{HH:mm:ss} [%X{correlationId}] %-5level %logger{36} - %msg%n
```

---

## 11) Common Interview Questions

### Fundamentals

**Q1: What is the difference between checked and unchecked exceptions?**

**Answer:**
- **Checked exceptions** are verified at compile-time. You must either handle them with try-catch or declare them with `throws`. Examples: `IOException`, `SQLException`.
- **Unchecked exceptions** (RuntimeException and its subclasses) are not checked at compile-time. Examples: `NullPointerException`, `IllegalArgumentException`.
- Use checked for recoverable conditions external to the program; use unchecked for programming errors.

**Q2: What is the difference between `throw` and `throws`?**

**Answer:**
- `throw` is used to explicitly throw an exception instance: `throw new IllegalArgumentException("Invalid");`
- `throws` is used in method signatures to declare exceptions that might be thrown: `public void read() throws IOException`

**Q3: Can we have a try block without catch?**

**Answer:** Yes, in two cases:
1. try-finally (without catch)
2. try-with-resources (automatically handles closing)

```java
try {
    // code
} finally {
    // cleanup
}

try (Resource r = new Resource()) {
    // auto-closed
}
```

### try-with-resources

**Q4: What is try-with-resources and why should we use it?**

**Answer:** try-with-resources (Java 7+) automatically closes resources that implement `AutoCloseable`. Benefits:
- Guarantees resource closure
- Handles exceptions in close() properly (suppressed exceptions)
- Reduces boilerplate code
- Prevents resource leaks

**Q5: What happens if both try block and close() throw exceptions?**

**Answer:** The exception from the try block is the primary exception. The exception from close() is added as a "suppressed exception" and can be accessed via `e.getSuppressed()`. This ensures no exceptions are lost.

### Exception Handling

**Q6: Why should we not catch `Throwable` or `Error`?**

**Answer:** 
- `Error` represents serious JVM problems (OutOfMemoryError, StackOverflowError) that applications typically cannot recover from.
- Catching them may prevent proper JVM shutdown or mask critical problems.
- If you must catch them (e.g., in a daemon thread), you should typically rethrow or terminate.

**Q7: Why is using exceptions for control flow considered bad practice?**

**Answer:**
- Exceptions are expensive (stack trace creation)
- Makes code harder to understand and debug
- Violates the principle that exceptions are for exceptional conditions
- Can mask real exceptions

**Q8: How do you create a custom exception?**

**Answer:**
```java
// Checked exception
public class BusinessException extends Exception {
    public BusinessException(String message) {
        super(message);
    }
    public BusinessException(String message, Throwable cause) {
        super(message, cause);
    }
}

// Unchecked exception
public class ValidationException extends RuntimeException {
    private final Map<String, String> errors;
    
    public ValidationException(Map<String, String> errors) {
        super("Validation failed");
        this.errors = errors;
    }
}
```

### Logging

**Q9: What are the log levels and when to use each?**

**Answer:**
| Level | Use Case |
|-------|----------|
| TRACE | Very detailed diagnostic (method entry/exit) |
| DEBUG | Detailed debugging information |
| INFO | General operational events |
| WARN | Potential problems, handled errors |
| ERROR | Errors requiring attention |

**Q10: What is the difference between logging facade (SLF4J) and implementation (Logback)?**

**Answer:**
- **SLF4J** is an abstraction/facade that your code depends on
- **Logback/Log4j2** are implementations that do the actual logging
- This separation allows switching implementations without changing code
- SLF4J provides a common API; the binding at runtime determines which implementation is used

### Best Practices

**Q11: What's wrong with this code?**

```java
try {
    riskyOperation();
} catch (Exception e) {
    throw new RuntimeException("Failed");
}
```

**Answer:** Multiple issues:
1. Catches too broadly (Exception)
2. Loses the original stack trace (should pass `e` as cause)
3. Uses generic RuntimeException instead of specific exception
4. Loses context about what failed

**Fixed version:**
```java
try {
    riskyOperation();
} catch (SpecificException e) {
    throw new ProcessingException("Risky operation failed", e);
}
```

**Q12: Should you log an exception and then rethrow it?**

**Answer:** Generally no - this leads to duplicate log entries as each layer logs the same exception. Best practice:
- Throw without logging in lower layers
- Log once at the system boundary (controller, message handler)
- Or log and handle (don't rethrow)

---

## 12) Quick Reference

### Exception Handling Syntax

```java
// Basic try-catch-finally
try {
    // risky code
} catch (SpecificException e) {
    // handle
} finally {
    // cleanup (always runs)
}

// Multi-catch (Java 7+)
try {
    // code
} catch (IOException | SQLException e) {
    // handle both
}

// try-with-resources (Java 7+)
try (Resource r = new Resource()) {
    // auto-closed
}

// Throw exception
throw new IllegalArgumentException("Invalid input");

// Declare throws
public void method() throws IOException { }
```

### Common Exceptions Cheat Sheet

| Exception | Type | Common Cause |
|-----------|------|--------------|
| `NullPointerException` | Unchecked | Operating on null reference |
| `IllegalArgumentException` | Unchecked | Invalid method argument |
| `IllegalStateException` | Unchecked | Invalid object state |
| `IndexOutOfBoundsException` | Unchecked | Invalid array/list index |
| `ClassCastException` | Unchecked | Invalid type cast |
| `IOException` | Checked | I/O operation failure |
| `SQLException` | Checked | Database operation failure |
| `FileNotFoundException` | Checked | File doesn't exist |
| `InterruptedException` | Checked | Thread interrupted |

### Logging Cheat Sheet

```java
// Logger declaration
private static final Logger logger = LoggerFactory.getLogger(MyClass.class);

// Parameterized logging (preferred)
logger.info("Processing user: {}", userId);
logger.error("Failed to process: {}", id, exception);

// Level checking for expensive operations
if (logger.isDebugEnabled()) {
    logger.debug("Data: {}", expensiveComputation());
}
```

### Exception Handling Decision Tree

```
Exception occurred?
├── Can you handle it meaningfully?
│   ├── YES → Catch and handle
│   └── NO → Let it propagate (maybe wrap in domain exception)
│
├── Is it a checked exception?
│   ├── YES → Must declare throws or catch
│   └── NO → Consider if catching adds value
│
├── Should you log it?
│   ├── At boundary? → YES, log with context
│   └── Middle layer? → NO, just propagate
│
└── Creating custom exception?
    ├── Need specific context? → YES, create custom
    └── Standard exception fits? → NO, use standard
```

---

*Good exception handling makes your code robust, maintainable, and debuggable!*
