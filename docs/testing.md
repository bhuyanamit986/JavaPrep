# Testing & Quality

Testing is fundamental to software quality. This guide covers testing concepts, JUnit, mocking, and best practices commonly asked in Java interviews.

## Definitions

- **Unit Test**: Tests a single unit (method/class) in complete isolation. Dependencies are mocked. Fast and focused.

- **Integration Test**: Tests how multiple components work together. May include real databases, APIs, or external services.

- **End-to-End (E2E) Test**: Tests complete user workflows through the entire system. Slowest but highest confidence.

- **Mock**: A test double that simulates behavior and can verify how it was called. Used to isolate the unit under test.

- **Stub**: A test double that returns predefined responses. Doesn't verify interactions.

- **Spy**: A partial mock that wraps a real object, allowing selective method stubbing.

- **Test Pyramid**: Strategy recommending many fast unit tests, fewer integration tests, and few slow E2E tests.

- **TDD (Test-Driven Development)**: Write failing test first, write minimal code to pass, refactor. Red → Green → Refactor.

- **Code Coverage**: Percentage of code executed by tests. High coverage doesn't guarantee quality but low coverage indicates risk.

## Illustrations

### Test Pyramid Explained

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         TEST PYRAMID                                     │
│                                                                          │
│                            /\                                            │
│                           /  \    E2E Tests                             │
│                          /    \   • Few tests                           │
│                         /      \  • Slow (minutes)                      │
│                        /        \ • High confidence                     │
│                       /    UI    \ • Expensive to maintain              │
│                      /____________\                                      │
│                     /              \                                     │
│                    /                \  Integration Tests                 │
│                   /                  \ • Medium count                    │
│                  /                    \ • Medium speed                   │
│                 /      Service         \ • Test component interactions  │
│                /________________________\                                │
│               /                          \                               │
│              /                            \  Unit Tests                  │
│             /                              \ • Many tests                │
│            /                                \ • Very fast (ms)           │
│           /              Unit                \ • Test one class/method  │
│          /____________________________________\                          │
│                                                                          │
│   Write MORE tests at the bottom, FEWER at the top!                     │
│                                                                          │
│   Anti-pattern: Ice Cream Cone (inverted pyramid)                       │
│   ┌──────────────────────┐                                              │
│   │     Many E2E         │  ← Too many slow, fragile tests             │
│   ├──────────────────────┤                                              │
│   │ Few Integration      │                                              │
│   ├──────────────────────┤                                              │
│   │ Fewer Unit           │  ← Should have more!                         │
│   └──────────────────────┘                                              │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Test Doubles Explained

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         TEST DOUBLES                                     │
│                                                                          │
│   Real Object                  Test Double                              │
│   ┌──────────────┐            ┌──────────────┐                          │
│   │  Database    │    ──▶     │   Mock DB    │  (Fake implementation)  │
│   │  (Slow, I/O) │            │  (Fast)      │                          │
│   └──────────────┘            └──────────────┘                          │
│                                                                          │
│   Types of Test Doubles:                                                │
│   ┌────────────────────────────────────────────────────────────────┐   │
│   │                                                                 │   │
│   │   DUMMY   - Passed but never used. Fills parameter lists.      │   │
│   │   ┌───────────────────────────────────────────────────┐        │   │
│   │   │ new Service(new DummyLogger());  // Never called  │        │   │
│   │   └───────────────────────────────────────────────────┘        │   │
│   │                                                                 │   │
│   │   STUB    - Returns canned answers. No verification.           │   │
│   │   ┌───────────────────────────────────────────────────┐        │   │
│   │   │ when(userRepo.findById(1)).thenReturn(testUser);  │        │   │
│   │   └───────────────────────────────────────────────────┘        │   │
│   │                                                                 │   │
│   │   MOCK    - Pre-programmed with expectations. Verifies calls.  │   │
│   │   ┌───────────────────────────────────────────────────┐        │   │
│   │   │ verify(emailService).sendEmail(any());            │        │   │
│   │   │ // Fails if not called!                           │        │   │
│   │   └───────────────────────────────────────────────────┘        │   │
│   │                                                                 │   │
│   │   SPY     - Wraps real object. Can stub specific methods.      │   │
│   │   ┌───────────────────────────────────────────────────┐        │   │
│   │   │ List<String> spy = spy(new ArrayList<>());        │        │   │
│   │   │ doReturn(100).when(spy).size();  // Stub one method│       │   │
│   │   └───────────────────────────────────────────────────┘        │   │
│   │                                                                 │   │
│   │   FAKE    - Working implementation, but simplified.            │   │
│   │   ┌───────────────────────────────────────────────────┐        │   │
│   │   │ class FakeUserRepository implements UserRepository│        │   │
│   │   │     Map<Long, User> data = new HashMap<>();       │        │   │
│   │   │     // Real logic, but in-memory                  │        │   │
│   │   │ }                                                 │        │   │
│   │   └───────────────────────────────────────────────────┘        │   │
│   │                                                                 │   │
│   └────────────────────────────────────────────────────────────────┘   │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### TDD Cycle

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    TDD: RED → GREEN → REFACTOR                           │
│                                                                          │
│            ┌─────────────────────────────────────────┐                  │
│            │                                         │                  │
│            ▼                                         │                  │
│   ┌─────────────────┐                                │                  │
│   │      RED        │  1. Write a failing test       │                  │
│   │   (Test fails)  │     for new functionality      │                  │
│   └────────┬────────┘                                │                  │
│            │                                         │                  │
│            ▼                                         │                  │
│   ┌─────────────────┐                                │                  │
│   │     GREEN       │  2. Write MINIMAL code to      │                  │
│   │   (Test passes) │     make the test pass         │                  │
│   └────────┬────────┘                                │                  │
│            │                                         │                  │
│            ▼                                         │                  │
│   ┌─────────────────┐                                │                  │
│   │    REFACTOR     │  3. Clean up code while        │                  │
│   │  (Still passes) │     keeping tests green        │                  │
│   └────────┬────────┘                                │                  │
│            │                                         │                  │
│            └─────────────────────────────────────────┘                  │
│                        (Repeat for next feature)                        │
│                                                                          │
│   Benefits:                                                             │
│   • Forces you to think about design first                              │
│   • Ensures high test coverage                                          │
│   • Provides safety net for refactoring                                 │
│   • Documents expected behavior                                         │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

