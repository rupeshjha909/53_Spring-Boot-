# Spring Boot Interview Questions 2024: Complete Guide

A comprehensive collection of Spring Boot interview questions commonly asked in technical interviews, organized by difficulty level and topics with detailed answers and code examples.

## Table of Contents
- [Basic Level Questions (1-2 Years)](#-basic-level-questions-1-2-years)
- [Intermediate Level Questions (3-5 Years)](#-intermediate-level-questions-3-5-years)
- [Advanced Level Questions (5+ Years)](#-advanced-level-questions-5-years)
- [Spring Boot 3.x Specific Questions](#spring-boot-3x-specific-questions)
- [Microservices with Spring Boot](#microservices-with-spring-boot)
- [Performance & Production Questions](#performance--production-questions)
- [Spring Boot Testing Questions](#spring-boot-testing-questions)
- [Security Questions](#security-questions)
- [Data & Database Questions](#data--database-questions)
- [DevOps & Deployment Questions](#devops--deployment-questions)

---

## 🔰 Basic Level Questions (1-2 Years)

### 1. What is Spring Boot and why is it popular?
**Expected Answer:**
Spring Boot is an opinionated framework built on top of Spring Framework that simplifies Java application development through:

- **Auto-configuration**: Automatically configures Spring applications based on classpath dependencies
- **Starter dependencies**: Pre-packaged dependency sets for common use cases
- **Embedded servers**: Built-in Tomcat, Jetty, or Undertow
- **Production-ready features**: Health checks, metrics, externalized configuration
- **Minimal boilerplate**: Reduces XML configuration and boilerplate code

```java
// Traditional Spring application
@Configuration
@EnableWebMvc
@ComponentScan
public class WebConfig implements WebMvcConfigurer {
    // Lots of configuration code
}

// Spring Boot application
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### 2. What are Spring Boot Starters?
**Expected Answer:**
Starters are convenient dependency descriptors that bring in all necessary dependencies for a particular functionality.

**Common Starters:**
```xml
<!-- Web applications -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- JPA/Database -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<!-- Security -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<!-- Testing -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

### 3. Explain @SpringBootApplication annotation
**Expected Answer:**
`@SpringBootApplication` is a composite annotation that combines:

```java
@SpringBootApplication
// Equivalent to:
@Configuration          // Marks class as configuration source
@EnableAutoConfiguration // Enables auto-configuration
@ComponentScan          // Enables component scanning
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

// You can customize behavior:
@SpringBootApplication(
    scanBasePackages = {"com.example.service", "com.example.controller"},
    exclude = {DataSourceAutoConfiguration.class}
)
public class CustomApplication { }
```

### 4. How does Spring Boot Auto-Configuration work?
**Expected Answer:**
Auto-configuration works through:

1. **Classpath scanning**: Detects libraries on classpath
2. **Conditional annotations**: Apply configuration only when conditions are met
3. **Configuration classes**: Pre-defined configurations for common scenarios

```java
// Example auto-configuration class
@Configuration
@ConditionalOnClass(DataSource.class)
@ConditionalOnMissingBean(DataSource.class)
@EnableConfigurationProperties(DataSourceProperties.class)
public class DataSourceAutoConfiguration {
    
    @Bean
    @ConfigurationProperties("spring.datasource")
    public DataSource dataSource() {
        return new HikariDataSource();
    }
}
```

### 5. What is application.properties vs application.yml?
**Expected Answer:**
Both are configuration files but with different formats:

```properties
# application.properties
server.port=8080
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=password
logging.level.com.example=DEBUG
```

```yaml
# application.yml
server:
  port: 8080

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: root
    password: password

logging:
  level:
    com.example: DEBUG
```

**YAML advantages**: More readable, supports hierarchical data, lists, and multi-document files.

### 6. How do you create a REST API in Spring Boot?
**Expected Answer:**
```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @Autowired
    private UserService userService;
    
    @GetMapping
    public List<User> getAllUsers() {
        return userService.findAll();
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<User> getUserById(@PathVariable Long id) {
        User user = userService.findById(id);
        return user != null ? ResponseEntity.ok(user) : ResponseEntity.notFound().build();
    }
    
    @PostMapping
    public ResponseEntity<User> createUser(@Valid @RequestBody User user) {
        User savedUser = userService.save(user);
        return ResponseEntity.status(HttpStatus.CREATED).body(savedUser);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<User> updateUser(@PathVariable Long id, @Valid @RequestBody User user) {
        User updatedUser = userService.update(id, user);
        return ResponseEntity.ok(updatedUser);
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

### 7. What are Spring Boot Profiles?
**Expected Answer:**
Profiles allow you to have different configurations for different environments:

```yaml
# application.yml
spring:
  profiles:
    active: dev

---
spring:
  profiles: dev
  datasource:
    url: jdbc:h2:mem:testdb
    
logging:
  level:
    root: DEBUG

---
spring:
  profiles: prod
  datasource:
    url: jdbc:mysql://prod-server:3306/proddb
    username: ${DB_USER}
    password: ${DB_PASSWORD}
    
logging:
  level:
    root: WARN
```

```java
@Component
@Profile("dev")
public class DevDataInitializer { }

@Component
@Profile("prod")
public class ProductionDataInitializer { }
```

### 8. How do you handle exceptions in Spring Boot?
**Expected Answer:**
```java
// Global exception handler
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleUserNotFound(UserNotFoundException ex) {
        ErrorResponse error = new ErrorResponse("USER_NOT_FOUND", ex.getMessage());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
    
    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<ErrorResponse> handleValidation(ValidationException ex) {
        ErrorResponse error = new ErrorResponse("VALIDATION_ERROR", ex.getMessage());
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(error);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneral(Exception ex) {
        ErrorResponse error = new ErrorResponse("INTERNAL_ERROR", "Something went wrong");
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}

// Custom exception
public class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(String message) {
        super(message);
    }
}
```

### 9. What is the difference between @Component, @Service, @Repository, and @Controller?
**Expected Answer:**
All are specializations of `@Component` with semantic meanings:

```java
@Component  // Generic component
public class UtilityComponent { }

@Controller // Web layer - handles HTTP requests
public class UserController { }

@Service    // Business logic layer
public class UserService { }

@Repository // Data access layer - includes exception translation
public class UserRepository { }
```

**Key differences:**
- `@Controller`: Web MVC controller, often returns views
- `@RestController`: REST controller, returns JSON/XML
- `@Service`: Business logic, no special behavior
- `@Repository`: Data access, provides exception translation
- `@Component`: Generic stereotype

### 10. How do you connect to a database in Spring Boot?
**Expected Answer:**
```yaml
# application.yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: root
    password: password
    driver-class-name: com.mysql.cj.jdbc.Driver
  
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MySQL8Dialect
```

```java
// Entity
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true)
    private String email;
    
    private String name;
    
    // Constructors, getters, setters
}

// Repository
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
    List<User> findByNameContainingIgnoreCase(String name);
}
```

---

## ⚙️ Intermediate Level Questions (3-5 Years)

### 11. Explain Spring Boot Actuator and its endpoints
**Expected Answer:**
Actuator provides production-ready features for monitoring and managing applications:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```yaml
# Configuration
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,env,configprops
  endpoint:
    health:
      show-details: always
  info:
    env:
      enabled: true
```

**Common endpoints:**
- `/actuator/health` - Application health
- `/actuator/info` - Application information  
- `/actuator/metrics` - Application metrics
- `/actuator/env` - Environment properties
- `/actuator/configprops` - Configuration properties

```java
// Custom health indicator
@Component
public class CustomHealthIndicator implements HealthIndicator {
    
    @Override
    public Health health() {
        // Custom health check logic
        boolean isHealthy = checkExternalService();
        
        if (isHealthy) {
            return Health.up()
                .withDetail("status", "Service is running")
                .withDetail("version", "1.0.0")
                .build();
        }
        
        return Health.down()
            .withDetail("status", "Service is down")
            .build();
    }
}
```

### 12. How do you implement caching in Spring Boot?
**Expected Answer:**
```java
// Enable caching
@SpringBootApplication
@EnableCaching
public class Application { }

// Service with caching
@Service
public class UserService {
    
    @Cacheable("users")
    public User findById(Long id) {
        // Expensive database operation
        return userRepository.findById(id);
    }
    
    @CacheEvict(value = "users", key = "#user.id")
    public User updateUser(User user) {
        return userRepository.save(user);
    }
    
    @CacheEvict(value = "users", allEntries = true)
    public void clearCache() {
        // Clears entire cache
    }
    
    @Cacheable(value = "usersByEmail", condition = "#email.length() > 3")
    public User findByEmail(String email) {
        return userRepository.findByEmail(email);
    }
}

// Redis configuration
@Configuration
public class CacheConfig {
    
    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory connectionFactory) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofMinutes(60))
            .serializeKeysWith(RedisSerializationContext.SerializationPair
                .fromSerializer(new StringRedisSerializer()))
            .serializeValuesWith(RedisSerializationContext.SerializationPair
                .fromSerializer(new GenericJackson2JsonRedisSerializer()));
        
        return RedisCacheManager.builder(connectionFactory)
            .cacheDefaults(config)
            .build();
    }
}
```

### 13. What is Spring Boot DevTools and how does it help in development?
**Expected Answer:**
DevTools provides development-time features:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
```

**Features:**
- **Automatic restart**: Restarts application when classpath files change
- **LiveReload**: Automatically refreshes browser
- **Property defaults**: Development-friendly defaults
- **H2 console**: Enables H2 database console

```yaml
# application-dev.yml
spring:
  devtools:
    restart:
      enabled: true
    livereload:
      enabled: true
    add-properties: true
```

### 14. How do you implement validation in Spring Boot?
**Expected Answer:**
```java
// Entity with validation annotations
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 50, message = "Name must be between 2 and 50 characters")
    private String name;
    
    @Email(message = "Email should be valid")
    @NotBlank(message = "Email is required")
    private String email;
    
    @Min(value = 18, message = "Age should not be less than 18")
    @Max(value = 65, message = "Age should not be greater than 65")
    private Integer age;
    
    @Pattern(regexp = "^\\d{10}$", message = "Phone number should be 10 digits")
    private String phone;
}

// Controller with validation
@RestController
public class UserController {
    
    @PostMapping("/users")
    public ResponseEntity<?> createUser(@Valid @RequestBody User user, 
                                       BindingResult result) {
        if (result.hasErrors()) {
            Map<String, String> errors = new HashMap<>();
            result.getFieldErrors().forEach(error -> 
                errors.put(error.getField(), error.getDefaultMessage()));
            return ResponseEntity.badRequest().body(errors);
        }
        
        User savedUser = userService.save(user);
        return ResponseEntity.ok(savedUser);
    }
}

// Custom validator
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = UniqueEmailValidator.class)
public @interface UniqueEmail {
    String message() default "Email already exists";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

@Component
public class UniqueEmailValidator implements ConstraintValidator<UniqueEmail, String> {
    
    @Autowired
    private UserRepository userRepository;
    
    @Override
    public boolean isValid(String email, ConstraintValidatorContext context) {
        return userRepository.findByEmail(email).isEmpty();
    }
}
```

### 15. How do you configure multiple data sources in Spring Boot?
**Expected Answer:**
```java
@Configuration
public class DataSourceConfig {
    
    @Bean
    @Primary
    @ConfigurationProperties("spring.datasource.primary")
    public DataSource primaryDataSource() {
        return DataSourceBuilder.create().build();
    }
    
    @Bean
    @ConfigurationProperties("spring.datasource.secondary")
    public DataSource secondaryDataSource() {
        return DataSourceBuilder.create().build();
    }
    
    @Bean
    @Primary
    public JdbcTemplate primaryJdbcTemplate(@Qualifier("primaryDataSource") DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
    
    @Bean
    public JdbcTemplate secondaryJdbcTemplate(@Qualifier("secondaryDataSource") DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}

// JPA configuration for multiple data sources
@Configuration
@EnableJpaRepositories(
    basePackages = "com.example.primary.repository",
    entityManagerFactoryRef = "primaryEntityManagerFactory",
    transactionManagerRef = "primaryTransactionManager"
)
public class PrimaryDataSourceConfig {
    
    @Bean
    @Primary
    public LocalContainerEntityManagerFactoryBean primaryEntityManagerFactory(
            @Qualifier("primaryDataSource") DataSource dataSource) {
        
        LocalContainerEntityManagerFactoryBean factory = new LocalContainerEntityManagerFactoryBean();
        factory.setDataSource(dataSource);
        factory.setPackagesToScan("com.example.primary.entity");
        factory.setJpaVendorAdapter(new HibernateJpaVendorAdapter());
        return factory;
    }
    
    @Bean
    @Primary
    public PlatformTransactionManager primaryTransactionManager(
            @Qualifier("primaryEntityManagerFactory") EntityManagerFactory factory) {
        return new JpaTransactionManager(factory);
    }
}
```

### 16. What are different ways to run Spring Boot applications?
**Expected Answer:**
1. **IDE**: Run main method directly
2. **Maven**: `mvn spring-boot:run`
3. **Gradle**: `gradle bootRun`
4. **JAR file**: `java -jar myapp.jar`
5. **WAR deployment**: Deploy to servlet container
6. **Docker**: Containerized deployment

```dockerfile
# Dockerfile
FROM openjdk:17-jre-slim
COPY target/myapp.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

```yaml
# docker-compose.yml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=docker
    depends_on:
      - mysql
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_DATABASE: myapp
      MYSQL_ROOT_PASSWORD: password
```

### 17. How do you implement logging in Spring Boot?
**Expected Answer:**
```yaml
# application.yml
logging:
  level:
    com.example: DEBUG
    org.springframework: INFO
    org.hibernate: DEBUG
  pattern:
    console: "%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n"
    file: "%d{ISO8601} [%thread] %-5level %logger{36} - %msg%n"
  file:
    name: logs/application.log
  logback:
    rollingpolicy:
      max-file-size: 10MB
      max-history: 30
```

```java
// Using SLF4J
@RestController
@Slf4j  // Lombok annotation
public class UserController {
    
    // Or manually create logger
    private static final Logger logger = LoggerFactory.getLogger(UserController.class);
    
    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        log.info("Fetching user with id: {}", id);
        
        try {
            User user = userService.findById(id);
            log.debug("Found user: {}", user.getName());
            return user;
        } catch (Exception e) {
            log.error("Error fetching user with id: {}", id, e);
            throw e;
        }
    }
}

// Custom logback configuration
<!-- logback-spring.xml -->
<configuration>
    <springProfile name="dev">
        <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
            <encoder>
                <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
            </encoder>
        </appender>
        <root level="DEBUG">
            <appender-ref ref="CONSOLE"/>
        </root>
    </springProfile>
    
    <springProfile name="prod">
        <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
            <file>logs/app.log</file>
            <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                <fileNamePattern>logs/app.%d{yyyy-MM-dd}.log</fileNamePattern>
                <maxHistory>30</maxHistory>
            </rollingPolicy>
            <encoder>
                <pattern>%d{ISO8601} [%thread] %-5level %logger{36} - %msg%n</pattern>
            </encoder>
        </appender>
        <root level="INFO">
            <appender-ref ref="FILE"/>
        </root>
    </springProfile>
</configuration>
```

### 18. How do you implement pagination and sorting in Spring Boot?
**Expected Answer:**
```java
// Repository
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Page<User> findByNameContainingIgnoreCase(String name, Pageable pageable);
    Page<User> findByAgeGreaterThan(Integer age, Pageable pageable);
}

// Controller
@RestController
public class UserController {
    
    @GetMapping("/users")
    public Page<User> getUsers(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size,
            @RequestParam(defaultValue = "id") String sortBy,
            @RequestParam(defaultValue = "ASC") String sortDir,
            @RequestParam(required = false) String search) {
        
        Sort sort = sortDir.equalsIgnoreCase("DESC") ? 
            Sort.by(sortBy).descending() : Sort.by(sortBy).ascending();
        
        Pageable pageable = PageRequest.of(page, size, sort);
        
        if (search != null && !search.isEmpty()) {
            return userRepository.findByNameContainingIgnoreCase(search, pageable);
        }
        
        return userRepository.findAll(pageable);
    }
    
    // Custom pagination response
    @GetMapping("/users/custom")
    public ResponseEntity<?> getUsersCustom(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size) {
        
        Page<User> userPage = userRepository.findAll(PageRequest.of(page, size));
        
        Map<String, Object> response = new HashMap<>();
        response.put("users", userPage.getContent());
        response.put("currentPage", userPage.getNumber());
        response.put("totalItems", userPage.getTotalElements());
        response.put("totalPages", userPage.getTotalPages());
        response.put("hasNext", userPage.hasNext());
        response.put("hasPrevious", userPage.hasPrevious());
        
        return ResponseEntity.ok(response);
    }
}
```

### 19. How do you implement file upload in Spring Boot?
**Expected Answer:**
```yaml
# application.yml
spring:
  servlet:
    multipart:
      max-file-size: 10MB
      max-request-size: 10MB
      enabled: true
```

```java
@RestController
public class FileUploadController {
    
    @Value("${file.upload-dir}")
    private String uploadDir;
    
    @PostMapping("/upload")
    public ResponseEntity<?> uploadFile(@RequestParam("file") MultipartFile file) {
        try {
            if (file.isEmpty()) {
                return ResponseEntity.badRequest().body("File is empty");
            }
            
            // Validate file type
            String contentType = file.getContentType();
            if (!isValidFileType(contentType)) {
                return ResponseEntity.badRequest().body("Invalid file type");
            }
            
            // Generate unique filename
            String originalFilename = file.getOriginalFilename();
            String filename = UUID.randomUUID().toString() + "_" + originalFilename;
            
            // Save file
            Path targetLocation = Paths.get(uploadDir).resolve(filename);
            Files.copy(file.getInputStream(), targetLocation, StandardCopyOption.REPLACE_EXISTING);
            
            Map<String, String> response = new HashMap<>();
            response.put("filename", filename);
            response.put("url", "/files/" + filename);
            response.put("size", String.valueOf(file.getSize()));
            
            return ResponseEntity.ok(response);
            
        } catch (IOException e) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body("Could not upload file: " + e.getMessage());
        }
    }
    
    @GetMapping("/files/{filename:.+}")
    public ResponseEntity<Resource> downloadFile(@PathVariable String filename) {
        try {
            Path filePath = Paths.get(uploadDir).resolve(filename).normalize();
            Resource resource = new UrlResource(filePath.toUri());
            
            if (resource.exists()) {
                return ResponseEntity.ok()
                    .contentType(MediaType.APPLICATION_OCTET_STREAM)
                    .header(HttpHeaders.CONTENT_DISPOSITION, 
                           "attachment; filename=\"" + resource.getFilename() + "\"")
                    .body(resource);
            } else {
                return ResponseEntity.notFound().build();
            }
        } catch (Exception e) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
        }
    }
    
    private boolean isValidFileType(String contentType) {
        return contentType != null && (
            contentType.equals("image/jpeg") ||
            contentType.equals("image/png") ||
            contentType.equals("application/pdf") ||
            contentType.equals("text/plain")
        );
    }
}
```

### 20. What is CommandLineRunner and ApplicationRunner?
**Expected Answer:**
Both interfaces allow you to run code after Spring Boot application starts:

```java
// CommandLineRunner - works with String arguments
@Component
public class DataInitializer implements CommandLineRunner {
    
    @Autowired
    private UserRepository userRepository;
    
    @Override
    public void run(String... args) throws Exception {
        if (userRepository.count() == 0) {
            userRepository.save(new User("admin", "admin@example.com"));
            userRepository.save(new User("user", "user@example.com"));
            System.out.println("Sample users created");
        }
    }
}

// ApplicationRunner - works with ApplicationArguments
@Component
@Order(1)  // Control execution order
public class ConfigurationChecker implements ApplicationRunner {
    
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("Non-option args: " + args.getNonOptionArgs());
        System.out.println("Option args: " + args.getOptionNames());
        
        if (args.containsOption("debug")) {
            System.out.println("Debug mode enabled");
        }
    }
}

