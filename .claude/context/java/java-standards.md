---
description: Comprehensive Java 17-21+ standards with modern language features, patterns, and code examples
globs: "**/*.java"
alwaysApply: false
---

# Java Standards and Implementation Patterns

Comprehensive code examples and implementation patterns for modern Java 17-21+ development with type safety, immutability, and production-ready practices.

## Prerequisites

- Java 17+ (LTS) or Java 21+ (Latest LTS)
- Gradle 8.0+ or Maven 3.9+
- Modern IDE with Java 21 support (IntelliJ IDEA, Eclipse, VS Code)

## Core Principle

**Write idiomatic, type-safe, immutable-by-default Java code using modern language features that eliminate entire categories of bugs while maintaining excellent performance.**

## Modern Java Language Features

### Records for Immutable Data

Records eliminate boilerplate and enforce immutability for DTOs and value objects.

#### Good: Using Records for DTOs

```java
/**
 * User data transfer object.
 * Records provide automatic equals/hashCode/toString.
 */
public record UserDTO(
    Long id,
    String username,
    String email,
    LocalDateTime createdAt
) {
    // Compact constructor for validation
    public UserDTO {
        Objects.requireNonNull(username, "Username cannot be null");
        Objects.requireNonNull(email, "Email cannot be null");
        if (!email.contains("@")) {
            throw new IllegalArgumentException("Invalid email format");
        }
    }

    // Custom methods allowed
    public boolean isRecent() {
        return createdAt.isAfter(LocalDateTime.now().minusDays(7));
    }
}
```

#### Good: Record Patterns for Decomposition (Java 21+)

```java
public record Point(int x, int y) {}

public record Rectangle(Point topLeft, Point bottomRight) {}

public double calculateArea(Rectangle rect) {
    // Record pattern matching with decomposition
    return switch (rect) {
        case Rectangle(Point(int x1, int y1), Point(int x2, int y2)) ->
            Math.abs((x2 - x1) * (y2 - y1));
    };
}
```

#### Bad: Traditional Boilerplate Classes

```java
// DON'T: Too much boilerplate for simple data
public class UserDTO {
    private final Long id;
    private final String username;
    private final String email;
    private final LocalDateTime createdAt;

    public UserDTO(Long id, String username, String email, LocalDateTime createdAt) {
        this.id = id;
        this.username = username;
        this.email = email;
        this.createdAt = createdAt;
    }

    // ... 50+ lines of getters, equals, hashCode, toString
}
```

### Sealed Classes for Type Hierarchies

Sealed classes enable exhaustive pattern matching and closed type hierarchies.

#### Good: Sealed Interface with Records

```java
/**
 * Payment method sealed hierarchy.
 * Compiler ensures all cases are handled.
 */
public sealed interface PaymentMethod
    permits CreditCard, BankTransfer, Cash {
}

public record CreditCard(
    String cardNumber,
    String expiryDate,
    String cvv
) implements PaymentMethod {}

public record BankTransfer(
    String accountNumber,
    String routingNumber
) implements PaymentMethod {}

public record Cash(
    BigDecimal amount
) implements PaymentMethod {}

// Exhaustive pattern matching
public BigDecimal calculateFee(PaymentMethod payment) {
    return switch (payment) {
        case CreditCard cc -> cc.amount().multiply(new BigDecimal("0.03"));
        case BankTransfer bt -> new BigDecimal("5.00");
        case Cash c -> BigDecimal.ZERO;
        // No default needed - compiler ensures exhaustiveness
    };
}
```

#### Good: Sealed Abstract Class for Behavior

```java
public sealed abstract class Result<T>
    permits Success, Failure {

    public abstract <U> Result<U> map(Function<T, U> mapper);
    public abstract T orElseThrow();
}

public final class Success<T> extends Result<T> {
    private final T value;

    public Success(T value) {
        this.value = Objects.requireNonNull(value);
    }

    @Override
    public <U> Result<U> map(Function<T, U> mapper) {
        return new Success<>(mapper.apply(value));
    }

    @Override
    public T orElseThrow() {
        return value;
    }
}

public final class Failure<T> extends Result<T> {
    private final Exception exception;

    public Failure(Exception exception) {
        this.exception = Objects.requireNonNull(exception);
    }

    @Override
    public <U> Result<U> map(Function<T, U> mapper) {
        return new Failure<>(exception);
    }

    @Override
    public T orElseThrow() {
        throw new RuntimeException(exception);
    }
}
```

