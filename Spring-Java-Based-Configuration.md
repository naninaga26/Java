# Spring Framework Java-Based Configuration - In-Depth Guide

## Table of Contents
1. [Introduction to Java-Based Configuration](#introduction-to-java-based-configuration)
2. [Configuration Class Basics](#configuration-class-basics)
3. [Bean Definition with @Bean](#bean-definition-with-bean)
4. [Dependency Injection](#dependency-injection)
5. [Constructor Injection](#constructor-injection)
6. [Setter/Method Injection](#settermethod-injection)
7. [Bean Scopes](#bean-scopes)
8. [Auto-wiring](#auto-wiring)
9. [Component Scanning](#component-scanning)
10. [Bean Lifecycle](#bean-lifecycle)
11. [Advanced Configuration](#advanced-configuration)
12. [Complete Examples](#complete-examples)

---

## Introduction to Java-Based Configuration

Java-based configuration allows you to define Spring beans using Java classes instead of XML. This approach provides:

- **Type Safety**: Compile-time checking
- **Refactoring Support**: IDE support for renaming, finding usages
- **Better IDE Support**: Auto-completion, navigation
- **No XML**: Pure Java approach
- **Conditional Configuration**: Programmatic bean creation logic

### Evolution of Spring Configuration

```
XML Configuration (2004)
    ↓
Annotation-based Configuration (2007) - @Component, @Autowired
    ↓
Java-based Configuration (2009) - @Configuration, @Bean
    ↓
Spring Boot Auto-configuration (2014)
```

---

## Configuration Class Basics

### Simple Configuration Class

```java
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {
    // Bean definitions go here
}
```

**Key Points:**
- `@Configuration` indicates this class contains bean definitions
- Configuration classes are processed by Spring container
- Methods annotated with `@Bean` produce beans managed by Spring

### Loading Configuration

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class Application {
    public static void main(String[] args) {
        // Load Java configuration
        ApplicationContext context =
            new AnnotationConfigApplicationContext(AppConfig.class);

        // Get bean from container
        UserService userService = context.getBean(UserService.class);

        // Use the bean
        userService.createUser(new User("John"));

        // Close context
        ((AnnotationConfigApplicationContext) context).close();
    }
}
```

---

## Bean Definition with @Bean

### Basic Bean Definition

```java
@Configuration
public class AppConfig {

    @Bean
    public UserService userService() {
        return new UserService();
    }

    @Bean
    public UserRepository userRepository() {
        return new UserRepository();
    }
}
```

**How it works:**
- Method name becomes bean name (by default)
- Return type is the bean type
- Spring calls the method and manages the returned object

### Custom Bean Name

```java
@Configuration
public class AppConfig {

    // Bean name is "userSvc" instead of "userService"
    @Bean(name = "userSvc")
    public UserService userService() {
        return new UserService();
    }

    // Multiple names/aliases
    @Bean(name = {"userManager", "userSvc", "userService"})
    public UserService userServiceWithAliases() {
        return new UserService();
    }

    // Using value attribute (same as name)
    @Bean(value = "customName")
    public UserService anotherUserService() {
        return new UserService();
    }
}
```

### Bean with Description

```java
@Configuration
public class AppConfig {

    @Bean
    @Description("This bean manages user-related operations")
    public UserService userService() {
        return new UserService();
    }
}
```

---

## Dependency Injection

### How DI Works in Java Config

1. **Container Initialization**: Spring scans configuration classes
2. **Bean Registration**: Methods annotated with @Bean are registered
3. **Dependency Resolution**: Spring identifies bean dependencies
4. **Bean Creation**: Spring creates beans in correct order
5. **Dependency Injection**: Dependencies are injected
6. **Bean Ready**: Fully configured beans are available

---

## Constructor Injection

Constructor injection is the recommended approach for mandatory dependencies.

### Example Classes

```java
public class UserRepository {
    private final String databaseUrl;
    private final String username;

    public UserRepository(String databaseUrl, String username) {
        this.databaseUrl = databaseUrl;
        this.username = username;
    }

    public void save(User user) {
        System.out.println("Saving user to: " + databaseUrl);
    }
}

public class UserService {
    private final UserRepository userRepository;
    private final EmailService emailService;

    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }

    public void createUser(User user) {
        userRepository.save(user);
        emailService.sendWelcomeEmail(user);
    }
}
```

### Method 1: Direct Method Calls

```java
@Configuration
public class AppConfig {

    @Bean
    public UserRepository userRepository() {
        return new UserRepository(
            "jdbc:mysql://localhost:3306/mydb",
            "admin"
        );
    }

    @Bean
    public EmailService emailService() {
        return new EmailService("smtp.gmail.com", 587);
    }

    @Bean
    public UserService userService() {
        // Direct constructor injection via method calls
        return new UserService(
            userRepository(),    // Spring ensures singleton behavior
            emailService()
        );
    }
}
```

**Important Note:**
When you call `userRepository()` multiple times within @Configuration class, Spring returns the **same instance** (singleton). This is achieved through CGLIB proxying.

### Method 2: Method Parameters (Recommended)

```java
@Configuration
public class AppConfig {

    @Bean
    public UserRepository userRepository() {
        return new UserRepository(
            "jdbc:mysql://localhost:3306/mydb",
            "admin"
        );
    }

    @Bean
    public EmailService emailService() {
        return new EmailService("smtp.gmail.com", 587);
    }

    @Bean
    public UserService userService(UserRepository userRepository,
                                   EmailService emailService) {
        // Spring injects parameters automatically
        return new UserService(userRepository, emailService);
    }
}
```

**Advantages:**
- Clearer dependency declaration
- Works even if methods are in different @Configuration classes
- No CGLIB proxy overhead for parameter injection
- More testable

### Method 3: Using @Autowired in Configuration

```java
@Configuration
public class AppConfig {

    @Autowired
    private DataSourceConfig dataSourceConfig;

    @Bean
    public UserRepository userRepository() {
        return new UserRepository(
            dataSourceConfig.getDatabaseUrl(),
            dataSourceConfig.getUsername()
        );
    }
}
```

### Constructor Injection with Collections

```java
@Configuration
public class AppConfig {

    @Bean
    public EmailService emailService() {
        List<String> recipients = Arrays.asList(
            "admin@example.com",
            "support@example.com"
        );

        Map<String, String> config = new HashMap<>();
        config.put("host", "smtp.gmail.com");
        config.put("port", "587");

        return new EmailService(recipients, config);
    }
}
```

### Constructor Injection with Conditional Logic

```java
@Configuration
public class AppConfig {

    @Bean
    public UserRepository userRepository(@Value("${app.env}") String environment) {
        if ("production".equals(environment)) {
            return new UserRepository("jdbc:mysql://prod-server:3306/db", "admin");
        } else {
            return new UserRepository("jdbc:h2:mem:testdb", "sa");
        }
    }
}
```

---

## Setter/Method Injection

Setter injection is useful for optional dependencies or when you need to reconfigure beans.

### Example Classes

```java
public class OrderService {
    private PaymentService paymentService;
    private NotificationService notificationService;
    private double taxRate;

    // Setter methods
    public void setPaymentService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    public void setNotificationService(NotificationService notificationService) {
        this.notificationService = notificationService;
    }

    public void setTaxRate(double taxRate) {
        this.taxRate = taxRate;
    }
}
```

### Setter Injection in Java Config

```java
@Configuration
public class AppConfig {

    @Bean
    public PaymentService paymentService() {
        return new PaymentService();
    }

    @Bean
    public NotificationService notificationService() {
        return new NotificationService();
    }

    @Bean
    public OrderService orderService(PaymentService paymentService,
                                     NotificationService notificationService) {
        OrderService orderService = new OrderService();

        // Setter injection
        orderService.setPaymentService(paymentService);
        orderService.setNotificationService(notificationService);
        orderService.setTaxRate(0.08);

        return orderService;
    }
}
```

### Builder Pattern with Java Config

```java
@Configuration
public class AppConfig {

    @Bean
    public DatabaseConfig databaseConfig() {
        return DatabaseConfig.builder()
            .url("jdbc:mysql://localhost:3306/mydb")
            .username("admin")
            .password("secret")
            .maxConnections(50)
            .timeout(30)
            .build();
    }
}
```

---

## Bean Scopes

### Available Scopes

| Scope | Description |
|-------|-------------|
| **singleton** | Single instance per container (default) |
| **prototype** | New instance for each request |
| **request** | Single instance per HTTP request |
| **session** | Single instance per HTTP session |
| **application** | Single instance per ServletContext |
| **websocket** | Single instance per WebSocket |

### 1. Singleton Scope (Default)

```java
@Configuration
public class AppConfig {

    @Bean
    @Scope("singleton")  // Default, can be omitted
    public UserService userService() {
        return new UserService();
    }

    // OR using constant
    @Bean
    @Scope(ConfigurableBeanFactory.SCOPE_SINGLETON)
    public ProductService productService() {
        return new ProductService();
    }
}
```

**Test:**

```java
ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
UserService service1 = context.getBean(UserService.class);
UserService service2 = context.getBean(UserService.class);
System.out.println(service1 == service2);  // true
```

### 2. Prototype Scope

```java
@Configuration
public class AppConfig {

    @Bean
    @Scope("prototype")
    public ShoppingCart shoppingCart() {
        return new ShoppingCart();
    }

    // OR using constant
    @Bean
    @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
    public SearchQuery searchQuery() {
        return new SearchQuery();
    }
}
```

**Test:**

```java
ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
ShoppingCart cart1 = context.getBean(ShoppingCart.class);
ShoppingCart cart2 = context.getBean(ShoppingCart.class);
System.out.println(cart1 == cart2);  // false
```

### 3. Request Scope (Web Application)

```java
@Configuration
public class WebConfig {

    @Bean
    @Scope(value = WebApplicationContext.SCOPE_REQUEST,
           proxyMode = ScopedProxyMode.TARGET_CLASS)
    public LoginAttempt loginAttempt() {
        return new LoginAttempt();
    }
}
```

**Proxy Mode Explanation:**
- `TARGET_CLASS`: Creates CGLIB proxy
- `INTERFACES`: Creates JDK dynamic proxy
- Required when injecting request/session scoped beans into singleton beans

### 4. Session Scope (Web Application)

```java
@Configuration
public class WebConfig {

    @Bean
    @Scope(value = WebApplicationContext.SCOPE_SESSION,
           proxyMode = ScopedProxyMode.TARGET_CLASS)
    public UserPreferences userPreferences() {
        return new UserPreferences();
    }

    @Bean
    @SessionScope  // Shortcut annotation
    public ShoppingCart sessionCart() {
        return new ShoppingCart();
    }
}
```

### 5. Custom Scope

```java
// Define custom scope
public class TenantScope implements Scope {
    private Map<String, Object> scopedObjects = new ConcurrentHashMap<>();

    @Override
    public Object get(String name, ObjectFactory<?> objectFactory) {
        String tenantId = TenantContext.getCurrentTenant();
        String key = tenantId + ":" + name;
        return scopedObjects.computeIfAbsent(key, k -> objectFactory.getObject());
    }

    @Override
    public Object remove(String name) {
        String tenantId = TenantContext.getCurrentTenant();
        return scopedObjects.remove(tenantId + ":" + name);
    }

    // Other methods...
}

// Register custom scope
@Configuration
public class ScopeConfig implements BeanFactoryPostProcessor {

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
        beanFactory.registerScope("tenant", new TenantScope());
    }
}

// Use custom scope
@Configuration
public class AppConfig {

    @Bean
    @Scope("tenant")
    public DataService dataService() {
        return new DataService();
    }
}
```

### Lazy Initialization

```java
@Configuration
public class AppConfig {

    @Bean
    @Lazy  // Bean created only when first requested
    public ExpensiveService expensiveService() {
        System.out.println("Creating expensive service...");
        return new ExpensiveService();
    }
}

// Lazy injection
@Service
public class MyService {

    @Lazy
    @Autowired
    private ExpensiveService expensiveService;  // Proxy injected, real bean created on first use
}
```

---

## Auto-wiring

Auto-wiring in Java configuration happens through constructor parameters, method parameters, and @Autowired annotation.

### 1. Auto-wiring via Method Parameters

```java
@Configuration
public class AppConfig {

    @Bean
    public UserRepository userRepository() {
        return new UserRepository();
    }

    @Bean
    public EmailService emailService() {
        return new EmailService();
    }

    @Bean
    // Spring automatically injects these parameters by type
    public UserService userService(UserRepository userRepository,
                                   EmailService emailService) {
        return new UserService(userRepository, emailService);
    }
}
```

### 2. Auto-wiring with @Autowired

```java
@Configuration
public class AppConfig {

    @Autowired
    private DataSource dataSource;  // Injected from another configuration

    @Bean
    public UserRepository userRepository() {
        return new UserRepository(dataSource);
    }
}
```

### 3. Auto-wiring with Qualifiers

When multiple beans of the same type exist:

```java
@Configuration
public class AppConfig {

    @Bean
    @Qualifier("mysqlDataSource")
    public DataSource mysqlDataSource() {
        return new MysqlDataSource();
    }

    @Bean
    @Qualifier("postgresDataSource")
    public DataSource postgresDataSource() {
        return new PostgresDataSource();
    }

    @Bean
    public UserRepository userRepository(@Qualifier("mysqlDataSource") DataSource dataSource) {
        return new UserRepository(dataSource);
    }
}
```

### 4. Auto-wiring with @Primary

```java
@Configuration
public class AppConfig {

    @Bean
    @Primary  // This bean is preferred when multiple candidates exist
    public DataSource primaryDataSource() {
        return new MysqlDataSource();
    }

    @Bean
    public DataSource secondaryDataSource() {
        return new PostgresDataSource();
    }

    @Bean
    // Injects primaryDataSource by default
    public UserRepository userRepository(DataSource dataSource) {
        return new UserRepository(dataSource);
    }
}
```

### 5. Auto-wiring Collections

```java
@Configuration
public class AppConfig {

    @Bean
    public MessageHandler emailHandler() {
        return new EmailHandler();
    }

    @Bean
    public MessageHandler smsHandler() {
        return new SmsHandler();
    }

    @Bean
    public MessageHandler pushHandler() {
        return new PushNotificationHandler();
    }

    @Bean
    // Spring injects all MessageHandler beans as a list
    public NotificationService notificationService(List<MessageHandler> handlers) {
        return new NotificationService(handlers);
    }

    @Bean
    // Spring injects all MessageHandler beans as a map (bean name -> bean)
    public MessageRouter messageRouter(Map<String, MessageHandler> handlers) {
        return new MessageRouter(handlers);
    }
}
```

### 6. Optional Dependencies

```java
@Configuration
public class AppConfig {

    @Bean
    public UserService userService(
            UserRepository userRepository,
            @Autowired(required = false) CacheService cacheService,
            Optional<MetricsService> metricsService) {

        UserService service = new UserService(userRepository);

        // Set optional dependencies if present
        if (cacheService != null) {
            service.setCacheService(cacheService);
        }

        metricsService.ifPresent(service::setMetricsService);

        return service;
    }
}
```

---

## Component Scanning

Component scanning automatically discovers and registers beans annotated with stereotype annotations.

### Stereotype Annotations

- `@Component`: Generic component
- `@Service`: Service layer component
- `@Repository`: Data access layer component
- `@Controller`: Web controller component
- `@RestController`: REST API controller
- `@Configuration`: Configuration class

### Enable Component Scanning

```java
@Configuration
@ComponentScan(basePackages = "com.example")
public class AppConfig {
    // Beans defined here
}

// Multiple packages
@Configuration
@ComponentScan(basePackages = {"com.example.service", "com.example.repository"})
public class AppConfig {
}

// Type-safe package scanning
@Configuration
@ComponentScan(basePackageClasses = {UserService.class, OrderService.class})
public class AppConfig {
}
```

### Component Classes

```java
@Component
public class UtilityService {
    public void doSomething() {
        System.out.println("Doing something...");
    }
}

@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    public void createUser(User user) {
        userRepository.save(user);
    }
}

@Repository
public class UserRepository {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    public void save(User user) {
        jdbcTemplate.update("INSERT INTO users...", user.getName());
    }
}

@RestController
@RequestMapping("/api/users")
public class UserController {

    @Autowired
    private UserService userService;

    @GetMapping
    public List<User> getUsers() {
        return userService.getAllUsers();
    }
}
```

### Custom Component Name

```java
@Component("myCustomService")
public class CustomService {
    // Bean name is "myCustomService"
}

@Service("userMgmtService")
public class UserManagementService {
    // Bean name is "userMgmtService"
}
```

### Filter Components

```java
@Configuration
@ComponentScan(
    basePackages = "com.example",
    includeFilters = @ComponentScan.Filter(
        type = FilterType.ANNOTATION,
        classes = CustomAnnotation.class
    ),
    excludeFilters = @ComponentScan.Filter(
        type = FilterType.REGEX,
        pattern = "com\\.example\\.exclude\\..*"
    )
)
public class AppConfig {
}
```

### Filter Types

```java
// 1. Annotation Type
@ComponentScan(
    includeFilters = @ComponentScan.Filter(
        type = FilterType.ANNOTATION,
        classes = MyCustomAnnotation.class
    )
)

// 2. Assignable Type
@ComponentScan(
    includeFilters = @ComponentScan.Filter(
        type = FilterType.ASSIGNABLE_TYPE,
        classes = MyInterface.class
    )
)

// 3. Regex Pattern
@ComponentScan(
    excludeFilters = @ComponentScan.Filter(
        type = FilterType.REGEX,
        pattern = "com\\.example\\.test\\..*"
    )
)

// 4. AspectJ Pattern
@ComponentScan(
    excludeFilters = @ComponentScan.Filter(
        type = FilterType.ASPECTJ,
        pattern = "com.example..*Test"
    )
)

// 5. Custom Filter
@ComponentScan(
    includeFilters = @ComponentScan.Filter(
        type = FilterType.CUSTOM,
        classes = MyCustomFilter.class
    )
)

// Custom Filter Implementation
public class MyCustomFilter implements TypeFilter {
    @Override
    public boolean match(MetadataReader metadataReader,
                        MetadataReaderFactory metadataReaderFactory) {
        // Custom logic to determine if class should be included
        return metadataReader.getClassMetadata()
                            .getClassName()
                            .contains("Custom");
    }
}
```

---

## Bean Lifecycle

### Lifecycle Callbacks

#### 1. Using initMethod and destroyMethod

```java
public class DatabaseConnection {
    private String url;

    public DatabaseConnection(String url) {
        this.url = url;
    }

    public void connect() {
        System.out.println("Connecting to: " + url);
    }

    public void disconnect() {
        System.out.println("Disconnecting from: " + url);
    }
}

@Configuration
public class AppConfig {

    @Bean(initMethod = "connect", destroyMethod = "disconnect")
    public DatabaseConnection databaseConnection() {
        return new DatabaseConnection("jdbc:mysql://localhost:3306/mydb");
    }
}
```

#### 2. Using @PostConstruct and @PreDestroy (JSR-250)

```java
import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

@Component
public class CacheService {

    @PostConstruct
    public void init() {
        System.out.println("Initializing cache...");
        // Load cache data
    }

    @PreDestroy
    public void cleanup() {
        System.out.println("Clearing cache...");
        // Clear cache data
    }
}
```

**Note:** Requires `javax.annotation-api` dependency:

```xml
<dependency>
    <groupId>javax.annotation</groupId>
    <artifactId>javax.annotation-api</artifactId>
    <version>1.3.2</version>
</dependency>
```

#### 3. Implementing InitializingBean and DisposableBean

```java
import org.springframework.beans.factory.InitializingBean;
import org.springframework.beans.factory.DisposableBean;

@Component
public class ResourceManager implements InitializingBean, DisposableBean {

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("Initializing resources...");
        // Initialization logic
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("Releasing resources...");
        // Cleanup logic
    }
}
```

### Complete Lifecycle Order

```java
@Component
public class LifecycleBean implements InitializingBean, DisposableBean,
                                     BeanNameAware, BeanFactoryAware,
                                     ApplicationContextAware {

    private String beanName;
    private BeanFactory beanFactory;
    private ApplicationContext applicationContext;

    // Constructor
    public LifecycleBean() {
        System.out.println("1. Constructor called");
    }

    // Setter injection
    @Autowired
    public void setDependency(SomeDependency dependency) {
        System.out.println("2. Dependencies injected");
    }

    // Aware interfaces
    @Override
    public void setBeanName(String name) {
        this.beanName = name;
        System.out.println("3. BeanNameAware: " + name);
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) {
        this.beanFactory = beanFactory;
        System.out.println("4. BeanFactoryAware called");
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) {
        this.applicationContext = applicationContext;
        System.out.println("5. ApplicationContextAware called");
    }

    // @PostConstruct
    @PostConstruct
    public void postConstruct() {
        System.out.println("6. @PostConstruct called");
    }

    // InitializingBean
    @Override
    public void afterPropertiesSet() {
        System.out.println("7. afterPropertiesSet called");
    }

    // Custom init method
    public void customInit() {
        System.out.println("8. Custom init method called");
    }

    // @PreDestroy
    @PreDestroy
    public void preDestroy() {
        System.out.println("9. @PreDestroy called");
    }

    // DisposableBean
    @Override
    public void destroy() {
        System.out.println("10. destroy() called");
    }

    // Custom destroy method
    public void customDestroy() {
        System.out.println("11. Custom destroy method called");
    }
}

@Configuration
public class AppConfig {

    @Bean(initMethod = "customInit", destroyMethod = "customDestroy")
    public LifecycleBean lifecycleBean() {
        return new LifecycleBean();
    }
}
```

### BeanPostProcessor

```java
@Component
public class CustomBeanPostProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) {
        System.out.println("Before initialization: " + beanName);
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) {
        System.out.println("After initialization: " + beanName);
        return bean;
    }
}
```

---

## Advanced Configuration

### 1. Importing Other Configurations

```java
@Configuration
public class DataSourceConfig {

    @Bean
    public DataSource dataSource() {
        return new HikariDataSource();
    }
}

@Configuration
public class ServiceConfig {

    @Bean
    public UserService userService() {
        return new UserService();
    }
}

@Configuration
@Import({DataSourceConfig.class, ServiceConfig.class})
public class AppConfig {
    // This configuration imports other configurations
}
```

### 2. Conditional Bean Registration

```java
// Custom condition
public class DatabaseCondition implements Condition {

    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        return context.getEnvironment().getProperty("database.enabled", Boolean.class, false);
    }
}

@Configuration
public class AppConfig {

    @Bean
    @Conditional(DatabaseCondition.class)
    public DataSource dataSource() {
        return new HikariDataSource();
    }

    // Spring Boot style conditions
    @Bean
    @ConditionalOnProperty(name = "cache.enabled", havingValue = "true")
    public CacheService cacheService() {
        return new CacheService();
    }

    @Bean
    @ConditionalOnClass(name = "com.mysql.cj.jdbc.Driver")
    public DataSource mysqlDataSource() {
        return new MysqlDataSource();
    }

    @Bean
    @ConditionalOnMissingBean(DataSource.class)
    public DataSource defaultDataSource() {
        return new H2DataSource();
    }
}
```

### 3. Property Sources

```java
@Configuration
@PropertySource("classpath:application.properties")
@PropertySource("classpath:database.properties")
public class AppConfig {

    @Value("${database.url}")
    private String databaseUrl;

    @Value("${database.username}")
    private String username;

    @Value("${database.password}")
    private String password;

    @Value("${app.max.connections:50}")  // Default value: 50
    private int maxConnections;

    @Bean
    public DataSource dataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl(databaseUrl);
        config.setUsername(username);
        config.setPassword(password);
        config.setMaximumPoolSize(maxConnections);
        return new HikariDataSource(config);
    }
}
```

#### application.properties

```properties
database.url=jdbc:mysql://localhost:3306/mydb
database.username=root
database.password=secret
app.max.connections=100
```

### 4. Environment Abstraction

```java
@Configuration
public class AppConfig {

    @Autowired
    private Environment env;

    @Bean
    public DataSource dataSource() {
        String url = env.getProperty("database.url");
        String username = env.getProperty("database.username");
        String password = env.getProperty("database.password");

        // Check if property exists
        if (env.containsProperty("database.pool.size")) {
            int poolSize = env.getProperty("database.pool.size", Integer.class);
        }

        // Get required property (throws exception if missing)
        String requiredProp = env.getRequiredProperty("database.url");

        return new HikariDataSource();
    }
}
```

### 5. Profiles

```java
@Configuration
public class AppConfig {

    @Bean
    @Profile("development")
    public DataSource devDataSource() {
        return new H2DataSource("jdbc:h2:mem:testdb");
    }

    @Bean
    @Profile("production")
    public DataSource prodDataSource() {
        return new MysqlDataSource("jdbc:mysql://prod-server:3306/proddb");
    }

    @Bean
    @Profile({"staging", "qa"})
    public DataSource stagingDataSource() {
        return new MysqlDataSource("jdbc:mysql://staging-server:3306/stagingdb");
    }

    @Bean
    @Profile("!production")  // Active for all profiles except production
    public DebugService debugService() {
        return new DebugService();
    }
}

// Activate profile programmatically
public class Application {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context =
            new AnnotationConfigApplicationContext();

        context.getEnvironment().setActiveProfiles("development");
        context.register(AppConfig.class);
        context.refresh();
    }
}

// Or via system property
// -Dspring.profiles.active=production

// Or via environment variable
// export SPRING_PROFILES_ACTIVE=production
```

### Profile-Specific Configuration Classes

```java
@Configuration
@Profile("development")
public class DevConfig {

    @Bean
    public DataSource dataSource() {
        return new H2DataSource();
    }

    @Bean
    public MockEmailService emailService() {
        return new MockEmailService();
    }
}

@Configuration
@Profile("production")
public class ProdConfig {

    @Bean
    public DataSource dataSource() {
        return new MysqlDataSource();
    }

    @Bean
    public RealEmailService emailService() {
        return new RealEmailService();
    }
}
```

### 6. Configuration Properties

```java
@Configuration
@ConfigurationProperties(prefix = "app.datasource")
public class DataSourceProperties {

    private String url;
    private String username;
    private String password;
    private PoolConfig pool;

    // Getters and setters
    public String getUrl() { return url; }
    public void setUrl(String url) { this.url = url; }

    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }

    public String getPassword() { return password; }
    public void setPassword(String password) { this.password = password; }

    public PoolConfig getPool() { return pool; }
    public void setPool(PoolConfig pool) { this.pool = pool; }

    public static class PoolConfig {
        private int maxSize;
        private int minIdle;

        public int getMaxSize() { return maxSize; }
        public void setMaxSize(int maxSize) { this.maxSize = maxSize; }

        public int getMinIdle() { return minIdle; }
        public void setMinIdle(int minIdle) { this.minIdle = minIdle; }
    }
}

@Configuration
@EnableConfigurationProperties(DataSourceProperties.class)
public class AppConfig {

    @Bean
    public DataSource dataSource(DataSourceProperties props) {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl(props.getUrl());
        config.setUsername(props.getUsername());
        config.setPassword(props.getPassword());
        config.setMaximumPoolSize(props.getPool().getMaxSize());
        config.setMinimumIdle(props.getPool().getMinIdle());
        return new HikariDataSource(config);
    }
}
```

#### application.properties

```properties
app.datasource.url=jdbc:mysql://localhost:3306/mydb
app.datasource.username=root
app.datasource.password=secret
app.datasource.pool.max-size=50
app.datasource.pool.min-idle=10
```

### 7. Factory Beans

```java
// FactoryBean implementation
public class ToolFactory implements FactoryBean<Tool> {

    private String toolId;

    public ToolFactory(String toolId) {
        this.toolId = toolId;
    }

    @Override
    public Tool getObject() throws Exception {
        return new Tool(toolId);
    }

    @Override
    public Class<?> getObjectType() {
        return Tool.class;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }
}

@Configuration
public class AppConfig {

    @Bean
    public ToolFactory toolFactory() {
        return new ToolFactory("tool-123");
    }

    // Usage
    @Bean
    public WorkerService workerService(Tool tool) {
        // Spring calls toolFactory.getObject() to get Tool
        return new WorkerService(tool);
    }
}
```

### 8. Method Injection (Lookup Method)

```java
public abstract class CommandProcessor {

    public void processCommand() {
        // Get a new instance each time
        Command command = createCommand();
        command.execute();
    }

    @Lookup
    protected abstract Command createCommand();
}

@Configuration
public class AppConfig {

    @Bean
    @Scope("prototype")
    public Command command() {
        return new Command();
    }

    @Bean
    public CommandProcessor commandProcessor() {
        return new CommandProcessor() {
            @Override
            protected Command createCommand() {
                return command();  // Returns new instance each time
            }
        };
    }
}
```

### 9. Depends-On

```java
@Configuration
public class AppConfig {

    @Bean
    public CacheManager cacheManager() {
        return new CacheManager();
    }

    @Bean
    @DependsOn("cacheManager")  // Ensures cacheManager is created first
    public UserService userService() {
        return new UserService();
    }

    @Bean
    @DependsOn({"cacheManager", "dataSource"})
    public OrderService orderService() {
        return new OrderService();
    }
}
```

### 10. Bean Validation

```java
@Configuration
@Validated
public class AppConfig {

    @Bean
    public UserService userService(
            @NotNull UserRepository userRepository,
            @Valid EmailConfig emailConfig) {
        return new UserService(userRepository, emailConfig);
    }
}

// EmailConfig with validation
public class EmailConfig {

    @NotBlank
    private String host;

    @Min(1)
    @Max(65535)
    private int port;

    @Email
    private String from;

    // Getters and setters
}
```

---

## Complete Examples

### Example 1: Simple Application

```java
// Domain Model
public class User {
    private String name;
    private String email;

    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }

    // Getters
    public String getName() { return name; }
    public String getEmail() { return email; }
}

// Repository
public class UserRepository {
    private final String databaseUrl;

    public UserRepository(String databaseUrl) {
        this.databaseUrl = databaseUrl;
    }

    public void save(User user) {
        System.out.println("Saving " + user.getName() + " to " + databaseUrl);
    }

    public void init() {
        System.out.println("UserRepository initialized with: " + databaseUrl);
    }

    public void cleanup() {
        System.out.println("UserRepository cleanup");
    }
}

// Service
public class EmailService {
    private final String smtpHost;
    private final int smtpPort;

    public EmailService(String smtpHost, int smtpPort) {
        this.smtpHost = smtpHost;
        this.smtpPort = smtpPort;
    }

    public void sendWelcomeEmail(User user) {
        System.out.println("Sending email to " + user.getEmail() +
                         " via " + smtpHost + ":" + smtpPort);
    }
}

public class UserService {
    private final UserRepository userRepository;
    private final EmailService emailService;

    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }

    public void registerUser(User user) {
        userRepository.save(user);
        emailService.sendWelcomeEmail(user);
    }
}