// Bean definition approach
@Configuration
public class StartupConfig {
    
    @Bean
    public CommandLineRunner init(UserService userService) {
        return args -> {
            // Initialization logic
            userService.initializeData();
        };
    }
}
```

---

## 🚀 Advanced Level Questions (5+ Years)

### 21. How do you implement custom auto-configuration in Spring Boot?
**Expected Answer:**
```java
// Custom auto-configuration class
@Configuration
@ConditionalOnClass(MyCustomService.class)
@ConditionalOnMissingBean(MyCustomService.class)
@EnableConfigurationProperties(MyCustomProperties.class)
public class MyCustomAutoConfiguration {
    
    @Bean
    @ConditionalOnProperty(prefix = "mycustom", name = "enabled", havingValue = "true", matchIfMissing = true)
    public MyCustomService myCustomService(MyCustomProperties properties) {
        return new MyCustomService(properties);
    }
    
    @Bean
    @ConditionalOnBean(MyCustomService.class)
    public MyCustomController myCustomController(MyCustomService service) {
        return new MyCustomController(service);
    }
}

// Properties class
@ConfigurationProperties(prefix = "mycustom")
@Data
public class MyCustomProperties {
    private boolean enabled = true;
    private String apiKey;
    private Duration timeout = Duration.ofSeconds(30);
    private List<String> allowedHosts = new ArrayList<>();
}