## Code Examples

```java
@org.junit.jupiter.api.Test
void addsNumbers() {
    org.junit.jupiter.api.Assertions.assertEquals(5, 2 + 3);
}
```

## Interview Questions

1. What is the difference between unit and integration tests?
2. Why do we prefer a testing pyramid?
3. When is mocking appropriate vs not appropriate?
4. What does TDD buy you in a team setting?
5. How do you test time or randomness in code?

---

## 1) Testing Fundamentals

### Types of Tests

| Type | Scope | Speed | Purpose |
|------|-------|-------|---------|
| **Unit** | Single class/method | Fast | Verify isolated logic |
| **Integration** | Multiple components | Medium | Verify interactions |
| **End-to-End (E2E)** | Entire system | Slow | Verify user flows |
| **Performance** | System | Varies | Verify speed/load |
| **Security** | System | Varies | Verify vulnerabilities |

### Testing Pyramid

```
        /\
       /  \  E2E Tests (few)
      /    \
     /      \  Integration Tests
    /        \
   /          \  Unit Tests (many)
  /____________\
```

**Key principle**: More unit tests (fast, cheap), fewer E2E tests (slow, expensive).

### Test Characteristics (F.I.R.S.T.)

| Letter | Principle | Meaning |
|--------|-----------|---------|
| F | **Fast** | Tests should run quickly |
| I | **Independent** | Tests shouldn't depend on each other |
| R | **Repeatable** | Same result every time |
| S | **Self-validating** | Pass or fail, no manual inspection |
| T | **Timely** | Written close to production code |

---

## 2) JUnit 5 (Jupiter)

### Basic Test Structure