// Configuration
@Configuration
@PropertySource("classpath:application.properties")
public class AppConfig {

    @Value("${db.url}")
    private String databaseUrl;

    @Value("${mail.host}")
    private String mailHost;

    @Value("${mail.port}")
    private int mailPort;

    @Bean(initMethod = "init", destroyMethod = "cleanup")
    public UserRepository userRepository() {
        return new UserRepository(databaseUrl);
    }

    @Bean
    public EmailService emailService() {
        return new EmailService(mailHost, mailPort);
    }

    @Bean
    public UserService userService(UserRepository userRepository,
                                   EmailService emailService) {
        return new UserService(userRepository, emailService);
    }
}

// Main Application
public class Application {
    public static void main(String[] args) {
        ApplicationContext context =
            new AnnotationConfigApplicationContext(AppConfig.class);

        UserService userService = context.getBean(UserService.class);

        User user = new User("John Doe", "john@example.com");
        userService.registerUser(user);

        ((AnnotationConfigApplicationContext) context).close();
    }
}
```

### Example 2: Multi-Layer Application with Component Scanning

```java
// Entity
public class Product {
    private Long id;
    private String name;
    private double price;

    // Constructor, getters, setters
}

// Repository Layer
@Repository
public class ProductRepository {

    private final JdbcTemplate jdbcTemplate;

