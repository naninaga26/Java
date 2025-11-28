# Spring Boot - Complete Fundamentals Guide

## Table of Contents
1. [Introduction to Spring Boot](#introduction-to-spring-boot)
2. [Dependency Injection and IoC](#dependency-injection-and-ioc)
3. [Spring Boot Auto-configuration](#spring-boot-auto-configuration)
4. [Application Properties](#application-properties)
5. [Spring Boot Starters](#spring-boot-starters)
6. [Bean Lifecycle](#bean-lifecycle)
7. [Spring Boot Actuator](#spring-boot-actuator)
8. [Common Pitfalls](#common-pitfalls)
9. [Interview Questions](#interview-questions)

---

## Introduction to Spring Boot

### What is Spring Boot?

Spring Boot is an opinionated framework built on top of Spring Framework that simplifies the development of production-ready applications.

**Key Benefits:**
- ✅ Auto-configuration
- ✅ Embedded servers (Tomcat, Jetty)
- ✅ Production-ready features (metrics, health checks)
- ✅ No XML configuration needed
- ✅ Starter dependencies for easy setup

### Spring Framework vs Spring Boot

| Feature | Spring Framework | Spring Boot |
|---------|------------------|-------------|
| Configuration | XML or Java Config (verbose) | Auto-configuration |
| Setup Time | Hours/Days | Minutes |
| Embedded Server | No (needs external server) | Yes (Tomcat embedded) |
| Production Features | Manual setup | Built-in (Actuator) |
| Dependency Management | Manual | Starters |
| Use Case | Large enterprise apps | Microservices, REST APIs |

### Creating Your First Spring Boot Application

#### Method 1: Spring Initializr (Recommended)

Visit [start.spring.io](https://start.spring.io) or use your IDE.

**Configuration:**
```
Project: Maven
Language: Java
Spring Boot: 3.2.x (latest)
Packaging: Jar
Java: 17

Dependencies:
- Spring Web
- Spring Data JPA
- PostgreSQL Driver
- Lombok
```

#### Method 2: Maven pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
        <relativePath/>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>demo-app</artifactId>
    <version>1.0.0</version>
    <name>Demo Application</name>

    <properties>
        <java.version>17</java.version>
    </properties>

    <dependencies>
        <!-- Spring Boot Web Starter -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Lombok for boilerplate reduction -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- Testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

#### Main Application Class

```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * @SpringBootApplication is a convenience annotation that combines:
 * - @Configuration: Marks class as source of bean definitions
 * - @EnableAutoConfiguration: Enables auto-configuration
 * - @ComponentScan: Scans for components in this package and sub-packages
 */
@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

#### Simple REST Controller

```java
package com.example.demo.controller;

import org.springframework.web.bind.annotation.*;

@RestController  // @Controller + @ResponseBody
@RequestMapping("/api")
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello, Spring Boot!";
    }

    @GetMapping("/hello/{name}")
    public String helloName(@PathVariable String name) {
        return "Hello, " + name + "!";
    }

    @PostMapping("/greet")
    public String greet(@RequestBody GreetingRequest request) {
        return "Hello, " + request.getName() + "! Welcome to Spring Boot.";
    }
}

// Request DTO
class GreetingRequest {
    private String name;

    // Getters and setters
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

**Run the application:**
```bash
mvn spring-boot:run

# Or
./mvnw spring-boot:run

# Application starts on http://localhost:8080
```

---

## Dependency Injection and IoC

### What is Dependency Injection?

**Dependency Injection (DI)** is a design pattern where objects receive their dependencies from external sources rather than creating them.

**Inversion of Control (IoC)** means the framework controls the object creation and lifecycle, not your code.

### Without Dependency Injection (Tight Coupling)

```java
// ❌ BAD: Tight coupling
public class UserService {
    private UserRepository userRepository = new UserRepository();  // Hard-coded dependency

    public User getUser(Long id) {
        return userRepository.findById(id);
    }
}

// Problems:
// 1. Cannot change implementation easily
// 2. Hard to test (cannot mock repository)
// 3. Creates new instance every time
```

### With Dependency Injection (Loose Coupling)

```java
// ✅ GOOD: Dependency Injection
@Service
public class UserService {
    private final UserRepository userRepository;

    // Constructor injection (recommended)
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public User getUser(Long id) {
        return userRepository.findById(id).orElseThrow();
    }
}

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    // Spring Data JPA provides implementation
}
```

### Types of Dependency Injection

#### 1. Constructor Injection (Recommended)

```java
@Service
public class OrderService {
    private final OrderRepository orderRepository;
    private final PaymentService paymentService;
    private final EmailService emailService;

    // Single constructor - @Autowired is optional
    public OrderService(
            OrderRepository orderRepository,
            PaymentService paymentService,
            EmailService emailService) {
        this.orderRepository = orderRepository;
        this.paymentService = paymentService;
        this.emailService = emailService;
    }

    public Order createOrder(OrderRequest request) {
        // Use dependencies
    }
}
```

**Why Constructor Injection?**
- ✅ Immutable dependencies (final fields)
- ✅ Required dependencies are clear
- ✅ Easy to test (can pass mocks)
- ✅ No need for @Autowired annotation
- ✅ Fails fast if dependency missing

#### 2. Setter Injection (Optional Dependencies)

```java
@Service
public class NotificationService {
    private EmailService emailService;
    private SmsService smsService;

    @Autowired  // Required for setter injection
    public void setEmailService(EmailService emailService) {
        this.emailService = emailService;
    }

    @Autowired
    public void setSmsService(SmsService smsService) {
        this.smsService = smsService;
    }
}

// Use only for optional dependencies!
```

#### 3. Field Injection (Not Recommended)

```java
@Service
public class ProductService {
    @Autowired  // ❌ Avoid: Hard to test, hidden dependencies
    private ProductRepository productRepository;

    public Product getProduct(Long id) {
        return productRepository.findById(id).orElseThrow();
    }
}

// Problems:
// - Cannot make fields final
// - Hard to write unit tests
// - Hidden dependencies
// - Circular dependency issues
```

### Component Scanning

Spring automatically discovers and registers beans with these annotations:

```java
// @Component - Generic stereotype
@Component
public class MyComponent {
    // Generic bean
}

// @Service - Business logic layer
@Service
public class UserService {
    // Business logic
}

// @Repository - Data access layer
@Repository
public class UserRepository {
    // Database operations
}

// @Controller - Web layer (returns views)
@Controller
public class WebController {
    // Handles web requests, returns HTML
}

// @RestController - REST API layer (returns JSON/XML)
@RestController
public class ApiController {
    // Handles REST requests, returns data
}

// @Configuration - Configuration class
@Configuration
public class AppConfig {
    @Bean
    public MyBean myBean() {
        return new MyBean();
    }
}
```

**Component Scan Configuration:**

```java
@SpringBootApplication
// Scans current package and sub-packages by default
public class DemoApplication { }

// Or specify packages explicitly
@SpringBootApplication
@ComponentScan(basePackages = {"com.example.service", "com.example.repository"})
public class DemoApplication { }
```

### Bean Scopes

```java
@Component
@Scope("singleton")  // Default scope
public class SingletonBean {
    // One instance per Spring container
}

@Component
@Scope("prototype")
public class PrototypeBean {
    // New instance every time bean is requested
}

@Component
@Scope("request")
public class RequestBean {
    // One instance per HTTP request (web apps only)
}

@Component
@Scope("session")
public class SessionBean {
    // One instance per HTTP session (web apps only)
}
```

**Common Pitfall #1: Prototype Bean in Singleton**

```java
// ❌ PROBLEM: Prototype bean injected into singleton
@Service  // Singleton scope
public class SingletonService {
    @Autowired
    private PrototypeBean prototypeBean;  // Only injected once!

    public void doSomething() {
        prototypeBean.process();  // Same instance used every time!
    }
}

// ✅ SOLUTION 1: Use ObjectProvider
@Service
public class SingletonService {
    @Autowired
    private ObjectProvider<PrototypeBean> prototypeBeanProvider;

    public void doSomething() {
        PrototypeBean bean = prototypeBeanProvider.getObject();  // New instance
        bean.process();
    }
}

// ✅ SOLUTION 2: Use ApplicationContext
@Service
public class SingletonService {
    @Autowired
    private ApplicationContext context;

    public void doSomething() {
        PrototypeBean bean = context.getBean(PrototypeBean.class);  // New instance
        bean.process();
    }
}
```

---

## Spring Boot Auto-configuration

### How Auto-configuration Works

Spring Boot examines:
1. Classpath dependencies
2. Existing beans
3. Property values

Then automatically configures beans.

**Example:**

```
If JPA dependency is present → Auto-configure DataSource, EntityManager
If Web dependency is present → Auto-configure DispatcherServlet, ViewResolver
If Security dependency is present → Auto-configure default security
```

### Viewing Auto-configuration Report

```properties
# application.properties
debug=true
```

```bash
# Run application to see auto-configuration report
# Shows:
# - Positive matches (what was auto-configured)
# - Negative matches (what was not)
# - Exclusions
# - Unconditional classes
```

### Conditional Annotations

```java
@Configuration
public class MyAutoConfiguration {

    // Bean created only if class is on classpath
    @Bean
    @ConditionalOnClass(DataSource.class)
    public DataSource dataSource() {
        return new HikariDataSource();
    }

    // Bean created only if bean doesn't exist
    @Bean
    @ConditionalOnMissingBean
    public MyService myService() {
        return new MyServiceImpl();
    }

    // Bean created only if property is set
    @Bean
    @ConditionalOnProperty(name = "feature.enabled", havingValue = "true")
    public FeatureService featureService() {
        return new FeatureService();
    }

    // Bean created only if another bean exists
    @Bean
    @ConditionalOnBean(DataSource.class)
    public JdbcTemplate jdbcTemplate(DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}
```

### Disabling Auto-configuration

```java
// Exclude specific auto-configurations
@SpringBootApplication(exclude = {
    DataSourceAutoConfiguration.class,
    SecurityAutoConfiguration.class
})
public class DemoApplication { }
```

```properties
# Or in application.properties
spring.autoconfigure.exclude=\
  org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
  org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration
```

---

## Application Properties

### application.properties vs application.yml

**application.properties:**
```properties
# Server configuration
server.port=8081
server.servlet.context-path=/api

# Database configuration
spring.datasource.url=jdbc:postgresql://localhost:5432/mydb
spring.datasource.username=postgres
spring.datasource.password=secret

# JPA configuration
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

# Logging
logging.level.root=INFO
logging.level.com.example=DEBUG
logging.file.name=application.log
```

**application.yml (YAML):**
```yaml
# Server configuration
server:
  port: 8081
  servlet:
    context-path: /api

# Database configuration
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/mydb
    username: postgres
    password: secret

  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true

# Logging
logging:
  level:
    root: INFO
    com.example: DEBUG
  file:
    name: application.log
```

### Environment-Specific Profiles

**application-dev.properties:**
```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/dev_db
logging.level.root=DEBUG
```

**application-prod.properties:**
```properties
spring.datasource.url=jdbc:postgresql://prod-server:5432/prod_db
logging.level.root=WARN
```

**Activating Profiles:**

```bash
# Command line
java -jar app.jar --spring.profiles.active=prod

# Or in application.properties
spring.profiles.active=dev

# Or as environment variable
export SPRING_PROFILES_ACTIVE=prod
```

**Profile-specific beans:**

```java
@Configuration
@Profile("dev")
public class DevConfiguration {
    @Bean
    public DataSource devDataSource() {
        // H2 in-memory database for development
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.H2)
            .build();
    }
}

@Configuration
@Profile("prod")
public class ProdConfiguration {
    @Bean
    public DataSource prodDataSource() {
        // Production database with connection pooling
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl("jdbc:postgresql://prod-server:5432/db");
        return dataSource;
    }
}
```

### Custom Properties

```properties
# application.properties
app.name=My Application
app.version=1.0.0
app.features.email-enabled=true
app.features.sms-enabled=false
app.max-upload-size=10485760
```

**@Value Annotation:**

```java
@Service
public class AppService {
    @Value("${app.name}")
    private String appName;

    @Value("${app.version}")
    private String appVersion;

    @Value("${app.max-upload-size:1048576}")  // Default value if not set
    private long maxUploadSize;

    public void printInfo() {
        System.out.println("App: " + appName + " v" + appVersion);
    }
}
```

**@ConfigurationProperties (Recommended for grouped properties):**

```java
@ConfigurationProperties(prefix = "app")
@Component
@Data  // Lombok for getters/setters
public class AppProperties {
    private String name;
    private String version;
    private Features features = new Features();
    private long maxUploadSize;

    @Data
    public static class Features {
        private boolean emailEnabled;
        private boolean smsEnabled;
    }
}

// Usage
@Service
public class AppService {
    private final AppProperties appProperties;

    public AppService(AppProperties appProperties) {
        this.appProperties = appProperties;
    }

    public void printInfo() {
        System.out.println("App: " + appProperties.getName());
        System.out.println("Email enabled: " + appProperties.getFeatures().isEmailEnabled());
    }
}
```

---

## Spring Boot Starters

Starters are curated dependency descriptors for specific features.

### Common Starters

```xml
<!-- Web Applications -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- RESTful Web Services -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- JPA and Hibernate -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<!-- Security -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<!-- Validation -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>

<!-- Testing -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>

<!-- Actuator (Monitoring) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>

<!-- Caching -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>

<!-- Mail -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>

<!-- WebSocket -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

---

## Bean Lifecycle

### Bean Creation Process

```
1. Constructor called
2. Dependencies injected
3. @PostConstruct method called (if present)
4. Bean ready for use
5. @PreDestroy method called (if present) when context closes
```

### Lifecycle Callbacks

```java
@Component
public class MyBean {
    private DataSource dataSource;

    // 1. Constructor
    public MyBean(DataSource dataSource) {
        System.out.println("1. Constructor called");
        this.dataSource = dataSource;
    }

    // 2. Post-construction initialization
    @PostConstruct
    public void init() {
        System.out.println("2. @PostConstruct: Bean initialized");
        // Perform initialization logic
        // e.g., validate configuration, open connections
    }

    // 3. Normal operation
    public void doWork() {
        System.out.println("3. Doing work...");
    }

    // 4. Pre-destruction cleanup
    @PreDestroy
    public void cleanup() {
        System.out.println("4. @PreDestroy: Cleaning up");
        // Perform cleanup logic
        // e.g., close connections, release resources
    }
}
```

### InitializingBean and DisposableBean Interfaces

```java
@Component
public class MyBean implements InitializingBean, DisposableBean {

    @Override
    public void afterPropertiesSet() throws Exception {
        // Called after properties are set
        System.out.println("InitializingBean: afterPropertiesSet");
    }

    @Override
    public void destroy() throws Exception {
        // Called before bean is destroyed
        System.out.println("DisposableBean: destroy");
    }
}
```

### @Bean with initMethod and destroyMethod

```java
@Configuration
public class AppConfig {

    @Bean(initMethod = "initialize", destroyMethod = "cleanup")
    public MyService myService() {
        return new MyService();
    }
}

public class MyService {
    public void initialize() {
        System.out.println("Custom init method");
    }

    public void cleanup() {
        System.out.println("Custom destroy method");
    }
}
```

---

## Spring Boot Actuator

Actuator provides production-ready features like monitoring, metrics, and health checks.

### Add Actuator Dependency

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### Configuration

```properties
# Expose all endpoints
management.endpoints.web.exposure.include=*

# Or specific endpoints
management.endpoints.web.exposure.include=health,info,metrics,env

# Base path for actuator endpoints
management.endpoints.web.base-path=/actuator

# Enable specific endpoints
management.endpoint.health.enabled=true
management.endpoint.info.enabled=true
```

### Common Actuator Endpoints

```bash
# Health check
GET http://localhost:8080/actuator/health

Response:
{
  "status": "UP",
  "components": {
    "db": {
      "status": "UP",
      "details": {
        "database": "PostgreSQL",
        "validationQuery": "isValid()"
      }
    },
    "diskSpace": {
      "status": "UP",
      "details": {
        "total": 250685575168,
        "free": 100000000000,
        "threshold": 10485760
      }
    }
  }
}

# Application info
GET http://localhost:8080/actuator/info

# Metrics
GET http://localhost:8080/actuator/metrics
GET http://localhost:8080/actuator/metrics/jvm.memory.used

# Environment properties
GET http://localhost:8080/actuator/env

# All beans
GET http://localhost:8080/actuator/beans

# Mappings (all endpoints)
GET http://localhost:8080/actuator/mappings
```

### Custom Health Indicator

```java
@Component
public class CustomHealthIndicator implements HealthIndicator {

    @Override
    public Health health() {
        // Check some custom condition
        boolean isHealthy = checkExternalService();

        if (isHealthy) {
            return Health.up()
                .withDetail("service", "External API")
                .withDetail("status", "Available")
                .build();
        } else {
            return Health.down()
                .withDetail("service", "External API")
                .withDetail("status", "Unavailable")
                .withDetail("error", "Connection timeout")
                .build();
        }
    }

    private boolean checkExternalService() {
        // Check external service
        return true;
    }
}
```

### Custom Info Contributor

```java
@Component
public class CustomInfoContributor implements InfoContributor {

    @Override
    public void contribute(Info.Builder builder) {
        builder.withDetail("app", Map.of(
            "name", "My Application",
            "version", "1.0.0",
            "description", "Demo application"
        ));

        builder.withDetail("team", Map.of(
            "name", "Backend Team",
            "size", 5
        ));
    }
}
```

---

## Common Pitfalls

### 1. Circular Dependency

```java
// ❌ PROBLEM
@Service
public class ServiceA {
    @Autowired
    private ServiceB serviceB;
}

@Service
public class ServiceB {
    @Autowired
    private ServiceA serviceA;  // Circular dependency!
}

// ✅ SOLUTION 1: Redesign (best)
// Break the circular dependency by introducing a third service

// ✅ SOLUTION 2: Use @Lazy
@Service
public class ServiceA {
    private final ServiceB serviceB;

    public ServiceA(@Lazy ServiceB serviceB) {
        this.serviceB = serviceB;
    }
}
```

### 2. Not Using Constructor Injection

```java
// ❌ AVOID
@Service
public class UserService {
    @Autowired
    private UserRepository repository;  // Field injection
}

// ✅ PREFER
@Service
public class UserService {
    private final UserRepository repository;

    public UserService(UserRepository repository) {
        this.repository = repository;
    }
}
```

### 3. Missing @Component Annotations

```java
// ❌ PROBLEM: Bean not discovered
public class MyService {  // No @Component/@Service annotation
    // Won't be managed by Spring
}

// ✅ SOLUTION
@Service
public class MyService {
    // Now Spring manages this bean
}
```

### 4. Autowiring Concrete Classes

```java
// ❌ BAD: Tight coupling
@Service
public class OrderService {
    @Autowired
    private PaymentServiceImpl paymentService;  // Concrete class
}

// ✅ GOOD: Program to interface
@Service
public class OrderService {
    private final PaymentService paymentService;  // Interface

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

### 5. Incorrect Property Names

```properties
# ❌ WRONG
server.port = 8080  # Spaces around =

# ✅ CORRECT
server.port=8080
```

---

## Interview Questions

### Q1: What is Spring Boot and how is it different from Spring?
**Answer:** Spring Boot is built on top of Spring Framework and provides:
- Auto-configuration (reduces boilerplate)
- Embedded servers (no need for external Tomcat)
- Starter dependencies (simplified dependency management)
- Production-ready features (Actuator)
- Opinionated defaults (convention over configuration)

### Q2: Explain Dependency Injection.
**Answer:** DI is a design pattern where objects receive dependencies from external sources rather than creating them. Spring IoC container manages object creation and injection. Benefits: loose coupling, easier testing, better maintainability.

### Q3: What are the types of Dependency Injection in Spring?
**Answer:**
1. Constructor Injection (recommended) - immutable, clear dependencies
2. Setter Injection - for optional dependencies
3. Field Injection (not recommended) - hard to test, hidden dependencies

### Q4: What is @SpringBootApplication annotation?
**Answer:** It's a convenience annotation combining:
- @Configuration - marks class as configuration
- @EnableAutoConfiguration - enables auto-configuration
- @ComponentScan - scans for components in package

### Q5: What are Spring Boot Starters?
**Answer:** Curated dependency descriptors for specific features. Example: spring-boot-starter-web includes Spring MVC, Tomcat, and Jackson. Benefits: simplified dependency management, compatible versions.

### Q6: Explain Spring Bean Scopes.
**Answer:**
- Singleton (default) - one instance per container
- Prototype - new instance per request
- Request - one per HTTP request (web apps)
- Session - one per HTTP session (web apps)
- Application - one per ServletContext
- WebSocket - one per WebSocket

### Q7: What is Spring Boot Actuator?
**Answer:** Production-ready features for monitoring and management:
- Health checks (/health)
- Metrics (/metrics)
- Environment info (/env)
- Bean information (/beans)
- HTTP trace (/httptrace)

### Q8: How does Auto-configuration work?
**Answer:** Spring Boot examines classpath, existing beans, and properties, then automatically configures beans. Uses @Conditional annotations to decide what to configure. Can be customized or disabled.

### Q9: What is @Value vs @ConfigurationProperties?
**Answer:**
- @Value: Individual property injection, supports SpEL
- @ConfigurationProperties: Type-safe, groups related properties, validation support, recommended for multiple properties

### Q10: How to handle circular dependencies?
**Answer:**
1. Redesign to eliminate circular dependency (best)
2. Use @Lazy on one dependency
3. Use setter injection instead of constructor
4. Use @PostConstruct for initialization

---

## Best Practices

1. **Use Constructor Injection** - Immutable, testable
2. **Program to Interfaces** - Loose coupling
3. **Use Profiles** - Environment-specific configuration
4. **Externalize Configuration** - application.properties
5. **Use Actuator** - Production monitoring
6. **Keep @SpringBootApplication at Root** - Proper component scanning
7. **Use Lombok** - Reduce boilerplate
8. **Enable Validation** - Use @Valid and @Validated
9. **Use Starters** - Simplified dependencies
10. **Follow Package Structure** - controller, service, repository, model

---

## Next Steps

1. **Build REST APIs** - [REST API Development Guide](../02-REST-API-Development/REST-API-Complete-Guide.md)
2. **Add Database** - [Spring Data JPA Guide](../03-Spring-Data-JPA/JPA-Complete-Guide.md)
3. **Secure Your API** - [Spring Security Guide](../04-Spring-Security/Spring-Security-Complete-Guide.md)

Would you like me to create the REST API Development guide next?