// META-INF/spring.factories (Spring Boot 2.x)
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.example.autoconfigure.MyCustomAutoConfiguration

// META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports (Spring Boot 3.x)
com.example.autoconfigure.MyCustomAutoConfiguration

// Starter module structure
my-custom-spring-boot-starter/
├── src/main/java/
│   └── com/example/starter/
│       └── MyCustomStarterApplication.java
├── src/main/resources/
│   └── META-INF/
│       └── spring/
│           └── org.springframework.boot.autoconfigure.AutoConfiguration.imports
└── pom.xml
```

### 22. How do you implement distributed tracing in Spring Boot?
**Expected Answer:**
```xml
<!-- Dependencies -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-sleuth-zipkin</artifactId>
</dependency>
```

```yaml
# application.yml
spring:
  sleuth:
    sampler:
      probability: 1.0  # Sample 100% of requests (use 0.1 in production)
    zipkin:
      base-url: http://localhost:9411
  application:
    name: user-service
```

```java
// Manual span creation
@Service
public class UserService {
    
    private final Tracer tracer;
    
    public UserService(Tracer tracer) {
        this.tracer = tracer;
    }
    
    public User processUser(User user) {
        Span span = tracer.nextSpan()
            .name("user-processing")
            .tag("user.id", String.valueOf(user.getId()))
            .start();
        
        try (Tracer.SpanInScope ws = tracer.withSpanInScope(span)) {
            // Business logic
            span.tag("operation", "validation");
            validateUser(user);
            
            span.tag("operation", "enrichment");
            enrichUserData(user);
            
            return user;
        } catch (Exception e) {
            span.tag("error", e.getMessage());
            throw e;
        } finally {
            span.end();
        }
    }
}

