# 4️⃣ Exception Handling

Exception handling is tested because it reveals how you design reliable code and how well you understand Java’s `Throwable` hierarchy.

---

## Checked vs unchecked exceptions

### Checked exceptions

- Subclasses of `Exception` excluding `RuntimeException`.
- Must be caught or declared with `throws`.

Use when:

- the caller can realistically recover (e.g., missing file, retryable operation) *and*
- you want recovery to be explicit in the API.

### Unchecked exceptions

- Subclasses of `RuntimeException`.
- Not enforced by compiler.

Use when:

- programming errors (`NullPointerException`, `IllegalArgumentException`)
- violating preconditions or invariants
- errors that can’t be reasonably handled at that layer

Interview-ready statement: “Checked vs unchecked is a design decision; checked communicates recoverability and forces handling, but can create boilerplate in layered systems.”

---

## Error vs Exception

- `Error`: serious JVM-level problems (often not meant to be caught), e.g., `OutOfMemoryError`.
- `Exception`: application-level exceptional conditions.

Rule: don’t catch `Error` unless you’re at a boundary and you’re terminating/logging.

---

## `try-catch-finally`

- `try`: code that might throw
- `catch`: handle specific exceptions
- `finally`: runs regardless of success/failure (except some termination cases)

Ordering matters: catch most specific first.

```java
try {
  // ...
} catch (FileNotFoundException e) {
  // specific
} catch (IOException e) {
  // general
} finally {
  // cleanup
}
```

---

## `try-with-resources`

Use for `AutoCloseable` resources. It guarantees close and is less error-prone.

```java
try (BufferedReader br = Files.newBufferedReader(path)) {
  return br.readLine();
}
```

### Suppressed exceptions

If both the try body and `close()` throw, the close exception becomes **suppressed**:

- `Throwable#getSuppressed()`

---

## Custom exceptions

Guidelines:

- name them by domain meaning (`InvalidUserStateException`)
- include a cause when wrapping
- include contextual fields (ids/state) but avoid secrets

```java
public class PaymentFailedException extends RuntimeException {
  public PaymentFailedException(String message, Throwable cause) {
    super(message, cause);
  }
}
```

---

## Exception propagation

If a method does not catch an exception, it propagates up the call stack until:

- caught by a caller, or
- reaches the thread boundary → uncaught exception handler → thread terminates

Interview point: frameworks often have top-level exception handlers (e.g., web controllers) that convert exceptions into error responses.

---

## Best practices

- Catch the **most specific** exception you can handle.
- Don’t swallow exceptions.
- Don’t lose the original cause.
- Use exceptions for exceptional conditions, not control flow.
- Prefer validation + clear error messages over letting NPEs happen.

---

## Common interview traps

### 1) `finally` overriding return

```java
static int f() {
  try { return 1; }
  finally { return 2; } // overrides
}
```

### 2) Throwing from `finally`

If `finally` throws, it can mask the original exception.

### 3) Catching `Exception` too early

Catching too broad exceptions can hide bugs and make debugging harder.

### 4) `throw e` vs `throw new X(e)`

- `throw e` preserves original stack trace
- wrapping should include the cause

---

## Quick questions to self-test

- Explain checked vs unchecked and when you’d choose each.
- What is try-with-resources? What are suppressed exceptions?
- Why is catching `Throwable` usually wrong?
- What are common problems with `finally`?

---

## Extra reference

For additional notes on logging practices and exception anti-patterns, see `docs/exceptions-logging.md`.

