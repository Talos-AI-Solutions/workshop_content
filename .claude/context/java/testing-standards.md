---
description: Comprehensive Java testing standards with JUnit 5, Mockito, AssertJ, Testcontainers, JaCoCo, and PITest
globs: "**/*Test.java, **/*IT.java, **/test/**/*.java"
alwaysApply: false
---

# Java Testing Standards and Implementation Patterns

Comprehensive testing patterns for Java applications with JUnit 5, Mockito, AssertJ, Testcontainers, coverage analysis, and mutation testing.

## Prerequisites

- JUnit 5.9+
- Mockito 5.0+
- AssertJ 3.24+
- Testcontainers 1.19+
- JaCoCo 0.8.10+
- PITest 1.14+

## Core Principle

**Write comprehensive, fast, maintainable tests with 90%+ code coverage and 80%+ mutation coverage using modern testing frameworks and practices.**

## JUnit 5 Patterns

### Good: Parameterized Tests

```java
@ParameterizedTest
@ValueSource(strings = {"", "  ", "\t", "\n"})
@DisplayName("Should reject blank usernames")
void shouldRejectBlankUsernames(String username) {
    var request = new CreateUserRequest(username, "test@example.com", "password", 25);

    assertThatThrownBy(() -> userService.createUser(request))
        .isInstanceOf(ValidationException.class)
        .hasMessageContaining("username");
}

@ParameterizedTest
@MethodSource("provideInvalidEmails")
@DisplayName("Should reject invalid email formats")
void shouldRejectInvalidEmails(String email, String expectedError) {
    var request = new CreateUserRequest("testuser", email, "password", 25);

    assertThatThrownBy(() -> userService.createUser(request))
        .isInstanceOf(ValidationException.class)
        .hasMessageContaining(expectedError);
}

private static Stream<Arguments> provideInvalidEmails() {
    return Stream.of(
        Arguments.of("", "Email is required"),
        Arguments.of("notanemail", "Invalid email format"),
        Arguments.of("missing@domain", "Invalid email format"),
        Arguments.of("@example.com", "Invalid email format")
    );
}
```

### Good: Nested Tests for Logical Grouping

```java
@DisplayName("User Service Tests")
class UserServiceTest {

    private UserService userService;
    private UserRepository userRepository;
    private PasswordEncoder passwordEncoder;

    @BeforeEach
    void setUp() {
        userRepository = mock(UserRepository.class);
        passwordEncoder = mock(PasswordEncoder.class);
        userService = new UserService(userRepository, passwordEncoder);
    }

    @Nested
    @DisplayName("Create User")
    class CreateUser {

        @Test
        @DisplayName("Should create user with valid data")
        void shouldCreateUserWithValidData() {
            var request = new CreateUserRequest(
                "testuser",
                "test@example.com",
                "password123",
                25
            );

            when(userRepository.findByUsername(request.username()))
                .thenReturn(Optional.empty());
            when(userRepository.findByEmail(request.email()))
                .thenReturn(Optional.empty());
            when(passwordEncoder.encode(request.password()))
                .thenReturn("hashed_password");

            var user = new User("testuser", "test@example.com", "hashed_password");
            when(userRepository.save(any(User.class))).thenReturn(user);

            var result = userService.createUser(request);

            assertThat(result)
                .isNotNull()
                .extracting(User::getUsername, User::getEmail)
                .containsExactly("testuser", "test@example.com");
        }

        @Test
        @DisplayName("Should throw exception when username exists")
        void shouldThrowExceptionWhenUsernameExists() {
            var request = new CreateUserRequest(
                "existing",
                "test@example.com",
                "password123",
                25
            );

            var existingUser = new User("existing", "other@example.com", "hash");
            when(userRepository.findByUsername(request.username()))
                .thenReturn(Optional.of(existingUser));

            assertThatThrownBy(() -> userService.createUser(request))
                .isInstanceOf(DuplicateUsernameException.class)
                .hasMessage("Username already exists: existing");

            verify(userRepository, never()).save(any());
        }

        @Test
        @DisplayName("Should throw exception when email exists")
        void shouldThrowExceptionWhenEmailExists() {
            var request = new CreateUserRequest(
                "testuser",
                "existing@example.com",
                "password123",
                25
            );

            when(userRepository.findByUsername(request.username()))
                .thenReturn(Optional.empty());

            var existingUser = new User("other", "existing@example.com", "hash");
            when(userRepository.findByEmail(request.email()))
                .thenReturn(Optional.of(existingUser));

            assertThatThrownBy(() -> userService.createUser(request))
                .isInstanceOf(DuplicateEmailException.class)
                .hasMessage("Email already exists: existing@example.com");

            verify(userRepository, never()).save(any());
        }
    }

    @Nested
    @DisplayName("Update User")
    class UpdateUser {

        @Test
        @DisplayName("Should update user when found")
        void shouldUpdateUserWhenFound() {
            var user = new User("testuser", "test@example.com", "hash");
            var request = new UpdateUserRequest("newusername", null, 30);

            when(userRepository.findById(1L)).thenReturn(Optional.of(user));

            var result = userService.updateUser(1L, request);

            assertThat(result)
                .isPresent()
                .get()
                .extracting(User::getUsername, User::getAge)
                .containsExactly("newusername", 30);

            assertThat(user.getEmail()).isEqualTo("test@example.com");
        }

        @Test
        @DisplayName("Should return empty when user not found")
        void shouldReturnEmptyWhenUserNotFound() {
            var request = new UpdateUserRequest("newusername", null, 30);

            when(userRepository.findById(999L)).thenReturn(Optional.empty());

            var result = userService.updateUser(999L, request);

            assertThat(result).isEmpty();
        }
    }
}
```