// Custom TraceFilter
@Component
public class CustomTraceFilter implements Filter {
    
    private final TraceContext.Injector<HttpServletRequest> injector;
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, 
                        FilterChain chain) throws IOException, ServletException {
        
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        
        // Add custom headers to trace
        String correlationId = httpRequest.getHeader("X-Correlation-ID");
        if (correlationId != null) {
            MDC.put("correlationId", correlationId);
        }
        
        try {
            chain.doFilter(request, response);
        } finally {
            MDC.clear();
        }
    }
}
```

### 23. How do you implement circuit breaker pattern in Spring Boot?
**Expected Answer:**
```xml
<!-- Resilience4j dependency -->
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-spring-boot2</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

```yaml
# application.yml
resilience4j:
  circuitbreaker:
    instances:
      backendService:
        registerHealthIndicator: true
        slidingWindowSize: 10
        minimumNumberOfCalls: 5
        permittedNumberOfCallsInHalfOpenState: 3
        waitDurationInOpenState: 60s
        failureRateThreshold: 50
        eventConsumerBufferSize: 10
        slowCallRateThreshold: 100
        slowCallDurationThreshold: 60000ms
        
  retry:
    instances:
      backendService:
        maxAttempts: 3
        waitDuration: 1s
        exponentialBackoffMultiplier: 2
        
  ratelimiter:
    instances:
      backendService:
        limitRefreshPeriod: 1s
        limitForPeriod: 10
        timeoutDuration: 0s
```

```java
@Service
public class ExternalService {
    
    @CircuitBreaker(name = "backendService", fallbackMethod = "fallbackMethod")
    @Retry(name = "backendService")
    @RateLimiter(name = "backendService")
    public String callExternalService(String data) {
        // Call to external service
        return restTemplate.postForObject("/external-api", data, String.class);
    }
    
    public String fallbackMethod(String data, Exception ex) {
        return "Fallback response: " + ex.getMessage();
    }
    
    // Programmatic approach
    private final CircuitBreaker circuitBreaker;
    private final Retry retry;
    
    public String callWithProgrammaticCircuitBreaker(String data) {
        Supplier<String> decoratedSupplier = CircuitBreaker
            .decorateSupplier(circuitBreaker, () -> {
                return restTemplate.postForObject("/external-api", data, String.class);
            });
            
        decoratedSupplier = Retry.decorateSupplier(retry, decoratedSupplier);
        
        return decoratedSupplier.get();
    }
}

// Circuit breaker event listener
@Component
public class CircuitBreakerEventListener {
    
    @EventListener
    public void onCircuitBreakerEvent(CircuitBreakerOnStateTransitionEvent event) {
        System.out.printf("Circuit breaker %s changed from %s to %s%n", 
            event.getCircuitBreakerName(), 
            event.getStateTransition().getFromState(), 
            event.getStateTransition().getToState());
    }
}
```

### 24. How do you implement custom metrics and monitoring?
**Expected Answer:**
```java
// Custom metrics with Micrometer
@Service
public class UserService {
    
    private final MeterRegistry meterRegistry;
    private final Counter userCreationCounter;
    private final Timer userProcessingTimer;
    private final Gauge activeUsersGauge;
    
    public UserService(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
        this.userCreationCounter = Counter.builder("user.creation")
            .description("Number of users created")
            .register(meterRegistry);
            
        this.userProcessingTimer = Timer.builder("user.processing.time")
            .description("Time taken to process user")
            .register(meterRegistry);
            
        this.activeUsersGauge = Gauge.builder("user.active")
            .description("Number of active users")
            .register(meterRegistry, this, UserService::getActiveUserCount);
    }
    
    public User createUser(User user) {
        Timer.Sample sample = Timer.start(meterRegistry);
        
        try {
            User savedUser = userRepository.save(user);
            userCreationCounter.increment(
                Tags.of("status", "success", "type", user.getType())
            );
            
            return savedUser;
        } catch (Exception e) {
            userCreationCounter.increment(
                Tags.of("status", "error", "error.type", e.getClass().getSimpleName())
            );
            throw e;
        } finally {
            sample.stop(userProcessingTimer);
        }
    }
    
    private double getActiveUserCount() {
        return userRepository.countByStatus("ACTIVE");
    }
}

// Custom health indicator
@Component
public class DatabaseHealthIndicator implements HealthIndicator {
    
    @Autowired
    private DataSource dataSource;
    
    @Override
    public Health health() {
        try (Connection connection = dataSource.getConnection()) {
            if (connection.isValid(2)) {
                return Health.up()
                    .withDetail("database", "Available")
                    .withDetail("validationQuery", connection.getMetaData().getURL())
                    .build();
            }
        } catch (Exception e) {
            return Health.down()
                .withDetail("database", "Unavailable")
                .withException(e)
                .build();
        }
        
        return Health.down()
            .withDetail("database", "Validation failed")
            .build();
    }
}

// Custom endpoint
@Component
@Endpoint(id = "custom")
public class CustomEndpoint {
    
    @ReadOperation
    public Map<String, Object> customInfo() {
        Map<String, Object> info = new HashMap<>();
        info.put("status", "operational");
        info.put("customMetric", calculateCustomMetric());
        info.put("timestamp", Instant.now());
        return info;
    }
    
    @WriteOperation
    public void configureCustom(@Selector String key, String value) {
        // Custom write operation
        System.out.println("Configuring " + key + " = " + value);
    }
    
    private double calculateCustomMetric() {
        // Custom calculation
        return Math.random() * 100;
    }
}
```