    @Autowired
    public ProductRepository(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public void save(Product product) {
        String sql = "INSERT INTO products (name, price) VALUES (?, ?)";
        jdbcTemplate.update(sql, product.getName(), product.getPrice());
    }

    public Product findById(Long id) {
        String sql = "SELECT * FROM products WHERE id = ?";
        return jdbcTemplate.queryForObject(sql,
            (rs, rowNum) -> new Product(
                rs.getLong("id"),
                rs.getString("name"),
                rs.getDouble("price")
            ),
            id);
    }
}

// Service Layer
@Service
public class ProductService {

    private final ProductRepository productRepository;
    private final NotificationService notificationService;

    @Autowired
    public ProductService(ProductRepository productRepository,
                         NotificationService notificationService) {
        this.productRepository = productRepository;
        this.notificationService = notificationService;
    }

    public void createProduct(Product product) {
        productRepository.save(product);
        notificationService.notifyProductCreated(product);
    }

    public Product getProduct(Long id) {
        return productRepository.findById(id);
    }
}

@Service
public class NotificationService {

    @Autowired(required = false)
    private EmailService emailService;

    public void notifyProductCreated(Product product) {
        System.out.println("Product created: " + product.getName());
        if (emailService != null) {
            emailService.sendNotification("New product: " + product.getName());
        }
    }
}

// Configuration
@Configuration
@ComponentScan(basePackages = "com.example")
@PropertySource("classpath:application.properties")
public class AppConfig {