### Good: Test Lifecycle and Setup

```java
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class DatabaseTest {

    private DataSource dataSource;

    @BeforeAll
    void setUpAll() {
        // Expensive setup once for all tests
        dataSource = createTestDataSource();
    }

    @BeforeEach
    void setUp() {
        // Reset state before each test
        cleanDatabase();
    }

    @AfterEach
    void tearDown() {
        // Cleanup after each test
        rollbackTransactions();
    }

    @AfterAll
    void tearDownAll() {
        // Cleanup after all tests
        closeDataSource();
    }

    @Test
    void testDatabaseOperation() {
        // Test uses shared dataSource
    }
}
```

## Mockito Patterns

### Good: Mocking External Dependencies

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock
    private OrderRepository orderRepository;

    @Mock
    private CustomerRepository customerRepository;

    @Mock
    private EmailService emailService;

    @InjectMocks
    private OrderService orderService;

    @Captor
    private ArgumentCaptor<Order> orderCaptor;

    @Test
    void shouldCreateOrderAndSendEmail() {
        var customer = new Customer(1L, "John Doe", "john@example.com");
        var request = new CreateOrderRequest(1L, List.of(
            new OrderItem("PROD-1", 2, new BigDecimal("10.00"))
        ));

        when(customerRepository.findById(1L))
            .thenReturn(Optional.of(customer));
        when(orderRepository.save(any(Order.class)))
            .thenAnswer(invocation -> {
                Order order = invocation.getArgument(0);
                order.setId(100L);
                return order;
            });

        var result = orderService.createOrder(request);

        assertThat(result.getId()).isEqualTo(100L);
        assertThat(result.getCustomer()).isEqualTo(customer);

        verify(orderRepository).save(orderCaptor.capture());
        var savedOrder = orderCaptor.getValue();
        assertThat(savedOrder.getItems()).hasSize(1);
        assertThat(savedOrder.getTotal()).isEqualTo(new BigDecimal("20.00"));

        verify(emailService).sendOrderConfirmation(
            eq("john@example.com"),
            eq(result)
        );
    }

    @Test
    void shouldThrowExceptionWhenCustomerNotFound() {
        var request = new CreateOrderRequest(999L, List.of());

        when(customerRepository.findById(999L))
            .thenReturn(Optional.empty());

        assertThatThrownBy(() -> orderService.createOrder(request))
            .isInstanceOf(CustomerNotFoundException.class)
            .hasMessage("Customer not found: 999");

        verify(orderRepository, never()).save(any());
        verify(emailService, never()).sendOrderConfirmation(any(), any());
    }
}
```

### Good: Argument Matchers

```java
@Test
void shouldHandleComplexArguments() {
    var service = new PaymentService(paymentGateway, auditLog);

    when(paymentGateway.processPayment(
        argThat(payment ->
            payment.getAmount().compareTo(BigDecimal.ZERO) > 0 &&
            payment.getCurrency().equals("USD")
        ),
        anyString()
    )).thenReturn(new PaymentResult(true, "SUCCESS"));

    var result = service.charge(
        new Payment(new BigDecimal("100.00"), "USD"),
        "card-123"
    );

    assertThat(result.isSuccess()).isTrue();
}
```

### Bad: Over-mocking

```java
// DON'T: Mocking value objects and simple classes
@Mock
private BigDecimal amount;  // BAD! Use real BigDecimal

