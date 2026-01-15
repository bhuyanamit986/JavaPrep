# Spring / Spring Boot (Interview Guide)

Spring is the most popular Java enterprise framework. This guide covers core Spring concepts and Spring Boot, which is frequently tested in Java interviews.

## Definitions

- **IoC (Inversion of Control)**: the framework manages object creation and wiring.
- **DI (Dependency Injection)**: dependencies are provided to a class instead of created inside it.
- **Bean**: a managed object in the Spring application context.
- **ApplicationContext**: the container that holds beans and configuration.
- **Auto-configuration**: Spring Boot feature that configures beans based on classpath and properties.
- **Component scanning**: automatic discovery of classes annotated with `@Component` and friends.
- **Starter**: a curated dependency set that pulls in common libraries.
- **Transaction**: a unit of work that must fully succeed or fully fail.

## Illustrations

- **Container**: like a restaurant kitchen that prepares and wires ingredients for you.
- **Auto-configuration**: a "starter kit" that sets up defaults so you can start quickly.
- **DI**: swapping a power adapter rather than rewiring a device.

## Code Examples

```java
@org.springframework.web.bind.annotation.RestController
class HelloController {
    @org.springframework.web.bind.annotation.GetMapping("/hello")
    String hello() {
        return "hello";
    }
}
```

## Interview Questions

1. What is the difference between IoC and DI?
2. How does component scanning work?
3. What does Spring Boot auto-configuration do?
4. What is the lifecycle of a Spring bean?
5. How do transactions work in Spring?

---

## 1) Spring Core Concepts

### What is Spring?

Spring is a comprehensive framework that provides:
- **Dependency Injection (DI)**: Inversion of Control
- **Aspect-Oriented Programming (AOP)**: Cross-cutting concerns
- **Transaction Management**: Declarative transactions
- **MVC Framework**: Web applications
- **Data Access**: JDBC, ORM integration
- **Security**: Authentication and authorization

### Inversion of Control (IoC)

Traditional approach:
```java
// You create dependencies
public class UserService {
    private UserRepository repository = new UserRepositoryImpl();
}
```

IoC approach:
```java
// Container provides dependencies
public class UserService {
    private final UserRepository repository;  // Injected
    
    public UserService(UserRepository repository) {
        this.repository = repository;
    }
}
```

**Benefits:**
- Loose coupling
- Easier testing (mock dependencies)
- Configuration flexibility

---

## 2) Dependency Injection

### Types of Injection

**1. Constructor Injection (Recommended)**

```java
@Service
public class UserService {
    private final UserRepository userRepository;
    private final EmailService emailService;

    // @Autowired optional for single constructor (Spring 4.3+)
    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }
}
```

**Why preferred:**
- Immutable dependencies (`final`)
- Required dependencies obvious
- Easy to test
- Fail-fast (at startup)

**2. Setter Injection**

```java
@Service
public class UserService {
    private UserRepository userRepository;

    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

**Use for:** Optional dependencies.

**3. Field Injection (Avoid)**

```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;  // ❌ Hard to test
}
```

**Why avoid:**
- Can't make fields `final`
- Harder to test (need reflection)
- Hides dependencies

### @Autowired vs @Inject vs @Resource

| Annotation | Source | By | Qualifier |
|------------|--------|-----|-----------|
| `@Autowired` | Spring | Type | `@Qualifier` |
| `@Inject` | JSR-330 | Type | `@Named` |
| `@Resource` | JSR-250 | Name | name attribute |

```java
// @Autowired with qualifier
@Autowired
@Qualifier("mysqlRepository")
private UserRepository repository;

// @Resource by name
@Resource(name = "mysqlRepository")
private UserRepository repository;
```

---

## 3) Spring Beans

### Bean Definition

A bean is an object managed by the Spring container.

### Stereotype Annotations

| Annotation | Purpose |
|------------|---------|
| `@Component` | Generic component |
| `@Service` | Business logic layer |
| `@Repository` | Data access layer (+ exception translation) |
| `@Controller` | Web controller (MVC) |
| `@RestController` | REST API controller |
| `@Configuration` | Configuration class |

```java
@Service
public class UserService {
    // Automatically detected and registered as bean
}