    @Value("${db.url}")
    private String dbUrl;

    @Value("${db.username}")
    private String dbUsername;

    @Value("${db.password}")
    private String dbPassword;

    @Bean
    public DataSource dataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl(dbUrl);
        config.setUsername(dbUsername);
        config.setPassword(dbPassword);
        return new HikariDataSource(config);
    }

    @Bean
    public JdbcTemplate jdbcTemplate(DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}

// Main Application
public class Application {
    public static void main(String[] args) {
        ApplicationContext context =
            new AnnotationConfigApplicationContext(AppConfig.class);

        ProductService productService = context.getBean(ProductService.class);

        Product product = new Product(null, "Laptop", 999.99);
        productService.createProduct(product);

        ((AnnotationConfigApplicationContext) context).close();
    }
}
```

### Example 3: Profile-Based Configuration

```java
// Configuration interface
public interface AppConfiguration {
    DataSource dataSource();
    EmailService emailService();
}

// Development Configuration
@Configuration
@Profile("development")
public class DevConfig implements AppConfiguration {

    @Bean
    @Override
    public DataSource dataSource() {
        System.out.println("Creating H2 in-memory database");
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.H2)
            .build();
    }

    @Bean
    @Override
    public EmailService emailService() {
        System.out.println("Creating mock email service");
        return new MockEmailService();
    }