@Mock
private LocalDateTime timestamp;  // BAD! Use real LocalDateTime

@Mock
private String email;  // BAD! Use real String

// DO: Only mock complex dependencies
@Mock
private EmailService emailService;  // GOOD! External service

@Mock
private PaymentGateway paymentGateway;  // GOOD! External system

@Mock
private UserRepository userRepository;  // GOOD! Database access
```

## AssertJ Patterns

### Good: Fluent Assertions

```java
@Test
void shouldCreateCompleteUserProfile() {
    var user = userService.createUser(validRequest);

    assertThat(user)
        .isNotNull()
        .extracting(
            User::getId,
            User::getUsername,
            User::getEmail,
            User::getActive
        )
        .containsExactly(
            1L,
            "testuser",
            "test@example.com",
            true
        );

    assertThat(user.getCreatedAt())
        .isNotNull()
        .isBefore(LocalDateTime.now().plusSeconds(1))
        .isAfter(LocalDateTime.now().minusSeconds(10));
}

@Test
void shouldReturnFilteredUsers() {
    var users = userService.findActiveUsers();

    assertThat(users)
        .isNotEmpty()
        .hasSize(3)
        .extracting(User::getUsername)
        .containsExactlyInAnyOrder("alice", "bob", "charlie")
        .doesNotContain("inactive_user");

    assertThat(users)
        .allSatisfy(user -> {
            assertThat(user.getActive()).isTrue();
            assertThat(user.getEmail()).contains("@");
        });
}
```

### Good: Collection Assertions

```java
@Test
void shouldGroupOrdersByStatus() {
    var orders = List.of(
        new Order(1L, OrderStatus.PENDING),
        new Order(2L, OrderStatus.COMPLETED),
        new Order(3L, OrderStatus.PENDING),
        new Order(4L, OrderStatus.CANCELLED)
    );

    var grouped = orderService.groupByStatus(orders);

    assertThat(grouped)
        .containsKeys(OrderStatus.PENDING, OrderStatus.COMPLETED, OrderStatus.CANCELLED)
        .hasEntrySatisfying(OrderStatus.PENDING,
            pendingOrders -> assertThat(pendingOrders).hasSize(2)
        )
        .hasEntrySatisfying(OrderStatus.COMPLETED,
            completedOrders -> assertThat(completedOrders).hasSize(1)
        );
}
```

### Good: Exception Assertions

```java
@Test
void shouldThrowDetailedException() {
    var invalidRequest = new CreateOrderRequest(-1L, List.of());

    assertThatThrownBy(() -> orderService.createOrder(invalidRequest))
        .isInstanceOf(InvalidOrderException.class)
        .hasMessage("Customer ID must be positive")
        .hasNoCause();

    assertThatExceptionOfType(InvalidOrderException.class)
        .isThrownBy(() -> orderService.createOrder(invalidRequest))
        .withMessage("Customer ID must be positive")
        .satisfies(ex -> {
            assertThat(ex.getErrorCode()).isEqualTo("INVALID_CUSTOMER_ID");
            assertThat(ex.getTimestamp()).isNotNull();
        });
}
```

## Testcontainers Patterns

### Good: PostgreSQL Integration Test

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@Testcontainers
class UserRepositoryIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15-alpine")
        .withDatabaseName("testdb")
        .withUsername("test")
        .withPassword("test")
        .withReuse(true);  // Reuse container across tests for speed

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private UserRepository userRepository;

    @Test
    void shouldPersistAndRetrieveUser() {
        var user = new User("testuser", "test@example.com", "hash");
        user.setAge(25);

        var saved = userRepository.save(user);
        assertThat(saved.getId()).isNotNull();

        var retrieved = userRepository.findById(saved.getId());
        assertThat(retrieved)
            .isPresent()
            .get()
            .usingRecursiveComparison()
            .ignoringFields("createdAt", "updatedAt")
            .isEqualTo(user);
    }

    @Test
    void shouldFindUserByUsername() {
        var user = new User("searchuser", "search@example.com", "hash");
        userRepository.save(user);

        var found = userRepository.findByUsername("searchuser");

        assertThat(found)
            .isPresent()
            .get()
            .extracting(User::getUsername, User::getEmail)
            .containsExactly("searchuser", "search@example.com");
    }

    @Test
    void shouldEnforceUniqueConstraints() {
        var user1 = new User("duplicate", "first@example.com", "hash");
        userRepository.save(user1);

        var user2 = new User("duplicate", "second@example.com", "hash");

        assertThatThrownBy(() -> userRepository.save(user2))
            .isInstanceOf(DataIntegrityViolationException.class);
    }
}
```

