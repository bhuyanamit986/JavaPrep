# Exceptions, Error Handling & Logging

## Definitions

- **Throwable**: the root of all errors and exceptions in Java.
- **Checked exception**: must be declared or handled; signals recoverable conditions.
- **Unchecked exception**: `RuntimeException` and subclasses; often represent programming errors.
- **Try-with-resources**: a `try` block that auto-closes `AutoCloseable` resources safely.
- **Suppressed exception**: an exception thrown during resource close that is attached to the primary failure.
- **Custom exception**: a domain-specific exception type that adds context and clarity.
- **Logging level**: severity (TRACE, DEBUG, INFO, WARN, ERROR) used to filter output.

## Illustrations

- **Exception flow**: an exception bubbles up the call stack like a balloon until someone catches it.
- **Checked vs unchecked**: checked exceptions are like a contract you must acknowledge; unchecked are like runtime surprises.
- **Logging levels**: think of a volume knob - TRACE is whispering details, ERROR is shouting problems.

## Code Examples

```java
private static final org.slf4j.Logger log =
        org.slf4j.LoggerFactory.getLogger(App.class);

try (java.io.BufferedReader br = java.nio.file.Files.newBufferedReader(path)) {
    return br.readLine();
} catch (java.io.IOException e) {
    log.warn("Failed to read {}", path, e);
    throw new IllegalStateException("Read failed", e);
}
```

## Interview Questions

1. What is the difference between `Error` and `Exception`?
2. When should you use checked vs unchecked exceptions?
3. How does try-with-resources handle multiple close failures?
4. What is exception wrapping and why is it useful?
5. Explain logging levels and how to choose them.
6. How do you design a meaningful custom exception?

---

## 1) Throwable hierarchy (interview level)

- `Throwable`
  - `Error`: serious problems the app typically shouldn’t catch (e.g., `OutOfMemoryError`)
  - `Exception`
    - Checked exceptions: must be declared/handled
    - Unchecked exceptions: `RuntimeException` and subclasses

Key idea: checked exceptions force handling at compile time; unchecked represent programming errors or unrecoverable conditions in many codebases.

## 2) Checked vs unchecked (trade-offs)

### Checked exceptions

Pros:

- Forces explicit handling, can make failure modes visible.

Cons:

- Can lead to noisy code (`throws` everywhere).
- Often wrapped into runtime exceptions in layered systems.

### Unchecked exceptions

Pros:

- Less boilerplate; propagate naturally.

Cons:

- Failure path can be implicit; must be documented by convention/tests.

Interview best answer: use checked exceptions for **recoverable** conditions when callers can reasonably act; use unchecked for **programming errors** or when recovery isn’t meaningful at that level.

## 3) `try-with-resources` (must know)

Preferred for closing resources implementing `AutoCloseable`.

```java
try (BufferedReader br = Files.newBufferedReader(path)) {
  return br.readLine();
}
```

### Suppressed exceptions

If both the try block and close throw, the close exception becomes **suppressed**.

Interview point: `Throwable#getSuppressed()` exists for diagnostics.

## 4) Designing custom exceptions

Guidelines:

- Use meaningful names (`InvalidOrderStateException`).
- Add context fields when helpful (ids, states), but avoid sensitive data.
- Preserve causes: `new XException("msg", cause)`.
- Don’t overuse; exceptions should represent **exceptional** conditions.

## 5) Common anti-patterns

- **Swallowing exceptions**:

```java
try { ... } catch (Exception e) { }
```

- **Catching too broad** (`Throwable`) unless you rethrow or terminate intentionally.
- **Using exceptions for control flow** (slow and unclear).
- **Losing stack trace** by creating new exception without cause.

## 6) Logging (interview-level)

### What interviewers expect

- You understand log levels: TRACE/DEBUG/INFO/WARN/ERROR.
- You avoid logging sensitive information (PII, tokens, passwords).
- You include context: request ids, user ids (if safe), correlation ids.

### Structured logging

Prefer key-value fields over concatenated strings when possible, e.g.:

- `orderId=... status=... latencyMs=...`

### Don’t log and throw the same exception repeatedly

If higher layers also log, you can get duplicate noisy logs.
Common practice: log once at the boundary (e.g., request handler), and propagate exceptions upward.

## 7) Interview questions

- Difference between checked and unchecked exceptions? Give examples and justify choices.
- What is try-with-resources and why is it safer?
- Explain suppressed exceptions.
- Best practices for logging exceptions in a service?