### Pattern Matching and Switch Expressions

#### Good: Type Patterns with Guards (Java 21+)

```java
public String formatValue(Object obj) {
    return switch (obj) {
        case Integer i when i > 0 -> "Positive: " + i;
        case Integer i when i < 0 -> "Negative: " + i;
        case Integer i -> "Zero";
        case String s when s.isEmpty() -> "Empty string";
        case String s -> "String: " + s;
        case List<?> list -> "List of size: " + list.size();
        case null -> "Null value";
        default -> "Unknown type: " + obj.getClass().getName();
    };
}
```

#### Good: Pattern Matching in instanceof

```java
public double calculateArea(Shape shape) {
    if (shape instanceof Circle c) {
        return Math.PI * c.radius() * c.radius();
    } else if (shape instanceof Rectangle r) {
        return r.width() * r.height();
    } else if (shape instanceof Triangle t) {
        return 0.5 * t.base() * t.height();
    }
    throw new IllegalArgumentException("Unknown shape");
}
```

#### Bad: Old-style instanceof

```java
// DON'T: Manual casting is error-prone
public double calculateArea(Shape shape) {
    if (shape instanceof Circle) {
        Circle c = (Circle) shape;  // Redundant cast
        return Math.PI * c.radius() * c.radius();
    }
    // ...
}
```

### Text Blocks for Multi-line Strings

#### Good: Text Blocks for JSON/SQL/HTML

```java
public class QueryBuilder {

    public String buildUserQuery(String username) {
        return """
            SELECT u.id, u.username, u.email, u.created_at
            FROM users u
            WHERE u.username = ?
              AND u.active = true
            ORDER BY u.created_at DESC
            LIMIT 1
            """;
    }

    public String buildJsonResponse(String message, int code) {
        return """
            {
              "status": "success",
              "code": %d,
              "message": "%s",
              "timestamp": "%s"
            }
            """.formatted(code, message, Instant.now());
    }
}
```

#### Bad: Concatenation Mess

```java
// DON'T: Hard to read and maintain
public String buildQuery() {
    return "SELECT u.id, u.username, u.email, u.created_at\n" +
           "FROM users u\n" +
           "WHERE u.username = ?\n" +
           "  AND u.active = true\n" +
           "ORDER BY u.created_at DESC\n" +
           "LIMIT 1";
}
```

## Immutability Patterns

### Immutable Collections

#### Good: Unmodifiable Collections

```java
public class UserService {
    private final Map<String, User> userCache;
    private final List<String> allowedRoles;

    public UserService(Map<String, User> initialCache, List<String> roles) {
        // Defensive copies with immutable wrappers
        this.userCache = Map.copyOf(initialCache);
        this.allowedRoles = List.copyOf(roles);
    }

    public List<String> getAllowedRoles() {
        return allowedRoles;  // Already immutable, safe to return
    }

    public Map<String, User> getUserCache() {
        return userCache;  // Already immutable, safe to return
    }
}
```

#### Good: Collection Factory Methods

```java
// Immutable collections (Java 9+)
List<String> roles = List.of("USER", "ADMIN", "MANAGER");
Set<Integer> ids = Set.of(1, 2, 3, 4, 5);
Map<String, Integer> scores = Map.of(
    "Alice", 100,
    "Bob", 95,
    "Charlie", 87
);

// For more than 10 entries
Map<String, String> largeMap = Map.ofEntries(
    Map.entry("key1", "value1"),
    Map.entry("key2", "value2"),
    // ... up to any size
);
```

#### Bad: Mutable Collections Exposed

```java
// DON'T: Exposes internal mutable state
public class UserService {
    private final List<User> users = new ArrayList<>();

    public List<User> getUsers() {
        return users;  // Caller can modify internal state!
    }
}
```

### Immutable Class Design

#### Good: Fully Immutable Class

```java
public final class Account {
    private final String accountId;
    private final BigDecimal balance;
    private final List<Transaction> transactions;
    private final LocalDateTime createdAt;

    public Account(
        String accountId,
        BigDecimal balance,
        List<Transaction> transactions
    ) {
        this.accountId = Objects.requireNonNull(accountId);
        this.balance = Objects.requireNonNull(balance);
        this.transactions = List.copyOf(transactions);  // Defensive copy
        this.createdAt = LocalDateTime.now();
    }

    // Return immutable view
    public List<Transaction> getTransactions() {
        return transactions;  // Already immutable
    }

    // Create new instance for mutations
    public Account withDeposit(BigDecimal amount) {
        var newBalance = balance.add(amount);
        var newTransaction = new Transaction(
            TransactionType.DEPOSIT,
            amount,
            LocalDateTime.now()
        );
        var newTransactions = new ArrayList<>(transactions);
        newTransactions.add(newTransaction);
        return new Account(accountId, newBalance, newTransactions);
    }
}
```

