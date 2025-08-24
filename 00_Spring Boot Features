# Digital Drop-Off Reminder System

A comprehensive Spring Boot application for managing digital drop-off reminders and notifications in the Paytm ecosystem.

## 🚀 Technology Stack

- **Java**: 17 (LTS)
- **Spring Boot**: 2.7.16
- **Spring Cloud**: 2021.0.3
- **Apache Kafka**: Message streaming
- **Apache Cassandra**: NoSQL database
- **MySQL**: Relational database
- **Maven**: Build tool

## 📁 Project Structure

```
digital-drop-off-reminder/
├── drop-off-app/                    # Main application module
├── drop-off-consumer/              # Kafka consumer module
├── drop-off-common/                # Shared utilities and models
└── drop-off-notification-consumer/ # Notification processing module
```

## 🔧 Spring Boot Annotations Reference

This document provides a comprehensive guide to all Spring Boot annotations used in this repository, including detailed explanations and related annotations.

### 🚀 Application Startup Process

Before diving into annotations, let's understand how a Spring Boot application starts:

1. **Main Method Execution**: The `main()` method is called by the JVM
2. **@SpringBootApplication Processing**: Spring Boot scans for this annotation
3. **Auto-Configuration**: Spring Boot automatically configures beans based on classpath
4. **Component Scanning**: Spring scans for components marked with stereotype annotations
5. **Bean Creation**: All beans are created and dependencies are injected
6. **Application Context**: Spring ApplicationContext is fully initialized
7. **Application Ready**: The application is ready to handle requests

**Example Startup Flow:**
```java
@EnableEncryptableProperties
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
public class DropOffAppMainApplication {
    public static void main(String[] args) {
        SpringApplication.run(DropOffAppMainApplication.class, args);
    }
}
```

**What happens during startup:**
- `@EnableEncryptableProperties` enables Jasypt encryption support
- `@SpringBootApplication` triggers auto-configuration and component scanning
- `SpringApplication.run()` starts the embedded web server and application context

### 🏗️ Application Configuration Annotations

#### `@SpringBootApplication`
**Purpose**: Main application entry point annotation that combines three annotations:
- `@Configuration`: Marks the class as a source of bean definitions
- `@EnableAutoConfiguration`: Tells Spring Boot to start adding beans based on classpath settings
- `@ComponentScan`: Tells Spring to look for other components, configurations, and services

**Usage**: Applied to the main class to enable auto-configuration, component scanning, and configuration properties

**Related Annotations:**
- `@Configuration`: Standalone configuration class
- `@EnableAutoConfiguration`: Explicit auto-configuration
- `@ComponentScan`: Explicit component scanning

```java
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
public class DropOffAppMainApplication {
    // Application entry point
    // exclude = DataSourceAutoConfiguration.class prevents automatic datasource configuration
}
```

**What @SpringBootApplication does:**
1. **Auto-Configuration**: Automatically configures beans based on classpath dependencies
2. **Component Scanning**: Scans for @Component, @Service, @Repository, @Controller annotations
3. **Property Sources**: Loads application.properties/application.yml files
4. **Embedded Server**: Starts embedded Tomcat/Jetty server if spring-boot-starter-web is present

#### `@EnableEncryptableProperties`
**Purpose**: Enables encrypted properties support using Jasypt (Java Simplified Encryption)
**Usage**: Applied to main application classes for secure configuration management

**Related Annotations:**
- `@EnableConfigurationProperties`: Enables @ConfigurationProperties support
- `@PropertySource`: Specifies property sources

**How it works:**
1. **Property Resolution**: Resolves encrypted properties like `ENC(encrypted_value)`
2. **Decryption**: Automatically decrypts values using configured encryption algorithm
3. **Integration**: Works seamlessly with @Value and @ConfigurationProperties

```java
@EnableEncryptableProperties
@SpringBootApplication()
public class DropOffNotificationConsumerApplication {
    // Application with encrypted properties support
}

// In application.yml:
// database:
//   password: ENC(encrypted_password_here)
```

**Security Benefits:**
- **Sensitive Data Protection**: Encrypts passwords, API keys, database credentials
- **Environment Safety**: Prevents accidental exposure of secrets in logs
- **Compliance**: Helps meet security compliance requirements

#### `@EnableJpaRepositories`
**Purpose**: Enables JPA repositories for database operations and specifies where to find them
**Usage**: Specifies base packages for repository scanning

**Related Annotations:**
- `@Repository`: Marks interfaces as repository components
- `@EntityScan`: Specifies entity scanning packages
- `@EnableTransactionManagement`: Enables transaction management

**Key Features:**
1. **Repository Scanning**: Automatically discovers repository interfaces
2. **Query Method Generation**: Generates queries from method names
3. **Transaction Support**: Provides transaction management for repository methods

```java
@EnableJpaRepositories(basePackages = {
    "com.paytm.recharges.reminder.repositories.reminderSql",
    "com.paytm.recharges.reminder.repository"
})
```

**What happens during startup:**
1. **Package Scanning**: Scans specified packages for repository interfaces
2. **Proxy Creation**: Creates proxy implementations for repository interfaces
3. **Bean Registration**: Registers repository beans in Spring context
4. **Query Analysis**: Analyzes method names to generate SQL queries

