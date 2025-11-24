---
description: Spring Boot 3.x standards with dependency injection, REST APIs, JPA, security, and production-ready patterns
globs: "**/*.java, **/application.yml, **/application.properties"
alwaysApply: false
---

# Spring Boot Standards and Implementation Patterns

Comprehensive patterns for Spring Boot 3.x applications with modern Java, type safety, and production-ready practices.

## Prerequisites

- Spring Boot 3.0+ (requires Java 17+)
- Spring Framework 6.0+
- Jakarta EE 9+ (jakarta.* packages)
- Gradle 8.0+ or Maven 3.9+

## Core Principle

**Build production-ready Spring Boot applications using constructor injection, immutable configuration, explicit validation, and comprehensive testing.**

## Dependency Injection Patterns

### Good: Constructor Injection (Recommended)

```java
@Service
public class OrderService {
    private final OrderRepository orderRepository;
    private final CustomerRepository customerRepository;
    private final EmailService emailService;
    private final Logger logger;

    // Constructor injection: Immutable, testable, explicit dependencies
    public OrderService(
        OrderRepository orderRepository,
        CustomerRepository customerRepository,
        EmailService emailService
    ) {
        this.orderRepository = orderRepository;
        this.customerRepository = customerRepository;
        this.emailService = emailService;
        this.logger = LoggerFactory.getLogger(OrderService.class);
    }

    @Transactional
    public Order createOrder(CreateOrderRequest request) {
        var customer = customerRepository.findById(request.customerId())
            .orElseThrow(() -> new CustomerNotFoundException(request.customerId()));

        var order = new Order(customer, request.items());
        var savedOrder = orderRepository.save(order);

        emailService.sendOrderConfirmation(customer.getEmail(), savedOrder);
        logger.info("Created order {} for customer {}", savedOrder.getId(), customer.getId());

        return savedOrder;
    }
}
```

### Good: Record-based Configuration

```java
@ConfigurationProperties(prefix = "app")
public record AppProperties(
    String name,
    String version,
    Security security,
    Database database
) {
    public record Security(
        String jwtSecret,
        Duration tokenExpiration
    ) {}

    public record Database(
        int maxConnections,
        Duration connectionTimeout
    ) {}
}

@Configuration
@EnableConfigurationProperties(AppProperties.class)
public class AppConfig {

    @Bean
    public JwtService jwtService(AppProperties properties) {
        return new JwtService(
            properties.security().jwtSecret(),
            properties.security().tokenExpiration()
        );
    }
}
```

### Bad: Field Injection

```java
// DON'T: Hard to test, mutable, implicit dependencies
@Service
public class OrderService {
    @Autowired  // BAD!
    private OrderRepository orderRepository;

    @Autowired  // BAD!
    private CustomerRepository customerRepository;

    // Cannot create instance without Spring container
    // Cannot ensure dependencies are set
    // Harder to write unit tests
}
```

## REST API Patterns

### Good: Controller with Validation

```java
@RestController
@RequestMapping("/api/v1/users")
@Validated
public class UserController {
    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping("/{id}")
    public ResponseEntity<UserResponse> getUser(@PathVariable Long id) {
        return userService.findById(id)
            .map(UserResponse::from)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }

    @PostMapping
    public ResponseEntity<UserResponse> createUser(
        @Valid @RequestBody CreateUserRequest request
    ) {
        var user = userService.createUser(request);
        var response = UserResponse.from(user);

        return ResponseEntity
            .created(URI.create("/api/v1/users/" + user.getId()))
            .body(response);
    }

    @PutMapping("/{id}")
    public ResponseEntity<UserResponse> updateUser(
        @PathVariable Long id,
        @Valid @RequestBody UpdateUserRequest request
    ) {
        return userService.updateUser(id, request)
            .map(UserResponse::from)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.noContent().build();
    }
}
```

### Good: Request/Response DTOs with Validation

```java
public record CreateUserRequest(
    @NotBlank(message = "Username is required")
    @Size(min = 3, max = 50, message = "Username must be between 3 and 50 characters")
    String username,

    @NotBlank(message = "Email is required")
    @Email(message = "Email must be valid")
    String email,

    @NotBlank(message = "Password is required")
    @Size(min = 8, message = "Password must be at least 8 characters")
    String password,

    @Min(value = 18, message = "Must be at least 18 years old")
    @Max(value = 120, message = "Invalid age")
    Integer age
) {}

public record UpdateUserRequest(
    @Size(min = 3, max = 50)
    String username,

    @Email
    String email,

    @Min(18)
    @Max(120)
    Integer age
) {}

public record UserResponse(
    Long id,
    String username,
    String email,
    Integer age,
    LocalDateTime createdAt,
    LocalDateTime updatedAt
) {
    public static UserResponse from(User user) {
        return new UserResponse(
            user.getId(),
            user.getUsername(),
            user.getEmail(),
            user.getAge(),
            user.getCreatedAt(),
            user.getUpdatedAt()
        );
    }
}
```