    @Bean
    public DebugService debugService() {
        return new DebugService();
    }
}

// Production Configuration
@Configuration
@Profile("production")
public class ProdConfig implements AppConfiguration {

    @Value("${db.url}")
    private String dbUrl;

    @Value("${db.username}")
    private String dbUsername;

    @Value("${db.password}")
    private String dbPassword;

    @Bean
    @Override
    public DataSource dataSource() {
        System.out.println("Creating production MySQL database");
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl(dbUrl);
        config.setUsername(dbUsername);
        config.setPassword(dbPassword);
        config.setMaximumPoolSize(50);
        return new HikariDataSource(config);
    }

    @Bean
    @Override
    public EmailService emailService() {
        System.out.println("Creating real email service");
        return new SmtpEmailService();
    }
}

// Main Configuration
@Configuration
@Import({DevConfig.class, ProdConfig.class})
@ComponentScan(basePackages = "com.example")
public class AppConfig {

    @Bean
    public UserService userService(DataSource dataSource,
                                   EmailService emailService) {
        return new UserService(dataSource, emailService);
    }
}

// Main Application
public class Application {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context =
            new AnnotationConfigApplicationContext();

        // Set active profile
        String profile = System.getProperty("app.profile", "development");
        context.getEnvironment().setActiveProfiles(profile);