### 25. How do you implement event-driven architecture in Spring Boot?
**Expected Answer:**
```java
// Domain event
public class UserRegisteredEvent extends ApplicationEvent {
    private final User user;
    
    public UserRegisteredEvent(Object source, User user) {
        super(source);
        this.user = user;
    }
    
    public User getUser() {
        return user;
    }
}

// Event publisher
@Service
public class UserService {
    
    private final ApplicationEventPublisher eventPublisher;
    private final UserRepository userRepository;
    
    public UserService(ApplicationEventPublisher eventPublisher, UserRepository userRepository) {
        this.eventPublisher = eventPublisher;
        this.userRepository = userRepository;
    }
    
    @Transactional
    public User registerUser(User user) {
        User savedUser = userRepository.save(user);
        
        // Publish event
        eventPublisher.publishEvent(new UserRegisteredEvent(this, savedUser));
        
        return savedUser;
    }
}

// Event listeners
@Component
public class UserEventListener {
    
    @EventListener
    @Async
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    public void handleUserRegistered(UserRegisteredEvent event) {
        User user = event.getUser();
        
        // Send welcome email
        emailService.sendWelcomeEmail(user);
        
        // Update analytics
        analyticsService.trackUserRegistration(user);
    }
    
    @EventListener(condition = "#event.user.premium == true")
    public void handlePremiumUserRegistered(UserRegisteredEvent event) {
        // Premium user specific handling
        premiumService.setupPremiumFeatures(event.getUser());
    }
}

// Async configuration
@Configuration
@EnableAsync
public class AsyncConfig {
    
    @Bean(name = "eventTaskExecutor")
    public Executor eventTaskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(2);
        executor.setMaxPoolSize(5);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("Event-");
        executor.initialize();
        return executor;
    }
}

// Message broker integration with RabbitMQ
@Configuration
@EnableRabbit
public class RabbitConfig {
    
    @Bean
    public TopicExchange userExchange() {
        return new TopicExchange("user.exchange");
    }
    
    @Bean
    public Queue userRegistrationQueue() {
        return QueueBuilder.durable("user.registration").build();
    }
    
    @Bean
    public Binding userRegistrationBinding() {
        return BindingBuilder.bind(userRegistrationQueue())
            .to(userExchange())
            .with("user.registered");
    }
    
    @Bean
    public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
        RabbitTemplate template = new RabbitTemplate(connectionFactory);
        template.setMessageConverter(new Jackson2JsonMessageConverter());
        return template;
    }
}

@Service
public class MessagePublisher {
    
    private final RabbitTemplate rabbitTemplate;
    
    @EventListener
    public void handleUserRegistered(UserRegisteredEvent event) {
        rabbitTemplate.convertAndSend("user.exchange", "user.registered", event.getUser());
    }
}

@RabbitListener(queues = "user.registration")
public class MessageListener {
    
    public void handleUserRegistration(@Payload User user, 
                                     @Header Map<String, Object> headers) {
        // Process the message
        System.out.println("Processing user registration: " + user.getName());
    }
}
```

---

## Spring Boot 3.x Specific Questions

### 26. What are the major changes in Spring Boot 3.x?
**Expected Answer:**
**Key Changes:**
1. **Java 17 minimum requirement**
2. **Spring Framework 6.x** with native image support
3. **Jakarta EE** instead of Java EE (javax → jakarta)
4. **Observability improvements** with Micrometer Tracing
5. **Native compilation** with GraalVM
6. **Auto-configuration registration changes**

```java
// Migration changes
// Before (Spring Boot 2.x)
import javax.servlet.http.HttpServletRequest;
import javax.persistence.Entity;
import javax.validation.Valid;

// After (Spring Boot 3.x)  
import jakarta.servlet.http.HttpServletRequest;
import jakarta.persistence.Entity;
import jakarta.validation.Valid;

// New auto-configuration registration
// META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
com.example.MyAutoConfiguration

// Observability (replaces Sleuth)
@RestController
public class UserController {
    
    @GetMapping("/users/{id}")
    @Observed(name = "user.get", contextualName = "get-user")
    public User getUser(@PathVariable Long id) {
        return userService.findById(id);
    }
}
```

### 27. How do you implement native image compilation with Spring Boot 3.x?
**Expected Answer:**
```xml
<!-- pom.xml -->
<plugin>
    <groupId>org.graalvm.buildtools</groupId>
    <artifactId>native-maven-plugin</artifactId>
</plugin>

<profiles>
    <profile>
        <id>native</id>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                    <configuration>
                        <image>
                            <builder>paketobuildpacks/builder:tiny</builder>
                            <env>
                                <BP_NATIVE_IMAGE>true</BP_NATIVE_IMAGE>
                            </env>
                        </image>
                    </configuration>
                </plugin>
            </plugins>
        </build>
    </profile>
</profiles>
```

```java
// Native hints for reflection
@RegisterReflectionForBinding(User.class)
@NativeHint(
    types = @TypeHint(types = MyService.class, access = AccessBits.ALL),
    resources = @ResourceHint(patterns = "application.yml")
)
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

// Runtime hints
@Configuration
public class MyRuntimeHintsRegistrar implements RuntimeHintsRegistrar {
    
    @Override
    public void registerHints(RuntimeHints hints, ClassLoader classLoader) {
        hints.reflection().registerType(MyClass.class, 
            MemberCategory.INVOKE_DECLARED_CONSTRUCTORS,
            MemberCategory.INVOKE_DECLARED_METHODS);
            
        hints.resources().registerPattern("templates/*.html");
    }
}
```

```bash
# Build commands
mvn -Pnative native:compile
mvn spring-boot:build-image -Pnative
```

---

## Microservices with Spring Boot

### 28. How do you implement service discovery in Spring Boot microservices?
**Expected Answer:**
```xml
<!-- Eureka Client -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

```yaml
# application.yml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka
    register-with-eureka: true
    fetch-registry: true
  instance:
    prefer-ip-address: true
    lease-renewal-interval-in-seconds: 10
    lease-expiration-duration-in-seconds: 20

spring:
  application:
    name: user-service
```

```java
// Service registration
@SpringBootApplication
@EnableEurekaClient
public class UserServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }
}

// Service consumption with LoadBalancer
@RestController
public class UserController {
    
    @Autowired
    private LoadBalancerClient loadBalancerClient;
    
    @Autowired
    @LoadBalanced
    private RestTemplate restTemplate;
    
    @GetMapping("/user/{id}/orders")
    public List<Order> getUserOrders(@PathVariable Long id) {
        // Using service name instead of hardcoded URL
        return restTemplate.getForObject(
            "http://order-service/orders/user/" + id, 
            List.class);
    }
    
    // Manual service discovery
    @GetMapping("/user/{id}/payments")  
    public List<Payment> getUserPayments(@PathVariable Long id) {
        ServiceInstance instance = loadBalancerClient.choose("payment-service");
        String url = String.format("http://%s:%s/payments/user/%s", 
            instance.getHost(), instance.getPort(), id);
        
        return restTemplate.getForObject(url, List.class);
    }
}

// OpenFeign client
@FeignClient(name = "order-service", fallback = OrderServiceFallback.class)
public interface OrderServiceClient {
    
    @GetMapping("/orders/user/{userId}")
    List<Order> getOrdersByUser(@PathVariable Long userId);
    
    @PostMapping("/orders")
    Order createOrder(@RequestBody Order order);
}

@Component
public class OrderServiceFallback implements OrderServiceClient {
    
    @Override
    public List<Order> getOrdersByUser(Long userId) {
        return Collections.emptyList();
    }
    
    @Override
    public Order createOrder(Order order) {
        throw new ServiceUnavailableException("Order service is unavailable");
    }
}
```

### 29. How do you implement API Gateway with Spring Boot?
**Expected Answer:**
```xml
<!-- Spring Cloud Gateway -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

```yaml
# application.yml
spring:
  cloud:
    gateway:
      routes:
        - id: user-service
          uri: lb://user-service
          predicates:
            - Path=/api/users/**
          filters:
            - StripPrefix=1
            - AddRequestHeader=X-Request-Source, gateway
            - name: CircuitBreaker
              args:
                name: user-service
                fallbackUri: forward:/fallback/users
                
        - id: order-service
          uri: lb://order-service
          predicates:
            - Path=/api/orders/**
            - Method=GET,POST
          filters:
            - StripPrefix=1
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10
                redis-rate-limiter.burstCapacity: 20
                key-resolver: "#{@userKeyResolver}"
                
      globalcors:
        cors-configurations:
          '[/**]':
            allowedOrigins: "*"
            allowedMethods: "*"
            allowedHeaders: "*"
```