## Stream API Patterns

### Good: Declarative Stream Operations

```java
public class OrderProcessor {

    public BigDecimal calculateTotalRevenue(List<Order> orders) {
        return orders.stream()
            .filter(Order::isPaid)
            .filter(order -> order.getStatus() == OrderStatus.COMPLETED)
            .map(Order::getAmount)
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }

    public Map<String, Long> countOrdersByStatus(List<Order> orders) {
        return orders.stream()
            .collect(Collectors.groupingBy(
                order -> order.getStatus().name(),
                Collectors.counting()
            ));
    }

    public List<CustomerSummary> getTopCustomers(List<Order> orders, int limit) {
        return orders.stream()
            .collect(Collectors.groupingBy(
                Order::getCustomerId,
                Collectors.summingDouble(order -> order.getAmount().doubleValue())
            ))
            .entrySet().stream()
            .sorted(Map.Entry.<String, Double>comparingByValue().reversed())
            .limit(limit)
            .map(entry -> new CustomerSummary(entry.getKey(), entry.getValue()))
            .toList();  // Java 16+: Immutable list
    }
}
```

### Good: Parallel Streams for CPU-Intensive Work

```java
public class DataProcessor {

    public List<ProcessedData> processLargeDataset(List<RawData> data) {
        // Use parallel streams for CPU-intensive operations
        return data.parallelStream()
            .filter(this::isValid)
            .map(this::transform)
            .map(this::enrich)
            .collect(Collectors.toList());
    }

    private boolean isValid(RawData data) {
        // CPU-intensive validation
        return data.getValue() > 0 && checkComplexCondition(data);
    }

    private ProcessedData transform(RawData data) {
        // CPU-intensive transformation
        return new ProcessedData(
            computeComplexMetric(data),
            data.getTimestamp()
        );
    }
}
```

### Bad: Stream Anti-patterns

```java
// DON'T: Side effects in streams
public void updateBalances(List<Account> accounts) {
    accounts.stream()
        .forEach(account -> {
            account.setBalance(account.getBalance().add(INTEREST));  // Side effect!
        });
}

// DON'T: Unnecessary stream for simple operations
public boolean hasActiveUsers(List<User> users) {
    return users.stream()
        .anyMatch(User::isActive);  // OK, but isEmpty() is clearer for this
}

// Better:
public boolean hasActiveUsers(List<User> users) {
    return users.stream().anyMatch(User::isActive);  // This is actually good
}

// DON'T: Collecting to list just to get first element
public Optional<User> findFirstActive(List<User> users) {
    return users.stream()
        .filter(User::isActive)
        .collect(Collectors.toList())  // Wasteful!
        .stream()
        .findFirst();
}

// DO: Use findFirst directly
public Optional<User> findFirstActive(List<User> users) {
    return users.stream()
        .filter(User::isActive)
        .findFirst();
}
```

## Optional Best Practices

### Good: Optional as Return Type

```java
public class UserRepository {

    public Optional<User> findById(Long id) {
        User user = queryDatabase(id);
        return Optional.ofNullable(user);
    }

    public Optional<User> findByEmail(String email) {
        return Optional.ofNullable(queryByEmail(email));
    }
}

public class UserService {
    private final UserRepository repository;

    public User getUserOrDefault(Long id) {
        return repository.findById(id)
            .orElse(User.GUEST_USER);
    }

    public User getUserOrThrow(Long id) {
        return repository.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id));
    }

    public String getUserEmail(Long id) {
        return repository.findById(id)
            .map(User::getEmail)
            .orElse("no-email@example.com");
    }

    public void processUser(Long id) {
        repository.findById(id)
            .ifPresent(user -> {
                sendNotification(user);
                logAccess(user);
            });
    }
}
```

### Bad: Optional Anti-patterns