### Good: Global Exception Handling

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(EntityNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(EntityNotFoundException ex) {
        var error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage(),
            Instant.now()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ValidationErrorResponse> handleValidation(
        MethodArgumentNotValidException ex
    ) {
        var errors = ex.getBindingResult()
            .getFieldErrors()
            .stream()
            .map(error -> new FieldError(
                error.getField(),
                error.getDefaultMessage()
            ))
            .toList();

        var response = new ValidationErrorResponse(
            HttpStatus.BAD_REQUEST.value(),
            "Validation failed",
            errors,
            Instant.now()
        );

        return ResponseEntity.badRequest().body(response);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneral(Exception ex) {
        var error = new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "An unexpected error occurred",
            Instant.now()
        );
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}

public record ErrorResponse(
    int status,
    String message,
    Instant timestamp
) {}

public record ValidationErrorResponse(
    int status,
    String message,
    List<FieldError> errors,
    Instant timestamp
) {}

public record FieldError(
    String field,
    String message
) {}
```

## JPA and Database Patterns

### Good: Entity with Explicit Annotations

```java
@Entity
@Table(
    name = "users",
    indexes = {
        @Index(name = "idx_username", columnList = "username"),
        @Index(name = "idx_email", columnList = "email")
    },
    uniqueConstraints = {
        @UniqueConstraint(name = "uk_username", columnNames = "username"),
        @UniqueConstraint(name = "uk_email", columnNames = "email")
    }
)
@EntityListeners(AuditingEntityListener.class)
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id", nullable = false)
    private Long id;

    @Column(name = "username", nullable = false, length = 50)
    private String username;

    @Column(name = "email", nullable = false, length = 100)
    private String email;

    @Column(name = "password_hash", nullable = false)
    private String passwordHash;

    @Column(name = "age")
    private Integer age;

    @Column(name = "active", nullable = false)
    private Boolean active = true;

    @CreatedDate
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;

    @OneToMany(
        mappedBy = "user",
        cascade = {CascadeType.PERSIST, CascadeType.MERGE},
        orphanRemoval = true,
        fetch = FetchType.LAZY
    )
    private List<Order> orders = new ArrayList<>();

    @ManyToMany(fetch = FetchType.LAZY)
    @JoinTable(
        name = "user_roles",
        joinColumns = @JoinColumn(name = "user_id"),
        inverseJoinColumns = @JoinColumn(name = "role_id"),
        indexes = {
            @Index(name = "idx_user_roles_user_id", columnList = "user_id"),
            @Index(name = "idx_user_roles_role_id", columnList = "role_id")
        }
    )
    private Set<Role> roles = new HashSet<>();

    protected User() {
        // JPA requires no-arg constructor
    }

    public User(String username, String email, String passwordHash) {
        this.username = username;
        this.email = email;
        this.passwordHash = passwordHash;
    }

    // Getters and setters
    public Long getId() { return id; }

    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }

    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }

    public String getPasswordHash() { return passwordHash; }
    public void setPasswordHash(String passwordHash) { this.passwordHash = passwordHash; }

    public Integer getAge() { return age; }
    public void setAge(Integer age) { this.age = age; }

    public Boolean getActive() { return active; }
    public void setActive(Boolean active) { this.active = active; }

    public LocalDateTime getCreatedAt() { return createdAt; }
    public LocalDateTime getUpdatedAt() { return updatedAt; }

    // Relationship helpers
    public void addOrder(Order order) {
        orders.add(order);
        order.setUser(this);
    }

    public void removeOrder(Order order) {
        orders.remove(order);
        order.setUser(null);
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof User user)) return false;
        return id != null && id.equals(user.id);
    }

    @Override
    public int hashCode() {
        return getClass().hashCode();
    }
}
```

### Good: Repository with Custom Queries

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    Optional<User> findByUsername(String username);

    Optional<User> findByEmail(String email);

    @Query("""
        SELECT u FROM User u
        LEFT JOIN FETCH u.roles
        WHERE u.username = :username
        """)
    Optional<User> findByUsernameWithRoles(@Param("username") String username);

    @Query("""
        SELECT u FROM User u
        WHERE u.active = true
          AND u.createdAt >= :since
        ORDER BY u.createdAt DESC
        """)
    List<User> findRecentActiveUsers(@Param("since") LocalDateTime since);

    @Modifying
    @Query("UPDATE User u SET u.active = false WHERE u.id = :id")
    void deactivateUser(@Param("id") Long id);
}
```

### Good: Service with Transaction Management