```java
// Custom filter
@Component
public class CustomGlobalFilter implements GlobalFilter, Ordered {
    
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();
        ServerHttpResponse response = exchange.getResponse();
        
        // Add correlation ID
        String correlationId = UUID.randomUUID().toString();
        ServerHttpRequest mutatedRequest = request.mutate()
            .header("X-Correlation-ID", correlationId)
            .build();
            
        ServerWebExchange mutatedExchange = exchange.mutate()
            .request(mutatedRequest)
            .build();
        
        // Log request
        System.out.println("Request: " + request.getMethod() + " " + request.getURI());
        
        return chain.filter(mutatedExchange)
            .doOnSuccess(aVoid -> {
                // Log response
                System.out.println("Response: " + response.getStatusCode());
            });
    }
    
    @Override
    public int getOrder() {
        return -1; // High priority
    }
}

// Custom predicate
@Component
public class CustomRoutePredicateFactory extends AbstractRoutePredicateFactory<CustomRoutePredicateFactory.Config> {
    
    public CustomRoutePredicateFactory() {
        super(Config.class);
    }
    
    @Override
    public Predicate<ServerWebExchange> apply(Config config) {
        return exchange -> {
            ServerHttpRequest request = exchange.getRequest();
            return request.getHeaders().containsKey(config.getHeaderName());
        };
    }
    
    public static class Config {
        private String headerName;
        
        // getter and setter
    }
}

// Rate limiting key resolver
@Bean
public KeyResolver userKeyResolver() {
    return exchange -> {
        String userId = exchange.getRequest().getHeaders().getFirst("X-User-ID");
        return Mono.just(userId != null ? userId : "anonymous");
    };
}

// Fallback controller
@RestController
public class FallbackController {
    
    @GetMapping("/fallback/users")
    public Mono<String> userFallback() {
        return Mono.just("User service is currently unavailable");
    }
    
    @GetMapping("/fallback/orders")
    public Mono<String> orderFallback() {
        return Mono.just("Order service is currently unavailable");
    }
}
```

### 30. How do you implement distributed configuration management?
**Expected Answer:**
```xml
<!-- Spring Cloud Config Client -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

```yaml
# bootstrap.yml (client)
spring:
  application:
    name: user-service
  cloud:
    config:
      uri: http://localhost:8888
      fail-fast: true
      retry:
        initial-interval: 1000
        multiplier: 1.1
        max-attempts: 6
      username: config-user
      password: config-password
  profiles:
    active: dev
```

```java
// Config Server
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}

// Configuration properties with refresh
@ConfigurationProperties(prefix = "app")
@RefreshScope
@Component
public class AppConfig {
    private String name;
    private String version;
    private Map<String, String> features = new HashMap<>();
    
    // getters and setters
}

// Using configuration
@RestController
@RefreshScope
public class ConfigController {
    
    @Value("${app.message:Default message}")
    private String message;
    
    @Autowired
    private AppConfig appConfig;
    
    @GetMapping("/config")
    public Map<String, Object> getConfig() {
        Map<String, Object> config = new HashMap<>();
        config.put("message", message);
        config.put("appName", appConfig.getName());
        config.put("version", appConfig.getVersion());
        config.put("features", appConfig.getFeatures());
        return config;
    }
    
    // Refresh configuration endpoint
    @PostMapping("/refresh")
    public String refresh() {
        return "Configuration refreshed";
    }
}

// Bus for broadcasting config changes
@Configuration
@EnableAutoConfiguration
public class BusConfig {
    
    // RabbitMQ configuration for config bus
    @Bean
    public ConnectionFactory connectionFactory() {
        CachingConnectionFactory factory = new CachingConnectionFactory("localhost");
        factory.setUsername("guest");
        factory.setPassword("guest");
        return factory;
    }
}

// Environment-specific configurations
# application-dev.yml
app:
  name: User Service
  version: 1.0.0-DEV
  features:
    new-ui: true
    beta-feature: true

# application-prod.yml  
app:
  name: User Service
  version: 1.0.0
  features:
    new-ui: false
    beta-feature: false
```

---

## Performance & Production Questions

### 31. How do you optimize Spring Boot application performance?
**Expected Answer:**
**Key optimization strategies:**

```yaml
# JVM tuning
JAVA_OPTS: "-Xmx2g -Xms2g -XX:+UseG1GC -XX:MaxGCPauseMillis=200"

# Connection pooling
spring:
  datasource:
    hikari:
      minimum-idle: 5
      maximum-pool-size: 20
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000
      
# Caching
  cache:
    type: redis
    redis:
      time-to-live: 600000
```

```java
// Lazy loading
@SpringBootApplication
@EnableCaching
public class Application {
    
    @Bean
    @Lazy
    public ExpensiveService expensiveService() {
        return new ExpensiveService();
    }
}

// Async processing
@Service
public class EmailService {
    
    @Async("taskExecutor")
    public CompletableFuture<Void> sendEmailAsync(String to, String subject, String body) {
        // Email sending logic
        return CompletableFuture.completedFuture(null);
    }
}

// Database optimization
@Entity
@Table(indexes = {
    @Index(name = "idx_email", columnList = "email"),
    @Index(name = "idx_created_date", columnList = "createdDate")
})
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    // Use appropriate fetch strategies
    @OneToMany(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    private List<Order> orders;
    
    // Use @Query for complex queries
    @Query("SELECT u FROM User u WHERE u.email = :email AND u.active = true")
    Optional<User> findActiveUserByEmail(@Param("email") String email);
}

// HTTP/2 and compression
server:
  http2:
    enabled: true
  compression:
    enabled: true
    mime-types: text/html,text/xml,text/plain,text/css,text/javascript,application/javascript,application/json
    min-response-size: 1024
```

### 32. How do you monitor Spring Boot applications in production?
**Expected Answer:**
```java
// Custom metrics
@Component
public class BusinessMetrics {
    
    private final MeterRegistry meterRegistry;
    private final Counter orderCounter;
    private final Timer responseTimer;
    private final DistributionSummary requestSizeSummary;
    
    public BusinessMetrics(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
        this.orderCounter = Counter.builder("orders.created")
            .description("Number of orders created")
            .register(meterRegistry);
            
        this.responseTimer = Timer.builder("api.response.time")
            .description("API response time")
            .register(meterRegistry);
            
        this.requestSizeSummary = DistributionSummary.builder("request.size")
            .description("Request payload size")
            .register(meterRegistry);
    }
    
    @EventListener
    public void onOrderCreated(OrderCreatedEvent event) {
        orderCounter.increment(
            Tags.of(
                "type", event.getOrder().getType(),
                "status", event.getOrder().getStatus()
            )
        );
    }
}

// APM integration
@Configuration
public class ObservabilityConfig {
    
    @Bean
    public ObservationRegistryCustomizer<ObservationRegistry> observationRegistryCustomizer() {
        return registry -> {
            registry.observationConfig()
                .observationHandler(new ObservationTextHandler())
                .observationHandler(new MicrometerObservationHandler(meterRegistry));
        };
    }
}

// Health checks
@Component
public class CustomHealthIndicators {
    
    @Bean
    public HealthIndicator externalServiceHealth() {
        return () -> {
            try {
                // Check external service
                restTemplate.getForObject("http://external-service/health", String.class);
                return Health.up().withDetail("external-service", "Available").build();
            } catch (Exception e) {
                return Health.down().withException(e).build();
            }
        };
    }
    
    @Bean
    public HealthIndicator diskSpaceHealth() {
        return new DiskSpaceHealthIndicator(Paths.get("/"), Duration.ofSeconds(2));
    }
}

// Structured logging
@RestController
public class UserController {
    
    private final Logger logger = LoggerFactory.getLogger(UserController.class);
    
    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        
        // Structured logging with MDC
        MDC.put("userId", String.valueOf(id));
        MDC.put("operation", "getUser");
        