        context.register(AppConfig.class);
        context.refresh();

        UserService userService = context.getBean(UserService.class);
        userService.doSomething();

        context.close();
    }
}

// Run with: java -Dapp.profile=production Application
```

---

## Best Practices

### 1. Constructor Injection for Mandatory Dependencies

```java
// GOOD - Immutable, testable
@Service
public class UserService {
    private final UserRepository userRepository;
    private final EmailService emailService;

    @Autowired  // Optional in single-constructor case
    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }
}

// BAD - Mutable, harder to test
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;

    @Autowired
    private EmailService emailService;
}
```

### 2. Use @Qualifier for Multiple Beans of Same Type

```java
@Configuration
public class AppConfig {

    @Bean
    @Qualifier("primaryCache")
    public CacheService primaryCache() {
        return new RedisCacheService();
    }

    @Bean
    @Qualifier("secondaryCache")
    public CacheService secondaryCache() {
        return new MemoryCacheService();
    }
}

@Service
public class ProductService {

    private final CacheService cacheService;

    public ProductService(@Qualifier("primaryCache") CacheService cacheService) {
        this.cacheService = cacheService;
    }
}
```

### 3. Use Profiles for Environment-Specific Configuration

```java
@Configuration
public class AppConfig {

    @Bean
    @Profile("!production")
    public DataSource devDataSource() {
        return new H2DataSource();
    }

