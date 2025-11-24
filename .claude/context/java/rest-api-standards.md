---
description: REST API implementation standards for Spring Boot 3.x microservices with resilience patterns, security, and observability
globs: "**/*Controller.java, **/*Service.java, **/application.yml"
alwaysApply: false
---

# REST API Implementation Standards for Spring Boot 3.x Microservices

Comprehensive patterns for building production-ready REST APIs using Spring Boot 3.x, Spring Cloud, and modern Java 21+ features with an opinionated technology stack.

## Prerequisites

- Spring Boot 3.2+ (requires Java 17+, recommended Java 21+)
- Spring Cloud 2023.x (2025 compatible)
- Spring Security 6.x
- Resilience4j 2.x
- OpenTelemetry + Micrometer for observability
- Gradle 8.0+ or Maven 3.9+

## Core Principle

**Build production-ready REST APIs for microservices using constructor injection, records for DTOs, resilience patterns (circuit breakers, rate limiting), comprehensive security (OAuth2/JWT), and full observability (metrics, tracing, structured logging).**

## REST Controller Implementation

### Good: Modern REST Controller with Records

```java
package com.example.api.controller;

import com.example.api.dto.CreateOrderRequest;
import com.example.api.dto.OrderResponse;
import com.example.api.dto.ErrorResponse;
import com.example.api.service.OrderService;
import io.github.resilience4j.circuitbreaker.annotation.CircuitBreaker;
import io.github.resilience4j.ratelimiter.annotation.RateLimiter;
import io.micrometer.observation.annotation.Observed;
import jakarta.validation.Valid;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

/**
 * REST controller for order management operations.
 * Implements resilience patterns and observability.
 */
@RestController
@RequestMapping("/api/v1/orders")
@Observed(name = "order.controller")
public class OrderController {

    private static final Logger logger = LoggerFactory.getLogger(OrderController.class);
    private final OrderService orderService;

    // Constructor injection for immutability and testability
    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }

    /**
     * Create a new order with circuit breaker and rate limiting.
     *
     * @param request validated order creation request
     * @return created order response with location header
     */
    @PostMapping
    @CircuitBreaker(name = "orderService", fallbackMethod = "createOrderFallback")
    @RateLimiter(name = "orderService")
    @Observed(name = "order.create")
    public ResponseEntity<OrderResponse> createOrder(@Valid @RequestBody CreateOrderRequest request) {
        logger.info("Creating order for customer: {}", request.customerId());

        var order = orderService.createOrder(request);

        return ResponseEntity
            .status(HttpStatus.CREATED)
            .header("Location", "/api/v1/orders/" + order.id())
            .body(order);
    }

    /**
     * Fallback method for order creation failures.
     */
    private ResponseEntity<OrderResponse> createOrderFallback(
        CreateOrderRequest request,
        Exception ex
    ) {
        logger.error("Order creation failed, returning fallback", ex);
        return ResponseEntity
            .status(HttpStatus.SERVICE_UNAVAILABLE)
            .build();
    }

    /**
     * Get order by ID with caching support.
     */
    @GetMapping("/{orderId}")
    @CircuitBreaker(name = "orderService")
    @Observed(name = "order.get")
    public ResponseEntity<OrderResponse> getOrder(@PathVariable Long orderId) {
        logger.debug("Fetching order: {}", orderId);

        return orderService.getOrderById(orderId)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }

    /**
     * List orders with pagination.
     */
    @GetMapping
    @Observed(name = "order.list")
    public ResponseEntity<List<OrderResponse>> listOrders(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "20") int size
    ) {
        logger.debug("Listing orders: page={}, size={}", page, size);

        var orders = orderService.listOrders(page, size);
        return ResponseEntity.ok(orders);
    }

    /**
     * Update order status.
     */
    @PatchMapping("/{orderId}/status")
    @CircuitBreaker(name = "orderService")
    @Observed(name = "order.updateStatus")
    public ResponseEntity<OrderResponse> updateOrderStatus(
        @PathVariable Long orderId,
        @RequestParam String status
    ) {
        logger.info("Updating order {} status to {}", orderId, status);

        return orderService.updateOrderStatus(orderId, status)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }

    /**
     * Delete order (soft delete).
     */
    @DeleteMapping("/{orderId}")
    @CircuitBreaker(name = "orderService")
    @Observed(name = "order.delete")
    public ResponseEntity<Void> deleteOrder(@PathVariable Long orderId) {
        logger.info("Deleting order: {}", orderId);

        orderService.deleteOrder(orderId);
        return ResponseEntity.noContent().build();
    }
}
```