        try {
            logger.info("Fetching user with id: {}", id);
            User user = userService.findById(id);
            
            logger.info("User found: {}", user.getEmail());
            return user;
            
        } catch (UserNotFoundException e) {
            logger.warn("User not found with id: {}", id);
            throw e;
        } catch (Exception e) {
            logger.error("Error fetching user", e);
            throw e;
        } finally {
            MDC.clear();
        }
    }
}
```

---

## Spring Boot Testing Questions

### 33. How do you write different types of tests in Spring Boot?
**Expected Answer:**
```java
// Unit tests
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @Mock
    private EmailService emailService;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    void shouldCreateUser() {
        // Given
        User user = new User("test@example.com", "Test User");
        when(userRepository.save(any(User.class))).thenReturn(user);
        
        // When
        User result = userService.createUser(user);
        
        // Then
        assertThat(result.getEmail()).isEqualTo("test@example.com");
        verify(emailService).sendWelcomeEmail(user);
    }
}

// Integration tests
@SpringBootTest
@AutoConfigureTestDatabase
@Transactional
class UserServiceIntegrationTest {
    
    @Autowired
    private UserService userService;
    
    @Autowired
    private UserRepository userRepository;
    
    @MockBean
    private EmailService emailService;
    
    @Test
    void shouldCreateAndRetrieveUser() {
        // Given
        User user = new User("test@example.com", "Test User");
        
        // When
        User created = userService.createUser(user);
        User retrieved = userService.findById(created.getId());
        
        // Then
        assertThat(retrieved.getEmail()).isEqualTo("test@example.com");
        verify(emailService).sendWelcomeEmail(any(User.class));
    }
}