@Repository
public class UserRepositoryImpl implements UserRepository {
    // Also handles persistence exceptions
}
```

### Bean Configuration

**1. Component Scanning (Annotation-based)**

```java
@Configuration
@ComponentScan("com.example")
public class AppConfig {
}
```

**2. Java Configuration**

```java
@Configuration
public class AppConfig {
    
    @Bean
    public UserService userService(UserRepository repository) {
        return new UserService(repository);
    }
    
    @Bean
    public UserRepository userRepository(DataSource dataSource) {
        return new JdbcUserRepository(dataSource);
    }
    
    @Bean(initMethod = "init", destroyMethod = "cleanup")
    public ExternalService externalService() {
        return new ExternalService();
    }
}
```

**3. XML Configuration (Legacy)**

```xml
<beans>
    <bean id="userService" class="com.example.UserService">
        <constructor-arg ref="userRepository"/>
    </bean>
</beans>
```

---

## 4) Bean Scopes

| Scope | Description | Default |
|-------|-------------|---------|
| `singleton` | One instance per container | ✓ |
| `prototype` | New instance each request | |
| `request` | One per HTTP request | Web |
| `session` | One per HTTP session | Web |
| `application` | One per ServletContext | Web |
| `websocket` | One per WebSocket | Web |

```java
@Component
@Scope("prototype")
public class PrototypeBean {
    // New instance each time
}

@Component
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class RequestScopedBean {
    // One per HTTP request
}
```

### Singleton vs Prototype

```java
@Service
public class SingletonService {
    // One instance - be careful with state!
}

@Service
@Scope("prototype")
public class PrototypeService {
    // New instance per injection - but injected once in singleton!
}

// Problem: Prototype injected into singleton
@Service
public class SingletonService {
    @Autowired
    private PrototypeService prototype;  // Always same instance!
}

// Solution: Provider or ObjectFactory
@Service
public class SingletonService {
    @Autowired
    private Provider<PrototypeService> prototypeProvider;
    
    public void doWork() {
        PrototypeService fresh = prototypeProvider.get();  // New each time
    }
}
```

---

## 5) Bean Lifecycle

### Lifecycle Callbacks

```java
@Component
public class LifecycleBean implements InitializingBean, DisposableBean {
    
    // 1. Constructor
    public LifecycleBean() {
        System.out.println("1. Constructor");
    }
    
    // 2. Dependencies injected
    @Autowired
    public void setDependency(SomeDependency dep) {
        System.out.println("2. Setter injection");
    }
    
    // 3. @PostConstruct
    @PostConstruct
    public void postConstruct() {
        System.out.println("3. @PostConstruct");
    }
    
    // 4. InitializingBean.afterPropertiesSet()
    @Override
    public void afterPropertiesSet() {
        System.out.println("4. afterPropertiesSet");
    }
    
    // 5. Custom init method (if defined)
    
    // Bean is ready for use...
    
    // 6. @PreDestroy (on shutdown)
    @PreDestroy
    public void preDestroy() {
        System.out.println("6. @PreDestroy");
    }
    
    // 7. DisposableBean.destroy()
    @Override
    public void destroy() {
        System.out.println("7. destroy");
    }
    
    // 8. Custom destroy method (if defined)
}
```

### Order

```
1. Instantiation (constructor)
2. Populate properties (DI)
3. BeanNameAware.setBeanName()
4. BeanFactoryAware.setBeanFactory()
5. ApplicationContextAware.setApplicationContext()
6. BeanPostProcessor.postProcessBeforeInitialization()
7. @PostConstruct
8. InitializingBean.afterPropertiesSet()
9. Custom init-method
10. BeanPostProcessor.postProcessAfterInitialization()
--- Bean is ready ---
11. @PreDestroy
12. DisposableBean.destroy()
13. Custom destroy-method
```

---

## 6) Spring Boot Basics

### What is Spring Boot?

Spring Boot is an opinionated framework that simplifies Spring development:
- **Auto-configuration**: Automatic bean configuration
- **Starter dependencies**: Curated dependency sets
- **Embedded servers**: No external server needed
- **Production-ready**: Actuator, metrics, health checks

### Main Application

```java
@SpringBootApplication  // = @Configuration + @EnableAutoConfiguration + @ComponentScan
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### Common Starters

| Starter | Purpose |
|---------|---------|
| `spring-boot-starter-web` | Web applications, REST APIs |
| `spring-boot-starter-data-jpa` | JPA with Hibernate |
| `spring-boot-starter-security` | Security |
| `spring-boot-starter-test` | Testing |
| `spring-boot-starter-actuator` | Production monitoring |
| `spring-boot-starter-validation` | Bean validation |