```java
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;

class CalculatorTest {

    private Calculator calculator;

    @BeforeAll
    static void setupAll() {
        // Runs once before all tests
        System.out.println("Starting Calculator tests");
    }

    @BeforeEach
    void setup() {
        // Runs before each test
        calculator = new Calculator();
    }

    @Test
    @DisplayName("Should add two positive numbers")
    void testAddition() {
        int result = calculator.add(2, 3);
        assertEquals(5, result, "2 + 3 should equal 5");
    }

    @Test
    void testDivision() {
        assertEquals(2, calculator.divide(6, 3));
    }

    @Test
    void testDivisionByZero() {
        assertThrows(ArithmeticException.class, () -> {
            calculator.divide(1, 0);
        });
    }

    @AfterEach
    void tearDown() {
        // Runs after each test
    }

    @AfterAll
    static void tearDownAll() {
        // Runs once after all tests
    }
}
```

### Lifecycle Annotations

| Annotation | Scope | Purpose |
|------------|-------|---------|
| `@BeforeAll` | Class | Setup once before all tests (static) |
| `@BeforeEach` | Test | Setup before each test |
| `@AfterEach` | Test | Cleanup after each test |
| `@AfterAll` | Class | Cleanup once after all tests (static) |

### Assertions

```java
// Basic assertions
assertEquals(expected, actual);
assertEquals(expected, actual, "Message on failure");
assertNotEquals(unexpected, actual);

// Boolean
assertTrue(condition);
assertFalse(condition);

// Null checks
assertNull(value);
assertNotNull(value);

// Reference equality
assertSame(expected, actual);      // Same object (==)
assertNotSame(unexpected, actual);

// Array equality
assertArrayEquals(expectedArray, actualArray);

// Exception assertions
assertThrows(IllegalArgumentException.class, () -> {
    service.process(null);
});

Exception ex = assertThrows(CustomException.class, () -> {
    service.doSomething();
});
assertEquals("Expected message", ex.getMessage());

// No exception
assertDoesNotThrow(() -> service.safeMethod());

// Timeout
assertTimeout(Duration.ofSeconds(2), () -> {
    slowOperation();
});

// Multiple assertions (all run, all must pass)
assertAll("person",
    () -> assertEquals("John", person.getFirstName()),
    () -> assertEquals("Doe", person.getLastName()),
    () -> assertEquals(30, person.getAge())
);
```

### Test Annotations

```java
@Test
void normalTest() {}

@Test
@Disabled("Bug #123 - temporarily disabled")
void disabledTest() {}

@Test
@DisplayName("Custom readable test name")
void testWithDisplayName() {}

@RepeatedTest(5)
void repeatedTest(RepetitionInfo info) {
    System.out.println("Repetition " + info.getCurrentRepetition());
}

@Tag("slow")
@Test
void slowTest() {}

@Test
@Timeout(value = 100, unit = TimeUnit.MILLISECONDS)
void testWithTimeout() {}
```

### Nested Tests

```java
@DisplayName("Calculator Tests")
class CalculatorTest {
    
    Calculator calculator = new Calculator();
    
    @Nested
    @DisplayName("Addition")
    class AdditionTests {
        
        @Test
        @DisplayName("positive numbers")
        void positiveNumbers() {
            assertEquals(5, calculator.add(2, 3));
        }
        
        @Test
        @DisplayName("negative numbers")
        void negativeNumbers() {
            assertEquals(-5, calculator.add(-2, -3));
        }
    }
    
    @Nested
    @DisplayName("Division")
    class DivisionTests {
        
        @Test
        void byNonZero() {
            assertEquals(2, calculator.divide(6, 3));
        }
        
        @Test
        void byZeroThrows() {
            assertThrows(ArithmeticException.class,
                () -> calculator.divide(1, 0));
        }
    }
}
```

---

## 3) Parameterized Tests

### Value Source

```java
@ParameterizedTest
@ValueSource(ints = {1, 2, 3, 4, 5})
void testIsPositive(int number) {
    assertTrue(number > 0);
}

@ParameterizedTest
@ValueSource(strings = {"", "  ", "\t", "\n"})
void testIsBlank(String input) {
    assertTrue(input.isBlank());
}

@ParameterizedTest
@NullSource
@EmptySource
@ValueSource(strings = {"  ", "\t"})
void testNullEmptyAndBlank(String input) {
    assertTrue(input == null || input.isBlank());
}

// Shortcut for null and empty
@ParameterizedTest
@NullAndEmptySource
void testNullAndEmpty(String input) {
    // input is null or ""
}
```