// Web layer tests
@WebMvcTest(UserController.class)
class UserControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private UserService userService;
    
    @Test
    void shouldReturnUser() throws Exception {
        // Given
        User user = new User("test@example.com", "Test User");
        when(userService.findById(1L)).thenReturn(user);
        
        // When & Then
        mockMvc.perform(get("/users/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.email").value("test@example.com"))
            .andExpected(jsonPath("$.name").value("Test User"));
    }
    
    @Test
    void shouldCreateUser() throws Exception {
        // Given
        User user = new User("test@example.com", "Test User");
        when(userService.createUser(any(User.class))).thenReturn(user);
        
        // When & Then
        mockMvc.perform(post("/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {
                        "email": "test@example.com",
                        "name": "Test User"
                    }
                    """))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.email").value("test@example.com"));
    }
}

// Data layer tests
@DataJpaTest
class UserRepositoryTest {
    
    @Autowired
    private TestEntityManager entityManager;
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    void shouldFindByEmail() {
        // Given
        User user = new User("test@example.com", "Test User");
        entityManager.persistAndFlush(user);
        
        // When
        Optional<User> result = userRepository.findByEmail("test@example.com");
        
        // Then
        assertThat(result).isPresent();
        assertThat(result.get().getName()).isEqualTo("Test User");
    }
}

// Test configuration
@TestConfiguration
public class TestConfig {
    
    @Bean
    @Primary
    public Clock testClock() {
        return Clock.fixed(Instant.parse("2024-01-01T00:00:00Z"), ZoneOffset.UTC);
    }
    
    @Bean
    @Primary
    public EmailService mockEmailService() {
        return Mockito.mock(EmailService.class);
    }
}

// Test containers
@SpringBootTest
@Testcontainers
class UserServiceContainerTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:13")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test");
    
    @Container
    static GenericContainer<?> redis = new GenericContainer<>("redis:6-alpine")
            .withExposedPorts(6379);
    
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
        registry.add("spring.redis.host", redis::getHost);
        registry.add("spring.redis.port", () -> redis.getMappedPort(6379));
    }
    
    @Autowired
    private UserService userService;
    
    @Test
    void shouldWorkWithRealDatabase() {
        User user = userService.createUser(new User("test@example.com", "Test"));
        assertThat(user.getId()).isNotNull();
    }
}
```

---

## Security Questions

### 34. How do you implement authentication and authorization in Spring Boot?
**Expected Answer:**
```java
// Security configuration
@Configuration
@EnableWebSecurity
@EnableMethodSecurity(prePostEnabled = true)
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .requestMatchers(HttpMethod.GET, "/api/users/**").hasAnyRole("USER", "ADMIN")
                .requestMatchers(HttpMethod.POST, "/api/users/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(jwt -> jwt
                    .jwtAuthenticationConverter(jwtAuthenticationConverter())));
        
        return http.build();
    }
    
    @Bean
    public JwtAuthenticationConverter jwtAuthenticationConverter() {
        JwtGrantedAuthoritiesConverter authoritiesConverter = new JwtGrantedAuthoritiesConverter();
        authoritiesConverter.setAuthorityPrefix("ROLE_");
        authoritiesConverter.setAuthoritiesClaimName("roles");
        
        JwtAuthenticationConverter authConverter = new JwtAuthenticationConverter();
        authConverter.setJwtGrantedAuthoritiesConverter(authoritiesConverter);
        return authConverter;
    }
    
    @Bean
    public JwtDecoder jwtDecoder() {
        return NimbusJwtDecoder.withJwkSetUri("http://localhost:8080/auth/realms/myrealm/protocol/openid-connect/certs").build();
    }
}

// Method-level security
@RestController
public class UserController {
    
    @GetMapping("/users/{id}")
    @PreAuthorize("hasRole('ADMIN') or (#id == authentication.principal.subject)")
    public User getUser(@PathVariable Long id) {
        return userService.findById(id);
    }
    
    @PostMapping("/users")
    @PreAuthorize("hasRole('ADMIN')")
    public User createUser(@RequestBody User user) {
        return userService.createUser(user);
    }
    
    @GetMapping("/users/me")
    public User getCurrentUser(Authentication authentication) {
        String userId = authentication.getName();
        return userService.findByUsername(userId);
    }
}

// Custom authentication provider
@Component
public class CustomAuthenticationProvider implements AuthenticationProvider {
    
    @Autowired
    private UserService userService;
    
    @Autowired
    private PasswordEncoder passwordEncoder;
    
    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        String username = authentication.getName();
        String password = authentication.getCredentials().toString();
        
        User user = userService.findByUsername(username);
        
        if (user != null && passwordEncoder.matches(password, user.getPassword())) {
            List<GrantedAuthority> authorities = user.getRoles().stream()
                .map(role -> new SimpleGrantedAuthority("ROLE_" + role.getName()))
                .collect(Collectors.toList());
            
            return new UsernamePasswordAuthenticationToken(username, password, authorities);
        }
        
        throw new BadCredentialsException("Authentication failed");
    }
    
    @Override
    public boolean supports(Class<?> authentication) {
        return authentication.equals(UsernamePasswordAuthenticationToken.class);
    }
}

// JWT token generation
@Service
public class JwtService {
    
    @Value("${jwt.secret}")
    private String secret;
    
    @Value("${jwt.expiration}")
    private Long expiration;
    
    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        return createToken(claims, userDetails.getUsername());
    }
    
    private String createToken(Map<String, Object> claims, String subject) {
        return Jwts.builder()
            .setClaims(claims)
            .setSubject(subject)
            .setIssuedAt(new Date(System.currentTimeMillis()))
            .setExpiration(new Date(System.currentTimeMillis() + expiration))
            .signWith(SignatureAlgorithm.HS512, secret)
            .compact();
    }
    
    public Boolean validateToken(String token, UserDetails userDetails) {
        final String username = getUsernameFromToken(token);
        return (username.equals(userDetails.getUsername()) && !isTokenExpired(token));
    }
}
```

---

## Data & Database Questions

### 35. How do you handle database migrations in Spring Boot?
**Expected Answer:**
```xml
<!-- Flyway dependency -->
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
```

```yaml
# application.yml
spring:
  flyway:
    enabled: true
    locations: classpath:db/migration
    baseline-on-migrate: true
    validate-on-migrate: true
```

```sql
-- V1__Initial_schema.sql
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- V2__Add_user_roles.sql  
CREATE TABLE roles (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL
);

CREATE TABLE user_roles (
    user_id BIGINT REFERENCES users(id),
    role_id BIGINT REFERENCES roles(id),
    PRIMARY KEY (user_id, role_id)
);

-- V3__Add_indexes.sql
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_created_at ON users(created_at);
```

```java
// Custom migration callback
@Component
public class FlywayCallback implements Callback {
    
    @Override
    public boolean supports(Event event, Context context) {
        return event == Event.AFTER_MIGRATE;
    }
    
    @Override
    public boolean canHandleInTransaction(Event event, Context context) {
        return true;
    }
    
    @Override
    public void handle(Event event, Context context) {
        System.out.println("Migration completed successfully");
    }
}

// Conditional migration
@Configuration
public class MigrationConfig {
    
    @Bean
    @ConditionalOnProperty("app.migration.enabled")
    public FlywayMigrationStrategy flywayMigrationStrategy() {
        return flyway -> {
            // Custom migration logic
            flyway.repair();
            flyway.migrate();
        };
    }
}
```

### 36. How do you implement database transactions in Spring Boot?
**Expected Answer:**
```java
// Declarative transactions
@Service
@Transactional
public class OrderService {
    
    @Autowired
    private OrderRepository orderRepository;
    
    @Autowired
    private InventoryService inventoryService;
    
    @Autowired
    private PaymentService paymentService;
    
    @Transactional
    public Order createOrder(Order order) {
        // All operations in single transaction
        Order savedOrder = orderRepository.save(order);
        inventoryService.reserveItems(order.getItems());
        paymentService.processPayment(order.getTotal());
        
        return savedOrder;
    }
    
    @Transactional(readOnly = true)
    public List<Order> findUserOrders(Long userId) {
        return orderRepository.findByUserId(userId);
    }
    
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void logOrderEvent(Long orderId, String event) {
        // Always runs in new transaction
        auditRepository.save(new AuditLog(orderId, event));
    }
    
    @Transactional(rollbackFor = {BusinessException.class, DataException.class})
    public void processOrderWithCustomRollback(Order order) {
        // Custom rollback conditions
        try {
            processOrder(order);
        } catch (BusinessException e) {
            // This will trigger rollback
            throw e;
        } catch (RuntimeException e) {
            // Convert to business exception to trigger rollback  
            throw new BusinessException("Order processing failed", e);
        }
    }
}

// Programmatic transactions
@Service
public class TransactionService {
    
    @Autowired
    private PlatformTransactionManager transactionManager;
    
    public void performComplexOperation() {
        TransactionDefinition def = new DefaultTransactionDefinition();
        TransactionStatus status = transactionManager.getTransaction(def);
        
        try {
            // Business logic
            performBusinessLogic();
            transactionManager.commit(status);
        } catch (Exception e) {
            transactionManager.rollback(status);
            throw e;
        }
    }
    
    // Using TransactionTemplate
    @Autowired
    private TransactionTemplate transactionTemplate;
    
    public String executeInTransaction() {
        return transactionTemplate.execute(status -> {
            // Business logic
            return performBusinessLogic();
        });
    }
}

// Multiple transaction managers
@Configuration
@EnableTransactionManagement
public class TransactionConfig {
    
    @Bean
    @Primary
    public PlatformTransactionManager primaryTransactionManager(
            @Qualifier("primaryEntityManagerFactory") EntityManagerFactory factory) {
        return new JpaTransactionManager(factory);
    }
    
    @Bean
    public PlatformTransactionManager secondaryTransactionManager(
            @Qualifier("secondaryEntityManagerFactory") EntityManagerFactory factory) {
        return new JpaTransactionManager(factory);
    }
}

// Using specific transaction manager
@Service
public class MultiDataSourceService {
    
    @Transactional("primaryTransactionManager")
    public void saveToPrimary(Entity entity) {
        primaryRepository.save(entity);
    }
    
    @Transactional("secondaryTransactionManager")
    public void saveToSecondary(Entity entity) {
        secondaryRepository.save(entity);
    }
}
```

---

## DevOps & Deployment Questions

### 37. How do you containerize a Spring Boot application?
**Expected Answer:**
```dockerfile
# Multi-stage Dockerfile
FROM eclipse-temurin:17-jdk-alpine as build
WORKDIR /workspace/app

COPY mvnw .
COPY .mvn .mvn
COPY pom.xml .
COPY src src

RUN ./mvnw install -DskipTests
RUN mkdir -p target/dependency && (cd target/dependency; jar -xf ../*.jar)

FROM eclipse-temurin:17-jre-alpine
VOLUME /tmp

ARG DEPENDENCY=/workspace/app/target/dependency
COPY --from=build ${DEPENDENCY}/BOOT-INF/lib /app/lib
COPY --from=build ${DEPENDENCY}/META-INF /app/META-INF  
COPY --from=build ${DEPENDENCY}/BOOT-INF/classes /app

ENTRYPOINT ["java","-cp","app:app/lib/*","com.example.Application"]
```

```yaml
# docker-compose.yml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=docker
      - SPRING_DATASOURCE_URL=jdbc:postgresql://postgres:5432/myapp
      - SPRING_DATASOURCE_USERNAME=postgres  
      - SPRING_DATASOURCE_PASSWORD=password
      - SPRING_REDIS_HOST=redis
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/actuator/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      
  postgres:
    image: postgres:13-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
      
  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

```yaml
# Kubernetes deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-boot-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: spring-boot-app
  template:
    metadata:
      labels:
        app: spring-boot-app
    spec:
      containers:
      - name: app
        image: myapp:latest
        ports:
        - containerPort: 8080
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: "kubernetes"
        - name: SPRING_DATASOURCE_URL
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: url
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /actuator/health/liveness
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          initialDelaySeconds: 20
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: spring-boot-service
spec:
  selector:
    app: spring-boot-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: LoadBalancer
```

---

## Summary

### Interview Preparation Tips

1. **Understand the Fundamentals**: Auto-configuration, starters, profiles
2. **Practice Coding**: Be ready to write REST controllers, services, repositories
3. **Know the Ecosystem**: Spring Cloud, Spring Security, Spring Data
4. **Production Readiness**: Monitoring, logging, testing, deployment
5. **Recent Updates**: Stay updated with Spring Boot 3.x features

### Key Topics to Focus On

- **Core Spring Boot**: Auto-configuration, starters, properties
- **Web Layer**: REST APIs, validation, exception handling  
- **Data Layer**: JPA, repositories, transactions, migrations
- **Security**: Authentication, authorization, JWT
- **Testing**: Unit, integration, web layer tests
- **Production**: Monitoring, logging, containerization
- **Microservices**: Service discovery, circuit breakers, API gateway

### Common Follow-up Questions

- "How would you scale this application?"
- "What would you do differently in production?"
- "How would you handle this error scenario?"
- "What are the trade-offs of this approach?"
- "How would you test this functionality?"

---

*This guide covers the most frequently asked Spring Boot interview questions in 2024. Practice implementing these concepts and be prepared to explain the reasoning behind your architectural decisions.*