---

## 7) REST Controllers

### Basic Controller

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping
    public List<User> getAllUsers() {
        return userService.findAll();
    }

    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        return userService.findById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public User createUser(@Valid @RequestBody CreateUserRequest request) {
        return userService.create(request);
    }

    @PutMapping("/{id}")
    public User updateUser(@PathVariable Long id, @Valid @RequestBody UpdateUserRequest request) {
        return userService.update(id, request);
    }

    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void deleteUser(@PathVariable Long id) {
        userService.delete(id);
    }
}
```

### Request Mapping Annotations

| Annotation | HTTP Method |
|------------|-------------|
| `@GetMapping` | GET |
| `@PostMapping` | POST |
| `@PutMapping` | PUT |
| `@DeleteMapping` | DELETE |
| `@PatchMapping` | PATCH |

### Parameter Annotations

```java
@GetMapping("/search")
public List<User> search(
    @RequestParam String name,                           // ?name=John
    @RequestParam(required = false) String email,        // Optional
    @RequestParam(defaultValue = "0") int page,          // With default
    @RequestHeader("Authorization") String auth,         // Header
    @CookieValue("sessionId") String sessionId           // Cookie
) {
    return userService.search(name, email, page);
}

@GetMapping("/{id}")
public User getUser(
    @PathVariable Long id,                               // /users/123
    @PathVariable("id") Long userId                      // Custom name
) {
    return userService.findById(id);
}
```

---

## 8) Request Validation

```java
// DTO with validation
public class CreateUserRequest {
    
    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 100)
    private String name;
    
    @NotNull
    @Email(message = "Invalid email format")
    private String email;
    
    @Min(0)
    @Max(150)
    private Integer age;
    
    @Pattern(regexp = "^\\+?[1-9]\\d{1,14}$")
    private String phone;
    
    // Getters/setters
}

// Controller
@PostMapping
public User createUser(@Valid @RequestBody CreateUserRequest request) {
    // Validation happens automatically
    return userService.create(request);
}

// Custom exception handler
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public Map<String, String> handleValidationErrors(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error ->
            errors.put(error.getField(), error.getDefaultMessage())
        );
        return errors;
    }
    
    @ExceptionHandler(EntityNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleNotFound(EntityNotFoundException ex) {
        return new ErrorResponse("NOT_FOUND", ex.getMessage());
    }
}
```

### Common Validation Annotations

| Annotation | Validates |
|------------|-----------|
| `@NotNull` | Not null |
| `@NotEmpty` | Not null/empty (collections, strings) |
| `@NotBlank` | Not null/empty/whitespace (strings) |
| `@Size(min, max)` | Size constraints |
| `@Min`, `@Max` | Numeric bounds |
| `@Email` | Email format |
| `@Pattern` | Regex pattern |
| `@Past`, `@Future` | Date constraints |
| `@Positive` | Positive number |

---

## 9) Spring Data JPA

### Entity

```java
@Entity
@Table(name = "users")
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, length = 100)
    private String name;
    
    @Column(unique = true, nullable = false)
    private String email;
    
    @Enumerated(EnumType.STRING)
    private UserStatus status;
    
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Order> orders = new ArrayList<>();
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "department_id")
    private Department department;
    
    @CreatedDate
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    private LocalDateTime updatedAt;
    
    // Getters/setters
}
```

### Repository

```java
public interface UserRepository extends JpaRepository<User, Long> {
    
    // Derived query methods
    Optional<User> findByEmail(String email);
    List<User> findByNameContainingIgnoreCase(String name);
    List<User> findByStatusAndDepartmentId(UserStatus status, Long deptId);
    boolean existsByEmail(String email);
    long countByStatus(UserStatus status);
    
    // Custom query with @Query
    @Query("SELECT u FROM User u WHERE u.status = :status ORDER BY u.createdAt DESC")
    List<User> findActiveUsersOrderByDate(@Param("status") UserStatus status);
    
    // Native query
    @Query(value = "SELECT * FROM users WHERE email LIKE %:domain", nativeQuery = true)
    List<User> findByEmailDomain(@Param("domain") String domain);
    
    // Modifying query
    @Modifying
    @Query("UPDATE User u SET u.status = :status WHERE u.id = :id")
    int updateStatus(@Param("id") Long id, @Param("status") UserStatus status);
    