**Repository Interface Example:**
```java
@Repository
public interface DigitalReminderNotificationConfigRepository extends JpaRepository<DigitalReminderConfigModel, Long> {
    // Spring Data JPA automatically implements methods like:
    // findByNameAndStatusAndKeyName(String name, Integer status, String keyName)
}
```

#### `@EntityScan`
**Purpose**: Specifies base packages for JPA entity scanning
**Usage**: Tells Spring where to find JPA entities for database mapping

**Related Annotations:**
- `@Entity`: Marks classes as JPA entities
- `@Table`: Specifies database table mapping
- `@Column`: Maps entity fields to database columns

**Key Features:**
1. **Entity Discovery**: Automatically discovers @Entity classes
2. **Table Creation**: Can create database tables automatically
3. **Schema Validation**: Validates entity mappings against database schema

```java
@EntityScan(basePackages = "com.paytm.recharges.reminder.models")
```

**Entity Example:**
```java
@Entity
@Table(name = "digital_reminder_config")
public class DigitalReminderConfigModel {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id", nullable = true)
    private Long id;
    
    @Column(name = "name", nullable = true)
    private String name;
    
    // Other fields...
}
```

**What happens during startup:**
1. **Package Scanning**: Scans specified packages for @Entity classes
2. **Metadata Processing**: Processes entity metadata and annotations
3. **Schema Validation**: Validates entity mappings against database
4. **Bean Registration**: Registers entity beans in Spring context

### 🎯 Component Stereotype Annotations

#### `@Service`
**Purpose**: Marks a class as a service layer component for business logic
**Usage**: Applied to business logic classes that contain application logic

**Related Annotations:**
- `@Component`: Base stereotype annotation
- `@Repository`: Data access layer stereotype
- `@Controller`: Web layer stereotype

**Key Features:**
1. **Business Logic**: Contains application business rules and logic
2. **Transaction Management**: Can participate in transactions
3. **Dependency Injection**: Can inject and be injected into other components

```java
@Service
public class DropOffHitService {
    @Autowired
    private DropOffHitEventProducer dropOffHitEventProducer;
    
    @Autowired
    private DropOffHitsRepository dropOffHitsRepository;
    
    public DropOffHitResponse processRequest(DropOffRequestBody dropOffRequestBody) {
        // Business logic implementation
        // Data validation, processing, and orchestration
    }
}
```

**Service Layer Responsibilities:**
- **Business Rules**: Implements application business logic
- **Data Orchestration**: Coordinates between different repositories
- **Transaction Management**: Manages transaction boundaries
- **Validation**: Performs business rule validation

#### `@Component`
**Purpose**: Marks a class as a Spring-managed component (base stereotype)
**Usage**: Applied to utility classes, configurations, and general Spring components

**Related Annotations:**
- `@Service`: Specialized component for business logic
- `@Repository`: Specialized component for data access
- `@Controller`: Specialized component for web controllers

**Key Features:**
1. **Spring Management**: Makes the class a Spring-managed bean
2. **Dependency Injection**: Can participate in dependency injection
3. **Lifecycle Management**: Spring manages the component lifecycle

```java
@Component
public class KafkaProducerConfiguration {
    @Value("${dropOff.kafka.bootstrapServers}")
    private String bootstrapServers;
    
    @Bean
    public KafkaTemplate<String, String> kafkaTemplate() {
        // Configuration component
        return new KafkaTemplate<>(producerFactory());
    }
}
```

**Component Lifecycle:**
1. **Instantiation**: Spring creates the component instance
2. **Dependency Injection**: Dependencies are injected
3. **Initialization**: @PostConstruct methods are called
4. **Ready**: Component is ready for use
5. **Destruction**: @PreDestroy methods are called during shutdown

**When to use @Component vs specialized annotations:**
- Use `@Component` for general utility classes
- Use `@Service` for business logic classes
- Use `@Repository` for data access classes
- Use `@Controller` for web controllers

#### `@RestController`
**Purpose**: Marks a class as a REST controller that handles HTTP requests
**Usage**: Applied to classes handling HTTP requests and returning JSON/XML responses

**Related Annotations:**
- `@Controller`: Traditional Spring MVC controller
- `@RequestMapping`: Maps HTTP requests to handler methods
- `@ResponseBody`: Indicates method return value should be serialized

**Key Features:**
1. **HTTP Request Handling**: Processes incoming HTTP requests
2. **Response Serialization**: Automatically serializes responses to JSON/XML
3. **Content Negotiation**: Handles different content types
4. **Exception Handling**: Provides centralized exception handling

```java
@RestController
@RequestMapping(path = Routes.DROP_OFF)
public class DropOffController {
    @Autowired
    private DropOffHitService dropOffHitService;
    
    @PostMapping(path = Routes.DROP_OFF_HIT)
    public ResponseEntity<Object> dropOffHit(
        HttpServletRequest request, 
        @RequestBody DropOffRequestBody dropOffRequestBody
    ) {
        // REST API endpoints
        return ResponseEntity.ok(dropOffHitService.processRequest(dropOffRequestBody));
    }
}
```

**@RestController vs @Controller:**
- `@RestController` = `@Controller` + `@ResponseBody`
- `@RestController` automatically serializes return values
- `@Controller` requires explicit `@ResponseBody` annotation

**Request Processing Flow:**
1. **Request Reception**: HTTP request arrives at controller
2. **Parameter Binding**: Request parameters are bound to method parameters
3. **Business Logic**: Service methods are called
4. **Response Generation**: Response is generated and serialized
5. **HTTP Response**: Serialized response is sent back to client