    @Bean
    @Profile("production")
    public DataSource prodDataSource() {
        return new MysqlDataSource();
    }
}
```

### 4. Externalize Configuration

```java
@Configuration
@PropertySource("classpath:application.properties")
public class AppConfig {

    @Bean
    public DataSource dataSource(
            @Value("${db.url}") String url,
            @Value("${db.username}") String username,
            @Value("${db.password}") String password) {
        // Configuration from properties file
    }
}
```

### 5. Use @Primary for Default Bean

```java
@Configuration
public class AppConfig {

    @Bean
    @Primary
    public MessageSender primarySender() {
        return new EmailSender();
    }

    @Bean
    public MessageSender smsSender() {
        return new SmsSender();
    }
}
```

### 6. Keep Configuration Classes Focused

```java
// GOOD - Focused configurations
@Configuration
public class DataSourceConfig { /* datasource beans */ }

@Configuration
public class SecurityConfig { /* security beans */ }

@Configuration
public class WebConfig { /* web beans */ }

@Configuration
@Import({DataSourceConfig.class, SecurityConfig.class, WebConfig.class})
public class AppConfig { }

// BAD - Monolithic configuration
@Configuration
public class AppConfig {
    // 100+ bean definitions in one file
}
```

### 7. Use Lazy Initialization Wisely

```java
@Configuration
public class AppConfig {