### Good: Multiple Containers with Network

```java
@SpringBootTest
@Testcontainers
class FullStackIntegrationTest {

    @Container
    static Network network = Network.newNetwork();

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15-alpine")
        .withNetwork(network)
        .withNetworkAliases("postgres");

    @Container
    static GenericContainer<?> redis = new GenericContainer<>("redis:7-alpine")
        .withNetwork(network)
        .withNetworkAliases("redis")
        .withExposedPorts(6379);

    @Container
    static KafkaContainer kafka = new KafkaContainer(DockerImageName.parse("confluentinc/cp-kafka:7.4.0"))
        .withNetwork(network);

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);

        registry.add("spring.redis.host", redis::getHost);
        registry.add("spring.redis.port", redis::getFirstMappedPort);

        registry.add("spring.kafka.bootstrap-servers", kafka::getBootstrapServers);
    }

    @Test
    void shouldIntegrateAllServices() {
        // Test with real database, cache, and message queue
    }
}
```

## Test Coverage with JaCoCo

### Good: Gradle Configuration

```groovy
plugins {
    id 'jacoco'
}

jacoco {
    toolVersion = "0.8.10"
}

jacocoTestReport {
    dependsOn test
    reports {
        xml.required = true
        html.required = true
        csv.required = false
    }

    afterEvaluate {
        classDirectories.setFrom(files(classDirectories.files.collect {
            fileTree(dir: it, exclude: [
                '**/config/**',
                '**/dto/**',
                '**/entity/**',
                '**/*Application.class'
            ])
        }))
    }
}

jacocoTestCoverageVerification {
    violationRules {
        rule {
            limit {
                minimum = 0.90  // 90% instruction coverage
                counter = 'INSTRUCTION'
            }
        }
        rule {
            limit {
                minimum = 0.85  // 85% branch coverage
                counter = 'BRANCH'
            }
        }
    }
}

test {
    useJUnitPlatform()
    finalizedBy jacocoTestReport
}

check {
    dependsOn jacocoTestCoverageVerification
}
```

## Mutation Testing with PITest

### Good: Gradle Configuration

```groovy
plugins {
    id 'info.solidsoft.pitest' version '1.14.0'
}

pitest {
    targetClasses = ['com.example.service.*', 'com.example.controller.*']
    excludedClasses = ['com.example.config.*', '**.*Application']
    threads = 4
    outputFormats = ['HTML', 'XML']
    timestampedReports = false
    mutationThreshold = 80  // 80% mutation coverage required
    coverageThreshold = 90  // 90% line coverage required
}
```

### Understanding Mutation Testing