#### `@Repository`
**Purpose**: Marks a class as a data access layer component
**Usage**: Applied to repository interfaces and classes that handle data persistence

**Related Annotations:**
- `@Transactional`: Defines transaction boundaries
- `@Query`: Custom query definitions
- `@Modifying`: Marks query methods that modify data

**Key Features:**
1. **Data Access**: Handles database operations and data persistence
2. **Exception Translation**: Translates database exceptions to Spring exceptions
3. **Transaction Support**: Provides transaction management capabilities

```java
@Repository
public interface DigitalReminderNotificationConfigRepository extends JpaRepository<DigitalReminderConfigModel, Long> {
    // Data access methods
    List<DigitalReminderConfigModel> findByNameAndStatusAndKeyName(String name, Integer status, String keyName);
    
    @Query("SELECT c FROM DigitalReminderConfigModel c WHERE c.status = :status")
    List<DigitalReminderConfigModel> findByStatus(@Param("status") Integer status);
}
```

**Repository Types:**
- **JpaRepository**: Provides JPA-specific methods
- **CrudRepository**: Basic CRUD operations
- **PagingAndSortingRepository**: Adds pagination and sorting
- **Custom Repository**: Custom data access logic

**Exception Translation:**
- Database-specific exceptions are translated to Spring's `DataAccessException`
- Provides consistent exception handling across different databases

#### `@Configuration`
**Purpose**: Marks a class as a configuration class that contains bean definitions
**Usage**: Applied to classes containing bean definitions and application configuration

**Related Annotations:**
- `@Bean`: Defines beans within configuration classes
- `@Import`: Imports other configuration classes
- `@Profile`: Conditionally applies configuration based on active profile

**Key Features:**
1. **Bean Definition**: Contains methods annotated with @Bean
2. **Configuration Management**: Centralizes application configuration
3. **Dependency Injection**: Can inject dependencies into @Bean methods

```java
@Configuration
public class ConfigSchedulerService {
    @Autowired
    private DigitalReminderNotificationConfigRepository digitalReminderNotificationConfigRepository;
    
    @Scheduled(initialDelay = 0, fixedRateString = "${spring.datasource.config-load-interval:300000}")
    public void getNotificationExpiryTemplateIdConfig() {
        // Configuration methods
        // Scheduled configuration loading
    }
}
```

**Configuration Class Benefits:**
- **Modular Configuration**: Organizes configuration into logical units
- **Bean Lifecycle Management**: Controls how beans are created and managed
- **Conditional Configuration**: Can conditionally create beans based on conditions
- **Environment-Specific Config**: Different configurations for different environments

**Configuration vs @Component:**
- `@Configuration` is specialized for configuration classes
- `@Configuration` provides additional features like @Bean method proxying
- `@Component` is more general-purpose

### 🔗 Dependency Injection Annotations

#### `@Autowired`
**Purpose**: Injects Spring-managed dependencies into components
**Usage**: Applied to fields, constructors, or methods to inject dependencies

**Related Annotations:**
- `@Qualifier`: Specifies which bean to inject when multiple candidates exist
- `@Primary`: Marks a bean as the primary candidate for injection
- `@Resource`: Alternative to @Autowired (JSR-250)

**Injection Types:**
1. **Field Injection**: Direct field injection (most common)
2. **Constructor Injection**: Injection through constructor (recommended)
3. **Setter Injection**: Injection through setter methods

```java
@Service
public class DropOffHitService {
    // Field Injection
    @Autowired
    private DropOffHitEventProducer dropOffHitEventProducer;
    
    // Constructor Injection (Recommended)
    private final ApplicationMetricsPublisher applicationMetricsPublisher;
    
    public DropOffHitService(@Autowired ApplicationMetricsPublisher applicationMetricsPublisher) {
        this.applicationMetricsPublisher = applicationMetricsPublisher;
    }
    
    // Setter Injection
    @Autowired
    public void setDropOffHitsRepository(DropOffHitsRepository dropOffHitsRepository) {
        this.dropOffHitsRepository = dropOffHitsRepository;
    }
}
```

**Dependency Resolution Process:**
1. **Type Matching**: Spring finds beans matching the required type
2. **Qualifier Resolution**: If multiple candidates exist, @Qualifier is used
3. **Primary Selection**: @Primary beans are preferred
4. **Injection**: Dependency is injected into the target

**Best Practices:**
- **Constructor Injection**: Preferred for required dependencies
- **Field Injection**: Convenient but harder to test
- **Setter Injection**: Good for optional dependencies

#### `@Value`
**Purpose**: Injects values from properties files, environment variables, or SpEL expressions
**Usage**: Applied to fields to inject configuration values

**Related Annotations:**
- `@ConfigurationProperties`: Binds properties to POJO classes
- `@PropertySource`: Specifies property sources
- `@EnableConfigurationProperties`: Enables @ConfigurationProperties

**Value Sources:**
1. **Properties Files**: application.yml, application.properties
2. **Environment Variables**: System environment variables
3. **SpEL Expressions**: Spring Expression Language expressions
4. **Default Values**: Fallback values when property is not found