```java
@Service
@Transactional(readOnly = true)
public class UserService {
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;

    public UserService(UserRepository userRepository, PasswordEncoder passwordEncoder) {
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
    }

    public Optional<User> findById(Long id) {
        return userRepository.findById(id);
    }

    public Optional<User> findByUsername(String username) {
        return userRepository.findByUsername(username);
    }

    @Transactional  // Override read-only for write operation
    public User createUser(CreateUserRequest request) {
        if (userRepository.findByUsername(request.username()).isPresent()) {
            throw new DuplicateUsernameException(request.username());
        }

        if (userRepository.findByEmail(request.email()).isPresent()) {
            throw new DuplicateEmailException(request.email());
        }

        var passwordHash = passwordEncoder.encode(request.password());
        var user = new User(request.username(), request.email(), passwordHash);
        user.setAge(request.age());

        return userRepository.save(user);
    }

    @Transactional
    public Optional<User> updateUser(Long id, UpdateUserRequest request) {
        return userRepository.findById(id)
            .map(user -> {
                if (request.username() != null) {
                    user.setUsername(request.username());
                }
                if (request.email() != null) {
                    user.setEmail(request.email());
                }
                if (request.age() != null) {
                    user.setAge(request.age());
                }
                return user;  // Automatic dirty checking updates DB
            });
    }

    @Transactional
    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }
}
```

## Security Patterns