    // Pagination
    Page<User> findByStatus(UserStatus status, Pageable pageable);
    
    // Projections
    @Query("SELECT u.name as name, u.email as email FROM User u")
    List<UserSummary> findAllSummaries();
}

// Projection interface
public interface UserSummary {
    String getName();
    String getEmail();
}
```

### Service Layer

```java
@Service
@Transactional(readOnly = true)
public class UserService {
    
    private final UserRepository userRepository;
    
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    public List<User> findAll() {
        return userRepository.findAll();
    }
    
    public Optional<User> findById(Long id) {
        return userRepository.findById(id);
    }
    
    public Page<User> findAll(Pageable pageable) {
        return userRepository.findAll(pageable);
    }
    
    @Transactional
    public User create(CreateUserRequest request) {
        if (userRepository.existsByEmail(request.getEmail())) {
            throw new DuplicateEmailException(request.getEmail());
        }
        User user = new User();
        user.setName(request.getName());
        user.setEmail(request.getEmail());
        user.setStatus(UserStatus.ACTIVE);
        return userRepository.save(user);
    }
    
    @Transactional
    public void delete(Long id) {
        userRepository.deleteById(id);
    }
}
```

---

## 10) Transactions

### @Transactional

```java
@Service
public class OrderService {
    
    // Read-only (optimization)
    @Transactional(readOnly = true)
    public Order findById(Long id) {
        return orderRepository.findById(id).orElseThrow();
    }
    
    // Default: REQUIRED propagation, RuntimeException rollback
    @Transactional
    public Order createOrder(CreateOrderRequest request) {
        Order order = new Order();
        // ... create order
        orderRepository.save(order);
        inventoryService.decreaseStock(order.getItems());  // Same transaction
        return order;
    }
    
    // Custom settings
    @Transactional(
        propagation = Propagation.REQUIRED,
        isolation = Isolation.READ_COMMITTED,
        timeout = 30,
        rollbackFor = BusinessException.class,
        noRollbackFor = NotificationException.class
    )
    public void processOrder(Long orderId) {
        // ...
    }
}
```

### Propagation Types

| Type | Behavior |
|------|----------|
| `REQUIRED` (default) | Join existing or create new |
| `REQUIRES_NEW` | Always create new (suspend existing) |
| `NESTED` | Nested transaction (savepoint) |
| `SUPPORTS` | Use existing, or run without |
| `NOT_SUPPORTED` | Suspend existing, run without |
| `MANDATORY` | Must have existing (exception if none) |
| `NEVER` | Must not have transaction (exception if exists) |

### Transaction Pitfalls

```java
@Service
public class UserService {
    
    // ❌ Self-invocation bypasses proxy!
    @Transactional
    public void updateUser(Long id) {
        // ...
        this.sendNotification();  // @Transactional ignored!
    }
    
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void sendNotification() {
        // This won't run in new transaction when called internally
    }
    
    // ✅ Solution: Inject self or use separate service
    @Autowired
    private UserService self;  // Or use ApplicationContext
    
    @Transactional
    public void updateUserFixed(Long id) {
        // ...
        self.sendNotification();  // Goes through proxy
    }
}

// ❌ Checked exceptions don't trigger rollback by default
@Transactional
public void process() throws IOException {
    // IOException won't rollback!
}

// ✅ Specify rollback for checked exceptions
@Transactional(rollbackFor = IOException.class)
public void processFixed() throws IOException {
    // Will rollback on IOException
}
```

---

## 11) Configuration Properties

### application.properties

```properties
# Server
server.port=8080
server.servlet.context-path=/api

# Database
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=secret
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# JPA
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

# Logging
logging.level.root=INFO
logging.level.com.example=DEBUG
logging.level.org.hibernate.SQL=DEBUG

# Custom properties
app.name=My Application
app.feature.enabled=true
app.limits.max-users=1000
```

### application.yml

```yaml
server:
  port: 8080
  servlet:
    context-path: /api

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: root
    password: secret
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: true

logging:
  level:
    root: INFO
    com.example: DEBUG

app:
  name: My Application
  feature:
    enabled: true
  limits:
    max-users: 1000
```

### Type-Safe Configuration

```java
@ConfigurationProperties(prefix = "app")
@Validated
public class AppProperties {
    