```java
@Component
public class KafkaProducerConfiguration {
    // Basic property injection
    @Value("${dropOff.kafka.bootstrapServers}")
    private String bootstrapServers;
    
    // With default value
    @Value("${dropOff.kafka.topicName:default-topic}")
    private String topicName;
    
    // Environment variable
    @Value("${KAFKA_BOOTSTRAP_SERVERS}")
    private String envBootstrapServers;
    
    // SpEL expression
    @Value("#{systemProperties['user.home']}")
    private String userHome;
    
    // Complex SpEL expression
    @Value("#{T(java.lang.Math).random() * 100.0}")
    private double randomValue;
}
```

**Property Resolution Order:**
1. **Command Line Arguments**: Highest priority
2. **Environment Variables**: System environment
3. **Application Properties**: application-{profile}.yml
4. **Default Properties**: application.yml
5. **Default Values**: Values specified in @Value annotation

**Configuration Properties Example:**
```java
@ConfigurationProperties(prefix = "dropoff.kafka")
public class KafkaProperties {
    private String bootstrapServers;
    private String topicName;
    private String consumerGroup;
    
    // Getters and setters
}
```

### 🫘 Bean Management Annotations

#### `@Bean`
**Purpose**: Defines a Spring bean in configuration classes
**Usage**: Applied to methods in configuration classes to create and configure beans

**Related Annotations:**
- `@Scope`: Defines bean scope (singleton, prototype, etc.)
- `@Lazy`: Lazy initialization of beans
- `@DependsOn`: Specifies bean dependencies

**Bean Scopes:**
1. **Singleton**: One instance per Spring container (default)
2. **Prototype**: New instance each time requested
3. **Request**: One instance per HTTP request
4. **Session**: One instance per HTTP session

```java
@Configuration
public class KafkaProducerConfiguration {
    @Value("${dropOff.kafka.bootstrapServers}")
    private String bootstrapServers;
    
    // Basic bean definition
    @Bean
    public KafkaTemplate<String, String> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }
    
    // Named bean with scope
    @Bean(name = ThreadPoolExecutorConstants.CLIENT_SERVICE_EXECUTOR)
    @Scope("singleton")
    public ThreadPoolTaskExecutor clientServiceExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(20);
        return executor;
    }
    
    // Bean with dependencies
    @Bean
    @DependsOn("clientServiceExecutor")
    public KafkaProducer<String, String> kafkaProducer() {
        return new KafkaProducer<>(producerConfigs());
    }
}
```

**Bean Lifecycle:**
1. **Instantiation**: Bean is created
2. **Dependency Injection**: Dependencies are injected
3. **Initialization**: @PostConstruct methods are called
4. **Ready**: Bean is ready for use
5. **Destruction**: @PreDestroy methods are called during shutdown

**Bean Configuration Options:**
- **Name**: Specify custom bean name
- **Scope**: Define bean scope
- **Dependencies**: Specify initialization order
- **Lazy Loading**: Defer bean creation until needed

#### `@PostConstruct`
**Purpose**: Marks a method to be executed after dependency injection is complete
**Usage**: Applied to initialization methods that need to run after all dependencies are injected

**Related Annotations:**
- `@PreDestroy`: Marks a method to be executed before bean destruction
- `@PostConstruct`: JSR-250 lifecycle annotation
- `InitializingBean`: Spring interface for initialization

**Lifecycle Order:**
1. **Bean Instantiation**: Bean is created
2. **Dependency Injection**: @Autowired fields are injected
3. **@PostConstruct**: Initialization method is called
4. **Bean Ready**: Bean is ready for use
5. **@PreDestroy**: Cleanup method is called during shutdown

```java
@Component
public class StatsDClientMetricPublisher {
    @Value("${metrics.prefix}")
    private String prefix;
    
    @Value("${metrics.statsdserver.host}")
    private String host;
    
    private StatsDClient statsDClient;
    
    @PostConstruct
    public void init() {
        // Initialization logic - runs after all @Value injections
        statsDClient = new NonBlockingStatsDClientBuilder()
                .prefix(prefix)
                .hostname(host)
                .port(port)
                .queueSize(queueSize)
                .build();
    }
    
    @PreDestroy
    public void cleanup() {
        // Cleanup logic - runs before bean destruction
        if (statsDClient != null) {
            statsDClient.close();
        }
    }
}
```

**Common Use Cases:**
- **Resource Initialization**: Initialize external resources (databases, connections)
- **Configuration Validation**: Validate configuration after injection
- **Cache Warming**: Pre-populate caches
- **Service Registration**: Register with external services

**Best Practices:**
- Keep initialization logic lightweight
- Handle exceptions gracefully
- Don't perform long-running operations
- Use for setup that depends on injected values

### 🌐 Web Layer Annotations

#### `@RequestMapping`
**Purpose**: Maps web requests to handler methods
**Usage**: Applied to controller classes or methods to define request mappings

**Related Annotations:**
- `@GetMapping`: Maps GET requests
- `@PostMapping`: Maps POST requests
- `@PutMapping`: Maps PUT requests
- `@DeleteMapping`: Maps DELETE requests
- `@PatchMapping`: Maps PATCH requests

**Mapping Attributes:**
- **path/value**: URL path pattern
- **method**: HTTP method (GET, POST, etc.)
- **consumes**: Content type the method consumes
- **produces**: Content type the method produces
- **headers**: Required HTTP headers
- **params**: Required request parameters