```java
// DON'T: Optional as parameter
public void updateUser(Optional<User> user) {  // BAD!
    user.ifPresent(this::save);
}

// DO: Use null or overloading
public void updateUser(User user) {
    if (user != null) {
        save(user);
    }
}

// DON'T: Calling get() without checking
public User getUser(Long id) {
    return repository.findById(id).get();  // Can throw NoSuchElementException!
}

// DO: Use orElseThrow with meaningful exception
public User getUser(Long id) {
    return repository.findById(id)
        .orElseThrow(() -> new UserNotFoundException(id));
}

// DON'T: Optional field
public class User {
    private Optional<String> middleName;  // BAD! Use null instead
}

// DO: Use nullable field
public class User {
    private String middleName;  // null is fine for optional fields
}
```

## Type System and Generics

### Good: Bounded Type Parameters

```java
public class Repository<T extends Entity> {
    private final Class<T> entityClass;

    public Repository(Class<T> entityClass) {
        this.entityClass = entityClass;
    }

    public Optional<T> findById(Long id) {
        // T is guaranteed to extend Entity
        return Optional.ofNullable(
            entityManager.find(entityClass, id)
        );
    }

    public List<T> findAll() {
        return entityManager
            .createQuery("SELECT e FROM " + entityClass.getSimpleName() + " e", entityClass)
            .getResultList();
    }
}

public interface Comparable<T> {
    int compareTo(T other);
}

public class NumberRange<T extends Number & Comparable<T>> {
    private final T min;
    private final T max;

    public NumberRange(T min, T max) {
        if (min.compareTo(max) > 0) {
            throw new IllegalArgumentException("min must be <= max");
        }
        this.min = min;
        this.max = max;
    }

    public boolean contains(T value) {
        return value.compareTo(min) >= 0 && value.compareTo(max) <= 0;
    }
}
```

### Good: Wildcards for Flexibility

```java
public class CollectionUtils {

    // Producer: Use extends (read-only)
    public static <T> void copy(
        List<? extends T> source,
        List<? super T> destination
    ) {
        for (T item : source) {
            destination.add(item);
        }
    }

    // Consumer: Use super (write-only)
    public static void addNumbers(List<? super Integer> list) {
        list.add(1);
        list.add(2);
        list.add(3);
    }

    public static double sum(List<? extends Number> numbers) {
        return numbers.stream()
            .mapToDouble(Number::doubleValue)
            .sum();
    }
}
```

## Functional Programming Patterns

### Good: Function Composition

```java
public class StringTransformer {

    public Function<String, String> createTransformer() {
        return ((Function<String, String>) String::trim)
            .andThen(String::toLowerCase)
            .andThen(s -> s.replaceAll("\\s+", "-"))
            .andThen(s -> s.replaceAll("[^a-z0-9-]", ""));
    }

    public List<String> transformAll(List<String> inputs) {
        var transformer = createTransformer();
        return inputs.stream()
            .map(transformer)
            .toList();
    }
}

public class ValidationChain {

    public static <T> Predicate<T> and(List<Predicate<T>> predicates) {
        return predicates.stream()
            .reduce(Predicate::and)
            .orElse(x -> true);
    }

    public boolean validateUser(User user) {
        var validations = List.<Predicate<User>>of(
            u -> u.getEmail() != null,
            u -> u.getEmail().contains("@"),
            u -> u.getAge() >= 18,
            u -> u.getUsername() != null && u.getUsername().length() >= 3
        );

        return and(validations).test(user);
    }
}
```

### Good: Pure Functions

```java
public class Calculator {

    // Pure function: Same input always produces same output, no side effects
    public static BigDecimal calculateTax(BigDecimal amount, BigDecimal rate) {
        return amount.multiply(rate).setScale(2, RoundingMode.HALF_UP);
    }

    // Pure function: No mutations, returns new objects
    public static List<Order> applyDiscount(List<Order> orders, BigDecimal discount) {
        return orders.stream()
            .map(order -> new Order(
                order.getId(),
                order.getCustomerId(),
                order.getAmount().multiply(BigDecimal.ONE.subtract(discount))
            ))
            .toList();
    }
}
```

## Exception Handling

### Good: Specific Exceptions