    @NotBlank
    private String name;
    
    private Feature feature = new Feature();
    private Limits limits = new Limits();
    
    public static class Feature {
        private boolean enabled = false;
        // Getters/setters
    }
    
    public static class Limits {
        @Min(1)
        private int maxUsers = 100;
        // Getters/setters
    }
    
    // Getters/setters
}

// Enable configuration properties
@Configuration
@EnableConfigurationProperties(AppProperties.class)
public class AppConfig {
}

// Usage
@Service
public class MyService {
    private final AppProperties properties;
    
    public MyService(AppProperties properties) {
        this.properties = properties;
    }
    
    public void doSomething() {
        if (properties.getFeature().isEnabled()) {
            // ...
        }
    }
}
```

### Profiles

```yaml
# application.yml (common settings)
spring:
  profiles:
    active: dev

---
# application-dev.yml
spring:
  config:
    activate:
      on-profile: dev
  datasource:
    url: jdbc:h2:mem:devdb

---
# application-prod.yml
spring:
  config:
    activate:
      on-profile: prod
  datasource:
    url: jdbc:mysql://prod-server:3306/proddb
```

```bash
# Activate profile
java -jar app.jar --spring.profiles.active=prod

# Environment variable
SPRING_PROFILES_ACTIVE=prod java -jar app.jar
```

---

## 12) Exception Handling

### Global Exception Handler

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    private static final Logger log = LoggerFactory.getLogger(GlobalExceptionHandler.class);
    
    @ExceptionHandler(EntityNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleNotFound(EntityNotFoundException ex) {
        return new ErrorResponse("NOT_FOUND", ex.getMessage());
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ValidationErrorResponse handleValidation(MethodArgumentNotValidException ex) {
        List<FieldError> errors = ex.getBindingResult().getFieldErrors().stream()
            .map(e -> new FieldError(e.getField(), e.getDefaultMessage()))
            .collect(Collectors.toList());
        return new ValidationErrorResponse("VALIDATION_ERROR", errors);
    }
    
    @ExceptionHandler(DataIntegrityViolationException.class)
    @ResponseStatus(HttpStatus.CONFLICT)
    public ErrorResponse handleDataIntegrity(DataIntegrityViolationException ex) {
        return new ErrorResponse("CONFLICT", "Data integrity violation");
    }
    
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleGeneral(Exception ex) {
        log.error("Unexpected error", ex);
        return new ErrorResponse("INTERNAL_ERROR", "An unexpected error occurred");
    }
}

// Error response DTOs
public record ErrorResponse(String code, String message) {}

public record ValidationErrorResponse(String code, List<FieldError> errors) {}

public record FieldError(String field, String message) {}
```

---

## 13) Spring Security Basics

### Configuration

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .requestMatchers("/api/**").authenticated()
                .anyRequest().permitAll()
            )
            .httpBasic(Customizer.withDefaults())
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .build();
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    
    @Bean
    public UserDetailsService userDetailsService() {
        UserDetails user = User.builder()
            .username("user")
            .password(passwordEncoder().encode("password"))
            .roles("USER")
            .build();
        
        UserDetails admin = User.builder()
            .username("admin")
            .password(passwordEncoder().encode("admin"))
            .roles("ADMIN", "USER")
            .build();
        
        return new InMemoryUserDetailsManager(user, admin);
    }
}
```

### Method Security

```java
@Configuration
@EnableMethodSecurity
public class MethodSecurityConfig {
}

@Service
public class UserService {
    
    @PreAuthorize("hasRole('ADMIN')")
    public void deleteUser(Long id) {
        // Only ADMIN can call this
    }
    
    @PreAuthorize("#userId == authentication.principal.id")
    public User getProfile(Long userId) {
        // Only user can access their own profile
    }
    
    @PostAuthorize("returnObject.owner == authentication.name")
    public Document getDocument(Long id) {
        // Check after method execution
    }
}
```

---

## 14) Actuator (Production Monitoring)

### Enable Actuator

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### Configuration

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  endpoint:
    health:
      show-details: always
  info:
    env:
      enabled: true

info:
  app:
    name: My Application
    version: 1.0.0
```

### Common Endpoints

| Endpoint | Purpose |
|----------|---------|
| `/actuator/health` | Health status |
| `/actuator/info` | Application info |
| `/actuator/metrics` | Metrics |
| `/actuator/env` | Environment properties |
| `/actuator/beans` | All beans |
| `/actuator/mappings` | Request mappings |
| `/actuator/loggers` | Logger configuration |