    @Bean
    @Lazy  // Only create when needed
    public ExpensiveService expensiveService() {
        return new ExpensiveService();
    }
}
```

### 8. Document Configuration Classes

```java
/**
 * Main application configuration.
 *
 * Configures:
 * - Database connections
 * - Email services
 * - Caching
 *
 * Active profiles:
 * - development: Uses H2 in-memory database
 * - production: Uses MySQL database
 */
@Configuration
@ComponentScan(basePackages = "com.example")
public class AppConfig {
    // Configuration
}
```

---

## Common Pitfalls

### 1. Circular Dependencies

```java
// PROBLEMATIC
@Configuration
public class AppConfig {

    @Bean
    public ServiceA serviceA(ServiceB serviceB) {
        return new ServiceA(serviceB);
    }

    @Bean
    public ServiceB serviceB(ServiceA serviceA) {
        return new ServiceB(serviceA);
    }
}

// SOLUTION 1: Use @Lazy
@Bean
public ServiceA serviceA(@Lazy ServiceB serviceB) {
    return new ServiceA(serviceB);
}

// SOLUTION 2: Refactor to remove circular dependency
```

### 2. Forgetting @Configuration

```java
// WRONG - Missing @Configuration
public class AppConfig {

    @Bean
    public UserService userService() {
        return new UserService(userRepository());  // Creates new instance!
    }

    @Bean
    public UserRepository userRepository() {
        return new UserRepository();
    }
}

// CORRECT
@Configuration
public class AppConfig {

    @Bean
    public UserService userService() {
        return new UserService(userRepository());  // Returns singleton
    }

    @Bean
    public UserRepository userRepository() {
        return new UserRepository();
    }
}
```

### 3. Scope Misuse

```java
// PROBLEMATIC - Singleton depending on prototype
@Configuration
public class AppConfig {

    @Bean
    @Scope("prototype")
    public ShoppingCart shoppingCart() {
        return new ShoppingCart();
    }

    @Bean
    public UserService userService() {
        // This captures ONE instance of ShoppingCart
        return new UserService(shoppingCart());
    }
}

// SOLUTION - Use Provider or lookup method
@Bean
public UserService userService(Provider<ShoppingCart> cartProvider) {
    return new UserService(cartProvider);
}
```

### 4. Not Closing Context

```java
// BAD
public static void main(String[] args) {
    ApplicationContext context =
        new AnnotationConfigApplicationContext(AppConfig.class);
    // Context never closed - resources may leak
}

// GOOD
public static void main(String[] args) {
    try (AnnotationConfigApplicationContext context =
            new AnnotationConfigApplicationContext(AppConfig.class)) {
        // Use context
    } // Automatically closed
}

// OR
context.registerShutdownHook();
```

### 5. Ambiguous Bean Resolution

```java
// PROBLEMATIC
@Configuration
public class AppConfig {

    @Bean
    public DataSource dataSource1() {
        return new HikariDataSource();
    }

    @Bean
    public DataSource dataSource2() {
        return new MysqlDataSource();
    }

    @Bean
    public UserRepository userRepository(DataSource dataSource) {
        // Which DataSource? Error!
        return new UserRepository(dataSource);
    }
}

// SOLUTION
@Bean
public UserRepository userRepository(@Qualifier("dataSource1") DataSource dataSource) {
    return new UserRepository(dataSource);
}
```

---

## Comparison: Java Config vs XML Config

| Aspect | Java Configuration | XML Configuration |
|--------|-------------------|-------------------|
| **Type Safety** | Compile-time checking | Runtime checking |
| **Refactoring** | IDE support | Manual |
| **Readability** | Java syntax | XML syntax |
| **Conditional Logic** | Easy | Difficult |
| **Learning Curve** | Java knowledge | XML + Spring XML schema |
| **Tooling** | Excellent | Good |
| **Flexibility** | High | Medium |
| **Verbosity** | Medium | High |

### Same Configuration in Both Styles

**Java Config:**
```java
@Configuration
public class AppConfig {

    @Bean
    public UserRepository userRepository() {
        return new UserRepository("jdbc:mysql://localhost:3306/mydb");
    }

    @Bean
    public UserService userService(UserRepository userRepository) {
        return new UserService(userRepository);
    }
}
```

**XML Config:**
```xml
<beans>
    <bean id="userRepository" class="com.example.UserRepository">
        <constructor-arg value="jdbc:mysql://localhost:3306/mydb"/>
    </bean>

    <bean id="userService" class="com.example.UserService">
        <constructor-arg ref="userRepository"/>
    </bean>
</beans>
```

---

## Conclusion

Java-based configuration is the modern approach for Spring applications, offering:

- **Type Safety**: Compile-time checking prevents many errors
- **Refactoring Support**: IDEs can safely rename and refactor
- **Programmatic Control**: Full Java power for bean creation
- **Better Testing**: Easier to test configuration classes
- **No XML**: Pure Java approach

### Key Takeaways

1. **Use @Configuration and @Bean** for explicit bean definitions
2. **Prefer Constructor Injection** for immutability and testability
3. **Use @ComponentScan** for automatic bean discovery
4. **Leverage @Profile** for environment-specific configuration
5. **Externalize with @PropertySource** and @Value
6. **Use @Qualifier and @Primary** for disambiguation
7. **Implement Lifecycle Callbacks** for resource management
8. **Keep Configurations Focused** and modular

### When to Use Each Approach

- **Java Config**: Modern applications, microservices, Spring Boot
- **XML Config**: Legacy applications, when external configuration is critical
- **Annotation Config**: Component-level configuration (@Component, @Service, etc.)
- **Hybrid**: Combine approaches as needed (@ImportResource for XML)

---

## Additional Resources

- Spring Framework Documentation: https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java
- Spring Java Configuration Reference
- @Configuration Javadoc
- Spring Boot Auto-configuration
- Spring Profiles Guide
- Testing Spring Applications