### Good: Security Configuration

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

    private final JwtAuthenticationFilter jwtAuthFilter;
    private final AuthenticationProvider authenticationProvider;

    public SecurityConfig(
        JwtAuthenticationFilter jwtAuthFilter,
        AuthenticationProvider authenticationProvider
    ) {
        this.jwtAuthFilter = jwtAuthFilter;
        this.authenticationProvider = authenticationProvider;
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf(AbstractHttpConfigurer::disable)
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/v1/auth/**").permitAll()
                .requestMatchers("/actuator/health").permitAll()
                .requestMatchers("/api/v1/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .authenticationProvider(authenticationProvider)
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);  // Strong work factor
    }

    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration config)
        throws Exception {
        return config.getAuthenticationManager();
    }

    @Bean
    public AuthenticationProvider authenticationProvider(
        UserDetailsService userDetailsService,
        PasswordEncoder passwordEncoder
    ) {
        var provider = new DaoAuthenticationProvider();
        provider.setUserDetailsService(userDetailsService);
        provider.setPasswordEncoder(passwordEncoder);
        return provider;
    }
}
```

### Good: JWT Service

```java
@Service
public class JwtService {
    private final String secret;
    private final Duration expiration;

    public JwtService(@Value("${app.security.jwt-secret}") String secret,
                     @Value("${app.security.token-expiration}") Duration expiration) {
        this.secret = secret;
        this.expiration = expiration;
    }

    public String generateToken(UserDetails userDetails) {
        return Jwts.builder()
            .setSubject(userDetails.getUsername())
            .setIssuedAt(new Date())
            .setExpiration(Date.from(Instant.now().plus(expiration)))
            .signWith(getSigningKey(), SignatureAlgorithm.HS256)
            .compact();
    }

    public boolean validateToken(String token, UserDetails userDetails) {
        var username = extractUsername(token);
        return username.equals(userDetails.getUsername()) && !isTokenExpired(token);
    }

    public String extractUsername(String token) {
        return extractClaim(token, Claims::getSubject);
    }

    private <T> T extractClaim(String token, Function<Claims, T> claimsResolver) {
        var claims = extractAllClaims(token);
        return claimsResolver.apply(claims);
    }

    private Claims extractAllClaims(String token) {
        return Jwts.parserBuilder()
            .setSigningKey(getSigningKey())
            .build()
            .parseClaimsJws(token)
            .getBody();
    }

    private boolean isTokenExpired(String token) {
        return extractClaim(token, Claims::getExpiration).before(new Date());
    }

    private Key getSigningKey() {
        byte[] keyBytes = Decoders.BASE64.decode(secret);
        return Keys.hmacShaKeyFor(keyBytes);
    }
}
```

## Testing Patterns

### Good: Integration Test with Testcontainers

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@Testcontainers
class UserControllerIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15-alpine")
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
    private TestRestTemplate restTemplate;

    @Autowired
    private UserRepository userRepository;

    @BeforeEach
    void setUp() {
        userRepository.deleteAll();
    }

    @Test
    void shouldCreateUser() {
        var request = new CreateUserRequest(
            "testuser",
            "test@example.com",
            "password123",
            25
        );

        var response = restTemplate.postForEntity(
            "/api/v1/users",
            request,
            UserResponse.class
        );

        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        assertThat(response.getBody()).isNotNull();
        assertThat(response.getBody().username()).isEqualTo("testuser");
        assertThat(response.getBody().email()).isEqualTo("test@example.com");
    }

    @Test
    void shouldReturnNotFoundForNonexistentUser() {
        var response = restTemplate.getForEntity(
            "/api/v1/users/999",
            UserResponse.class
        );

        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.NOT_FOUND);
    }
}
```

### Good: Service Unit Test

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @Mock
    private PasswordEncoder passwordEncoder;

    @InjectMocks
    private UserService userService;

    @Test
    void shouldCreateUser() {
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
        user.setAge(25);
        when(userRepository.save(any(User.class))).thenReturn(user);

        var result = userService.createUser(request);

        assertThat(result.getUsername()).isEqualTo("testuser");
        assertThat(result.getEmail()).isEqualTo("test@example.com");
        assertThat(result.getAge()).isEqualTo(25);

        verify(userRepository).findByUsername("testuser");
        verify(userRepository).findByEmail("test@example.com");
        verify(passwordEncoder).encode("password123");
        verify(userRepository).save(any(User.class));
    }

    @Test
    void shouldThrowExceptionWhenUsernameExists() {
        var request = new CreateUserRequest(
            "existinguser",
            "test@example.com",
            "password123",
            25
        );

        var existingUser = new User("existinguser", "other@example.com", "hash");
        when(userRepository.findByUsername(request.username()))
            .thenReturn(Optional.of(existingUser));

        assertThatThrownBy(() -> userService.createUser(request))
            .isInstanceOf(DuplicateUsernameException.class)
            .hasMessageContaining("existinguser");

        verify(userRepository, never()).save(any());
    }
}
```

## Configuration Files

### Good: application.yml

```yaml
spring:
  application:
    name: user-service

  datasource:
    url: ${DATABASE_URL:jdbc:postgresql://localhost:5432/userdb}
    username: ${DATABASE_USER:postgres}
    password: ${DATABASE_PASSWORD:postgres}
    driver-class-name: org.postgresql.Driver
    hikari:
      maximum-pool-size: 10
      minimum-idle: 2
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000

  jpa:
    hibernate:
      ddl-auto: validate
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
        format_sql: true
        show_sql: false
        jdbc:
          batch_size: 20
        order_inserts: true
        order_updates: true
    open-in-view: false

  flyway:
    enabled: true
    baseline-on-migrate: true
    locations: classpath:db/migration

server:
  port: ${PORT:8080}
  compression:
    enabled: true
  error:
    include-message: always
    include-binding-errors: always

app:
  name: User Service
  version: 1.0.0
  security:
    jwt-secret: ${JWT_SECRET}
    token-expiration: PT24H
  database:
    max-connections: 10
    connection-timeout: PT30S

logging:
  level:
    root: INFO
    com.example: DEBUG
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} - %msg%n"

management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  endpoint:
    health:
      show-details: always
```

## Quality Indicators

### Configuration
- Constructor injection everywhere
- No `@Autowired` on fields
- Records for configuration properties
- Environment-specific profiles

### API Design
- DTOs separate from entities
- Validation on request DTOs
- Global exception handling
- Consistent response formats

### Database
- Explicit JPA annotations
- Foreign key indexes on PostgreSQL
- Lazy loading by default
- Transaction boundaries at service layer

### Security
- BCrypt with high work factor
- JWT with appropriate expiration
- HTTPS in production
- Input validation and sanitization

## Common Anti-Patterns

### DON'T: Entity as DTO

```java
// BAD: Exposing entity directly
@GetMapping("/{id}")
public User getUser(@PathVariable Long id) {
    return userRepository.findById(id).orElseThrow();  // Exposes entity!
}

// GOOD: Use DTO
@GetMapping("/{id}")
public UserResponse getUser(@PathVariable Long id) {
    return userRepository.findById(id)
        .map(UserResponse::from)
        .orElseThrow();
}
```

### DON'T: N+1 Query Problems

```java
// BAD: Causes N+1 queries
@GetMapping("/users")
public List<UserResponse> getUsers() {
    return userRepository.findAll().stream()
        .map(user -> new UserResponse(
            user.getId(),
            user.getUsername(),
            user.getOrders().size()  // Lazy load triggers N queries!
        ))
        .toList();
}

// GOOD: Fetch join
@Query("""
    SELECT u FROM User u
    LEFT JOIN FETCH u.orders
    """)
List<User> findAllWithOrders();
```

## Supporting Components

For complete Spring Boot development:
- **Java Standards**: `.claude/context/java/java-standards.md`
- **Testing Patterns**: `.claude/context/java/testing-standards.md`
- **Database Patterns**: `.claude/context/database/java-hibernate-standards.md`
- **Security Best Practices**: OWASP Top 10 compliance