### Custom Health Indicator

```java
@Component
public class DatabaseHealthIndicator implements HealthIndicator {
    
    private final DataSource dataSource;
    
    public DatabaseHealthIndicator(DataSource dataSource) {
        this.dataSource = dataSource;
    }
    
    @Override
    public Health health() {
        try (Connection conn = dataSource.getConnection()) {
            if (conn.isValid(1)) {
                return Health.up()
                    .withDetail("database", "Available")
                    .build();
            }
        } catch (SQLException e) {
            return Health.down()
                .withDetail("database", "Unavailable")
                .withException(e)
                .build();
        }
        return Health.down().build();
    }
}
```

---

## 15) Testing Spring Applications

### Unit Test (Service Layer)

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    void shouldCreateUser() {
        // Given
        CreateUserRequest request = new CreateUserRequest("John", "john@example.com");
        when(userRepository.existsByEmail(anyString())).thenReturn(false);
        when(userRepository.save(any(User.class))).thenAnswer(inv -> {
            User user = inv.getArgument(0);
            user.setId(1L);
            return user;
        });
        
        // When
        User result = userService.create(request);
        
        // Then
        assertThat(result.getId()).isEqualTo(1L);
        assertThat(result.getName()).isEqualTo("John");
        verify(userRepository).save(any(User.class));
    }
}
```

### Integration Test (Controller)

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
        String requestBody = """
            {
                "name": "John",
                "email": "john@example.com"
            }
            """;
        
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(requestBody))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.name").value("John"))
            .andExpect(jsonPath("$.email").value("john@example.com"));
        
        assertThat(userRepository.count()).isEqualTo(1);
    }
    
    @Test
    void shouldReturn404WhenUserNotFound() throws Exception {
        mockMvc.perform(get("/api/users/999"))
            .andExpect(status().isNotFound());
    }
}
```

### WebMvcTest (Controller Only)

```java
@WebMvcTest(UserController.class)
class UserControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private UserService userService;
    
    @Test
    void shouldReturnUser() throws Exception {
        User user = new User(1L, "John", "john@example.com");
        when(userService.findById(1L)).thenReturn(Optional.of(user));
        
        mockMvc.perform(get("/api/users/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name").value("John"));
    }
}
```

### DataJpaTest (Repository)

```java
@DataJpaTest
class UserRepositoryTest {
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    void shouldFindByEmail() {
        User user = new User();
        user.setName("John");
        user.setEmail("john@example.com");
        userRepository.save(user);
        
        Optional<User> found = userRepository.findByEmail("john@example.com");
        
        assertThat(found).isPresent();
        assertThat(found.get().getName()).isEqualTo("John");
    }
}
```

---

## 16) Interview Questions

### Spring Core
1. What is Dependency Injection? What are its benefits?
2. What is the difference between BeanFactory and ApplicationContext?
3. Explain bean scopes in Spring.
4. What is the bean lifecycle in Spring?
5. What is the difference between constructor and setter injection?
6. Why is field injection not recommended?

### Spring Boot
7. What is Spring Boot? How is it different from Spring?
8. What is auto-configuration?
9. What are Spring Boot starters?
10. How do you externalize configuration?
11. What is the difference between @Controller and @RestController?

### Data & Transactions
12. How does Spring Data JPA work?
13. Explain @Transactional annotation.
14. What are transaction propagation levels?
15. Why doesn't @Transactional work on self-invocation?

### Security
16. How does Spring Security work?
17. What is the difference between authentication and authorization?
18. How do you secure REST APIs?

### Testing
19. How do you test Spring applications?
20. What is the difference between @SpringBootTest and @WebMvcTest?

---

## 17) Quick Reference

### Common Annotations

| Annotation | Purpose |
|------------|---------|
| `@SpringBootApplication` | Main class |
| `@RestController` | REST controller |
| `@Service` | Service layer |
| `@Repository` | Data access layer |
| `@Autowired` | Dependency injection |
| `@Transactional` | Transaction management |
| `@Valid` | Enable validation |
| `@ExceptionHandler` | Handle exceptions |
| `@Configuration` | Configuration class |
| `@Bean` | Define bean |
| `@Value` | Inject property value |
| `@Profile` | Environment-specific bean |