```java
public class OrderService {

    public Order createOrder(OrderRequest request) {
        validateRequest(request);

        var customer = customerRepository.findById(request.getCustomerId())
            .orElseThrow(() -> new CustomerNotFoundException(request.getCustomerId()));

        var product = productRepository.findById(request.getProductId())
            .orElseThrow(() -> new ProductNotFoundException(request.getProductId()));

        if (product.getStock() < request.getQuantity()) {
            throw new InsufficientStockException(
                product.getId(),
                product.getStock(),
                request.getQuantity()
            );
        }

        return orderRepository.save(new Order(
            customer,
            product,
            request.getQuantity()
        ));
    }

    private void validateRequest(OrderRequest request) {
        if (request.getQuantity() <= 0) {
            throw new InvalidOrderException("Quantity must be positive");
        }
        if (request.getCustomerId() == null) {
            throw new InvalidOrderException("Customer ID is required");
        }
        if (request.getProductId() == null) {
            throw new InvalidOrderException("Product ID is required");
        }
    }
}

// Custom exceptions
public class CustomerNotFoundException extends RuntimeException {
    public CustomerNotFoundException(Long customerId) {
        super("Customer not found: " + customerId);
    }
}

public class ProductNotFoundException extends RuntimeException {
    public ProductNotFoundException(Long productId) {
        super("Product not found: " + productId);
    }
}

public class InsufficientStockException extends RuntimeException {
    public InsufficientStockException(Long productId, int available, int requested) {
        super(String.format(
            "Insufficient stock for product %d: %d available, %d requested",
            productId, available, requested
        ));
    }
}
```

### Good: Try-with-resources

```java
public class FileProcessor {

    public List<String> readLines(Path path) throws IOException {
        try (var reader = Files.newBufferedReader(path)) {
            return reader.lines().toList();
        }
        // Reader automatically closed, even if exception occurs
    }

    public void processFile(Path input, Path output) throws IOException {
        try (
            var reader = Files.newBufferedReader(input);
            var writer = Files.newBufferedWriter(output)
        ) {
            reader.lines()
                .map(String::toUpperCase)
                .forEach(line -> {
                    try {
                        writer.write(line);
                        writer.newLine();
                    } catch (IOException e) {
                        throw new UncheckedIOException(e);
                    }
                });
        }
    }
}
```

## Quality Indicators

### Code Metrics
- **Cyclomatic Complexity**: < 10 per method (SpotBugs)
- **Method Length**: < 50 lines per method
- **Class Length**: < 500 lines per class
- **Test Coverage**: > 90% instruction coverage (JaCoCo)
- **Mutation Coverage**: > 80% (PITest)

### Type Safety
- No raw types warnings
- No unchecked cast warnings
- Proper generic bounds
- Explicit variance annotations

### Immutability
- Final fields by default
- Unmodifiable collections for public APIs
- No setters on domain objects
- Records for DTOs

## Common Anti-Patterns

### DON'T: Null Collections

```java
// BAD
public List<User> getUsers() {
    return null;  // Caller must null-check
}

// GOOD
public List<User> getUsers() {
    return List.of();  // Empty list, no null-check needed
}
```

### DON'T: Primitive Obsession

```java
// BAD
public void createAccount(String accountId, double balance, int type) {
    // What does type=1 mean?
}

// GOOD
public void createAccount(AccountId accountId, Money balance, AccountType type) {
    // Type-safe value objects
}

public record AccountId(String value) {
    public AccountId {
        if (value == null || value.isBlank()) {
            throw new IllegalArgumentException("Account ID cannot be blank");
        }
    }
}

public record Money(BigDecimal amount, Currency currency) {
    public Money {
        if (amount.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("Amount cannot be negative");
        }
    }
}

public enum AccountType {
    CHECKING, SAVINGS, INVESTMENT
}
```

### DON'T: String Concatenation in Loops

```java
// BAD
public String buildReport(List<String> items) {
    String report = "";
    for (String item : items) {
        report += item + "\n";  // Creates new String each iteration!
    }
    return report;
}

// GOOD
public String buildReport(List<String> items) {
    return String.join("\n", items);
}

// GOOD (for complex building)
public String buildComplexReport(List<Item> items) {
    var sb = new StringBuilder();
    for (Item item : items) {
        sb.append("ID: ").append(item.id())
          .append(", Name: ").append(item.name())
          .append("\n");
    }
    return sb.toString();
}
```

## Supporting Components

For comprehensive Java development:
- **Spring Boot Patterns**: `.claude/context/java/spring-boot-standards.md`
- **Testing Standards**: `.claude/context/java/testing-standards.md`
- **Build Configuration**: Gradle/Maven setup examples
- **Static Analysis**: SpotBugs, PMD, Checkstyle configurations