### Enum Source

```java
@ParameterizedTest
@EnumSource(Month.class)
void testAllMonths(Month month) {
    assertNotNull(month);
}

@ParameterizedTest
@EnumSource(value = Month.class, names = {"JANUARY", "FEBRUARY", "MARCH"})
void testFirstQuarter(Month month) {
    int monthNumber = month.getValue();
    assertTrue(monthNumber <= 3);
}

@ParameterizedTest
@EnumSource(value = Month.class, mode = EnumSource.Mode.EXCLUDE, names = {"JUNE", "JULY", "AUGUST"})
void testNonSummerMonths(Month month) {
    // All months except summer
}
```

### CSV Source

```java
@ParameterizedTest
@CsvSource({
    "1, 1, 2",
    "2, 3, 5",
    "10, 20, 30"
})
void testAddition(int a, int b, int expected) {
    assertEquals(expected, calculator.add(a, b));
}

@ParameterizedTest
@CsvSource({
    "apple, APPLE",
    "hello world, HELLO WORLD",
    "'foo, bar', 'FOO, BAR'"  // Quote for values with commas
})
void testToUpperCase(String input, String expected) {
    assertEquals(expected, input.toUpperCase());
}

// From CSV file
@ParameterizedTest
@CsvFileSource(resources = "/test-data.csv", numLinesToSkip = 1)
void testFromFile(String input, int expected) {
    // Read from src/test/resources/test-data.csv
}
```

### Method Source

```java
@ParameterizedTest
@MethodSource("provideStringsForIsBlank")
void testIsBlank(String input, boolean expected) {
    assertEquals(expected, Strings.isBlank(input));
}

private static Stream<Arguments> provideStringsForIsBlank() {
    return Stream.of(
        Arguments.of(null, true),
        Arguments.of("", true),
        Arguments.of("  ", true),
        Arguments.of("hello", false),
        Arguments.of(" hello ", false)
    );
}

// External method source
@ParameterizedTest
@MethodSource("com.example.TestDataProvider#provideData")
void testWithExternalSource(String data) {
    // ...
}
```

### Arguments Aggregator

```java
@ParameterizedTest
@CsvSource({
    "John, Doe, 30",
    "Jane, Smith, 25"
})
void testPerson(
    @AggregateWith(PersonAggregator.class) Person person) {
    assertNotNull(person.getFullName());
}

static class PersonAggregator implements ArgumentsAggregator {
    @Override
    public Person aggregateArguments(ArgumentsAccessor accessor, ParameterContext context) {
        return new Person(
            accessor.getString(0),
            accessor.getString(1),
            accessor.getInteger(2)
        );
    }
}
```

---

## 4) Mocking with Mockito

### Basic Mocking

```java
import static org.mockito.Mockito.*;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private UserService userService;

    @Test
    void testGetUser() {
        // Arrange
        User mockUser = new User(1, "John");
        when(userRepository.findById(1)).thenReturn(Optional.of(mockUser));

        // Act
        User result = userService.getUser(1);

        // Assert
        assertEquals("John", result.getName());
        verify(userRepository).findById(1);
    }
}
```

### Creating Mocks

```java
// Annotation-based (with @ExtendWith)
@Mock
private UserRepository userRepository;

// Programmatic
UserRepository userRepository = mock(UserRepository.class);

// Deep stubs (auto-mock return values)
Foo foo = mock(Foo.class, RETURNS_DEEP_STUBS);
when(foo.getBar().getBaz()).thenReturn("value");
```

### Stubbing Behavior