```java
@RestController
@RequestMapping(
    path = Routes.DROP_OFF,
    produces = MediaType.APPLICATION_JSON_VALUE,
    consumes = MediaType.APPLICATION_JSON_VALUE
)
public class DropOffController {
    // Controller methods
    
    @RequestMapping(
        path = "/health",
        method = RequestMethod.GET,
        produces = MediaType.TEXT_PLAIN_VALUE
    )
    public String health() {
        return "OK";
    }
}
```

**Request Mapping Resolution:**
1. **URL Pattern Matching**: Matches request URL to path patterns
2. **HTTP Method Matching**: Matches HTTP method
3. **Content Type Matching**: Matches Accept and Content-Type headers
4. **Parameter Matching**: Matches required request parameters
5. **Handler Selection**: Selects the most specific matching handler

#### `@PostMapping`
**Purpose**: Maps POST requests to handler methods
**Usage**: Applied to methods handling POST requests (composed annotation of @RequestMapping with method = POST)

**Related Annotations:**
- `@GetMapping`: Maps GET requests
- `@PutMapping`: Maps PUT requests
- `@DeleteMapping`: Maps DELETE requests
- `@PatchMapping`: Maps PATCH requests

**Common Use Cases:**
- **Data Creation**: Creating new resources
- **Form Submission**: Processing form data
- **File Upload**: Handling file uploads
- **API Endpoints**: REST API endpoints

```java
@PostMapping(
    path = Routes.DROP_OFF_HIT,
    consumes = MediaType.APPLICATION_JSON_VALUE,
    produces = MediaType.APPLICATION_JSON_VALUE
)
public ResponseEntity<Object> dropOffHit(
    HttpServletRequest request, 
    @RequestBody DropOffRequestBody dropOffRequestBody
) {
    // POST request handler
    DropOffHitResponse response = dropOffHitService.processRequest(dropOffRequestBody);
    return ResponseEntity.ok(response);
}
```

**Request Processing Flow:**
1. **Request Reception**: HTTP POST request arrives
2. **Content Type Validation**: Validates Content-Type header
3. **Body Deserialization**: Deserializes request body to DTO
4. **Business Logic**: Calls service layer methods
5. **Response Generation**: Creates HTTP response
6. **Serialization**: Serializes response to JSON

**Response Status Codes:**
- **200 OK**: Successful processing
- **201 Created**: Resource created successfully
- **400 Bad Request**: Invalid request data
- **500 Internal Server Error**: Server error

#### `@RequestBody`
**Purpose**: Binds HTTP request body to method parameters
**Usage**: Applied to method parameters to deserialize request body

**Related Annotations:**
- `@RequestParam`: Binds request parameters
- `@PathVariable`: Binds path variables
- `@RequestHeader`: Binds request headers
- `@Valid`: Triggers validation on bound object

**Binding Process:**
1. **Content Type Detection**: Determines content type from Content-Type header
2. **Message Converter Selection**: Selects appropriate HttpMessageConverter
3. **Deserialization**: Converts request body to Java object
4. **Validation**: Applies validation if @Valid is present
5. **Parameter Binding**: Binds object to method parameter

```java
@PostMapping(path = Routes.DROP_OFF_HIT)
public ResponseEntity<Object> dropOffHit(
    @RequestBody @Valid DropOffRequestBody dropOffRequestBody,
    @RequestParam(required = false) String source,
    @RequestHeader("User-Agent") String userAgent
) {
    // Request body binding with validation
    // dropOffRequestBody contains deserialized JSON data
    // @Valid triggers validation annotations on DropOffRequestBody
}
```

**Supported Content Types:**
- **application/json**: JSON data (most common)
- **application/xml**: XML data
- **application/x-www-form-urlencoded**: Form data
- **multipart/form-data**: File uploads

**Validation Example:**
```java
public class DropOffRequestBody {
    @NotNull(message = "Customer ID is required")
    private String customerId;
    
    @NotBlank(message = "Category cannot be empty")
    private String category;
    
    @Min(value = 1, message = "Amount must be positive")
    private BigDecimal amount;
    
    // Getters and setters
}
```

### 📨 Kafka Integration Annotations

#### `@KafkaListener`
**Purpose**: Marks a method as a Kafka message listener
**Usage**: Applied to methods that consume Kafka messages from specified topics

**Related Annotations:**
- `@Payload`: Marks message payload parameter
- `@Header`: Binds Kafka headers
- `@KafkaHandler`: Marks handler methods in @KafkaListener classes

**Listener Configuration:**
- **topics**: List of topics to consume from
- **groupId**: Consumer group ID for partition assignment
- **containerFactory**: Custom container factory bean name
- **autoStartup**: Whether to start listening automatically
- **concurrency**: Number of concurrent consumers

```java
@Component
@Conditional(KafkaConsumerEnableCondition.class)
public class OMSKafkaConsumer {
    
    @KafkaListener(
        topics = "${oms.kafka.topicName}",
        groupId = "${oms.kafka.consumerGroup}",
        containerFactory = "kafkaListenerContainerFactory",
        autoStartup = "true",
        concurrency = "3"
    )
    public void listen(@Payload List<String> messages, Acknowledgment acknowledgment) {
        // Kafka message processing
        for (String message : messages) {
            try {
                processMessage(message);
                acknowledgment.acknowledge(); // Manual acknowledgment
            } catch (Exception e) {
                // Handle error - message will be retried
                logger.error("Error processing message: {}", e.getMessage());
            }
        }
    }
}
```