### Good: DTO with Records (Java 21+)

```java
package com.example.api.dto;

import jakarta.validation.constraints.*;
import java.math.BigDecimal;
import java.time.Instant;
import java.util.List;

/**
 * Request to create a new order.
 * Uses record for immutability and conciseness.
 */
public record CreateOrderRequest(
    @NotNull(message = "Customer ID is required")
    @Positive(message = "Customer ID must be positive")
    Long customerId,

    @NotEmpty(message = "Order items cannot be empty")
    List<@Valid OrderItemRequest> items,

    @Size(max = 500, message = "Notes cannot exceed 500 characters")
    String notes
) {
    // Compact constructor for additional validation
    public CreateOrderRequest {
        if (items != null && items.isEmpty()) {
            throw new IllegalArgumentException("Order must contain at least one item");
        }
    }
}

/**
 * Individual order item in a request.
 */
public record OrderItemRequest(
    @NotNull(message = "Product ID is required")
    @Positive
    Long productId,

    @NotNull(message = "Quantity is required")
    @Min(value = 1, message = "Quantity must be at least 1")
    Integer quantity,

    @DecimalMin(value = "0.0", inclusive = false, message = "Price must be greater than zero")
    BigDecimal unitPrice
) {}

/**
 * Order response DTO.
 */
public record OrderResponse(
    Long id,
    Long customerId,
    String customerName,
    List<OrderItemResponse> items,
    BigDecimal totalAmount,
    String status,
    Instant createdAt,
    Instant updatedAt
) {}

/**
 * Order item in a response.
 */
public record OrderItemResponse(
    Long id,
    Long productId,
    String productName,
    Integer quantity,
    BigDecimal unitPrice,
    BigDecimal totalPrice
) {}
```

## Global Exception Handling

### Good: @RestControllerAdvice with RFC 7807 Problem Details

```java
package com.example.api.exception;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.HttpStatus;
import org.springframework.http.ProblemDetail;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import org.springframework.web.context.request.WebRequest;

import java.net.URI;
import java.time.Instant;
import java.util.HashMap;
import java.util.Map;

/**
 * Global exception handler for REST API.
 * Returns RFC 7807 Problem Details for consistent error responses.
 */
@RestControllerAdvice
public class GlobalExceptionHandler {

    private static final Logger logger = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    /**
     * Handle validation errors from @Valid.
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ProblemDetail> handleValidationErrors(
        MethodArgumentNotValidException ex,
        WebRequest request
    ) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getAllErrors().forEach(error -> {
            String fieldName = ((FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        });

        ProblemDetail problemDetail = ProblemDetail.forStatusAndDetail(
            HttpStatus.BAD_REQUEST,
            "Validation failed for one or more fields"
        );
        problemDetail.setType(URI.create("https://api.example.com/errors/validation-error"));
        problemDetail.setTitle("Validation Error");
        problemDetail.setProperty("timestamp", Instant.now());
        problemDetail.setProperty("errors", errors);

        logger.warn("Validation errors: {}", errors);
        return ResponseEntity.badRequest().body(problemDetail);
    }

    /**
     * Handle resource not found errors.
     */
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ProblemDetail> handleResourceNotFound(
        ResourceNotFoundException ex,
        WebRequest request
    ) {
        ProblemDetail problemDetail = ProblemDetail.forStatusAndDetail(
            HttpStatus.NOT_FOUND,
            ex.getMessage()
        );
        problemDetail.setType(URI.create("https://api.example.com/errors/not-found"));
        problemDetail.setTitle("Resource Not Found");
        problemDetail.setProperty("timestamp", Instant.now());
        problemDetail.setProperty("resourceType", ex.getResourceType());
        problemDetail.setProperty("resourceId", ex.getResourceId());

        logger.warn("Resource not found: {} with id {}", ex.getResourceType(), ex.getResourceId());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(problemDetail);
    }

    /**
     * Handle business logic exceptions.
     */
    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<ProblemDetail> handleBusinessException(
        BusinessException ex,
        WebRequest request
    ) {
        ProblemDetail problemDetail = ProblemDetail.forStatusAndDetail(
            HttpStatus.UNPROCESSABLE_ENTITY,
            ex.getMessage()
        );
        problemDetail.setType(URI.create("https://api.example.com/errors/business-error"));
        problemDetail.setTitle("Business Rule Violation");
        problemDetail.setProperty("timestamp", Instant.now());
        problemDetail.setProperty("errorCode", ex.getErrorCode());

        logger.error("Business exception: {}", ex.getMessage(), ex);
        return ResponseEntity.status(HttpStatus.UNPROCESSABLE_ENTITY).body(problemDetail);
    }

    /**
     * Handle all other exceptions.
     */
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ProblemDetail> handleGlobalException(
        Exception ex,
        WebRequest request
    ) {
        ProblemDetail problemDetail = ProblemDetail.forStatusAndDetail(
            HttpStatus.INTERNAL_SERVER_ERROR,
            "An unexpected error occurred"
        );
        problemDetail.setType(URI.create("https://api.example.com/errors/internal-error"));
        problemDetail.setTitle("Internal Server Error");
        problemDetail.setProperty("timestamp", Instant.now());

        logger.error("Unexpected exception", ex);
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(problemDetail);
    }
}

/**
 * Custom exception for resource not found scenarios.
 */
public class ResourceNotFoundException extends RuntimeException {
    private final String resourceType;
    private final Object resourceId;

    public ResourceNotFoundException(String resourceType, Object resourceId) {
        super(String.format("%s not found with id: %s", resourceType, resourceId));
        this.resourceType = resourceType;
        this.resourceId = resourceId;
    }

    public String getResourceType() { return resourceType; }
    public Object getResourceId() { return resourceId; }
}

/**
 * Custom exception for business logic violations.
 */
public class BusinessException extends RuntimeException {
    private final String errorCode;

    public BusinessException(String message, String errorCode) {
        super(message);
        this.errorCode = errorCode;
    }

    public String getErrorCode() { return errorCode; }
}
```