```java
// Return value
when(mock.method()).thenReturn("value");
when(mock.method(anyString())).thenReturn("value");

// Return different values on subsequent calls
when(mock.method())
    .thenReturn("first")
    .thenReturn("second")
    .thenReturn("third");

// Throw exception
when(mock.method()).thenThrow(new RuntimeException("Error"));
when(mock.method()).thenThrow(RuntimeException.class);

// Void methods
doNothing().when(mock).voidMethod();
doThrow(new RuntimeException()).when(mock).voidMethod();

// Answer (dynamic response)
when(mock.method(anyString())).thenAnswer(invocation -> {
    String arg = invocation.getArgument(0);
    return arg.toUpperCase();
});

// Call real method (partial mock)
when(mock.method()).thenCallRealMethod();
```

### Argument Matchers

```java
// Any matchers
when(mock.method(any())).thenReturn("result");
when(mock.method(anyInt())).thenReturn("result");
when(mock.method(anyString())).thenReturn("result");
when(mock.method(anyList())).thenReturn("result");

// Specific matchers
when(mock.method(eq("specific"))).thenReturn("result");
when(mock.method(isNull())).thenReturn("default");
when(mock.method(notNull())).thenReturn("result");

// Custom matchers
when(mock.method(argThat(s -> s.length() > 5))).thenReturn("long");

// Combining (all args must use matchers if any does)
when(mock.method(eq("value"), anyInt())).thenReturn("result");
// WRONG: when(mock.method("value", anyInt()))  // Mixing raw and matcher
```

### Verification

```java
// Verify called
verify(mock).method();
verify(mock, times(2)).method();
verify(mock, atLeast(1)).method();
verify(mock, atMost(3)).method();
verify(mock, never()).method();

// Verify argument
verify(mock).method("expectedArg");
verify(mock).method(argThat(arg -> arg.startsWith("prefix")));

// Argument capture
ArgumentCaptor<User> captor = ArgumentCaptor.forClass(User.class);
verify(mock).save(captor.capture());
User savedUser = captor.getValue();
assertEquals("John", savedUser.getName());

// Verify order
InOrder inOrder = inOrder(mock1, mock2);
inOrder.verify(mock1).method1();
inOrder.verify(mock2).method2();

// Verify no more interactions
verifyNoMoreInteractions(mock);
verifyNoInteractions(mock);
```

### Spies (Partial Mocks)

```java
// Spy wraps real object
List<String> realList = new ArrayList<>();
List<String> spyList = spy(realList);

// Calls real methods by default
spyList.add("one");
assertEquals(1, spyList.size());

// But can stub specific methods
doReturn("stubbed").when(spyList).get(0);
assertEquals("stubbed", spyList.get(0));

// With annotation
@Spy
private ArrayList<String> spyList;
```

### Mocking Static Methods (Mockito 3.4+)

```java
@Test
void testStaticMethod() {
    try (MockedStatic<Utils> mocked = mockStatic(Utils.class)) {
        mocked.when(Utils::getCurrentTime).thenReturn("12:00");
        
        assertEquals("12:00", Utils.getCurrentTime());
    }
    // Static mock is closed, original behavior restored
}
```

---

## 5) Test Doubles

| Type | Purpose | Example |
|------|---------|---------|
| **Dummy** | Fill parameter lists | `null`, empty object |
| **Stub** | Provide canned answers | `when().thenReturn()` |
| **Spy** | Record calls + real behavior | `spy(realObject)` |
| **Mock** | Verify interactions | `verify(mock).method()` |
| **Fake** | Working implementation (simple) | In-memory database |

---

## 6) Test Organization

### Arrange-Act-Assert (AAA)

```java
@Test
void shouldCalculateTotalPrice() {
    // Arrange - setup test data
    ShoppingCart cart = new ShoppingCart();
    cart.addItem(new Item("Book", 20));
    cart.addItem(new Item("Pen", 5));

    // Act - perform the action
    double total = cart.calculateTotal();

    // Assert - verify the result
    assertEquals(25.0, total, 0.001);
}
```

### Given-When-Then (BDD Style)

```java
@Test
void shouldApplyDiscountForPremiumCustomers() {
    // Given
    Customer customer = new Customer("John", CustomerType.PREMIUM);
    Order order = new Order(customer, 100.0);

    // When
    double finalPrice = order.calculateFinalPrice();

    // Then
    assertEquals(90.0, finalPrice);  // 10% discount
}
```

### Test Naming Conventions

