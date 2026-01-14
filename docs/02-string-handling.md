# 2️⃣ String Handling (High-Frequency Area)

Strings are among the most frequently asked Java topics because they test multiple fundamentals at once:

- immutability and object identity
- hashing and equality
- performance and memory behavior
- thread-safety trade-offs (`StringBuilder` vs `StringBuffer`)

---

## 2.1 String Class Deep Dive

### String immutability

`String` is immutable: once created, its content cannot change.

When you “modify” a string, you create a **new** string:

```java
String s = "abc";
s = s + "d";           // creates a new String "abcd"
```

Why Java made `String` immutable (interview-worthy reasons):

- **Security**: safe to share in sensitive contexts (classloading paths, URLs, config keys).
- **String pool** works effectively only with immutability (shared storage).
- **Hash caching**: many JVMs cache `String.hashCode()` because content doesn’t change.
- **Thread-safety**: immutable objects are safe to share across threads.

Illustration:

```
s ---> "abc"   (object #1)
        |
        +-- after concatenation
             s ---> "abcd" (object #2)
```

### String pool & `intern()`

The JVM maintains a **string pool** for string literals (and interned strings).

- String literals like `"java"` are typically stored once in the pool.
- Two references to the same literal often point to the same pooled object.

```java
String a = "java";
String b = "java";
System.out.println(a == b); // true (same pooled literal)
```

But:

```java
String a = "java";
String b = new String("java");
System.out.println(a == b); // false (b is a different object)
```

`intern()` returns the pooled canonical representation:

```java
String a = "java";
String b = new String("java").intern();
System.out.println(a == b); // true
```

Interview advice:

- Don’t use `intern()` as a default optimization; it can increase pressure on the pool and complicate memory behavior.

### `==` vs `equals()`

- `==` checks **reference identity** (same object?)
- `equals()` checks **logical equality** (same content?) for `String`

```java
String a = new String("x");
String b = new String("x");

System.out.println(a == b);      // false
System.out.println(a.equals(b)); // true
```

### `hashCode()` contract (and why it matters)

Contract:

- If `a.equals(b)` is true, then `a.hashCode() == b.hashCode()` must be true.

Why interviewers care:

- `HashMap` and `HashSet` depend on it.
- With strings, hashing is extremely common (map keys, caching, interning-like behavior).

Important note:

- equal hash codes do **not** imply equality (collisions exist).

### `StringBuilder` vs `StringBuffer`

#### `StringBuilder`

- Mutable sequence of characters
- **Not synchronized** → faster in single-threaded contexts

#### `StringBuffer`

- Mutable sequence of characters
- **Synchronized** methods → thread-safe but slower (contention)

Rule of thumb:

- Use **`StringBuilder`** unless you specifically need synchronized mutation (rare today).

### Performance implications (what to say in interviews)

#### Concatenation in loops

Bad (creates many intermediate strings):

```java
String s = "";
for (int i = 0; i < n; i++) {
  s += i; // many allocations
}
```

Better:

```java
StringBuilder sb = new StringBuilder();
for (int i = 0; i < n; i++) {
  sb.append(i);
}
String s = sb.toString();
```

#### Compiler optimization nuance

`"a" + "b" + "c"` at compile time becomes a single constant `"abc"`.
But concatenation across variables (especially in loops) typically allocates.

#### Substring memory nuance (historical)

In older Java versions, `substring()` could share the original char array, potentially causing memory retention.
Modern Java creates new arrays for substring to avoid that class of leaks.

### Common string interview problems (with patterns)

#### 1) Reverse a string

```java
String reversed = new StringBuilder(s).reverse().toString();
```

If asked to do it manually, show two-pointer swap on a char array.

#### 2) Check palindrome

Two-pointer approach:

```java
int i = 0, j = s.length() - 1;
while (i < j) {
  if (s.charAt(i++) != s.charAt(j--)) return false;
}
return true;
```

#### 3) Check anagrams

Approach options:

- sort both strings and compare
- frequency array (for lowercase a-z)
- `Map<Character, Integer>` frequency (general)

#### 4) First non-repeating character

Use frequency counting + second pass.

#### 5) Longest substring without repeating characters

Sliding window with a map of last seen indices.

Interview expectation: you can explain *why* it’s O(n).

---

## Quick interview checklist (Strings)

- Can you explain string pool and why `==` sometimes “works” on literals?
- Can you explain why `String` is immutable with at least 2 strong reasons?
- Can you explain why `StringBuilder` is preferred in loops?
- Do you know `equals`/`hashCode` contract and why it matters for map/set keys?