## Service Layer with Virtual Threads

### Good: Service with CompletableFuture and Virtual Threads

```java
package com.example.api.service;

import com.example.api.dto.*;
import com.example.api.exception.ResourceNotFoundException;
import com.example.api.repository.OrderRepository;
import com.example.api.repository.CustomerRepository;
import com.example.api.repository.ProductRepository;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.slf4j.MDC;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.Optional;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * Order service with virtual threads for I/O-bound operations.
 */
@Service
public class OrderService {

    private static final Logger logger = LoggerFactory.getLogger(OrderService.class);

    private final OrderRepository orderRepository;
    private final CustomerRepository customerRepository;
    private final ProductRepository productRepository;
    private final NotificationService notificationService;

    // Virtual thread executor for I/O-bound operations (Java 21+)
    private final ExecutorService virtualExecutor =
        Executors.newVirtualThreadPerTaskExecutor();

    public OrderService(
        OrderRepository orderRepository,
        CustomerRepository customerRepository,
        ProductRepository productRepository,
        NotificationService notificationService
    ) {
        this.orderRepository = orderRepository;
        this.customerRepository = customerRepository;
        this.productRepository = productRepository;
        this.notificationService = notificationService;
    }

    /**
     * Create order with async notifications using virtual threads.
     */
    @Transactional
    public OrderResponse createOrder(CreateOrderRequest request) {
        // Preserve MDC context for virtual threads
        var mdcContext = MDC.getCopyOfContextMap();

        logger.info("Creating order for customer: {}", request.customerId());

        // Validate customer exists
        var customer = customerRepository.findById(request.customerId())
            .orElseThrow(() -> new ResourceNotFoundException("Customer", request.customerId()));

        // Validate products and calculate totals
        var orderItems = request.items().stream()
            .map(item -> {
                var product = productRepository.findById(item.productId())
                    .orElseThrow(() -> new ResourceNotFoundException("Product", item.productId()));

                return new OrderItem(
                    product.getId(),
                    product.getName(),
                    item.quantity(),
                    product.getPrice()
                );
            })
            .toList();

        // Create and save order
        var order = new Order(customer.getId(), orderItems, request.notes());
        var savedOrder = orderRepository.save(order);

        // Send notifications asynchronously with virtual threads
        CompletableFuture.runAsync(() -> {
            // Restore MDC context in virtual thread
            if (mdcContext != null) {
                MDC.setContextMap(mdcContext);
            }
            try {
                notificationService.sendOrderConfirmation(customer.getEmail(), savedOrder);
                logger.info("Order confirmation sent for order: {}", savedOrder.getId());
            } catch (Exception e) {
                logger.error("Failed to send order confirmation", e);
            } finally {
                MDC.clear();
            }
        }, virtualExecutor);

        logger.info("Order created successfully: {}", savedOrder.getId());
        return mapToResponse(savedOrder);
    }

    /**
     * Get order by ID with optional caching.
     */
    @Transactional(readOnly = true)
    public Optional<OrderResponse> getOrderById(Long orderId) {
        logger.debug("Fetching order: {}", orderId);

        return orderRepository.findById(orderId)
            .map(this::mapToResponse);
    }

    /**
     * List orders with pagination.
     */
    @Transactional(readOnly = true)
    public List<OrderResponse> listOrders(int page, int size) {
        logger.debug("Listing orders: page={}, size={}", page, size);

        return orderRepository.findAllWithPagination(page, size).stream()
            .map(this::mapToResponse)
            .toList();
    }

    /**
     * Update order status.
     */
    @Transactional
    public Optional<OrderResponse> updateOrderStatus(Long orderId, String status) {
        logger.info("Updating order {} status to {}", orderId, status);

        return orderRepository.findById(orderId)
            .map(order -> {
                order.setStatus(status);
                var updated = orderRepository.save(order);
                return mapToResponse(updated);
            });
    }

    /**
     * Soft delete order.
     */
    @Transactional
    public void deleteOrder(Long orderId) {
        logger.info("Soft deleting order: {}", orderId);

        orderRepository.findById(orderId)
            .ifPresent(order -> {
                order.setDeleted(true);
                orderRepository.save(order);
            });
    }

    private OrderResponse mapToResponse(Order order) {
        // Map entity to DTO (consider using MapStruct for complex mappings)
        return new OrderResponse(
            order.getId(),
            order.getCustomerId(),
            order.getCustomerName(),
            order.getItems().stream()
                .map(item -> new OrderItemResponse(
                    item.getId(),
                    item.getProductId(),
                    item.getProductName(),
                    item.getQuantity(),
                    item.getUnitPrice(),
                    item.getTotalPrice()
                ))
                .toList(),
            order.getTotalAmount(),
            order.getStatus(),
            order.getCreatedAt(),
            order.getUpdatedAt()
        );
    }
}
```