```java
// Pattern: methodName_stateUnderTest_expectedBehavior
@Test
void calculateTotal_emptyCart_returnsZero() {}

@Test
void calculateTotal_singleItem_returnsItemPrice() {}

// Pattern: should_expectedBehavior_when_stateUnderTest
@Test
void should_returnZero_when_cartIsEmpty() {}

// Pattern: givenX_whenY_thenZ
@Test
void givenEmptyCart_whenCalculateTotal_thenReturnsZero() {}

// Use @DisplayName for readable names
@Test
@DisplayName("Should return zero for empty cart")
void testEmptyCart() {}
```

---

## 7) Testing Best Practices

### Do's

```java
// ✅ Test one thing per test
@Test
void shouldAddItemToCart() {
    cart.addItem(item);
    assertEquals(1, cart.getItemCount());
}

// ✅ Use meaningful names
@Test
void shouldThrowExceptionWhenDividingByZero() {}

// ✅ Test edge cases
@Test
void shouldHandleEmptyList() {}
@Test
void shouldHandleNullInput() {}
@Test
void shouldHandleMaxIntValue() {}

// ✅ Keep tests independent
@BeforeEach
void setup() {
    // Fresh state for each test
    repository = new InMemoryRepository();
}

// ✅ Use assertions wisely
assertEquals(expected, actual);  // Not assertEquals(actual, expected)!

// ✅ Test behavior, not implementation
@Test
void shouldPersistUser() {
    userService.createUser("John");
    assertTrue(userService.userExists("John"));
    // Don't verify internal method calls unless necessary
}
```

### Don'ts

```java
// ❌ Don't test multiple things
@Test
void testUserOperations() {
    // Don't combine create, update, delete in one test
}

// ❌ Don't use logic in tests
@Test
void testCalculation() {
    int expected = 2 + 3;  // ❌ Logic duplicates production code
    assertEquals(expected, calculator.add(2, 3));
    
    assertEquals(5, calculator.add(2, 3));  // ✅ Hardcoded expected value
}

// ❌ Don't test trivial code
@Test
void testGetter() {
    user.setName("John");
    assertEquals("John", user.getName());  // Usually unnecessary
}

// ❌ Don't use Thread.sleep() for timing
@Test
void testAsyncOperation() {
    service.startAsync();
    Thread.sleep(1000);  // ❌ Flaky!
    // Use Awaitility or CompletableFuture instead
}

// ❌ Don't test private methods directly
// Test through public API instead

// ❌ Don't catch exceptions unnecessarily
@Test
void testThrowsException() {
    try {
        service.process(null);
        fail();  // ❌ Hard to read
    } catch (IllegalArgumentException e) {
        // expected
    }
    
    // ✅ Better
    assertThrows(IllegalArgumentException.class, 
        () -> service.process(null));
}
```

---

## 8) Test Anti-Patterns

### The Giant Test

```java
// ❌ Tests too much
@Test
void testEntireUserWorkflow() {
    // Creates user
    // Updates user
    // Deletes user
    // Verifies notifications
    // 200 lines...
}

// ✅ Split into focused tests
@Test void shouldCreateUser() {}
@Test void shouldUpdateUser() {}
@Test void shouldDeleteUser() {}
```

### The Mockery

```java
// ❌ Too many mocks
@Test
void testWithTenMocks() {
    // If you need this many mocks, class probably has too many dependencies
}

// ✅ Redesign for testability
```

### The Inspector

```java
// ❌ Tests implementation details
@Test
void testInternals() {
    service.process(data);
    verify(internalHelper, times(3)).doSomething();
    verify(privateField).update();
}

// ✅ Test behavior/outcomes
@Test
void shouldProcessDataSuccessfully() {
    Result result = service.process(data);
    assertEquals(Status.SUCCESS, result.getStatus());
}
```

### The Slow Poke

```java
// ❌ Slow test
@Test
void slowIntegrationTest() {
    // Connects to real database
    // Makes HTTP calls
    // Takes 30 seconds
}

// ✅ Use mocks for unit tests, run integration tests separately
```

### The Flickering Test