**Message Processing Flow:**
1. **Message Reception**: Kafka consumer receives messages
2. **Deserialization**: Messages are deserialized to Java objects
3. **Batch Processing**: Messages are processed in batches
4. **Business Logic**: Application logic processes each message
5. **Acknowledgment**: Messages are acknowledged after successful processing
6. **Error Handling**: Failed messages are handled according to retry policy

**Consumer Configuration:**
```java
@Bean
public ConcurrentKafkaListenerContainerFactory<String, String> kafkaListenerContainerFactory() {
    ConcurrentKafkaListenerContainerFactory<String, String> factory = 
        new ConcurrentKafkaListenerContainerFactory<>();
    factory.setConsumerFactory(consumerFactory());
    factory.setConcurrency(3);
    factory.getContainerProperties().setAckMode(ContainerProperties.AckMode.MANUAL);
    return factory;
}
```

#### `@Payload`
**Purpose**: Marks a parameter as the message payload in Kafka listeners
**Usage**: Applied to Kafka listener method parameters to bind message content

**Related Annotations:**
- `@Header`: Binds Kafka message headers
- `@Headers`: Binds all headers as a Map
- `@KafkaListener`: Marks the listener method

**Parameter Binding:**
- **@Payload**: Message body content
- **@Header**: Individual header values
- **@Headers**: All headers as Map<String, Object>
- **Acknowledgment**: Manual acknowledgment control

```java
@KafkaListener(topics = "${oms.kafka.topicName}")
public void listen(
    @Payload List<String> messages,
    @Header(KafkaHeaders.RECEIVED_TOPIC) String topic,
    @Header(KafkaHeaders.RECEIVED_PARTITION_ID) int partition,
    @Header(KafkaHeaders.OFFSET) long offset,
    @Headers Map<String, Object> headers,
    Acknowledgment acknowledgment
) {
    // Message payload processing with header information
    for (String message : messages) {
        logger.info("Processing message from topic: {}, partition: {}, offset: {}", 
                   topic, partition, offset);
        processMessage(message);
    }
    acknowledgment.acknowledge();
}
```

**Message Structure:**
```java
// Kafka message structure
{
    "headers": {
        "message-id": "12345",
        "timestamp": "2024-01-01T10:00:00Z",
        "source": "drop-off-app"
    },
    "payload": {
        "customerId": "CUST123",
        "category": "mobile",
        "amount": 100.00
    }
}
```

**Deserialization Process:**
1. **Message Reception**: Raw Kafka message received
2. **Header Extraction**: Headers are extracted and bound
3. **Payload Deserialization**: Message body is deserialized
4. **Parameter Binding**: Parameters are bound to method arguments
5. **Method Execution**: Listener method is called with bound parameters

### ⏰ Scheduling Annotations

#### `@Scheduled`
**Purpose**: Marks a method for scheduled execution
**Usage**: Applied to methods that need periodic execution at fixed intervals

**Related Annotations:**
- `@EnableScheduling`: Enables scheduling support
- `@Async`: Combines with scheduling for asynchronous execution
- `@Conditional`: Conditionally enables scheduled tasks

**Scheduling Types:**
1. **fixedRate**: Fixed rate execution (every X milliseconds)
2. **fixedDelay**: Fixed delay execution (X milliseconds after completion)
3. **cron**: Cron expression for complex scheduling
4. **initialDelay**: Initial delay before first execution

```java
@Configuration
@EnableScheduling
public class ConfigSchedulerService {
    
    @Autowired
    private DigitalReminderNotificationConfigRepository repository;
    
    // Fixed rate - executes every 5 minutes
    @Scheduled(fixedRateString = "${spring.datasource.config-load-interval:300000}")
    public void getNotificationExpiryTemplateIdConfig() {
        // Scheduled task execution
        List<DigitalReminderConfigModel> configs = repository.findByNameAndStatusAndKeyName(
            "TEMPLATE_CONFIG", 1, "NOTIFICATION_EXPIRY"
        );
        // Process configurations...
    }
    
    // Cron expression - executes at specific times
    @Scheduled(cron = "0 0 2 * * ?") // Daily at 2 AM
    public void dailyCleanup() {
        // Daily cleanup task
    }
    
    // Fixed delay - executes 30 seconds after completion
    @Scheduled(fixedDelay = 30000)
    public void periodicHealthCheck() {
        // Health check task
    }
    
    // With initial delay
    @Scheduled(initialDelay = 60000, fixedRate = 300000)
    public void delayedStartup() {
        // Task that starts 1 minute after application startup
    }
}
```

**Cron Expression Format:**
```
┌───────────── second (0-59)
│ ┌───────────── minute (0-59)
│ │ ┌───────────── hour (0-23)
│ │ │ ┌───────────── day of month (1-31)
│ │ │ │ ┌───────────── month (1-12)
│ │ │ │ │ ┌───────────── day of week (0-7) (0 and 7 are Sunday)
│ │ │ │ │ │
* * * * * *
```

**Common Cron Examples:**
- `0 0 2 * * ?` - Daily at 2 AM
- `0 0 */6 * * ?` - Every 6 hours
- `0 30 9 * * MON-FRI` - Weekdays at 9:30 AM
- `0 0 12 1 * ?` - Monthly on 1st at noon