## Resilience4j Configuration

### Good: Comprehensive Resilience Configuration

```yaml
# application.yml - Resilience4j configuration
resilience4j:
  circuitbreaker:
    instances:
      orderService:
        registerHealthIndicator: true
        slidingWindowSize: 10
        minimumNumberOfCalls: 5
        permittedNumberOfCallsInHalfOpenState: 3
        automaticTransitionFromOpenToHalfOpenEnabled: true
        waitDurationInOpenState: 10s
        failureRateThreshold: 50
        slowCallRateThreshold: 50
        slowCallDurationThreshold: 2s
        recordExceptions:
          - java.net.ConnectException
          - java.net.SocketTimeoutException
          - org.springframework.web.client.ResourceAccessException
        ignoreExceptions:
          - com.example.api.exception.BusinessException

  ratelimiter:
    instances:
      orderService:
        registerHealthIndicator: true
        limitForPeriod: 100
        limitRefreshPeriod: 1s
        timeoutDuration: 0s

  retry:
    instances:
      orderService:
        maxAttempts: 3
        waitDuration: 1s
        enableExponentialBackoff: true
        exponentialBackoffMultiplier: 2
        retryExceptions:
          - java.net.ConnectException
          - java.net.SocketTimeoutException
        ignoreExceptions:
          - com.example.api.exception.ResourceNotFoundException

  bulkhead:
    instances:
      orderService:
        maxConcurrentCalls: 25
        maxWaitDuration: 100ms

  timelimiter:
    instances:
      orderService:
        timeoutDuration: 5s
        cancelRunningFuture: true
```

## Spring Security with OAuth2/JWT

### Good: Security Configuration for Resource Server