```java
// ❌ Flaky test (passes sometimes, fails sometimes)
@Test
void testWithRandomData() {
    // Uses System.currentTimeMillis()
    // Uses Math.random()
    // Depends on timing
}

// ✅ Use deterministic data, inject time/random sources
```

---

## 9) Integration Testing

### Spring Boot Test

```java
@SpringBootTest
@AutoConfigureMockMvc
class UserControllerIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private UserRepository userRepository;

    @BeforeEach
    void setup() {
        userRepository.deleteAll();
    }

    @Test
    void shouldCreateUser() throws Exception {
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"name\": \"John\", \"email\": \"john@example.com\"}"))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.name").value("John"));

        assertEquals(1, userRepository.count());
    }

    @Test
    void shouldReturnUserById() throws Exception {
        User saved = userRepository.save(new User("John", "john@example.com"));

        mockMvc.perform(get("/api/users/" + saved.getId()))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name").value("John"))
            .andExpect(jsonPath("$.email").value("john@example.com"));
    }
}
```

### Database Testing with Testcontainers

```java
@SpringBootTest
@Testcontainers
class UserRepositoryTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:14")
        .withDatabaseName("testdb")
        .withUsername("test")
        .withPassword("test");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private UserRepository userRepository;

    @Test
    void shouldSaveAndFindUser() {
        User user = new User("John", "john@example.com");
        User saved = userRepository.save(user);

        Optional<User> found = userRepository.findById(saved.getId());
        assertTrue(found.isPresent());
        assertEquals("John", found.get().getName());
    }
}
```

---

## 10) Test Coverage

### What to Measure

| Metric | Description | Target |
|--------|-------------|--------|
| **Line Coverage** | Lines executed | 70-80% |
| **Branch Coverage** | Decision paths | 70-80% |
| **Method Coverage** | Methods tested | 80%+ |
| **Mutation Coverage** | Tests catch mutations | Varies |

### Coverage Caveats

- **100% coverage ≠ bug-free**
- **Coverage doesn't measure test quality**
- **Some code doesn't need testing** (getters, generated code)
- **Focus on critical paths** rather than percentage

### JaCoCo Configuration (Maven)

```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.8</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

---

## 11) Test Data Management

### Test Fixtures

```java
class OrderTestFixtures {
    
    public static Order createValidOrder() {
        return Order.builder()
            .id(1L)
            .customerId(100L)
            .items(List.of(createDefaultItem()))
            .status(OrderStatus.PENDING)
            .build();
    }
    
    public static OrderItem createDefaultItem() {
        return new OrderItem("Product", 1, 10.0);
    }
    
    public static Customer createPremiumCustomer() {
        return new Customer("John", CustomerType.PREMIUM);
    }
}

// Usage in tests
@Test
void shouldCalculateTotal() {
    Order order = OrderTestFixtures.createValidOrder();
    // ...
}
```

### Builder Pattern for Tests

```java
public class TestUserBuilder {
    private String name = "John";
    private String email = "john@example.com";
    private int age = 30;
    
    public TestUserBuilder withName(String name) {
        this.name = name;
        return this;
    }
    
    public TestUserBuilder withEmail(String email) {
        this.email = email;
        return this;
    }
    
    public TestUserBuilder withAge(int age) {
        this.age = age;
        return this;
    }
    
    public User build() {
        return new User(name, email, age);
    }
}

// Usage
User user = new TestUserBuilder()
    .withName("Jane")
    .withAge(25)
    .build();
```

---

## 12) Assertions Libraries

### AssertJ (Recommended)

```java
import static org.assertj.core.api.Assertions.*;

// Basic
assertThat(name).isEqualTo("John");
assertThat(name).isNotNull().startsWith("J").endsWith("n");

// Numbers
assertThat(age).isPositive().isLessThan(100);

// Collections
assertThat(list).hasSize(3).contains("a", "b").doesNotContain("x");
assertThat(list).extracting("name").containsExactly("John", "Jane");

// Exceptions
assertThatThrownBy(() -> service.process(null))
    .isInstanceOf(IllegalArgumentException.class)
    .hasMessageContaining("null");