```java
// Original code
public boolean isAdult(int age) {
    return age >= 18;  // PITest will mutate to: age > 18, age <= 18, etc.
}

// Weak test (won't catch all mutations)
@Test
void shouldIdentifyAdult() {
    assertThat(calculator.isAdult(25)).isTrue();  // Misses boundary!
}

// Strong test (catches mutations)
@Test
void shouldIdentifyAdults() {
    assertThat(calculator.isAdult(18)).isTrue();   // Boundary
    assertThat(calculator.isAdult(17)).isFalse();  // Just below boundary
    assertThat(calculator.isAdult(25)).isTrue();   // Well above
}
```

## Performance Testing

### Good: JMH Benchmarks

```java
@State(Scope.Thread)
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.MICROSECONDS)
@Warmup(iterations = 3, time = 1)
@Measurement(iterations = 5, time = 1)
@Fork(1)
public class SerializationBenchmark {

    private User user;
    private ObjectMapper objectMapper;

    @Setup
    public void setup() {
        user = new User("testuser", "test@example.com", "hash");
        objectMapper = new ObjectMapper();
    }

    @Benchmark
    public String jacksonSerialization() throws Exception {
        return objectMapper.writeValueAsString(user);
    }

    @Benchmark
    public User jacksonDeserialization() throws Exception {
        String json = objectMapper.writeValueAsString(user);
        return objectMapper.readValue(json, User.class);
    }
}
```

## Quality Indicators

### Coverage Metrics
- **Line Coverage**: 90%+ (traditional metric)
- **Branch Coverage**: 85%+ (critical for logic)
- **Instruction Coverage**: 90%+ (most accurate)
- **Mutation Coverage**: 80%+ (test effectiveness)

### Test Quality
- Clear test names describing behavior
- Arrange-Act-Assert structure
- No test interdependencies
- Fast execution (< 5 minutes for unit tests)
- Minimal mocking (only external dependencies)

### Test Organization
- One test class per production class
- Nested test classes for logical grouping
- Parameterized tests for similar scenarios
- Integration tests with real dependencies

## Common Anti-Patterns

### DON'T: Testing Implementation Details

```java
// BAD: Testing internal state
@Test
void shouldSetInternalCache() {
    service.loadData();
    assertThat(service.getCache()).isNotEmpty();  // Implementation detail!
}

// GOOD: Testing behavior
@Test
void shouldLoadDataFasterOnSecondCall() {
    var firstCallDuration = measureDuration(() -> service.loadData());
    var secondCallDuration = measureDuration(() -> service.loadData());
    assertThat(secondCallDuration).isLessThan(firstCallDuration);
}
```

### DON'T: Fragile Tests

```java
// BAD: Depends on exact string format
@Test
void shouldFormatMessage() {
    var message = service.formatWelcomeMessage(user);
    assertThat(message).isEqualTo("Welcome, John Doe! You have 5 messages.");
}

// GOOD: Tests essential behavior
@Test
void shouldIncludeUsernameAndMessageCount() {
    var message = service.formatWelcomeMessage(user);
    assertThat(message)
        .contains(user.getName())
        .contains("5 messages");
}
```

### DON'T: Slow Tests

```java
// BAD: Thread.sleep in tests
@Test
void shouldProcessAsync() throws InterruptedException {
    service.startAsyncProcess();
    Thread.sleep(5000);  // BAD! Slow and flaky
    assertThat(service.isComplete()).isTrue();
}

// GOOD: Use Awaitility
@Test
void shouldCompleteProcessing() {
    service.startAsyncProcess();

    await()
        .atMost(Duration.ofSeconds(5))
        .pollInterval(Duration.ofMillis(100))
        .until(() -> service.isComplete());
}
```

## Supporting Components

For complete testing ecosystem:
- **Java Standards**: `.claude/context/java/java-standards.md`
- **Spring Boot Testing**: `.claude/context/java/spring-boot-standards.md`
- **CI/CD Integration**: Run coverage and mutation testing in pipelines
- **IDE Integration**: IntelliJ IDEA coverage runner, Eclipse EclEmma