```java
package com.example.api.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.oauth2.server.resource.authentication.JwtAuthenticationConverter;
import org.springframework.security.oauth2.server.resource.authentication.JwtGrantedAuthoritiesConverter;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.CorsConfigurationSource;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;

import java.util.List;

/**
 * Security configuration for OAuth2 Resource Server with JWT.
 */
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .cors(cors -> cors.configurationSource(corsConfigurationSource()))
            .csrf(csrf -> csrf.disable()) // Disabled for stateless REST API
            .sessionManagement(session ->
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .authorizeHttpRequests(auth -> auth
                // Public endpoints
                .requestMatchers("/actuator/health", "/actuator/info").permitAll()
                .requestMatchers("/swagger-ui/**", "/v3/api-docs/**").permitAll()

                // API endpoints require authentication
                .requestMatchers("/api/v1/orders/**").hasRole("USER")
                .requestMatchers("/api/v1/admin/**").hasRole("ADMIN")

                // All other requests require authentication
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(jwt -> jwt
                    .jwtAuthenticationConverter(jwtAuthenticationConverter())
                )
            );

        return http.build();
    }

    /**
     * Convert JWT claims to Spring Security authorities.
     */
    @Bean
    public JwtAuthenticationConverter jwtAuthenticationConverter() {
        var grantedAuthoritiesConverter = new JwtGrantedAuthoritiesConverter();
        grantedAuthoritiesConverter.setAuthoritiesClaimName("roles");
        grantedAuthoritiesConverter.setAuthorityPrefix("ROLE_");

        var jwtAuthenticationConverter = new JwtAuthenticationConverter();
        jwtAuthenticationConverter.setJwtGrantedAuthoritiesConverter(grantedAuthoritiesConverter);

        return jwtAuthenticationConverter;
    }

    /**
     * CORS configuration for cross-origin requests.
     */
    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        var configuration = new CorsConfiguration();
        configuration.setAllowedOrigins(List.of("https://example.com", "http://localhost:3000"));
        configuration.setAllowedMethods(List.of("GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS"));
        configuration.setAllowedHeaders(List.of("Authorization", "Content-Type", "X-Request-Id"));
        configuration.setExposedHeaders(List.of("X-Total-Count", "Location"));
        configuration.setAllowCredentials(true);
        configuration.setMaxAge(3600L);

        var source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/api/**", configuration);
        return source;
    }
}
```

### Good: Method-Level Security

```java
package com.example.api.controller;

import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.security.core.Authentication;
import org.springframework.web.bind.annotation.*;

/**
 * Admin controller with method-level security.
 */
@RestController
@RequestMapping("/api/v1/admin")
public class AdminController {

    /**
     * Only users with ADMIN role can access.
     */
    @GetMapping("/users")
    @PreAuthorize("hasRole('ADMIN')")
    public ResponseEntity<List<UserResponse>> listAllUsers() {
        // Implementation
    }

    /**
     * Check if user owns the resource or is admin.
     */
    @PutMapping("/orders/{orderId}")
    @PreAuthorize("hasRole('ADMIN') or @orderSecurity.isOrderOwner(authentication, #orderId)")
    public ResponseEntity<OrderResponse> updateOrder(
        @PathVariable Long orderId,
        @RequestBody UpdateOrderRequest request,
        Authentication authentication
    ) {
        // Implementation
    }
}

/**
 * Custom security expression handler.
 */
@Component("orderSecurity")
public class OrderSecurityExpressionHandler {

    private final OrderRepository orderRepository;

    public OrderSecurityExpressionHandler(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }

    public boolean isOrderOwner(Authentication authentication, Long orderId) {
        var username = authentication.getName();
        return orderRepository.findById(orderId)
            .map(order -> order.getCustomerUsername().equals(username))
            .orElse(false);
    }
}
```

## Observability with OpenTelemetry and Micrometer

### Good: Observability Configuration

```java
package com.example.api.config;

import io.micrometer.observation.ObservationRegistry;
import io.micrometer.observation.aop.ObservedAspect;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * Observability configuration for metrics and tracing.
 */
@Configuration
public class ObservabilityConfig {

    /**
     * Enable @Observed annotation support.
     */
    @Bean
    public ObservedAspect observedAspect(ObservationRegistry observationRegistry) {
        return new ObservedAspect(observationRegistry);
    }
}
```

### Good: Structured Logging with MDC

