# Best practices in Spring Boot application

- [1. Project Structure and Modularization](#1-project-structure-and-modularization)
- [2. Dependency Management](#2-dependency-management)
- [3. Logging](#3-logging)
- [4. Exception Handling](#4-exception-handling)
- [5. Database Interaction](#5-database-interaction)
- [6. Security](#6-security)
- [7. Testing](#7-testing)
- [8. Performance Monitoring](#8-performance-monitoring)
- [9. Documentation](#9-documentation)
- [10. Transaction Management](#10-transaction-management)
- [11. Caching](#11-caching)
- [12. Internationalization and Localization](#12-internationalization-and-localization)
- [13. Dependency Injection](#13-dependency-injection)
- [14. Versioning APIs](#14-versioning-apis)
- [15. Java 17 Code Standard](#15-java-17-code-standard)

## **1. Project Structure and Modularization**:
   - **Tip**: Organize your project structure thoughtfully, following the principles of modularity.
   - **Best Practice**: Use packages to group related classes, and consider modularizing your application for easier maintenance and scalability.
```
src
└── main
    ├── java
    │   └── com
    │       └── example
    │           └── projectname
    │               ├── annotation      # Annotation classes
    │               ├── config          # Configuration classes
    │               ├── controller      # REST controllers
    │               └── service         # Service layer interfaces
    │                   └── impl        # Service layer implementations
    │               ├── repository      # JPA repositories
    │               ├── dto             # Data Transfer Objects (DTOs)
    │               ├── entity          # JPA entities
    │               ├── exception       # Custom exception classes
    │               ├── mapper          # MapStruct mappers
    │               ├── security        # Security-related classes
    │               ├── validation      # Validation classes
    │               └── util            # Utility classes
    ├── resources
    │   ├── static                      # Static files (e.g., HTML, CSS, JS)
    │   ├── templates                   # Thymeleaf or other templates
    │   ├── application.properties      # Default application configuration
    │   └── application.yml             # YAML configuration (optional)
    └── test
        └── java
            └── com
                └── example
                    └── projectname     # Unit and integration tests
```
---
## **2. Dependency Management**:
   - **Tip**: Be mindful of your project’s dependencies.
   - **Best Practice**: Regularly review and update dependencies using tools like Spring Boot’s built-in dependency management. Avoid unnecessary dependencies to keep your application lightweight.

### **Gradle Build File Example**

Here's a sample `build.gradle` file with **Spring Boot 3** and effective dependency management:

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.2.0'
    id 'io.spring.dependency-management' version '1.1.3'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '21'

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web' // Core Web Dependency
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa' // JPA for database
    implementation 'org.springframework.boot:spring-boot-starter-security' // Spring Security
    implementation 'com.fasterxml.jackson.core:jackson-databind' // JSON handling
    runtimeOnly 'org.postgresql:postgresql' // PostgreSQL Driver
    testImplementation 'org.springframework.boot:spring-boot-starter-test' // Testing Dependencies

    // Avoid unnecessary dependencies and keep versions aligned via Spring Boot's BOM
}

tasks.withType(JavaCompile) {
    options.encoding = 'UTF-8'
}

tasks.withType(Test) {
    useJUnitPlatform() // Ensure you're using JUnit 5
}
```

### **Regular Dependency Updates**

You can use tools like `Gradle Versions Plugin` to check for outdated dependencies:

#### Add the plugin to `build.gradle`:
```groovy
plugins {
    id 'com.github.ben-manes.versions' version '0.48.0'
}
```

#### Run the task:
```bash
./gradlew dependencyUpdates
```
This will generate a report showing which dependencies can be updated.

### **Exclude Unnecessary Dependencies**

To keep the application lightweight, exclude unused transitive dependencies. For example:

```groovy
implementation('org.springframework.boot:spring-boot-starter-data-jpa') {
    exclude group: 'org.hibernate', module: 'hibernate-validator' // Exclude Hibernate Validator
}
```
---
## **3. Logging**:
   - **Tip**: Use logging effectively for troubleshooting and monitoring.
   - **Best Practice**: RUtilize SLF4J with a logging implementation like Logback. Configure log levels appropriately and use structured logging for better readability.

### **Add Dependencies to `build.gradle`**

Add the required dependencies for SLF4J and Logback:

```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter'
    implementation 'org.springframework.boot:spring-boot-starter-logging'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

### **Use SLF4J in Your Spring Boot Application**

Use `org.slf4j.Logger` for logging in your application.

#### Example: `ExampleController.java`

```java
package com.example.logging;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@RestController
public class LoggingExampleController {

    private static final Logger logger = LoggerFactory.getLogger(LoggingExampleController.class);

    @GetMapping("/log-example")
    public String logExample() {
        logger.info("Info level log message");
        logger.debug("Debug level log message");
        logger.warn("Warn level log message");
        logger.error("Error level log message");

        return "Check the logs for log messages!";
    }
}
```

### **Set Log Levels in `application.yml`**

You can override the log levels dynamically via `application.yml`:

```yaml
logging:
  level:
    root: INFO
    com.example.logging: DEBUG
  file:
    name: logs/application.log
```
---
## **4. Exception Handling**:
   - **Tip**: Plan for graceful error handling.
   - **Best Practice**: Implement a centralized exception handling strategy using @ControllerAdvice and provide meaningful error responses. Log exceptions with relevant details for debugging.

### **Create a Custom Exception Class**
```java
package com.example.exception;

public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```


### **Create a Global Exception Handler using `@ControllerAdvice`**
```java
package com.example.exception;

@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<Map<String, Object>> handleResourceNotFoundException(ResourceNotFoundException ex,
WebRequest request) {
        Map<String, Object> response = new HashMap<>();
        response.put("timestamp", LocalDateTime.now());
        response.put("message", ex.getMessage());
        response.put("status", HttpStatus.NOT_FOUND.value());
        response.put("error", HttpStatus.NOT_FOUND.getReasonPhrase());
        return new ResponseEntity<>(response, HttpStatus.NOT_FOUND);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<Map<String, Object>> handleGlobalException(Exception ex, WebRequest request) {
        Map<String, Object> response = new HashMap<>();
        response.put("timestamp", LocalDateTime.now());
        response.put("message", "An unexpected error occurred");
        response.put("status", HttpStatus.INTERNAL_SERVER_ERROR.value());
        response.put("error", HttpStatus.INTERNAL_SERVER_ERROR.getReasonPhrase());
        return new ResponseEntity<>(response, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

### **Update a Sample Service, Controller**
```java
@Service
public class SampleService {

    public String getResourceById(String id) throws ResourceNotFoundException {
        if ("123".equals(id)) {
            return "Resource found!";
        } else {
            throw new ResourceNotFoundException("Resource with ID " + id + " not found.");
        }
    }
}
```
```java
@RestController
public class SampleController {

    @GetMapping("/api/resource")
    public String getResource(@RequestParam String id) {
        return sampleService.getResourceById(id);
    }
}
```
---
## **5. Database Interaction**:
   - **Tip**: Optimize database interactions for performance.
   - **Best Practice**: Use Spring Data JPA for efficient database access. Implement proper indexing, caching, and connection pooling. Leverage query optimization techniques and be cautious with lazy loading.

### **Gradle Build Configuration (`build.gradle`)**

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-cache'
    implementation 'com.h2database:h2' // For testing; use proper DB in production
}
```

### **Application Properties (`application.yml`)**

```yaml
spring:
  datasource:
    url: jdbc:h2:mem:testdb
    driver-class-name: org.h2.Driver
    username: sa
    password: password
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
  cache:
    type: simple # Simple cache implementation
```

### **Create the Entity (`User.java`)**

```java
@Entity
@Table(name = "users", indexes = {
    @Index(name = "idx_email", columnList = "email", unique = true)
})
@Getter
@Setter
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false, unique = true)
    private String email;
}
```

### **Create the Repository (`UserRepository.java`)**

```java
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
}
```
### **Create the Service (`UserService.java`)**

```java
@Service
public class UserService {
    private final UserRepository userRepository;

   /**
     * Caches the user data to improve performance. 
     * While there are various caching strategies available, 
     * we specifically use @Cacheable here to store the result 
     * and avoid redundant database calls for the same email.
     */
    @Cacheable("users")
    public Optional<User> findUserByEmail(String email) {
        return userRepository.findByEmail(email);
    }
}
```
---
## **6. Security**:
   - **Tip**: Prioritize application security from the start.
   - **Best Practice**: Implement secure coding practices, use Spring Security for authentication and authorization, and stay updated on security patches. Regularly conduct security audits.

```
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-security'
    implementation 'io.jsonwebtoken:jjwt-api:0.11.5'
    runtimeOnly 'io.jsonwebtoken:jjwt-impl:0.11.5'
    runtimeOnly 'io.jsonwebtoken:jjwt-jackson:0.11.5'
}
```

### **`SecurityConfig.java`**
```java
package com.example.demo.config;

import com.example.securitydemo.service.JwtService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfig {

    private final UserDetailsService userDetailsService;

    public SecurityConfig(UserDetailsService userDetailsService) {
        this.userDetailsService = userDetailsService;
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http.csrf().disable()
                   .authorizeHttpRequests()
                   .requestMatchers("/auth/**").permitAll()
                   .anyRequest().authenticated()
                   .and()
                   .build();
    }

    @Bean
    public AuthenticationManager authenticationManager(HttpSecurity http) throws Exception {
        return http.getSharedObject(AuthenticationManagerBuilder.class)
                   .userDetailsService(userDetailsService)
                   .passwordEncoder(passwordEncoder())
                   .and()
                   .build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### **`JwtService.java`**
```java
package com.example.demo.service;

import com.example.securitydemo.util.JwtUtil;
import org.springframework.stereotype.Service;

@Service
public class JwtService {
    private final JwtUtil jwtUtil;

    public JwtService(JwtUtil jwtUtil) {
        this.jwtUtil = jwtUtil;
    }

    public String generateToken(String username) {
        return jwtUtil.generateToken(username);
    }
}
```

### **`JwtUtil.java`**
```java
package com.example.demo.util;

import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import org.springframework.stereotype.Component;

import java.util.Date;

@Component
public class JwtUtil {
    // Example secret key; in production, store securely in a vault or environment variable.
    private static final String SECRET_KEY = "mysecretkey";
    private static final long EXPIRATION_TIME = 86400000; // 1 day

    public String generateToken(String username) {
        return Jwts.builder()
                   .setSubject(username)
                   .setIssuedAt(new Date())
                   .setExpiration(new Date(System.currentTimeMillis() + EXPIRATION_TIME))
                   .signWith(SignatureAlgorithm.HS256, SECRET_KEY)
                   .compact();
    }
}
```
---
## **7. Testing**:
   - **Tip**: Embrace a comprehensive testing strategy.
   - **Best Practice**: Write unit tests, integration tests, and end-to-end tests. Leverage tools like JUnit and Mockito. Use Spring Boot’s testing annotations for effective testing.

### **Project Setup with Gradle**
```gradle
dependencies {
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation 'org.mockito:mockito-core'
    testImplementation 'org.mockito:mockito-junit-jupiter'
}
```

### **Unit Testing**
#### `CalculatorService.java`

```java
package com.example.demo.service;

import org.springframework.stereotype.Service;

@Service
public class CalculatorService {

    public int add(int a, int b) {
        return a + b;
    }

    public int subtract(int a, int b) {
        return a - b;
    }
}
```

#### `CalculatorServiceTest.java`

```java
package com.example.demo.service;

import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.assertEquals;

class CalculatorServiceTest {

    private final CalculatorService calculatorService = new CalculatorService();

    @Test
    void testAdd() {
        int result = calculatorService.add(3, 7);
        assertEquals(10, result);
    }

    @Test
    void testSubtract() {
        int result = calculatorService.subtract(10, 3);
        assertEquals(7, result);
    }
}
```

### **Integration Testing**

**Scenario:** Test a REST API endpoint with an in-memory database.

#### `CalculatorController.java`

```java
package com.example.demo.controller;

import com.example.demo.service.CalculatorService;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/calculator")
public class CalculatorController {

    private final CalculatorService calculatorService;

    public CalculatorController(CalculatorService calculatorService) {
        this.calculatorService = calculatorService;
    }

    @GetMapping("/add")
    public int add(@RequestParam int a, @RequestParam int b) {
        return calculatorService.add(a, b);
    }

    @GetMapping("/subtract")
    public int subtract(@RequestParam int a, @RequestParam int b) {
        return calculatorService.subtract(a, b);
    }
}
```

#### `CalculatorControllerIT.java`

```java
package com.example.demo.controller;

import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.boot.test.web.server.LocalServerPort;

import static org.junit.jupiter.api.Assertions.assertEquals;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class CalculatorControllerIT {

    @LocalServerPort
    private int port;

    private final TestRestTemplate restTemplate = new TestRestTemplate();

    @Test
    void testAddEndpoint() {
        String url = "http://localhost:" + port + "/api/calculator/add?a=5&b=3";
        int result = restTemplate.getForObject(url, Integer.class);
        assertEquals(8, result);
    }

    @Test
    void testSubtractEndpoint() {
        String url = "http://localhost:" + port + "/api/calculator/subtract?a=10&b=4";
        int result = restTemplate.getForObject(url, Integer.class);
        assertEquals(6, result);
    }
}
```
.
### **End-to-End Testing**

**Scenario:** Simulate full application flow with `MockMvc`.

#### `CalculatorControllerE2ETest.java`

```java
package com.example.demo.controller;

import com.example.demo.service.CalculatorService;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.test.web.servlet.MockMvc;

import static org.mockito.Mockito.when;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;

@WebMvcTest(CalculatorController.class)
class CalculatorControllerE2ETest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private CalculatorService calculatorService;

    @BeforeEach
    void setUp() {
        // Set up common mock behavior here
        when(calculatorService.add(3, 5)).thenReturn(8);
        when(calculatorService.subtract(10, 4)).thenReturn(6);
    }

    @Test
    void testAddEndpoint() throws Exception {
        mockMvc.perform(get("/api/calculator/add")
                .param("a", "3")
                .param("b", "5"))
                .andExpect(status().isOk())
                .andExpect(content().string("8"));
    }

    @Test
    void testSubtractEndpoint() throws Exception {
        mockMvc.perform(get("/api/calculator/subtract")
                .param("a", "10")
                .param("b", "4"))
                .andExpect(status().isOk())
                .andExpect(content().string("6"));
    }
}
```

### **Running the Tests**

Run tests with Gradle:

```bash
./gradlew test
```
---
## **8. Performance Monitoring**:
   - **Tip**: Monitor your application’s performance in real time.
   - **Best Practice**: Integrate Spring Boot Actuator for production-ready features like health checks, metrics, and monitoring endpoints. Use tools like Micrometer for custom metrics.

### **Add Dependencies**

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    implementation 'io.micrometer:micrometer-registry-prometheus'
}
```

### **Configure Actuator Endpoints**

#### `application.yml` Example:
```yaml
management:
  endpoints:
    web:
      exposure:
        include: "health,info,metrics,prometheus" # Expose specific endpoints
  health:
    livenessstate:
      enabled: true
    readinessstate:
      enabled: true
  metrics:
    export:
      prometheus:
        enabled: true # Enable Prometheus metrics
```

### **Customize Metrics with Micrometer**
#### Custom Metrics Example:
```java
import io.micrometer.core.instrument.MeterRegistry;
import org.springframework.boot.CommandLineRunner;
import org.springframework.context.annotation.Bean;
import org.springframework.stereotype.Component;

@Component
public class CustomMetricsExample {

    private final MeterRegistry meterRegistry;

    public CustomMetricsExample(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
    }

    @Bean
    public CommandLineRunner initializeCustomMetrics() {
        return args -> {
            // Example: Custom Counter
            meterRegistry.counter("custom.metrics.example.counter").increment();

            // Example: Timer
            meterRegistry.timer("custom.metrics.example.timer").record(() -> {
                // Simulate some operation
                try {
                    Thread.sleep(200);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        };
    }
}
```

### **Health Checks**
#### Custom Health Indicator Example:
```java
import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;

@Component
public class CustomHealthIndicator implements HealthIndicator {

    @Override
    public Health health() {
        // Example: Check if a service or resource is reachable
        boolean serviceUp = checkServiceHealth();
        if (serviceUp) {
            return Health.up().withDetail("CustomService", "Available").build();
        }
        return Health.down().withDetail("CustomService", "Unavailable").build();
    }

    private boolean checkServiceHealth() {
        // Simulate a health check (e.g., database, API call)
        return true; // Replace with actual check
    }
}
```

### **Testing the Setup**
Run the application and visit the Actuator endpoints:

- **Health Check**: `http://localhost:8080/actuator/health`
- **Metrics**: `http://localhost:8080/actuator/metrics`
- **Prometheus**: `http://localhost:8080/actuator/prometheus`
---
## **9. Documentation**:
   - **Tip**: Document your code and APIs.
   - **Best Practice**: Generate API documentation using tools like Swagger. Maintain clear and up-to-date documentation for both code and external APIs.

### Add Dependencies in `build.gradle`
```groovy
dependencies {
    implementation 'org.springdoc:springdoc-openapi-starter-webmvc-ui:2.2.0'
}
```

### Configure OpenAPI in the Application

```java
package com.example.documentation;

import io.swagger.v3.oas.models.ExternalDocumentation;
import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Contact;
import io.swagger.v3.oas.models.info.Info;
import io.swagger.v3.oas.models.info.License;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class OpenApiConfig {

    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
                .info(new Info()
                        .title("My Spring Boot API")
                        .description("This is the API documentation.")
                        .version("1.0.0")
                        .contact(new Contact()
                                .name("Nazran Khondokar")
                                .email("nazran@example.com")
                                .url("https://example.com"))
                        .license(new License()
                                .name("Apache 2.0")
                                .url("https://www.apache.org/licenses/LICENSE-2.0")))
                .externalDocs(new ExternalDocumentation()
                        .description("Full Documentation")
                        .url("https://example.com/docs"));
    }
}
```

### Annotate Controllers for API Documentation

```java
package com.example.documentation.controller;

import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.media.Content;
import io.swagger.v3.oas.annotations.media.Schema;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.tags.Tag;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/v1/users")
@Tag(name = "User Management", description = "APIs for managing users")
public class UserController {

    @Operation(summary = "Get User by ID", 
               description = "Retrieve a user's details by their unique ID", 
               responses = {
                   @ApiResponse(responseCode = "200", description = "User found", 
                                content = @Content(schema = @Schema(implementation = UserDTO.class))),
                   @ApiResponse(responseCode = "404", description = "User not found", content = @Content)
               })
    @GetMapping("/{id}")
    public ResponseEntity<UserDTO> getUserById(@PathVariable Long id) {
        // Sample implementation
        UserDTO user = new UserDTO(id, "John Doe", "john.doe@example.com");
        return ResponseEntity.ok(user);
    }

    @Operation(summary = "Create User", description = "Add a new user")
    @PostMapping
    public ResponseEntity<String> createUser(@RequestBody UserDTO user) {
        // Sample implementation
        return new ResponseEntity<>("User created successfully", HttpStatus.CREATED);
    }
}
```

### Create DTOs for Documentation

```java
package com.example.documentation.controller;

import io.swagger.v3.oas.annotations.media.Schema;

@Schema(description = "User data transfer object")
public class UserDTO {

    @Schema(description = "Unique identifier of the user", example = "1")
    private Long id;

    @Schema(description = "Full name of the user", example = "John Doe")
    private String name;

    @Schema(description = "Email address of the user", example = "john.doe@example.com")
    private String email;
}
```

### Access Swagger UI

Run your application and navigate to:

- Swagger UI: `http://localhost:8080/swagger-ui.html`
- OpenAPI JSON: `http://localhost:8080/v3/api-docs`
---
## **10. Transaction Management**:
   - **Tip**: Understand transaction boundaries and isolation levels.
   - **Best Practice**: Use Spring’s declarative transaction management for consistency. Be cautious with transaction boundaries and avoid long-running transactions.
        - **Use declarative transaction management** with `@Transactional` annotation.
        - **Set appropriate isolation levels** to control concurrency (e.g., `READ_COMMITTED`, `REPEATABLE_READ`).
        - **Define transaction boundaries clearly**—keep transactions short and limit scope to prevent long-running transactions.
        - **Avoid nested transactions** when possible and rely on Spring's automatic rollback on exceptions.
### **Gradle Dependency**
```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-jdbc'
}
```
### **Add @Transactional in service**
```java
@Service
public class UserService {

    @Transactional(isolation = Isolation.READ_COMMITTED, rollbackFor = Exception.class)
    public void createUser(User user) {
        // Business logic here
    }
}
```
---
## **11. Caching**:
   - **Tip**: Optimize performance with caching strategies.
   - **Best Practice**: Integrate caching mechanisms, such as Spring’s caching annotations or third-party caching solutions, to reduce redundant computations and database calls.

### **Gradle Dependency**

```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-cache'
    implementation 'com.github.ben-manes.caffeine:caffeine'
}
```

### **Enable Caching in the Application**

```java
package com.example.cache;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cache.annotation.EnableCaching;

@SpringBootApplication
@EnableCaching
public class CacheApplication {
    public static void main(String[] args) {
        SpringApplication.run(CacheApplication.class, args);
    }
}
```

### **Cache Configuration**

```java
package com.example.cache.config;

import com.github.benmanes.caffeine.cache.Caffeine;
import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.cache.caffeine.CaffeineCacheManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.concurrent.TimeUnit;

@Configuration
@EnableCaching
public class CacheConfig {

    @Bean
    public Caffeine<Object, Object> caffeineConfig() {
        return Caffeine.newBuilder()
                .expireAfterWrite(10, TimeUnit.MINUTES)
                .maximumSize(100);
    }

    @Bean
    public CacheManager cacheManager(Caffeine<Object, Object> caffeine) {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager();
        cacheManager.setCaffeine(caffeine);
        return cacheManager;
    }
}
```

### **Use Caching in a Service**

```java
package com.example.cache.service;

import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
public class ProductService {

    @Cacheable(value = "products", key = "#productId")
    public String getProductById(String productId) {
        simulateSlowService();
        return "Product details for ID: " + productId;
    }

    private void simulateSlowService() {
        try {
            Thread.sleep(3000); // Simulate a delay
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```
---
## **12. Internationalization and Localization**:
   - **Tip**: Plan for multi-language support from the beginning.
   - **Best Practice**: Utilize Spring’s internationalization features to support different languages and locales in your application.
### **Configure `MessageSource` for Resource Bundles**  

```java
package com.example.config;

import org.springframework.context.MessageSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.support.ReloadableResourceBundleMessageSource;
import org.springframework.web.servlet.LocaleResolver;
import org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver;

import java.util.Locale;

@Configuration
public class InternationalizationConfig {

    @Bean
    public MessageSource messageSource() {
        ReloadableResourceBundleMessageSource messageSource = new ReloadableResourceBundleMessageSource();
        messageSource.setBasename("classpath:messages");
        messageSource.setDefaultEncoding("UTF-8");
        return messageSource;
    }

    @Bean
    public LocaleResolver localeResolver() {
        AcceptHeaderLocaleResolver localeResolver = new AcceptHeaderLocaleResolver();
        localeResolver.setDefaultLocale(Locale.ENGLISH);
        return localeResolver;
    }
}
```

### **Create Message Resource Files**  
Add message files in `src/main/resources`.  
- **`messages.properties`** (default locale, e.g., English):  
  ```properties
  greeting=Hello, {0}!
  ```
  
- **`messages_fr.properties`** (French locale):  
  ```properties
  greeting=Bonjour, {0}!
  ```

- **`messages_es.properties`** (Spanish locale):  
  ```properties
  greeting=¡Hola, {0}!
  ```

### **Implement a Controller for Testing**  

```java
package com.example.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.MessageSource;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestHeader;
import org.springframework.web.bind.annotation.RestController;

import java.util.Locale;

@RestController
public class GreetingController {

    @Autowired
    private MessageSource messageSource;

    @GetMapping("/greet")
    public String greet(
            @RequestHeader(name = "Accept-Language", required = false) Locale locale) {
        return messageSource.getMessage("greeting", new Object[]{"Nazran"}, locale);
    }
}
```

### **Testing the Application**  

#### Example Requests:  
- **Default (English):**
  ```bash
  curl -H "Accept-Language: en" http://localhost:8080/greet
  ```
  **Response:**  
  ```
  Hello, Nazran!
  ```

- **French:**  
  ```bash
  curl -H "Accept-Language: fr" http://localhost:8080/greet
  ```
  **Response:**  
  ```
  Bonjour, Nazran!
  ```

- **Spanish:**  
  ```bash
  curl -H "Accept-Language: es" http://localhost:8080/greet
  ```
  **Response:**  
  ```
  ¡Hola, Nazran!
  ```
---
## **13. Dependency Injection**:
   - **Tip**: Leverage dependency injection for loose coupling.
   - **Best Practice**: Use Spring’s dependency injection to achieve inversion of control (IoC). Follow the SOLID principles to design modular and maintainable code.

**Define an Interface**  
This promotes loose coupling by allowing different implementations.

```java
package com.example.service;

public interface GreetingService {
    String getGreeting(String name);
}
```

**Provide an Implementation**

```java
package com.example.service;

import org.springframework.stereotype.Service;

@Service
public class GreetingServiceImpl implements GreetingService {

    @Override
    public String getGreeting(String name) {
        return "Hello, " + name + "!";
    }
}
```

- The `@Service` annotation marks this class as a Spring-managed bean.

**Inject the Service into a Controller**

```java

@RestController
public class GreetingController {

    private final GreetingService greetingService;

    // Constructor-based Dependency Injection
    public GreetingController(GreetingService greetingService) {
        this.greetingService = greetingService;
    }

    @GetMapping("/greet")
    public String greet(@RequestParam String name) {
        return greetingService.getGreeting(name);
    }
}
```

- **Best Practice**: Use constructor injection for required dependencies, which ensures immutability and testability.
---
## **14. Versioning APIs**:
   - **Tip**: Plan for future changes in your API design.
   - **Best Practice**: Implement versioning strategies for APIs to ensure backward compatibility and smooth transitions when introducing changes.
### Key Best Practices for API Versioning:
1. **Use URI versioning (Recommended)**: Include the version in the URL path (`/v1/resource`).
2. **Header versioning**: Use a custom header (`X-API-Version: 1`).
3. **Media Type versioning**: Define a version as part of the `Content-Type` (`application/vnd.api.v1+json`).

Here, we'll focus on **URI versioning**, which is simple and widely adopted.

### API Versioning with URI

#### Project Structure
```
src/main/java
└── com/example/versioning
    ├── controller
    │   ├── UserControllerV1.java
    │   └── UserControllerV2.java
    ├── dto
    │   └── User.java
    ├── service
    │   ├── UserServiceV1.java
    │   └── UserServiceV2.java
    └── VersioningApplication.java
```
#### Implementation

##### DTOs

###### `User.java`
```java
public class User {
    private String firstName;
    private String lastName;

    // Constructor for version 1 (name only)
    public User(String name) {
        this.firstName = name;
        this.lastName = "";
    }

    // Constructor for version 2 (first and last name)
    public User(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }

    // Getters and setters
}
```
##### Controllers

###### `UserControllerV1.java`
```java
import com.example.versioning.dto.User;

@RestController
@RequestMapping("/api/v1/users")
public class UserControllerV1 {

    @GetMapping
    public UserV1 getUser() {
        return new User("John Doe");
    }
}
```

###### `UserControllerV2.java`
```java
import com.example.versioning.dto.User;

@RestController
@RequestMapping("/api/v2/users")
public class UserControllerV2 {

    @GetMapping
    public UserV2 getUser() {
        return new User("John", "Doe");
    }
}
```

#### Testing the API

**Version 1 Endpoint:**
```bash
curl http://localhost:8080/api/v1/users
# Response: {"firstName":"John Doe","lastName":""}
```

**Version 2 Endpoint:**
```bash
curl http://localhost:8080/api/v2/users
# Response: {"firstName":"John","lastName":"Doe"}
```
---
## **15. Java 17 Code Standard**:

### **Database Field Types & Mapping (JPA Best Practices)**
- Use proper data types matching business requirements.
- Avoid `VARCHAR(MAX)`, prefer `VARCHAR(255)` unless large storage is needed.
- Use `@Enumerated(EnumType.STRING)` instead of `@Enumerated(EnumType.ORDINAL)` for Enums.
- Use `@Column(nullable = false)` for mandatory fields.

```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(length = 100, nullable = false)
    private String name;

    @Enumerated(EnumType.STRING)
    private Role role;
}
```

### **Avoid Hardcoded Values (Use Constants & Config)**
- Use constants or properties instead of hardcoded values.

```java
public class AppConfig {
    public static final String BASE_URL = "https://api.example.com";
}
```

```java
@Value("${app.base-url}")
private String baseUrl;
```

### **Avoid Coding Smells**
- **Long Methods →** Break into smaller methods.
- **Feature Envy →** Move logic closer to the related class.
- **Magic Numbers →** Use named constants.

```java
public static final int MAX_RETRIES = 3;

public void processRequest() {
    for (int i = 0; i < MAX_RETRIES; i++) {
        // retry logic
    }
}
```

### **Unused Imports (Optimize Imports)**
- Remove unnecessary imports.

Before:
```java
import java.util.List;
import java.util.ArrayList;
import java.util.Random; // Unused
```
After:
```java
import java.util.List;
import java.util.ArrayList;
```

**Use IntelliJ IDEA/VS Code shortcut:**  
`Ctrl + Alt + O` (Windows/Linux) or `Cmd + Option + O` (Mac)  


### **Use Records for DTOs (Java 17)**
- Instead of traditional DTOs, use **Java Records** for immutable objects.

```java
public record UserDTO(Long id, String name, Role role) {}
```

### **Use Optional Instead of Null Checks**
- Avoid `null` checks with `Optional`.

```java
public Optional<User> findUserById(Long id) {
    return userRepository.findById(id);
}
```
### **Use Clear and Intuitive Naming Conventions**
- **Classes**: Use nouns, e.g., `UserService`.
- **Packages**: Use lowercase, e.g., `com.example.service`.
- **Interfaces**: Use CamelCase, e.g., `UserRepository`.
- **Variables**: Use mixedCase, e.g., `userCount`.
- **Methods**: Use verbs, e.g., `calculateSum()`.


### **Comment and Write Self-Documenting Code**
- **Comment Example**:  
  ```java
  // Check if user is active
  if (user.isActive()) { ... }
  ```

- **Self-Documenting Code**:  
  ```java
  if (user.isEligibleForDiscount()) { ... }
  ```


### **Write Descriptive Commit Messages**
- Example:  
  ```bash
  git commit -m "Fix NullPointerException in user registration"
  ```
- Keep messages brief and focus on **what changed**.


### **Avoid Empty Catch Blocks**
- **Bad**:  
  ```java
  try {
      int result = Integer.parseInt("abc");
  } catch (NumberFormatException e) {
      // Silent failure
  }
  ```
- **Good**:  
  ```java
  int result;
  try {
      int result = Integer.parseInt("abc");
  } catch (NumberFormatException e) {
      logger.error("Invalid number format: " + e.getMessage());
  }
  ```

### **Handle NullPointerException Properly**
- **Bad**:  
  ```java
  int count = company.getEmployees().size();
  ```
- **Good**:  
  ```java
  int count;
  if (company != null && company.getEmployees() != null) {
      count = company.getEmployees().size();
  }
  ```


### **Use Java Libraries Wisely**
- Avoid overusing libraries; choose reliable ones.
- Example: Prefer `java.util.stream` over adding new dependencies:
  ```java
  var sum = numbers.stream().mapToInt(Integer::intValue).sum();
  ```

### **Access Class Members Privately**
- **Bad**:  
  ```java
  public String name;
  ```
- **Good**:  
  ```java
  private String name;

  public void setName(String name) {
      if (name == null || name.isBlank()) {
          throw new IllegalArgumentException("Invalid name");
      }
      this.name = name;
  }
  ```

### **Avoid Redundant Initializations**
- **Bad**:  
  ```java
  private boolean active = false; // Default is already false
  ```
- **Good**:  
  ```java
  private boolean active;
  ```

### **Prevent Memory Leaks**
- Always release resources:
  ```java
  try (var connection = dataSource.getConnection()) {
      // Use connection
  } catch (SQLException e) {
      e.printStackTrace();
  }
  ```
- Use tools like IntelliJ Memory Tab or Eclipse MAT to detect leaks.

---
