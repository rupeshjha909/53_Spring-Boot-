# Dependency Injection: Complete Interview Guide

A comprehensive guide covering Dependency Injection concepts, implementation patterns, constructor vs setter injection, and common interview questions with practical Spring examples.

## Table of Contents
- [Core Concepts](#core-concepts)
- [Types of Dependency Injection](#types-of-dependency-injection)
- [Constructor vs Setter vs Field Injection](#constructor-vs-setter-vs-field-injection)
- [When to Use Which Type](#when-to-use-which-type)
- [Spring DI Implementation](#spring-di-implementation)
- [Best Practices](#best-practices)
- [Common Interview Questions](#common-interview-questions)
- [Advanced Topics](#advanced-topics)
- [Anti-Patterns to Avoid](#anti-patterns-to-avoid)

---

## Core Concepts

### 🎯 What is Dependency Injection?

**Definition**: Dependency Injection (DI) is a design pattern where objects receive their dependencies from external sources rather than creating them internally. It's a technique for achieving Inversion of Control (IoC).

### The Problem DI Solves

```java
// Without DI - Tight Coupling (BAD)
public class UserService {
    private final EmailService emailService;
    private final UserRepository userRepository;
    
    public UserService() {
        // Hard-coded dependencies - tight coupling
        this.emailService = new EmailService();
        this.userRepository = new DatabaseUserRepository();
    }
    
    public void createUser(User user) {
        userRepository.save(user);
        emailService.sendWelcomeEmail(user);
    }
}

// Problems with above approach:
// 1. Hard to test (cannot mock dependencies)
// 2. Hard to change implementation
// 3. Violates Single Responsibility Principle
// 4. Tight coupling between classes
```

```java
// With DI - Loose Coupling (GOOD)
public class UserService {
    private final EmailService emailService;
    private final UserRepository userRepository;
    
    // Dependencies injected via constructor
    public UserService(EmailService emailService, UserRepository userRepository) {
        this.emailService = emailService;
        this.userRepository = userRepository;
    }
    
    public void createUser(User user) {
        userRepository.save(user);
        emailService.sendWelcomeEmail(user);
    }
}

// Benefits:
// 1. Easy to test with mock dependencies
// 2. Implementation can be changed without modifying UserService
// 3. Follows Dependency Inversion Principle
// 4. Loose coupling between classes
```

### 🏗️ Key Benefits of DI

1. **Testability**: Easy to inject mock dependencies for unit testing
2. **Loose Coupling**: Classes depend on abstractions, not concrete implementations
3. **Flexibility**: Easy to swap implementations without code changes
4. **Single Responsibility**: Classes focus on business logic, not object creation
5. **Configuration Management**: Centralized dependency configuration

---

## Types of Dependency Injection

### 1. **Constructor Injection** 🏗️

```java
@Service
public class OrderService {
    private final PaymentService paymentService;
    private final InventoryService inventoryService;
    private final EmailService emailService;
    
    // All dependencies injected via constructor
    public OrderService(PaymentService paymentService, 
                       InventoryService inventoryService,
                       EmailService emailService) {
        this.paymentService = paymentService;
        this.inventoryService = inventoryService;
        this.emailService = emailService;
    }
    
    public void processOrder(Order order) {
        // Check inventory
        if (!inventoryService.isAvailable(order.getItems())) {
            throw new OutOfStockException();
        }
        
        // Process payment
        PaymentResult result = paymentService.processPayment(order.getTotal());
        
        if (result.isSuccess()) {
            inventoryService.reserveItems(order.getItems());
            emailService.sendOrderConfirmation(order);
        }
    }
}
```

**Characteristics**:
- Dependencies are required and immutable
- Object is fully initialized after construction
- Supports final fields
- Fails fast if dependencies are missing

### 2. **Setter Injection** ⚙️

```java
@Service
public class NotificationService {
    private EmailProvider emailProvider;
    private SmsProvider smsProvider;
    private PushNotificationProvider pushProvider;
    
    // Default constructor
    public NotificationService() {}
    
    // Setter methods for dependency injection
    @Autowired
    public void setEmailProvider(EmailProvider emailProvider) {
        this.emailProvider = emailProvider;
    }
    
    @Autowired
    public void setSmsProvider(SmsProvider smsProvider) {
        this.smsProvider = smsProvider;
    }
    
    @Autowired(required = false)  // Optional dependency
    public void setPushProvider(PushNotificationProvider pushProvider) {
        this.pushProvider = pushProvider;
    }
    
    public void sendNotification(String message, User user, NotificationType type) {
        switch (type) {
            case EMAIL:
                if (emailProvider != null) {
                    emailProvider.send(message, user.getEmail());
                }
                break;
            case SMS:
                if (smsProvider != null) {
                    smsProvider.send(message, user.getPhone());
                }
                break;
            case PUSH:
                if (pushProvider != null) {
                    pushProvider.send(message, user.getDeviceId());
                }
                break;
        }
    }
}
```

**Characteristics**:
- Dependencies can be optional
- Allows partial dependency injection
- Object can be created without all dependencies
- Dependencies can be changed after object creation

### 3. **Field Injection** 🔌

```java
@Service
public class ReportService {
    @Autowired
    private DataService dataService;
    
    @Autowired
    private TemplateEngine templateEngine;
    
    @Autowired
    private FileStorageService fileStorageService;
    
    public Report generateReport(ReportRequest request) {
        // Get data
        List<Data> data = dataService.fetchData(request.getCriteria());
        
        // Generate report
        String content = templateEngine.process(request.getTemplate(), data);
        
        // Save report
        String fileUrl = fileStorageService.save(content, request.getFileName());
        
        return new Report(fileUrl, content.length());
    }
}
```

**Characteristics**:
- Most concise syntax
- Uses reflection (slower)
- Hard to unit test
- Violates encapsulation
- **Generally not recommended**

### 4. **Method Injection** 📋

```java
@Service
public class CacheService {
    private CacheProvider cacheProvider;
    
    // Method injection - can have multiple parameters
    @Autowired
    public void initializeCacheProviders(CacheProvider cacheProvider, 
                                       @Qualifier("redis") CacheProvider redisProvider,
                                       @Value("${cache.ttl}") long ttl) {
        this.cacheProvider = cacheProvider;
        this.cacheProvider.setTtl(ttl);
        
        // Additional initialization logic
        if (redisProvider.isAvailable()) {
            this.cacheProvider = redisProvider;
        }
    }
    
    public <T> void put(String key, T value) {
        cacheProvider.put(key, value);
    }
    
    public <T> T get(String key, Class<T> type) {
        return cacheProvider.get(key, type);
    }
}
```

**Characteristics**:
- Can have complex initialization logic
- Can inject multiple dependencies in one method
- Less common than constructor/setter injection
- Useful for configuration-based initialization

---

## Constructor vs Setter vs Field Injection

### Detailed Comparison Table

| Aspect | Constructor Injection | Setter Injection | Field Injection |
|--------|----------------------|------------------|-----------------|
| **Immutability** | ✅ Supports final fields | ❌ Cannot use final | ❌ Cannot use final |
| **Required Dependencies** | ✅ All dependencies required | ❌ Dependencies optional | ⚠️ Depends on @Autowired(required) |
| **Partial Injection** | ❌ All or nothing | ✅ Can inject selectively | ✅ Can inject selectively |
| **Testability** | ✅ Easy to test with mocks | ✅ Easy to test | ❌ Requires reflection/Spring |
| **Fail-Fast** | ✅ Fails at creation time | ❌ May fail at runtime | ❌ May fail at runtime |
| **Circular Dependencies** | ❌ Prevents circular deps | ✅ Allows circular deps | ✅ Allows circular deps |
| **Code Verbosity** | ⚠️ More verbose | ⚠️ Moderate verbosity | ✅ Least verbose |
| **Encapsulation** | ✅ Good encapsulation | ✅ Good encapsulation | ❌ Breaks encapsulation |
| **Spring Recommendation** | ✅ **Recommended** | ✅ For optional deps | ❌ **Not recommended** |

### Real-World Examples

#### Constructor Injection Example (Recommended)
```java
@Service
public class PaymentProcessingService {
    private final PaymentGateway paymentGateway;
    private final AuditLogger auditLogger;
    private final FraudDetectionService fraudService;
    
    // Constructor injection - all dependencies required
    public PaymentProcessingService(PaymentGateway paymentGateway,
                                   AuditLogger auditLogger,
                                   FraudDetectionService fraudService) {
        this.paymentGateway = Objects.requireNonNull(paymentGateway);
        this.auditLogger = Objects.requireNonNull(auditLogger);
        this.fraudService = Objects.requireNonNull(fraudService);
    }
    
    public PaymentResult processPayment(PaymentRequest request) {
        // All dependencies guaranteed to be available
        auditLogger.logPaymentAttempt(request);
        
        if (!fraudService.isLegitimate(request)) {
            auditLogger.logFraudAttempt(request);
            return PaymentResult.rejected("Fraud detected");
        }
        
        PaymentResult result = paymentGateway.process(request);
        auditLogger.logPaymentResult(request, result);
        
        return result;
    }
}

// Easy unit testing
@Test
void shouldProcessPaymentSuccessfully() {
    // Arrange
    PaymentGateway mockGateway = mock(PaymentGateway.class);
    AuditLogger mockLogger = mock(AuditLogger.class);
    FraudDetectionService mockFraud = mock(FraudDetectionService.class);
    
    PaymentProcessingService service = new PaymentProcessingService(
        mockGateway, mockLogger, mockFraud);
    
    // Test with mocked dependencies - no Spring context needed
}
```

#### Setter Injection Example (Optional Dependencies)
```java
@Service
public class EmailNotificationService {
    private EmailTemplateService templateService;
    private EmailDeliveryService deliveryService;
    private EmailAnalyticsService analyticsService;  // Optional
    
    // Required dependencies via setter
    @Autowired
    public void setTemplateService(EmailTemplateService templateService) {
        this.templateService = templateService;
    }
    
    @Autowired
    public void setDeliveryService(EmailDeliveryService deliveryService) {
        this.deliveryService = deliveryService;
    }
    
    // Optional dependency
    @Autowired(required = false)
    public void setAnalyticsService(EmailAnalyticsService analyticsService) {
        this.analyticsService = analyticsService;
    }
    
    public void sendEmail(String to, String subject, Map<String, Object> data) {
        // Required dependencies must be checked
        if (templateService == null || deliveryService == null) {
            throw new IllegalStateException("Required dependencies not injected");
        }
        
        String content = templateService.generateContent(subject, data);
        EmailResult result = deliveryService.send(to, subject, content);
        
        // Optional analytics
        if (analyticsService != null) {
            analyticsService.trackEmailSent(to, subject, result.isSuccess());
        }
    }
}
```

#### Field Injection Example (Not Recommended)
```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;  // Breaks encapsulation
    
    @Autowired
    private PasswordEncoder passwordEncoder;  // Hard to test
    
    public User createUser(String username, String password) {
        // Dependencies might be null if injection fails
        String encodedPassword = passwordEncoder.encode(password);
        User user = new User(username, encodedPassword);
        return userRepository.save(user);
    }
}

// Testing problems with field injection
@Test
void testUserCreation() {
    UserService service = new UserService();
    
    // Cannot easily inject mock dependencies
    // Need to use reflection or Spring test context
    ReflectionTestUtils.setField(service, "userRepository", mockRepository);
    ReflectionTestUtils.setField(service, "passwordEncoder", mockEncoder);
    
    // Test logic...
}
```

---

## When to Use Which Type

### Decision Matrix

```java
// Use CONSTRUCTOR injection when:
// ✅ Dependencies are REQUIRED for object to function
// ✅ Dependencies should be IMMUTABLE
// ✅ You want FAIL-FAST behavior
// ✅ Object should be FULLY INITIALIZED after construction

@Service
public class BankingService {
    private final AccountRepository accountRepository;  // Required
    private final TransactionLogger transactionLogger;  // Required
    
    public BankingService(AccountRepository accountRepository,
                         TransactionLogger transactionLogger) {
        this.accountRepository = accountRepository;
        this.transactionLogger = transactionLogger;
    }
    // Banking service cannot function without these dependencies
}

// Use SETTER injection when:
// ✅ Dependencies are OPTIONAL
// ✅ You need DEFAULT behavior when dependency is missing
// ✅ Dependencies might need to be CHANGED after creation
// ✅ Handling CIRCULAR dependencies (rare, usually indicates design issue)

@Service
public class NotificationDispatcher {
    private EmailService emailService;
    private SmsService smsService;
    private SlackService slackService;  // Optional - not all companies use Slack
    
    @Autowired(required = false)
    public void setSlackService(SlackService slackService) {
        this.slackService = slackService;
    }
    
    public void sendNotification(String message, NotificationChannel... channels) {
        for (NotificationChannel channel : channels) {
            switch (channel) {
                case SLACK:
                    if (slackService != null) {  // Graceful degradation
                        slackService.send(message);
                    }
                    break;
                // ... other channels
            }
        }
    }
}

// AVOID FIELD injection because:
// ❌ Hard to unit test without Spring context
// ❌ Breaks encapsulation
// ❌ Violates explicit dependency principle
// ❌ Can lead to NullPointerException if injection fails
// ❌ Makes dependencies hidden from class interface
```

### Real-World Scenarios

#### Scenario 1: Core Service Dependencies
```java
// Core business service - use constructor injection
@Service
public class OrderFulfillmentService {
    private final InventoryService inventoryService;    // Always needed
    private final PaymentService paymentService;        // Always needed
    private final ShippingService shippingService;      // Always needed
    
    public OrderFulfillmentService(InventoryService inventoryService,
                                  PaymentService paymentService,
                                  ShippingService shippingService) {
        this.inventoryService = inventoryService;
        this.paymentService = paymentService;
        this.shippingService = shippingService;
    }
    
    // Service cannot function without all three dependencies
}
```

#### Scenario 2: Feature Flags and Optional Services
```java
@Service
public class ProductRecommendationService {
    private RecommendationEngine basicEngine;
    private AIRecommendationEngine aiEngine;  // Optional - premium feature
    private UserBehaviorTracker tracker;      // Optional - analytics feature
    
    @Autowired
    public void setBasicEngine(RecommendationEngine basicEngine) {
        this.basicEngine = basicEngine;
    }
    
    @Autowired(required = false)
    public void setAiEngine(AIRecommendationEngine aiEngine) {
        this.aiEngine = aiEngine;
    }
    
    @Autowired(required = false)
    public void setTracker(UserBehaviorTracker tracker) {
        this.tracker = tracker;
    }
    
    public List<Product> recommend(User user) {
        List<Product> recommendations;
        
        // Use AI engine if available (premium feature)
        if (aiEngine != null && user.isPremiumMember()) {
            recommendations = aiEngine.recommend(user);
        } else {
            recommendations = basicEngine.recommend(user);
        }
        
        // Track behavior if analytics enabled
        if (tracker != null) {
            tracker.trackRecommendationView(user, recommendations);
        }
        
        return recommendations;
    }
}
```

#### Scenario 3: Mixed Injection Strategy
```java
@Service
public class ComprehensiveUserService {
    // Required dependencies - constructor injection
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    private final EmailValidator emailValidator;
    
    // Optional dependencies - setter injection
    private UserActivityTracker activityTracker;
    private SocialMediaIntegration socialIntegration;
    
    // Constructor for required dependencies
    public ComprehensiveUserService(UserRepository userRepository,
                                   PasswordEncoder passwordEncoder,
                                   EmailValidator emailValidator) {
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
        this.emailValidator = emailValidator;
    }
    
    // Optional dependency setters
    @Autowired(required = false)
    public void setActivityTracker(UserActivityTracker activityTracker) {
        this.activityTracker = activityTracker;
    }
    
    @Autowired(required = false)
    public void setSocialIntegration(SocialMediaIntegration socialIntegration) {
        this.socialIntegration = socialIntegration;
    }
    
    public User createUser(UserRegistrationRequest request) {
        // Required validation
        if (!emailValidator.isValid(request.getEmail())) {
            throw new InvalidEmailException();
        }
        
        // Create user with required components
        String encodedPassword = passwordEncoder.encode(request.getPassword());
        User user = new User(request.getEmail(), encodedPassword);
        User savedUser = userRepository.save(user);
        
        // Optional features
        if (activityTracker != null) {
            activityTracker.trackUserRegistration(savedUser);
        }
        
        if (socialIntegration != null && request.getSocialProfiles() != null) {
            socialIntegration.linkProfiles(savedUser, request.getSocialProfiles());
        }
        
        return savedUser;
    }
}
```

---

## Spring DI Implementation

### Configuration Options

#### 1. **Annotation-Based Configuration**
```java
@Configuration
@ComponentScan("com.example")
@EnableAutoConfiguration
public class ApplicationConfig {
    // Beans will be auto-discovered and auto-wired
}

@Service
public class BookService {
    private final BookRepository bookRepository;
    private final AuthorService authorService;
    
    // Spring auto-detects constructor and injects dependencies
    public BookService(BookRepository bookRepository, AuthorService authorService) {
        this.bookRepository = bookRepository;
        this.authorService = authorService;
    }
}
```

#### 2. **Explicit Bean Configuration**
```java
@Configuration
public class DatabaseConfig {
    
    @Bean
    @Primary
    public DataSource primaryDataSource() {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl("jdbc:postgresql://localhost/primary");
        return dataSource;
    }
    
    @Bean
    @Qualifier("readonly")
    public DataSource readOnlyDataSource() {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl("jdbc:postgresql://localhost/readonly");
        return dataSource;
    }
    
    @Bean
    public UserRepository userRepository(@Qualifier("readonly") DataSource dataSource) {
        return new JdbcUserRepository(dataSource);
    }
}
```

#### 3. **Mixed Configuration Strategy**
```java
@Service
public class OrderService {
    private final PaymentService paymentService;
    private final ShippingService shippingService;
    private final AuditService auditService;
    
    // Constructor injection for required dependencies
    public OrderService(PaymentService paymentService, 
                       ShippingService shippingService) {
        this.paymentService = paymentService;
        this.shippingService = shippingService;
    }
    
    // Setter injection for optional dependency with custom configuration
    @Autowired(required = false)
    @Qualifier("orderAuditService")
    public void setAuditService(AuditService auditService) {
        this.auditService = auditService;
    }
    
    public Order processOrder(OrderRequest request) {
        Order order = new Order(request);
        
        // Required operations
        PaymentResult payment = paymentService.processPayment(order);
        ShippingResult shipping = shippingService.arrangeShipping(order);
        
        order.setPaymentResult(payment);
        order.setShippingResult(shipping);
        
        // Optional audit
        if (auditService != null) {
            auditService.auditOrderProcessing(order);
        }
        
        return order;
    }
}
```

### Handling Multiple Implementations

#### Using @Qualifier
```java
// Multiple implementations of the same interface
@Service
@Qualifier("emailNotification")
public class EmailNotificationService implements NotificationService {
    public void send(String message, String recipient) {
        // Email implementation
    }
}

@Service
@Qualifier("smsNotification") 
public class SmsNotificationService implements NotificationService {
    public void send(String message, String recipient) {
        // SMS implementation
    }
}

// Injecting specific implementation
@Service
public class UserRegistrationService {
    private final NotificationService emailNotification;
    private final NotificationService smsNotification;
    
    public UserRegistrationService(@Qualifier("emailNotification") NotificationService emailNotification,
                                  @Qualifier("smsNotification") NotificationService smsNotification) {
        this.emailNotification = emailNotification;
        this.smsNotification = smsNotification;
    }
    
    public void registerUser(User user, NotificationPreference preference) {
        // Register user logic...
        
        switch (preference) {
            case EMAIL:
                emailNotification.send("Welcome!", user.getEmail());
                break;
            case SMS:
                smsNotification.send("Welcome!", user.getPhone());
                break;
            case BOTH:
                emailNotification.send("Welcome!", user.getEmail());
                smsNotification.send("Welcome!", user.getPhone());
                break;
        }
    }
}
```

#### Using @Primary and Collections
```java
@Service
@Primary
public class PrimaryPaymentGateway implements PaymentGateway {
    // Primary implementation
}

@Service
public class BackupPaymentGateway implements PaymentGateway {
    // Backup implementation
}

@Service
public class PaymentService {
    private final PaymentGateway primaryGateway;
    private final List<PaymentGateway> allGateways;
    
    public PaymentService(PaymentGateway primaryGateway,           // Injects @Primary
                         List<PaymentGateway> allGateways) {       // Injects all implementations
        this.primaryGateway = primaryGateway;
        this.allGateways = allGateways;
    }
    
    public PaymentResult processPayment(PaymentRequest request) {
        try {
            return primaryGateway.process(request);
        } catch (PaymentException e) {
            // Fallback to other gateways
            for (PaymentGateway gateway : allGateways) {
                if (gateway != primaryGateway) {
                    try {
                        return gateway.process(request);
                    } catch (PaymentException fallbackException) {
                        // Log and continue to next gateway
                    }
                }
            }
            throw new AllPaymentGatewaysFailedException();
        }
    }
}
```

---

## Best Practices

### 1. **Prefer Constructor Injection**
```java
// ✅ GOOD - Constructor injection
@Service
public class UserService {
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    
    public UserService(UserRepository userRepository, PasswordEncoder passwordEncoder) {
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
    }
    
    // Benefits:
    // - Immutable dependencies (final fields)
    // - Fail-fast if dependencies missing
    // - Easy to unit test
    // - Clear dependency requirements
}

// ❌ BAD - Field injection
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;  // Mutable, hard to test
    
    @Autowired 
    private PasswordEncoder passwordEncoder;  // Hidden dependencies
    
    // Problems:
    // - Hard to unit test without Spring
    // - Dependencies not clear from class interface
    // - Violates encapsulation
    // - May fail at runtime if injection fails
}
```

### 2. **Validate Dependencies**
```java
@Service
public class SecureUserService {
    private final UserRepository userRepository;
    private final EncryptionService encryptionService;
    
    public SecureUserService(UserRepository userRepository, EncryptionService encryptionService) {
        // Validate required dependencies
        this.userRepository = Objects.requireNonNull(userRepository, "UserRepository cannot be null");
        this.encryptionService = Objects.requireNonNull(encryptionService, "EncryptionService cannot be null");
        
        // Additional validation if needed
        if (!encryptionService.isInitialized()) {
            throw new IllegalStateException("EncryptionService must be initialized");
        }
    }
}
```

### 3. **Keep Constructor Parameters Reasonable**
```java
// ❌ BAD - Too many dependencies (code smell)
public class MonolithicService {
    public MonolithicService(ServiceA a, ServiceB b, ServiceC c, ServiceD d, 
                           ServiceE e, ServiceF f, ServiceG g, ServiceH h) {
        // Too many dependencies suggests violation of Single Responsibility
    }
}

// ✅ GOOD - Refactored into focused services
@Service
public class UserService {
    private final UserRepository userRepository;
    private final UserValidator userValidator;
    private final UserEventPublisher eventPublisher;
    
    public UserService(UserRepository userRepository,
                      UserValidator userValidator,
                      UserEventPublisher eventPublisher) {
        this.userRepository = userRepository;
        this.userValidator = userValidator;
        this.eventPublisher = eventPublisher;
    }
}

@Service
public class UserNotificationService {
    private final EmailService emailService;
    private final SmsService smsService;
    
    public UserNotificationService(EmailService emailService, SmsService smsService) {
        this.emailService = emailService;
        this.smsService = smsService;
    }
}
```

### 4. **Handle Optional Dependencies Gracefully**
```java
@Service
public class ComprehensiveService {
    private final RequiredService requiredService;
    private final OptionalService optionalService;
    
    public ComprehensiveService(RequiredService requiredService,
                               @Autowired(required = false) OptionalService optionalService) {
        this.requiredService = requiredService;
        this.optionalService = optionalService;  // May be null
    }
    
    public void performOperation() {
        // Always available
        requiredService.doRequiredWork();
        
        // Check before using optional service
        if (optionalService != null) {
            optionalService.doOptionalWork();
        } else {
            // Provide default behavior or skip optional functionality
            logger.info("Optional service not available, skipping optional work");
        }
    }
}
```

### 5. **Use Interfaces for Loose Coupling**
```java
// Define interfaces for dependencies
public interface PaymentGateway {
    PaymentResult process(PaymentRequest request);
}

public interface NotificationService {
    void send(String message, String recipient);
}

// Depend on interfaces, not concrete classes
@Service
public class OrderService {
    private final PaymentGateway paymentGateway;        // Interface
    private final NotificationService notificationService;  // Interface
    
    public OrderService(PaymentGateway paymentGateway,
                       NotificationService notificationService) {
        this.paymentGateway = paymentGateway;
        this.notificationService = notificationService;
    }
    
    // Benefits:
    // - Easy to swap implementations
    // - Easy to mock for testing
    // - Loose coupling
    // - Follows Dependency Inversion Principle
}
```

---

## Common Interview Questions

### Q1: What is Dependency Injection and why is it important?
**Answer**: Dependency Injection is a design pattern where objects receive their dependencies from external sources rather than creating them internally. It's important because:

- **Testability**: Easy to inject mock dependencies for unit testing
- **Loose Coupling**: Classes depend on abstractions, not concrete implementations
- **Single Responsibility**: Classes focus on business logic, not object creation
- **Flexibility**: Easy to change implementations without modifying dependent classes

```java
// Without DI - Hard to test and tightly coupled
public class OrderService {
    private EmailService emailService = new EmailService(); // Hard-coded dependency
    
    public void processOrder(Order order) {
        // Cannot easily mock EmailService for testing
        emailService.sendConfirmation(order);
    }
}

// With DI - Easy to test and loosely coupled
public class OrderService {
    private final EmailService emailService;
    
    public OrderService(EmailService emailService) {
        this.emailService = emailService; // Dependency injected
    }
    
    public void processOrder(Order order) {
        emailService.sendConfirmation(order);
    }
}
```

### Q2: What are the different types of dependency injection?
**Answer**: There are three main types:

1. **Constructor Injection**: Dependencies passed via constructor parameters
2. **Setter Injection**: Dependencies set via setter methods
3. **Field Injection**: Dependencies injected directly into fields using annotations

```java
// Constructor Injection (Recommended)
@Service
public class UserService {
    private final UserRepository repository;
    
    public UserService(UserRepository repository) {
        this.repository = repository;
    }
}

// Setter Injection
@Service
public class UserService {
    private UserRepository repository;
    
    @Autowired
    public void setRepository(UserRepository repository) {
        this.repository = repository;
    }
}

// Field Injection (Not recommended)
@Service
public class UserService {
    @Autowired
    private UserRepository repository;
}
```

### Q3: When should you use constructor injection vs setter injection?
**Answer**:

**Use Constructor Injection when**:
- Dependencies are required for the object to function
- You want immutable dependencies (final fields)
- You want fail-fast behavior
- Dependencies should not change after object creation

**Use Setter Injection when**:
- Dependencies are optional
- You need default behavior when dependencies are missing
- You need to change dependencies after object creation
- Handling circular dependencies (though this usually indicates a design issue)

```java
// Constructor for required dependencies
public class PaymentService {
    private final PaymentGateway gateway;  // Required, immutable
    
    public PaymentService(PaymentGateway gateway) {
        this.gateway = Objects.requireNonNull(gateway);
    }
}

// Setter for optional dependencies
public class NotificationService {
    private SlackIntegration slackIntegration;  // Optional
    
    @Autowired(required = false)
    public void setSlackIntegration(SlackIntegration slack) {
        this.slackIntegration = slack;
    }
    
    public void sendNotification(String message) {
        // Graceful degradation if optional dependency missing
        if (slackIntegration != null) {
            slackIntegration.send(message);
        }
    }
}
```

### Q4: What are the disadvantages of field injection?
**Answer**: Field injection has several problems:

1. **Hard to Unit Test**: Requires Spring context or reflection to inject mocks
2. **Breaks Encapsulation**: Makes private fields accessible to framework
3. **Hidden Dependencies**: Dependencies not visible in class interface
4. **Immutability**: Cannot use final fields
5. **Runtime Failures**: May fail silently if injection fails

```java
// Field injection problems
@Service
public class UserService {
    @Autowired
    private UserRepository repository;  // Hidden dependency, not final
    
    public User findUser(Long id) {
        return repository.findById(id);  // May throw NPE if injection failed
    }
}

// Testing problems
@Test
void testUserService() {
    UserService service = new UserService();
    // Cannot easily inject mock repository
    // Need to use reflection or Spring test context
    ReflectionTestUtils.setField(service, "repository", mockRepository);
}

// Constructor injection solution
@Service
public class UserService {
    private final UserRepository repository;  // Visible, immutable
    
    public UserService(UserRepository repository) {
        this.repository = repository;  // Clear dependency
    }
}

// Easy testing
@Test
void testUserService() {
    UserRepository mockRepo = mock(UserRepository.class);
    UserService service = new UserService(mockRepo);  // Easy mock injection
}
```

### Q5: How do you handle circular dependencies?
**Answer**: Circular dependencies occur when two or more beans depend on each other. Solutions:

1. **Refactor Design** (Best approach - eliminate circular dependency)
2. **Use @Lazy annotation**
3. **Use setter injection instead of constructor injection**
4. **Use @PostConstruct for initialization**

```java
// Circular dependency problem
@Service
public class ServiceA {
    private final ServiceB serviceB;
    
    public ServiceA(ServiceB serviceB) { // ServiceB depends on ServiceA
        this.serviceB = serviceB;
    }
}

@Service
public class ServiceB {
    private final ServiceA serviceA;
    
    public ServiceB(ServiceA serviceA) { // Creates circular dependency
        this.serviceA = serviceA;
    }
}

// Solution 1: Refactor design (Best)
@Service
public class ServiceA {
    private final CommonService commonService;
    
    public ServiceA(CommonService commonService) {
        this.commonService = commonService;
    }
}

@Service
public class ServiceB {
    private final CommonService commonService;
    
    public ServiceB(CommonService commonService) {
        this.commonService = commonService;
    }
}

// Solution 2: Use @Lazy
@Service
public class ServiceA {
    private final ServiceB serviceB;
    
    public ServiceA(@Lazy ServiceB serviceB) {
        this.serviceB = serviceB;
    }
}

// Solution 3: Setter injection
@Service
public class ServiceA {
    private ServiceB serviceB;
    
    @Autowired
    public void setServiceB(ServiceB serviceB) {
        this.serviceB = serviceB;
    }
}
```

### Q6: How do you inject multiple implementations of the same interface?
**Answer**: Several approaches:

1. **@Qualifier annotation**
2. **@Primary annotation** 
3. **Inject collections (List, Map)**
4. **Use @Profile for environment-specific beans**

```java
// Multiple implementations
@Service
@Qualifier("email")
public class EmailNotificationService implements NotificationService { }

@Service
@Qualifier("sms")
public class SmsNotificationService implements NotificationService { }

// Using @Qualifier
@Service
public class UserService {
    private final NotificationService emailService;
    private final NotificationService smsService;
    
    public UserService(@Qualifier("email") NotificationService emailService,
                      @Qualifier("sms") NotificationService smsService) {
        this.emailService = emailService;
        this.smsService = smsService;
    }
}

// Using collections
@Service
public class NotificationDispatcher {
    private final List<NotificationService> allServices;
    private final Map<String, NotificationService> serviceMap;
    
    public NotificationDispatcher(List<NotificationService> allServices,
                                 Map<String, NotificationService> serviceMap) {
        this.allServices = allServices;      // All implementations
        this.serviceMap = serviceMap;        // Bean name -> implementation
    }
    
    public void sendToAll(String message) {
        allServices.forEach(service -> service.send(message));
    }
}
```

### Q7: What is the difference between @Autowired and @Resource?
**Answer**: 

| Aspect | @Autowired | @Resource |
|--------|------------|-----------|
| **Origin** | Spring-specific | Java standard (JSR-250) |
| **Matching** | By type first, then by name | By name first, then by type |
| **Required** | Can be optional with required=false | Always required |
| **Usage** | Constructor, setter, field | Setter and field only |

```java
public class ExampleService {
    // @Autowired - matches by type first
    @Autowired
    @Qualifier("primaryDatabase")
    private DataSource dataSource;
    
    // @Resource - matches by name first
    @Resource(name = "primaryDatabase")
    private DataSource dataSourceByName;
    
    // @Autowired with optional
    @Autowired(required = false)
    private OptionalService optionalService;
    
    // @Resource cannot be optional
    @Resource(name = "requiredService")
    private RequiredService requiredService;
}
```

### Q8: How do you test classes with dependency injection?
**Answer**: Several testing strategies:

```java
// 1. Constructor injection makes testing easy
@Service
public class UserService {
    private final UserRepository repository;
    private final EmailService emailService;
    
    public UserService(UserRepository repository, EmailService emailService) {
        this.repository = repository;
        this.emailService = emailService;
    }
    
    public User createUser(String name, String email) {
        User user = new User(name, email);
        User saved = repository.save(user);
        emailService.sendWelcome(saved);
        return saved;
    }
}

// Unit test - no Spring context needed
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    @Mock private UserRepository repository;
    @Mock private EmailService emailService;
    @InjectMocks private UserService userService;
    
    @Test
    void shouldCreateUser() {
        // Given
        User user = new User("John", "john@example.com");
        when(repository.save(any(User.class))).thenReturn(user);
        
        // When
        User result = userService.createUser("John", "john@example.com");
        
        // Then
        assertThat(result.getName()).isEqualTo("John");
        verify(emailService).sendWelcome(user);
    }
}

// 2. Integration test with Spring context
@SpringBootTest
class UserServiceIntegrationTest {
    @Autowired private UserService userService;
    @MockBean private EmailService emailService;  // Mock specific beans
    
    @Test
    void shouldCreateUserWithRealDatabase() {
        User result = userService.createUser("John", "john@example.com");
        
        assertThat(result.getId()).isNotNull();
        verify(emailService).sendWelcome(result);
    }
}

// 3. Test configuration
@TestConfiguration
public class TestConfig {
    @Bean
    @Primary
    public EmailService mockEmailService() {
        return Mockito.mock(EmailService.class);
    }
}
```

---

## Advanced Topics

### 1. **Lazy Initialization**
```java
@Service
@Lazy  // Bean created only when first accessed
public class ExpensiveService {
    public ExpensiveService() {
        // Expensive initialization
        System.out.println("ExpensiveService initialized");
    }
}

@Service
public class ClientService {
    private final ExpensiveService expensiveService;
    
    // Lazy proxy injected, real bean created when first used
    public ClientService(@Lazy ExpensiveService expensiveService) {
        this.expensiveService = expensiveService;
        System.out.println("ClientService initialized");
    }
    
    public void useExpensiveService() {
        expensiveService.doWork();  // Now ExpensiveService is initialized
    }
}
```

### 2. **Conditional Bean Creation**
```java
@Service
@ConditionalOnProperty(name = "feature.analytics.enabled", havingValue = "true")
public class AnalyticsService {
    // Only created if property is true
}

@Service
@ConditionalOnMissingBean(PaymentService.class)
public class DefaultPaymentService implements PaymentService {
    // Created only if no other PaymentService bean exists
}

@Service
@Profile("production")
public class ProductionEmailService implements EmailService {
    // Only created in production profile
}

@Service
@Profile("!production")
public class MockEmailService implements EmailService {
    // Created in all profiles except production
}
```

### 3. **Custom Dependency Resolution**
```java
@Configuration
public class CustomDependencyConfig {
    
    @Bean
    @Scope("prototype")  // New instance for each injection
    public TaskProcessor taskProcessor() {
        return new TaskProcessor();
    }
    
    @Bean
    public TaskService taskService(List<TaskProcessor> processors) {
        return new TaskService(processors);  // Injects all TaskProcessor beans
    }
    
    @Bean
    @ConditionalOnProperty(name = "cache.type", havingValue = "redis")
    public CacheService redisCacheService(@Value("${redis.url}") String redisUrl) {
        return new RedisCacheService(redisUrl);
    }
    
    @Bean
    @ConditionalOnProperty(name = "cache.type", havingValue = "memory", matchIfMissing = true)
    public CacheService memoryCacheService() {
        return new MemoryCacheService();
    }
}
```

### 4. **Factory Pattern with DI**
```java
public interface PaymentProcessor {
    PaymentResult process(PaymentRequest request);
}

@Component
public class PaymentProcessorFactory {
    private final Map<PaymentType, PaymentProcessor> processors;
    
    // Spring injects all PaymentProcessor implementations as a map
    public PaymentProcessorFactory(Map<String, PaymentProcessor> processors) {
        this.processors = processors.entrySet().stream()
            .collect(Collectors.toMap(
                entry -> PaymentType.valueOf(entry.getKey().toUpperCase()),
                Map.Entry::getValue
            ));
    }
    
    public PaymentProcessor getProcessor(PaymentType type) {
        PaymentProcessor processor = processors.get(type);
        if (processor == null) {
            throw new UnsupportedPaymentTypeException("No processor for: " + type);
        }
        return processor;
    }
}

@Service("credit_card")
public class CreditCardProcessor implements PaymentProcessor {
    // Implementation for credit card payments
}

@Service("paypal")
public class PayPalProcessor implements PaymentProcessor {
    // Implementation for PayPal payments
}
```

---

## Anti-Patterns to Avoid

### 1. **Service Locator Anti-Pattern**
```java
// ❌ BAD - Service Locator (anti-pattern)
@Service
public class OrderService {
    public void processOrder(Order order) {
        // Manually looking up dependencies - tight coupling
        PaymentService paymentService = ApplicationContext.getBean(PaymentService.class);
        EmailService emailService = ApplicationContext.getBean(EmailService.class);
        
        paymentService.processPayment(order);
        emailService.sendConfirmation(order);
    }
}

// ✅ GOOD - Dependency Injection
@Service
public class OrderService {
    private final PaymentService paymentService;
    private final EmailService emailService;
    
    public OrderService(PaymentService paymentService, EmailService emailService) {
        this.paymentService = paymentService;
        this.emailService = emailService;
    }
    
    public void processOrder(Order order) {
        paymentService.processPayment(order);
        emailService.sendConfirmation(order);
    }
}
```

### 2. **Constructor Over-Injection**
```java
// ❌ BAD - Too many dependencies (code smell)
@Service
public class MonolithicUserService {
    public MonolithicUserService(UserRepository repo, EmailService email, 
                               SmsService sms, AuditService audit,
                               SecurityService security, CacheService cache,
                               MetricsService metrics, LoggingService logging) {
        // Too many dependencies suggests SRP violation
    }
}

// ✅ GOOD - Focused, single-responsibility services
@Service
public class UserService {
    private final UserRepository userRepository;
    private final UserValidator userValidator;
    
    public UserService(UserRepository userRepository, UserValidator userValidator) {
        this.userRepository = userRepository;
        this.userValidator = userValidator;
    }
}

@Service
public class UserNotificationService {
    private final EmailService emailService;
    private final SmsService smsService;
    
    public UserNotificationService(EmailService emailService, SmsService smsService) {
        this.emailService = emailService;
        this.smsService = smsService;
    }
}
```

### 3. **Temporal Coupling**
```java
// ❌ BAD - Order of setter calls matters
@Service
public class ConfigurableService {
    private DatabaseConfig dbConfig;
    private CacheConfig cacheConfig;
    private boolean initialized = false;
    
    @Autowired
    public void setDbConfig(DatabaseConfig dbConfig) {
        this.dbConfig = dbConfig;
    }
    
    @Autowired  
    public void setCacheConfig(CacheConfig cacheConfig) {
        this.cacheConfig = cacheConfig;
    }
    
    @PostConstruct
    public void initialize() {
        // Temporal coupling - relies on setter call order
        if (dbConfig == null || cacheConfig == null) {
            throw new IllegalStateException("Configuration not complete");
        }
        this.initialized = true;
    }
}

// ✅ GOOD - All dependencies provided at construction
@Service
public class ConfigurableService {
    private final DatabaseConfig dbConfig;
    private final CacheConfig cacheConfig;
    
    public ConfigurableService(DatabaseConfig dbConfig, CacheConfig cacheConfig) {
        this.dbConfig = Objects.requireNonNull(dbConfig);
        this.cacheConfig = Objects.requireNonNull(cacheConfig);
        // Object is fully initialized after construction
    }
}
```

---

## Summary

### Key Takeaways for Interviews

1. **DI Benefits**: Testability, loose coupling, flexibility, single responsibility
2. **Constructor Injection**: Preferred for required, immutable dependencies
3. **Setter Injection**: Used for optional dependencies
4. **Field Injection**: Avoid - hard to test, breaks encapsulation
5. **Circular Dependencies**: Usually design smell, can be resolved with @Lazy
6. **Testing**: Constructor injection makes unit testing without Spring context easy

### Decision Framework
```java
// Use this decision tree in interviews:

// Is the dependency REQUIRED for the object to function?
if (required) {
    // Use CONSTRUCTOR injection
    public Service(RequiredDependency dependency) { ... }
} else {
    // Is it an optional feature or configuration?
    if (optional) {
        // Use SETTER injection with required=false
        @Autowired(required = false)
        public void setOptionalDependency(OptionalDependency dependency) { ... }
    }
}

// NEVER use field injection unless you have a very specific reason
// and can justify the trade-offs
```

### Interview Success Tips
```java
// Show understanding of principles
"I prefer constructor injection because it ensures dependencies are 
immutable, makes testing easier without Spring context, and provides 
fail-fast behavior if required dependencies are missing."

// Demonstrate practical knowledge  
"In our microservices, we used constructor injection for core 
business dependencies like repositories and services, but setter 
injection for optional features like analytics tracking that 
might not be available in all environments."

// Show awareness of trade-offs
"While field injection is more concise, it makes unit testing 
harder because you need Spring context or reflection to inject 
mocks, and it violates encapsulation by making private fields 
accessible to the framework."
```

---

*This comprehensive guide covers all essential Dependency Injection concepts for technical interviews. Focus on understanding the principles behind each approach and being able to justify your design decisions with practical examples.*