```java
package com.example.api.filter;

import jakarta.servlet.Filter;
import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.ServletRequest;
import jakarta.servlet.ServletResponse;
import jakarta.servlet.http.HttpServletRequest;
import org.slf4j.MDC;
import org.springframework.stereotype.Component;

import java.io.IOException;
import java.util.UUID;

/**
 * Filter to add correlation ID to MDC for request tracing.
 */
@Component
public class CorrelationIdFilter implements Filter {

    private static final String CORRELATION_ID_HEADER = "X-Correlation-Id";
    private static final String CORRELATION_ID_MDC_KEY = "correlationId";

    @Override
    public void doFilter(
        ServletRequest request,
        ServletResponse response,
        FilterChain chain
    ) throws IOException, ServletException {
        var httpRequest = (HttpServletRequest) request;

        // Get or generate correlation ID
        var correlationId = httpRequest.getHeader(CORRELATION_ID_HEADER);
        if (correlationId == null || correlationId.isBlank()) {
            correlationId = UUID.randomUUID().toString();
        }

        // Add to MDC for all logs in this request
        MDC.put(CORRELATION_ID_MDC_KEY, correlationId);
        MDC.put("requestUri", httpRequest.getRequestURI());
        MDC.put("method", httpRequest.getMethod());

        try {
            chain.doFilter(request, response);
        } finally {
            MDC.clear();
        }
    }
}
```

### Good: Logback Configuration with MDC

```xml
<!-- logback-spring.xml -->
<configuration>
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>

    <springProperty scope="context" name="applicationName" source="spring.application.name"/>

    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="net.logstash.logback.encoder.LogstashEncoder">
            <includeMdcKeyName>correlationId</includeMdcKeyName>
            <includeMdcKeyName>requestUri</includeMdcKeyName>
            <includeMdcKeyName>method</includeMdcKeyName>
            <customFields>{"application":"${applicationName}"}</customFields>
        </encoder>
    </appender>

    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/application.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/application-%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder class="net.logstash.logback.encoder.LogstashEncoder">
            <includeMdcKeyName>correlationId</includeMdcKeyName>
            <includeMdcKeyName>requestUri</includeMdcKeyName>
            <includeMdcKeyName>method</includeMdcKeyName>
        </encoder>
    </appender>

    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
    </root>

    <logger name="com.example.api" level="DEBUG"/>
</configuration>
```

## OpenAPI Documentation

### Good: OpenAPI Configuration

```java
package com.example.api.config;

import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Contact;
import io.swagger.v3.oas.models.info.Info;
import io.swagger.v3.oas.models.info.License;
import io.swagger.v3.oas.models.security.SecurityRequirement;
import io.swagger.v3.oas.models.security.SecurityScheme;
import io.swagger.v3.oas.models.Components;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * OpenAPI 3 documentation configuration.
 */
@Configuration
public class OpenApiConfig {

    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
            .info(new Info()
                .title("Order Management API")
                .version("v1")
                .description("REST API for order management in microservices architecture")
                .contact(new Contact()
                    .name("API Support")
                    .email("api-support@example.com")
                    .url("https://example.com/support"))
                .license(new License()
                    .name("Apache 2.0")
                    .url("https://www.apache.org/licenses/LICENSE-2.0")))
            .components(new Components()
                .addSecuritySchemes("bearer-jwt", new SecurityScheme()
                    .type(SecurityScheme.Type.HTTP)
                    .scheme("bearer")
                    .bearerFormat("JWT")
                    .description("JWT token authentication")))
            .addSecurityItem(new SecurityRequirement().addList("bearer-jwt"));
    }
}
```

## Spring Cloud Gateway Configuration

### Good: API Gateway with Circuit Breaker

```yaml
# application.yml - Spring Cloud Gateway configuration
spring:
  cloud:
    gateway:
      routes:
        # Order Service Route
        - id: order-service
          uri: lb://order-service
          predicates:
            - Path=/api/v1/orders/**
          filters:
            - name: CircuitBreaker
              args:
                name: orderServiceCircuitBreaker
                fallbackUri: forward:/fallback/orders
            - name: RequestRateLimiter
              args:
                redis-rate-limiter:
                  replenishRate: 100
                  burstCapacity: 200
            - RewritePath=/api/v1/orders/(?<segment>.*), /$\{segment}
            - AddRequestHeader=X-Gateway, SpringCloudGateway
            - AddResponseHeader=X-Response-Time, ${responseTime}

        # Customer Service Route
        - id: customer-service
          uri: lb://customer-service
          predicates:
            - Path=/api/v1/customers/**
            - Method=GET,POST,PUT,DELETE
          filters:
            - name: CircuitBreaker
              args:
                name: customerServiceCircuitBreaker
            - StripPrefix=2

      # Default filters applied to all routes
      default-filters:
        - AddRequestHeader=X-Request-Timestamp, ${requestTimestamp}
        - RemoveRequestHeader=Cookie

      # Global CORS configuration
      globalcors:
        cors-configurations:
          '[/**]':
            allowedOrigins: "https://example.com"
            allowedMethods:
              - GET
              - POST
              - PUT
              - DELETE
            allowedHeaders: "*"
            exposedHeaders:
              - X-Total-Count
              - Location
            maxAge: 3600

# Service Discovery
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
    fetchRegistry: true
    registerWithEureka: true
```