**Best Practices:**
- **Error Handling**: Implement proper error handling in scheduled tasks
- **Logging**: Add comprehensive logging for monitoring
- **Resource Management**: Be mindful of resource usage in long-running tasks
- **Idempotency**: Make tasks idempotent to handle failures gracefully

### 🔄 Asynchronous Processing Annotations

#### `@Async`
**Purpose**: Marks a method for asynchronous execution
**Usage**: Applied to methods that should run in background threads
```java
@Async(Constants.ThreadPoolExecutorConstants.CLIENT_SERVICE_EXECUTOR)
public CompletableFuture<Boolean> processPayload(
    SendNotificationKafkaPayload sendNotificationKafkaPayload,
    List<UUID> alreadyProcessedNotifications
) {
    // Asynchronous processing
}
```

### 🎛️ Conditional Annotations

#### `@Conditional`
**Purpose**: Conditionally creates beans based on conditions
**Usage**: Applied to components that should be created conditionally
```java
@Component
@Conditional(SendNotificationKafkaConsumerEnableCondition.class)
public class SendNotificationKafkaConsumer {
    // Conditional component
}
```

### 🗄️ Database Annotations

#### `@PrimaryKeyColumn`
**Purpose**: Defines primary key columns for Cassandra entities
**Usage**: Applied to fields in Cassandra entity classes
```java
@PrimaryKeyColumn(name = "paytype", ordinal = 0, type = PrimaryKeyType.PARTITIONED)
private String paytype;

@PrimaryKeyColumn(name = "service", ordinal = 1, type = PrimaryKeyType.CLUSTERED)
private String service;
```

#### `@Column`
**Purpose**: Maps entity fields to database columns
**Usage**: Applied to fields in entity classes
```java
@Column("amount")
private String amount;

@Column("customer_id")
private String customerId;
```

### 🧪 Testing Annotations

#### `@Test`
**Purpose**: Marks a method as a test method
**Usage**: Applied to JUnit test methods
```java
@Test
public void testDropOffHitResponse() {
    // Test implementation
}
```

#### `@Mock`
**Purpose**: Creates mock objects for testing
**Usage**: Applied to fields in test classes
```java
@Mock
private DropOffHitsRepository dropOffHitsRepository;
```

#### `@InjectMocks`
**Purpose**: Injects mocks into the class under test
**Usage**: Applied to the class being tested
```java
@InjectMocks
private DropOffHitService dropOffHitService;
```

#### `@Before`
**Purpose**: Marks a method to run before each test
**Usage**: Applied to setup methods in test classes
```java
@Before
public void setUp() {
    // Test setup
}
```

### 📝 Lombok Annotations

#### `@Getter` and `@Setter`
**Purpose**: Automatically generates getter and setter methods
**Usage**: Applied to class fields
```java
@Getter
@Setter
public class DropOffHitResponse {
    // Auto-generated getters and setters
}
```

#### `@Data`
**Purpose**: Generates getters, setters, toString, equals, and hashCode
**Usage**: Applied to data classes
```java
@Data
public class BillsRecentCacheModel {
    // Complete data class with all methods
}
```

## 🚀 Getting Started

### Prerequisites
- Java 17
- Maven 3.8+
- Apache Kafka
- Apache Cassandra
- MySQL