assertThatCode(() -> service.safeMethod())
    .doesNotThrowAnyException();

// Objects
assertThat(user)
    .hasFieldOrPropertyWithValue("name", "John")
    .hasFieldOrPropertyWithValue("age", 30);

// Soft assertions (all run, report all failures)
SoftAssertions.assertSoftly(soft -> {
    soft.assertThat(user.getName()).isEqualTo("John");
    soft.assertThat(user.getAge()).isEqualTo(30);
    soft.assertThat(user.getEmail()).contains("@");
});
```

### Hamcrest

```java
import static org.hamcrest.MatcherAssert.assertThat;
import static org.hamcrest.Matchers.*;

assertThat(name, is("John"));
assertThat(name, equalTo("John"));
assertThat(name, startsWith("J"));
assertThat(age, greaterThan(0));
assertThat(list, hasSize(3));
assertThat(list, contains("a", "b", "c"));
assertThat(list, hasItem("a"));
```

---

## 13) Asynchronous Testing

### Awaitility

```java
import static org.awaitility.Awaitility.*;

@Test
void shouldProcessAsync() {
    service.startAsyncProcess();
    
    await()
        .atMost(5, TimeUnit.SECONDS)
        .pollInterval(100, TimeUnit.MILLISECONDS)
        .until(() -> service.isComplete());
        
    // Or with AssertJ
    await()
        .atMost(Duration.ofSeconds(5))
        .untilAsserted(() -> 
            assertThat(service.getResult()).isEqualTo("done"));
}
```

### CompletableFuture Testing

```java
@Test
void shouldReturnFutureResult() throws Exception {
    CompletableFuture<String> future = service.getAsync();
    
    String result = future.get(5, TimeUnit.SECONDS);
    assertEquals("expected", result);
}

@Test
void shouldCompleteExceptionally() {
    CompletableFuture<String> future = service.getFailingAsync();
    
    assertThrows(ExecutionException.class, () -> 
        future.get(1, TimeUnit.SECONDS));
}
```

---

## 14) Interview Questions

### General Testing
1. What is the difference between unit and integration tests?
2. Explain the testing pyramid.
3. What makes a good unit test?
4. What is TDD (Test-Driven Development)?

### JUnit
5. Explain JUnit 5 lifecycle annotations.
6. What is `@ParameterizedTest`? Give an example.
7. How do you test exceptions in JUnit 5?
8. What is the difference between `@BeforeAll` and `@BeforeEach`?

### Mocking
9. What is the difference between mock and stub?
10. When would you use a spy vs a mock?
11. Explain argument matchers in Mockito.
12. How do you verify method calls with Mockito?
13. How do you mock static methods?

### Best Practices
14. What are common test anti-patterns?
15. How much code coverage is enough?
16. How do you test private methods?
17. What is flaky test? How do you prevent them?

### Integration Testing
18. How do you test database interactions?
19. What is Testcontainers? When would you use it?
20. How do you test REST endpoints?

---

## 15) Quick Reference

### JUnit 5 Annotations

| Annotation | Purpose |
|------------|---------|
| `@Test` | Mark test method |
| `@ParameterizedTest` | Parameterized test |
| `@RepeatedTest` | Repeat test N times |
| `@DisplayName` | Custom test name |
| `@Disabled` | Skip test |
| `@Nested` | Group tests |
| `@Tag` | Categorize tests |
| `@Timeout` | Fail if too slow |
| `@BeforeEach/All` | Setup |
| `@AfterEach/All` | Cleanup |
| `@ExtendWith` | Register extensions |

### Mockito Cheat Sheet

```java
// Create
mock(Class.class)
spy(realObject)
@Mock, @Spy, @InjectMocks

// Stub
when(mock.method()).thenReturn(value)
when(mock.method()).thenThrow(exception)
doReturn(value).when(mock).method()
doThrow(exception).when(mock).voidMethod()

// Matchers
any(), anyInt(), anyString(), anyList()
eq(value), isNull(), notNull()
argThat(predicate)

// Verify
verify(mock).method()
verify(mock, times(n)).method()
verify(mock, never()).method()
verifyNoInteractions(mock)
```