## Event-Driven Architecture with Spring Cloud Stream

### Good: Kafka Event Producer

```java
package com.example.api.event;

import com.example.api.dto.OrderCreatedEvent;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.cloud.stream.function.StreamBridge;
import org.springframework.messaging.Message;
import org.springframework.messaging.support.MessageBuilder;
import org.springframework.stereotype.Component;

/**
 * Event publisher for order domain events.
 */
@Component
public class OrderEventPublisher {

    private static final Logger logger = LoggerFactory.getLogger(OrderEventPublisher.class);
    private static final String ORDER_CREATED_BINDING = "orderCreated-out-0";

    private final StreamBridge streamBridge;

    public OrderEventPublisher(StreamBridge streamBridge) {
        this.streamBridge = streamBridge;
    }

    /**
     * Publish order created event to Kafka.
     */
    public void publishOrderCreated(OrderCreatedEvent event) {
        logger.info("Publishing order created event: orderId={}", event.orderId());

        Message<OrderCreatedEvent> message = MessageBuilder
            .withPayload(event)
            .setHeader("eventType", "ORDER_CREATED")
            .setHeader("eventVersion", "v1")
            .setHeader("correlationId", event.correlationId())
            .build();

        boolean sent = streamBridge.send(ORDER_CREATED_BINDING, message);

        if (sent) {
            logger.info("Order created event published successfully: orderId={}", event.orderId());
        } else {
            logger.error("Failed to publish order created event: orderId={}", event.orderId());
        }
    }
}

/**
 * Order created event DTO.
 */
public record OrderCreatedEvent(
    String correlationId,
    Long orderId,
    Long customerId,
    BigDecimal totalAmount,
    Instant createdAt
) {}
```

### Good: Kafka Event Consumer

```java
package com.example.api.event;

import com.example.api.dto.OrderCreatedEvent;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.Message;

import java.util.function.Consumer;

/**
 * Event consumer for order domain events.
 */
@Configuration
public class OrderEventConsumer {

    private static final Logger logger = LoggerFactory.getLogger(OrderEventConsumer.class);

    private final NotificationService notificationService;
    private final AnalyticsService analyticsService;

    public OrderEventConsumer(
        NotificationService notificationService,
        AnalyticsService analyticsService
    ) {
        this.notificationService = notificationService;
        this.analyticsService = analyticsService;
    }

    /**
     * Consume order created events from Kafka.
     */
    @Bean
    public Consumer<Message<OrderCreatedEvent>> orderCreated() {
        return message -> {
            var event = message.getPayload();
            var correlationId = message.getHeaders().get("correlationId", String.class);

            logger.info("Received order created event: orderId={}, correlationId={}",
                event.orderId(), correlationId);

            try {
                // Process event
                notificationService.sendOrderNotification(event);
                analyticsService.recordOrderCreated(event);

                logger.info("Order created event processed successfully: orderId={}",
                    event.orderId());
            } catch (Exception e) {
                logger.error("Failed to process order created event: orderId={}",
                    event.orderId(), e);
                throw e; // Trigger retry or DLQ
            }
        };
    }
}
```

### Good: Spring Cloud Stream Configuration

```yaml
# application.yml - Spring Cloud Stream + Kafka configuration
spring:
  cloud:
    stream:
      # Kafka binder configuration
      kafka:
        binder:
          brokers: localhost:9092
          auto-create-topics: true
          configuration:
            key:
              serializer: org.apache.kafka.common.serialization.StringSerializer
            value:
              serializer: org.springframework.kafka.support.serializer.JsonSerializer

      # Function bindings
      function:
        definition: orderCreated

      # Binding configurations
      bindings:
        # Producer binding
        orderCreated-out-0:
          destination: order.events
          content-type: application/json
          producer:
            partition-key-expression: headers['orderId']

        # Consumer binding
        orderCreated-in-0:
          destination: order.events
          content-type: application/json
          group: notification-service
          consumer:
            max-attempts: 3
            back-off-initial-interval: 1000
            back-off-multiplier: 2.0
```