### Installation

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd digital-drop-off-reminder
   ```

2. **Build the project**
   ```bash
   mvn clean install
   ```

3. **Configure application properties**
   - Update `application-dev.yml` for development
   - Update `application-staging.yml` for staging
   - Update `application-production.yml` for production

4. **Run the applications**
   ```bash
   # Run drop-off-app
   mvn spring-boot:run -pl drop-off-app
   
   # Run drop-off-consumer
   mvn spring-boot:run -pl drop-off-consumer
   
   # Run drop-off-notification-consumer
   mvn spring-boot:run -pl drop-off-notification-consumer
   ```

## 📊 Architecture Overview

The system consists of four main modules working together to provide a comprehensive digital drop-off reminder solution:

### 🏗️ Module Architecture

1. **drop-off-app**: REST API for receiving drop-off events
   - **Purpose**: Entry point for drop-off event ingestion
   - **Responsibilities**: 
     - Receive HTTP requests with drop-off data
     - Validate incoming data
     - Store events in Cassandra
     - Publish events to Kafka for processing
   - **Key Components**:
     - `DropOffController`: REST endpoints
     - `DropOffHitService`: Business logic
     - `DropOffHitEventProducer`: Kafka producer

2. **drop-off-consumer**: Processes drop-off events and manages notifications
   - **Purpose**: Core business logic for drop-off processing
   - **Responsibilities**:
     - Consume drop-off events from Kafka
     - Apply business rules and priority logic
     - Manage notification scheduling
     - Handle first and second interval processing
   - **Key Components**:
     - `DropOffKafkaConsumer`: Kafka message processing
     - `DropOffConsumerBusinessLogic`: Core business logic
     - `OMSKafkaConsumer`: Order management system integration

3. **drop-off-common**: Shared utilities, models, and configurations
   - **Purpose**: Common code shared across modules
   - **Responsibilities**:
     - Data models and DTOs
     - Configuration classes
     - Utility classes
     - Database repositories
   - **Key Components**:
     - `DropOffHitsRepository`: Cassandra data access
     - `DynamicTableRepositoryImpl`: Dynamic table operations
     - `ApplicationMetricsPublisher`: Metrics collection

4. **drop-off-notification-consumer**: Handles notification sending
   - **Purpose**: Notification delivery and management
   - **Responsibilities**:
     - Consume notification requests from Kafka
     - Send notifications via various channels
     - Handle notification retries and failures
     - Manage notification configuration
   - **Key Components**:
     - `SendNotificationKafkaConsumer`: Notification processing
     - `SendNotificationService`: Notification delivery
     - `ConfigSchedulerService`: Dynamic configuration management

### 🔄 Application Startup Process

#### **Phase 1: Application Context Initialization**
```java
@EnableEncryptableProperties
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
public class DropOffAppMainApplication {
    public static void main(String[] args) {
        SpringApplication.run(DropOffAppMainApplication.class, args);
    }
}
```

**Startup Steps:**
1. **JVM Startup**: Main method is called
2. **SpringApplication Creation**: SpringApplication instance is created
3. **Environment Setup**: Application environment is configured
4. **Context Creation**: ApplicationContext is created
5. **Bean Definition Loading**: Bean definitions are loaded

#### **Phase 2: Auto-Configuration**
```java
// Spring Boot automatically configures based on classpath
@ConditionalOnClass(KafkaTemplate.class)
@EnableConfigurationProperties(KafkaProperties.class)
public class KafkaAutoConfiguration {
    // Auto-configuration of Kafka components
}
```

**Auto-Configuration Process:**
1. **Classpath Scanning**: Spring Boot scans classpath for dependencies
2. **Conditional Bean Creation**: Beans are created based on conditions
3. **Property Binding**: Configuration properties are bound
4. **Default Configuration**: Default configurations are applied

#### **Phase 3: Component Scanning**
```java
// Spring scans for components marked with stereotype annotations
@ComponentScan(basePackages = "com.paytm.recharges.reminder")
```

**Component Discovery:**
1. **Package Scanning**: Scans specified packages for components
2. **Annotation Processing**: Processes @Component, @Service, @Repository, @Controller
3. **Bean Registration**: Registers discovered components as beans
4. **Dependency Resolution**: Resolves dependencies between components

#### **Phase 4: Bean Initialization**
```java
@Component
public class StatsDClientMetricPublisher {
    @PostConstruct
    public void init() {
        // Initialize after dependency injection
        statsDClient = new NonBlockingStatsDClientBuilder()
                .prefix(prefix)
                .hostname(host)
                .port(port)
                .build();
    }
}
```

**Initialization Process:**
1. **Bean Creation**: Beans are instantiated
2. **Dependency Injection**: Dependencies are injected
3. **@PostConstruct Execution**: Initialization methods are called
4. **Bean Ready**: Beans are ready for use

#### **Phase 5: Application Ready**
```java
@Component
public class DropOffKafkaConsumer {
    @KafkaListener(topics = "${dropOff.kafka.topicName}")
    public void listen(@Payload List<String> messages) {
        // Kafka consumer starts listening
    }
}
```

**Final Steps:**
1. **Embedded Server Start**: Tomcat/Jetty server starts
2. **Kafka Consumers Start**: Kafka listeners begin consuming
3. **Scheduled Tasks Start**: @Scheduled methods begin execution
4. **Application Ready**: Application is ready to handle requests

### 🔧 Configuration Management

#### **Environment-Specific Configuration**
```yaml
# application-dev.yml
dropOff:
  kafka:
    bootstrapServers: localhost:9092
    topicName: 'drop_off_hit'

# application-production.yml  
dropOff:
  kafka:
    bootstrapServers: 10.4.33.86:9092,10.4.33.247:9092,10.4.33.251:9092
    topicName: 'drop_off_hit'
```

#### **Dynamic Configuration Loading**
```java
@Scheduled(fixedRateString = "${spring.datasource.config-load-interval:300000}")
public void getNotificationExpiryTemplateIdConfig() {
    // Dynamically loads configuration from database
    List<DigitalReminderConfigModel> configs = repository.findByNameAndStatusAndKeyName(
        "TEMPLATE_CONFIG", 1, "NOTIFICATION_EXPIRY"
    );
    // Updates in-memory configuration
}
```

## 🔄 Data Flow

1. **Drop-off Event Capture**: REST API receives drop-off events
2. **Event Processing**: Kafka consumers process events based on business rules
3. **Notification Management**: System sends notifications based on priority and timing
4. **Data Persistence**: Events and notifications are stored in Cassandra and MySQL

## 📈 Monitoring and Metrics

The application uses StatsD for metrics collection:
- API latency and response counts
- Kafka message processing metrics
- Database operation metrics
- Custom business metrics

## 🔒 Security

- **Encrypted Properties**: Uses Jasypt for configuration encryption
- **Vault Integration**: Secure secret management
- **DND Hours**: Respects Do Not Disturb hours (7 AM - 11 PM)

## 🧪 Testing

Run tests using Maven:
```bash
mvn test
```

The project includes:
- Unit tests with JUnit and Mockito
- Integration tests
- Code coverage with JaCoCo

## 📝 Contributing

1. Follow the existing code style
2. Add appropriate tests for new features
3. Update documentation as needed
4. Ensure all tests pass before submitting

## 📄 License

This project is proprietary to Paytm. All rights reserved.

## 🤝 Support

For questions and support, please contact the development team or create an issue in the repository.