## Common Anti-Patterns

### Anti-Pattern: Field Injection

```java
// DON'T: Field injection is hard to test and hides dependencies
@RestController
public class OrderController {
    @Autowired
    private OrderService orderService; // Bad: mutable, hard to test

    @Autowired
    private CustomerService customerService;
}

// DO: Constructor injection for immutability
@RestController
public class OrderController {
    private final OrderService orderService;
    private final CustomerService customerService;

    public OrderController(
        OrderService orderService,
        CustomerService customerService
    ) {
        this.orderService = orderService;
        this.customerService = customerService;
    }
}
```

### Anti-Pattern: Missing Error Handling

```java
// DON'T: Let exceptions propagate without proper handling
@GetMapping("/{id}")
public Order getOrder(@PathVariable Long id) {
    return orderRepository.findById(id).get(); // Throws NoSuchElementException
}

// DO: Handle errors gracefully with proper HTTP status
@GetMapping("/{id}")
public ResponseEntity<OrderResponse> getOrder(@PathVariable Long id) {
    return orderRepository.findById(id)
        .map(this::mapToResponse)
        .map(ResponseEntity::ok)
        .orElse(ResponseEntity.notFound().build());
}
```

### Anti-Pattern: No API Versioning

```java
// DON'T: No versioning strategy
@RestController
@RequestMapping("/orders") // Will break clients when API changes
public class OrderController {
}

// DO: Include version in path
@RestController
@RequestMapping("/api/v1/orders") // Clear versioning
public class OrderController {
}
```

### Anti-Pattern: Exposing Entities Directly

```java
// DON'T: Return JPA entities directly
@GetMapping("/{id}")
public Order getOrder(@PathVariable Long id) {
    return orderRepository.findById(id).get(); // Exposes internal structure
}

// DO: Use DTOs to decouple API from persistence
@GetMapping("/{id}")
public ResponseEntity<OrderResponse> getOrder(@PathVariable Long id) {
    return orderRepository.findById(id)
        .map(this::mapToResponse) // Map to DTO
        .map(ResponseEntity::ok)
        .orElse(ResponseEntity.notFound().build());
}
```

### Anti-Pattern: Missing Resilience Patterns

```java
// DON'T: No circuit breaker or timeout protection
@GetMapping("/{id}")
public OrderResponse getOrder(@PathVariable Long id) {
    // Can hang indefinitely if downstream service is slow
    return externalService.fetchOrder(id);
}

// DO: Add circuit breaker and rate limiting
@GetMapping("/{id}")
@CircuitBreaker(name = "orderService", fallbackMethod = "getOrderFallback")
@RateLimiter(name = "orderService")
public ResponseEntity<OrderResponse> getOrder(@PathVariable Long id) {
    return ResponseEntity.ok(externalService.fetchOrder(id));
}

private ResponseEntity<OrderResponse> getOrderFallback(Long id, Exception ex) {
    logger.error("Circuit breaker fallback for order: {}", id, ex);
    return ResponseEntity.status(HttpStatus.SERVICE_UNAVAILABLE).build();
}
```

## Quality Indicators

- All endpoints have proper HTTP status codes (200, 201, 204, 400, 404, 500)
- DTOs use records for immutability and conciseness
- Global exception handling with @RestControllerAdvice and RFC 7807 Problem Details
- API versioning: /api/v1/resource pattern
- Constructor injection throughout (NEVER field injection)
- Resilience patterns: circuit breakers, rate limiting, timeouts on all external calls
- Comprehensive validation with @Valid and Jakarta Bean Validation
- Security: OAuth2 Resource Server with JWT, role-based access control
- Observability: @Observed annotations, MDC logging, correlation IDs
- OpenAPI 3 documentation auto-generated
- Virtual threads for I/O-bound operations (Java 21+)
- Event-driven integration with Spring Cloud Stream + Kafka
- Comprehensive test coverage: @WebMvcTest, @DataJpaTest, integration tests

## Supporting Components

- **Implementation Guidance**: `.claude/agents/java/java-api-engineer.md`
- **General Java Standards**: `.claude/context/java/java-standards.md`
- **Spring Boot Standards**: `.claude/context/java/spring-boot-standards.md`
- **Testing Patterns**: `.claude/context/java/testing-standards.md`
- **Quality Validation**: `/java:lint-all` and `/java:test-coverage` commands
