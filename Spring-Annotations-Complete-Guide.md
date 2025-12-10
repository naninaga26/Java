# Spring Framework Annotations - Complete Guide

## Table of Contents
1. [Core Stereotype Annotations](#core-stereotype-annotations)
2. [Configuration Annotations](#configuration-annotations)
3. [Dependency Injection Annotations](#dependency-injection-annotations)
4. [Bean Lifecycle Annotations](#bean-lifecycle-annotations)
5. [Scope and Lazy Annotations](#scope-and-lazy-annotations)
6. [Conditional Annotations](#conditional-annotations)
7. [Profile Annotations](#profile-annotations)
8. [Property and Value Annotations](#property-and-value-annotations)
9. [AOP Annotations](#aop-annotations)
10. [Transaction Annotations](#transaction-annotations)
11. [Validation Annotations](#validation-annotations)
12. [Spring Web Annotations](#spring-web-annotations)
13. [Spring Boot Annotations](#spring-boot-annotations)
14. [Advanced Annotations](#advanced-annotations)

---

## Core Stereotype Annotations

### @Component

Base stereotype annotation for any Spring-managed component.

```java
@Component
public class UtilityService {

    public String generateId() {
        return UUID.randomUUID().toString();
    }
}

// Custom bean name
@Component("myUtil")
public class CustomUtility {
    // Bean name is "myUtil" instead of "customUtility"
}
```

**Usage:**
- Generic Spring-managed bean
- Base annotation for other stereotypes
- Auto-detected by component scanning

---

### @Service

Specialization of @Component for service layer.

```java
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    public User createUser(String name, String email) {
        User user = new User(name, email);
        return userRepository.save(user);
    }

    public List<User> getAllUsers() {
        return userRepository.findAll();
    }
}
```

**Purpose:**
- Indicates service layer component
- Contains business logic
- Semantic meaning for better organization
- Can be used for AOP pointcuts

---

### @Repository

Specialization of @Component for data access layer.

```java
@Repository
public class UserRepository {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    public User save(User user) {
        String sql = "INSERT INTO users (name, email) VALUES (?, ?)";
        jdbcTemplate.update(sql, user.getName(), user.getEmail());
        return user;
    }

    public List<User> findAll() {
        String sql = "SELECT * FROM users";
        return jdbcTemplate.query(sql, new UserRowMapper());
    }

    public Optional<User> findById(Long id) {
        String sql = "SELECT * FROM users WHERE id = ?";
        try {
            User user = jdbcTemplate.queryForObject(sql, new UserRowMapper(), id);
            return Optional.of(user);
        } catch (EmptyResultDataAccessException e) {
            return Optional.empty();
        }
    }
}
```

**Special Feature:**
- Automatic exception translation (SQLException → DataAccessException)
- Persistence layer semantic
- Better error handling

---

### @Controller

Specialization of @Component for Spring MVC controllers.

```java
@Controller
@RequestMapping("/users")
public class UserController {

    @Autowired
    private UserService userService;

    @GetMapping
    public String listUsers(Model model) {
        model.addAttribute("users", userService.getAllUsers());
        return "user-list";  // Returns view name
    }

    @GetMapping("/{id}")
    public String viewUser(@PathVariable Long id, Model model) {
        User user = userService.getUserById(id);
        model.addAttribute("user", user);
        return "user-detail";
    }

    @PostMapping
    public String createUser(@ModelAttribute User user) {
        userService.createUser(user);
        return "redirect:/users";
    }
}
```

**Purpose:**
- Web layer component
- Handles HTTP requests
- Returns views or model data

---

### @RestController

Combines @Controller and @ResponseBody.

```java
@RestController
@RequestMapping("/api/users")
public class UserRestController {

    @Autowired
    private UserService userService;

    @GetMapping
    public List<User> getAllUsers() {
        return userService.getAllUsers();  // Automatically serialized to JSON
    }

    @GetMapping("/{id}")
    public ResponseEntity<User> getUserById(@PathVariable Long id) {
        return userService.getUserById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }

    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody User user) {
        User created = userService.createUser(user);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }

    @PutMapping("/{id}")
    public ResponseEntity<User> updateUser(@PathVariable Long id,
                                          @RequestBody User user) {
        User updated = userService.updateUser(id, user);
        return ResponseEntity.ok(updated);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.noContent().build();
    }
}
```

**Equivalent to:**
```java
@Controller
@ResponseBody
public class UserRestController {
    // Same methods
}
```

---

## Configuration Annotations

### @Configuration

Indicates that a class declares one or more @Bean methods.

```java
@Configuration
public class AppConfig {

    @Bean
    public UserService userService() {
        return new UserService();
    }

    @Bean
    public DataSource dataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:mysql://localhost:3306/mydb");
        return new HikariDataSource(config);
    }
}
```

**Key Features:**
- Methods annotated with @Bean produce Spring beans
- Spring uses CGLIB to proxy the class
- Method calls within the class return singleton beans

```java
@Configuration
public class DatabaseConfig {

    @Bean
    public DataSource dataSource() {
        return new HikariDataSource();
    }

    @Bean
    public JdbcTemplate jdbcTemplate() {
        // Multiple calls to dataSource() return the same instance
        return new JdbcTemplate(dataSource());
    }

    @Bean
    public TransactionManager transactionManager() {
        return new DataSourceTransactionManager(dataSource());
    }
}
```

---

### @Bean

Indicates that a method produces a bean managed by Spring.

```java
@Configuration
public class AppConfig {

    // Simple bean
    @Bean
    public UserService userService() {
        return new UserService();
    }

    // Custom bean name
    @Bean(name = "userSvc")
    public UserService customNamedService() {
        return new UserService();
    }

    // Multiple names
    @Bean(name = {"userService", "userManager", "userSvc"})
    public UserService multiNamedService() {
        return new UserService();
    }

    // With initialization and destruction methods
    @Bean(initMethod = "init", destroyMethod = "cleanup")
    public DatabaseConnection databaseConnection() {
        return new DatabaseConnection();
    }

    // Dependency injection via parameters
    @Bean
    public OrderService orderService(UserService userService,
                                    PaymentService paymentService) {
        return new OrderService(userService, paymentService);
    }
}
```

---

### @ComponentScan

Configures component scanning directives.

```java
// Simple scan
@Configuration
@ComponentScan(basePackages = "com.example")
public class AppConfig {
}

// Multiple packages
@Configuration
@ComponentScan(basePackages = {"com.example.service", "com.example.repository"})
public class AppConfig {
}

// Type-safe package scanning
@Configuration
@ComponentScan(basePackageClasses = {UserService.class, ProductService.class})
public class AppConfig {
}

// With filters
@Configuration
@ComponentScan(
    basePackages = "com.example",
    includeFilters = @ComponentScan.Filter(
        type = FilterType.ANNOTATION,
        classes = CustomComponent.class
    ),
    excludeFilters = @ComponentScan.Filter(
        type = FilterType.REGEX,
        pattern = "com\\.example\\.exclude\\..*"
    )
)
public class AppConfig {
}

// Exclude @Configuration classes
@Configuration
@ComponentScan(
    basePackages = "com.example",
    excludeFilters = @ComponentScan.Filter(
        type = FilterType.ANNOTATION,
        classes = Configuration.class
    )
)
public class AppConfig {
}
```

**Filter Types:**
```java
// ANNOTATION - Filter by annotation type
@ComponentScan.Filter(type = FilterType.ANNOTATION, classes = MyAnnotation.class)

// ASSIGNABLE_TYPE - Filter by class/interface
@ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = MyInterface.class)

// ASPECTJ - Filter using AspectJ pattern
@ComponentScan.Filter(type = FilterType.ASPECTJ, pattern = "com.example..*Service+")

// REGEX - Filter using regex pattern
@ComponentScan.Filter(type = FilterType.REGEX, pattern = ".*Test.*")

// CUSTOM - Custom filter implementation
@ComponentScan.Filter(type = FilterType.CUSTOM, classes = MyCustomFilter.class)
```

---

### @Import

Imports additional configuration classes.

```java
@Configuration
public class DatabaseConfig {
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

// Import other configurations
@Configuration
@Import({DatabaseConfig.class, ServiceConfig.class})
public class AppConfig {
    // This configuration includes beans from imported configs
}

// Import with ImportSelector
@Configuration
@Import(MyImportSelector.class)
public class AppConfig {
}

class MyImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata metadata) {
        return new String[] {
            "com.example.config.DatabaseConfig",
            "com.example.config.ServiceConfig"
        };
    }
}
```

---

### @ImportResource

Imports XML configuration files.

```java
@Configuration
@ImportResource("classpath:beans.xml")
public class AppConfig {
    // Mix Java config with XML config
}

// Multiple XML files
@Configuration
@ImportResource({
    "classpath:database-config.xml",
    "classpath:service-config.xml"
})
public class AppConfig {
}
```

---

### @PropertySource

Adds property sources to Spring Environment.

```java
@Configuration
@PropertySource("classpath:application.properties")
public class AppConfig {

    @Value("${database.url}")
    private String databaseUrl;

    @Bean
    public DataSource dataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl(databaseUrl);
        return new HikariDataSource(config);
    }
}

// Multiple property files
@Configuration
@PropertySource("classpath:application.properties")
@PropertySource("classpath:database.properties")
public class AppConfig {
}

// Or using array
@Configuration
@PropertySources({
    @PropertySource("classpath:application.properties"),
    @PropertySource("classpath:database.properties")
})
public class AppConfig {
}

// Ignore if file not found
@Configuration
@PropertySource(value = "classpath:optional.properties", ignoreResourceNotFound = true)
public class AppConfig {
}
```

---

## Dependency Injection Annotations

### @Autowired

Marks a dependency to be injected by Spring.

```java
// Constructor injection (Recommended)
@Service
public class UserService {

    private final UserRepository userRepository;
    private final EmailService emailService;

    @Autowired  // Optional in single-constructor case (Spring 4.3+)
    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }
}

// Setter injection
@Service
public class OrderService {

    private PaymentService paymentService;

    @Autowired
    public void setPaymentService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}

// Field injection (Not recommended)
@Service
public class ProductService {

    @Autowired
    private ProductRepository productRepository;
}

// Method injection
@Service
public class ReportService {

    private DataService dataService;
    private FormatterService formatterService;

    @Autowired
    public void initialize(DataService dataService, FormatterService formatterService) {
        this.dataService = dataService;
        this.formatterService = formatterService;
    }
}

// Optional dependency
@Service
public class NotificationService {

    @Autowired(required = false)
    private SmsService smsService;  // May be null

    @Autowired
    private Optional<PushService> pushService;  // Java 8 Optional
}

// Collection injection
@Service
public class MessageRouter {

    @Autowired
    private List<MessageHandler> handlers;  // All MessageHandler beans

    @Autowired
    private Map<String, MessageHandler> handlerMap;  // Bean name -> Bean
}
```

---

### @Qualifier

Specifies which bean to inject when multiple candidates exist.

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
}

@Service
public class UserService {

    private final DataSource dataSource;

    @Autowired
    public UserService(@Qualifier("mysqlDataSource") DataSource dataSource) {
        this.dataSource = dataSource;
    }
}

// Custom qualifier annotation
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface MySQL {
}

@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Postgres {
}

// Usage
@Configuration
public class AppConfig {

    @Bean
    @MySQL
    public DataSource mysqlDataSource() {
        return new MysqlDataSource();
    }

    @Bean
    @Postgres
    public DataSource postgresDataSource() {
        return new PostgresDataSource();
    }
}

@Service
public class UserService {

    @Autowired
    @MySQL
    private DataSource dataSource;
}
```

---

### @Primary

Indicates a bean should be preferred when multiple candidates exist.

```java
@Configuration
public class AppConfig {

    @Bean
    @Primary  // This will be injected by default
    public DataSource primaryDataSource() {
        return new MysqlDataSource();
    }

    @Bean
    public DataSource secondaryDataSource() {
        return new PostgresDataSource();
    }
}

@Service
public class UserService {

    @Autowired
    private DataSource dataSource;  // primaryDataSource will be injected
}
```

---

### @Value

Injects values from property files or SpEL expressions.

```java
@Component
public class AppProperties {

    // Simple property injection
    @Value("${app.name}")
    private String appName;

    // With default value
    @Value("${app.timeout:30}")
    private int timeout;

    // System property
    @Value("${java.home}")
    private String javaHome;

    // Environment variable
    @Value("${USER}")
    private String user;

    // SpEL expression
    @Value("#{systemProperties['user.name']}")
    private String userName;

    // Mathematical expression
    @Value("#{10 * 5}")
    private int result;

    // Bean property
    @Value("#{databaseConfig.maxConnections}")
    private int maxConnections;

    // Collection from properties
    @Value("${app.servers}")
    private List<String> servers;  // app.servers=server1,server2,server3

    // Array
    @Value("${app.ports}")
    private int[] ports;

    // Map (requires SpEL)
    @Value("#{${app.settings}}")
    private Map<String, String> settings;  // app.settings={key1:'value1',key2:'value2'}
}

// Constructor injection with @Value
@Service
public class EmailService {

    private final String smtpHost;
    private final int smtpPort;

    public EmailService(
            @Value("${mail.smtp.host}") String smtpHost,
            @Value("${mail.smtp.port}") int smtpPort) {
        this.smtpHost = smtpHost;
        this.smtpPort = smtpPort;
    }
}
```

**Complex SpEL Examples:**

```java
@Component
public class SpELExamples {

    // Conditional
    @Value("#{${app.enabled} ? 'Enabled' : 'Disabled'}")
    private String status;

    // Null-safe navigation
    @Value("#{config.database?.url ?: 'default-url'}")
    private String databaseUrl;

    // Regular expression
    @Value("#{'${app.email}' matches '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}'}")
    private boolean validEmail;

    // List filtering
    @Value("#{users.?[age > 18]}")
    private List<User> adults;

    // Calling methods
    @Value("#{T(java.lang.Math).random() * 100}")
    private double randomNumber;

    // Static field
    @Value("#{T(java.lang.Math).PI}")
    private double pi;
}
```

---

### @Resource (JSR-250)

Java standard annotation for dependency injection (by name).

```java
@Service
public class UserService {

    @Resource(name = "userRepository")
    private UserRepository repository;

    @Resource  // Injects by field name
    private EmailService emailService;
}
```

---

### @Inject (JSR-330)

Java standard alternative to @Autowired.

```java
@Service
public class OrderService {

    @Inject
    private PaymentService paymentService;

    @Inject
    @Named("primaryCache")
    private CacheService cacheService;
}
```

**Requires dependency:**
```xml
<dependency>
    <groupId>javax.inject</groupId>
    <artifactId>javax.inject</artifactId>
    <version>1</version>
</dependency>
```

---

## Bean Lifecycle Annotations

### @PostConstruct

Executed after dependency injection.

```java
@Component
public class DatabaseConnection {

    @Value("${db.url}")
    private String url;

    private Connection connection;

    @PostConstruct
    public void init() {
        System.out.println("Initializing database connection to: " + url);
        try {
            connection = DriverManager.getConnection(url);
        } catch (SQLException e) {
            throw new RuntimeException("Failed to connect", e);
        }
    }

    public void executeQuery(String sql) {
        // Use connection
    }
}
```

**Execution Order:**
1. Constructor
2. Dependency Injection (@Autowired, @Value)
3. **@PostConstruct**
4. Bean ready for use

---

### @PreDestroy

Executed before bean destruction.

```java
@Component
public class CacheService {

    private Map<String, Object> cache = new ConcurrentHashMap<>();

    public void put(String key, Object value) {
        cache.put(key, value);
    }

    @PreDestroy
    public void cleanup() {
        System.out.println("Clearing cache before shutdown");
        cache.clear();
        // Release resources, close connections, etc.
    }
}
```

**Use Cases:**
- Close database connections
- Release file handles
- Clear caches
- Shutdown thread pools
- Flush pending writes

---

## Scope and Lazy Annotations

### @Scope

Defines the scope of a bean.

```java
// Singleton (default)
@Component
@Scope("singleton")
public class ConfigService {
    // Single instance per container
}

// Prototype
@Component
@Scope("prototype")
public class ShoppingCart {
    // New instance for each request
}

// Using constants
@Component
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class SearchQuery {
}

// Request scope (Web)
@Component
@Scope(value = WebApplicationContext.SCOPE_REQUEST,
       proxyMode = ScopedProxyMode.TARGET_CLASS)
public class LoginAttempt {
    // New instance per HTTP request
}

// Session scope (Web)
@Component
@Scope(value = WebApplicationContext.SCOPE_SESSION,
       proxyMode = ScopedProxyMode.TARGET_CLASS)
public class UserPreferences {
    // Single instance per HTTP session
}

// Application scope (Web)
@Component
@Scope(value = WebApplicationContext.SCOPE_APPLICATION,
       proxyMode = ScopedProxyMode.TARGET_CLASS)
public class AppMetrics {
    // Single instance per ServletContext
}
```

**Proxy Modes:**
```java
// TARGET_CLASS - CGLIB proxy
@Scope(value = "session", proxyMode = ScopedProxyMode.TARGET_CLASS)

// INTERFACES - JDK dynamic proxy
@Scope(value = "session", proxyMode = ScopedProxyMode.INTERFACES)

// NO - No proxy (default)
@Scope(value = "singleton", proxyMode = ScopedProxyMode.NO)
```

---

### @Lazy

Delays bean initialization until first use.

```java
// Lazy singleton
@Component
@Lazy
public class ExpensiveService {

    public ExpensiveService() {
        System.out.println("ExpensiveService created");
        // Expensive initialization
    }
}

// Lazy injection
@Service
public class UserService {

    @Lazy
    @Autowired
    private ExpensiveService expensiveService;  // Proxy injected, real bean created on first use
}

// Lazy in configuration
@Configuration
public class AppConfig {

    @Bean
    @Lazy
    public DataSource dataSource() {
        System.out.println("Creating DataSource");
        return new HikariDataSource();
    }
}
```

---

### @RequestScope / @SessionScope / @ApplicationScope

Shortcut annotations for web scopes.

```java
@Component
@RequestScope  // Equivalent to @Scope(value = "request", proxyMode = TARGET_CLASS)
public class LoginRequest {
    private String username;
    private String password;
    // Getters and setters
}

@Component
@SessionScope  // Equivalent to @Scope(value = "session", proxyMode = TARGET_CLASS)
public class UserSession {
    private User currentUser;
    private List<String> visitedPages = new ArrayList<>();
    // Methods
}

@Component
@ApplicationScope  // Equivalent to @Scope(value = "application", proxyMode = TARGET_CLASS)
public class ApplicationStats {
    private long requestCount = 0;
    private long activeUsers = 0;
    // Methods
}
```

---

## Conditional Annotations

### @Conditional

Creates beans conditionally based on custom logic.

```java
// Custom condition
public class WindowsCondition implements Condition {

    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        return context.getEnvironment()
                     .getProperty("os.name")
                     .toLowerCase()
                     .contains("windows");
    }
}

public class LinuxCondition implements Condition {

    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        return context.getEnvironment()
                     .getProperty("os.name")
                     .toLowerCase()
                     .contains("linux");
    }
}

// Usage
@Configuration
public class AppConfig {

    @Bean
    @Conditional(WindowsCondition.class)
    public FileService windowsFileService() {
        return new WindowsFileService();
    }

    @Bean
    @Conditional(LinuxCondition.class)
    public FileService linuxFileService() {
        return new LinuxFileService();
    }
}
```

**Spring Boot Conditional Annotations:**

```java
// Based on class presence
@Bean
@ConditionalOnClass(name = "com.mysql.cj.jdbc.Driver")
public DataSource mysqlDataSource() {
    return new MysqlDataSource();
}

// Based on missing class
@Bean
@ConditionalOnMissingClass("org.postgresql.Driver")
public DataSource defaultDataSource() {
    return new H2DataSource();
}

// Based on bean presence
@Bean
@ConditionalOnBean(DataSource.class)
public JdbcTemplate jdbcTemplate(DataSource dataSource) {
    return new JdbcTemplate(dataSource);
}

// Based on missing bean
@Bean
@ConditionalOnMissingBean(DataSource.class)
public DataSource defaultDataSource() {
    return new H2DataSource();
}

// Based on property
@Bean
@ConditionalOnProperty(name = "cache.enabled", havingValue = "true")
public CacheService cacheService() {
    return new RedisCacheService();
}

@Bean
@ConditionalOnProperty(
    name = "feature.advanced",
    havingValue = "true",
    matchIfMissing = false  // Default when property is missing
)
public AdvancedFeature advancedFeature() {
    return new AdvancedFeature();
}

// Based on resource
@Bean
@ConditionalOnResource(resources = "classpath:schema.sql")
public SchemaInitializer schemaInitializer() {
    return new SchemaInitializer();
}

// Based on expression
@Bean
@ConditionalOnExpression("${cache.enabled:false} and ${cache.type:''} == 'redis'")
public CacheService redisCacheService() {
    return new RedisCacheService();
}

// Based on web application type
@Bean
@ConditionalOnWebApplication
public WebSecurityConfig webSecurityConfig() {
    return new WebSecurityConfig();
}

@Bean
@ConditionalOnNotWebApplication
public ConsoleService consoleService() {
    return new ConsoleService();
}
```

---

## Profile Annotations

### @Profile

Activates beans only when specific profiles are active.

```java
@Configuration
@Profile("development")
public class DevConfig {

    @Bean
    public DataSource dataSource() {
        return new H2DataSource("jdbc:h2:mem:testdb");
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
        return new MysqlDataSource("jdbc:mysql://prod-server:3306/proddb");
    }

    @Bean
    public SmtpEmailService emailService() {
        return new SmtpEmailService();
    }
}

// Multiple profiles
@Configuration
@Profile({"staging", "qa"})
public class TestConfig {
}

// NOT profile (active for all except production)
@Configuration
@Profile("!production")
public class NonProdConfig {

    @Bean
    public DebugService debugService() {
        return new DebugService();
    }
}

// Complex expressions
@Profile("production & region-us")  // production AND region-us

@Profile("dev | test")  // dev OR test

@Profile("!production & !staging")  // NOT production AND NOT staging
```

**Method-level @Profile:**

```java
@Configuration
public class AppConfig {

    @Bean
    @Profile("development")
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

**Activating Profiles:**

```java
// 1. Programmatically
AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
context.getEnvironment().setActiveProfiles("development", "debug");
context.register(AppConfig.class);
context.refresh();

// 2. System property
// -Dspring.profiles.active=production

// 3. Environment variable
// export SPRING_PROFILES_ACTIVE=production

// 4. application.properties
// spring.profiles.active=production

// 5. @ActiveProfiles (Testing)
@SpringBootTest
@ActiveProfiles("test")
public class UserServiceTest {
}
```

---

## Property and Value Annotations

### @ConfigurationProperties

Binds external properties to a POJO.

```java
@Component
@ConfigurationProperties(prefix = "app")
public class AppProperties {

    private String name;
    private String version;
    private Security security = new Security();
    private List<String> servers;
    private Map<String, String> metadata;

    // Getters and setters
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public String getVersion() { return version; }
    public void setVersion(String version) { this.version = version; }

    public Security getSecurity() { return security; }
    public void setSecurity(Security security) { this.security = security; }

    public List<String> getServers() { return servers; }
    public void setServers(List<String> servers) { this.servers = servers; }

    public Map<String, String> getMetadata() { return metadata; }
    public void setMetadata(Map<String, String> metadata) { this.metadata = metadata; }

    public static class Security {
        private boolean enabled;
        private String algorithm;

        public boolean isEnabled() { return enabled; }
        public void setEnabled(boolean enabled) { this.enabled = enabled; }

        public String getAlgorithm() { return algorithm; }
        public void setAlgorithm(String algorithm) { this.algorithm = algorithm; }
    }
}
```

**application.properties:**
```properties
app.name=MyApplication
app.version=1.0.0
app.security.enabled=true
app.security.algorithm=SHA256
app.servers=server1,server2,server3
app.metadata.author=John
app.metadata.department=IT
```

**Enable Configuration Properties:**

```java
@Configuration
@EnableConfigurationProperties(AppProperties.class)
public class AppConfig {

    @Autowired
    private AppProperties appProperties;

    @Bean
    public void displayConfig() {
        System.out.println("App Name: " + appProperties.getName());
        System.out.println("Security Enabled: " + appProperties.getSecurity().isEnabled());
    }
}
```

---

### @Validated

Enables validation on configuration properties.

```java
@Component
@ConfigurationProperties(prefix = "server")
@Validated
public class ServerProperties {

    @NotBlank
    private String host;

    @Min(1)
    @Max(65535)
    private int port;

    @NotEmpty
    private List<String> allowedOrigins;

    // Getters and setters
}
```

---

## AOP Annotations

### @Aspect

Declares a class as an aspect.

```java
@Aspect
@Component
public class LoggingAspect {

    private static final Logger logger = LoggerFactory.getLogger(LoggingAspect.class);

    @Before("execution(* com.example.service.*.*(..))")
    public void logBefore(JoinPoint joinPoint) {
        logger.info("Executing: " + joinPoint.getSignature().getName());
    }

    @AfterReturning(
        pointcut = "execution(* com.example.service.*.*(..))",
        returning = "result"
    )
    public void logAfterReturning(JoinPoint joinPoint, Object result) {
        logger.info("Method {} returned: {}", joinPoint.getSignature().getName(), result);
    }

    @AfterThrowing(
        pointcut = "execution(* com.example.service.*.*(..))",
        throwing = "error"
    )
    public void logAfterThrowing(JoinPoint joinPoint, Throwable error) {
        logger.error("Method {} threw exception: {}",
                    joinPoint.getSignature().getName(),
                    error.getMessage());
    }
}
```

---

### @Before

Advice executed before method execution.

```java
@Aspect
@Component
public class SecurityAspect {

    @Before("@annotation(Secured)")
    public void checkSecurity(JoinPoint joinPoint) {
        // Check security before method execution
        String methodName = joinPoint.getSignature().getName();
        System.out.println("Security check for: " + methodName);

        // Throw exception if unauthorized
        if (!SecurityContext.isAuthorized()) {
            throw new UnauthorizedException("Access denied");
        }
    }
}

// Custom annotation
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Secured {
}

// Usage
@Service
public class UserService {

    @Secured
    public void deleteUser(Long id) {
        // Method implementation
    }
}
```

---

### @After

Advice executed after method execution (regardless of outcome).

```java
@Aspect
@Component
public class AuditAspect {

    @After("execution(* com.example.service.*.save*(..))")
    public void auditSave(JoinPoint joinPoint) {
        System.out.println("Audit: Save operation completed");
        // Log to audit table
    }
}
```

---

### @AfterReturning

Advice executed after successful method execution.

```java
@Aspect
@Component
public class CacheAspect {

    @AfterReturning(
        pointcut = "execution(* com.example.repository.*.findById(..))",
        returning = "result"
    )
    public void cacheResult(JoinPoint joinPoint, Object result) {
        if (result != null) {
            Object[] args = joinPoint.getArgs();
            Long id = (Long) args[0];
            CacheManager.put(id, result);
        }
    }
}
```

---

### @AfterThrowing

Advice executed when method throws an exception.

```java
@Aspect
@Component
public class ExceptionHandlingAspect {

    @AfterThrowing(
        pointcut = "execution(* com.example.service.*.*(..))",
        throwing = "ex"
    )
    public void handleException(JoinPoint joinPoint, Exception ex) {
        String methodName = joinPoint.getSignature().toShortString();
        logger.error("Exception in {}: {}", methodName, ex.getMessage());

        // Send alert
        alertService.sendAlert("Exception in " + methodName);

        // Store in error log
        errorLog.save(new ErrorEntry(methodName, ex));
    }
}
```

---

### @Around

Most powerful advice that surrounds method execution.

```java
@Aspect
@Component
public class PerformanceAspect {

    @Around("@annotation(Timed)")
    public Object measureExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();

        try {
            // Proceed with method execution
            Object result = joinPoint.proceed();
            return result;
        } finally {
            long duration = System.currentTimeMillis() - start;
            System.out.println(joinPoint.getSignature().getName() +
                             " executed in " + duration + "ms");
        }
    }
}

// Custom annotation
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Timed {
}

// Usage
@Service
public class UserService {

    @Timed
    public List<User> getAllUsers() {
        // Method implementation
    }
}
```

**Advanced @Around Example:**

```java
@Aspect
@Component
public class RetryAspect {

    @Around("@annotation(retry)")
    public Object retry(ProceedingJoinPoint joinPoint, Retry retry) throws Throwable {
        int maxAttempts = retry.maxAttempts();
        int attempt = 0;

        while (attempt < maxAttempts) {
            try {
                return joinPoint.proceed();
            } catch (Exception e) {
                attempt++;
                if (attempt >= maxAttempts) {
                    throw e;
                }
                Thread.sleep(retry.delay());
            }
        }
        return null;
    }
}

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Retry {
    int maxAttempts() default 3;
    long delay() default 1000;
}
```

---

### @Pointcut

Defines reusable pointcut expressions.

```java
@Aspect
@Component
public class SystemAspect {

    // Define pointcuts
    @Pointcut("execution(* com.example.service.*.*(..))")
    public void serviceMethods() {}

    @Pointcut("execution(* com.example.repository.*.*(..))")
    public void repositoryMethods() {}

    @Pointcut("serviceMethods() || repositoryMethods()")
    public void allBusinessMethods() {}

    @Pointcut("@annotation(com.example.annotations.Loggable)")
    public void loggableMethods() {}

    // Use pointcuts
    @Before("serviceMethods()")
    public void beforeService() {
        System.out.println("Before service method");
    }

    @Around("allBusinessMethods()")
    public Object aroundBusiness(ProceedingJoinPoint joinPoint) throws Throwable {
        // Logic
        return joinPoint.proceed();
    }

    @After("loggableMethods()")
    public void afterLoggable() {
        System.out.println("After loggable method");
    }
}
```

---

## Transaction Annotations

### @Transactional

Declares transaction boundaries.

```java
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private EmailService emailService;

    // Simple transaction
    @Transactional
    public void createUser(User user) {
        userRepository.save(user);
        emailService.sendWelcomeEmail(user);
    }

    // Read-only transaction (optimization)
    @Transactional(readOnly = true)
    public List<User> getAllUsers() {
        return userRepository.findAll();
    }

    // Custom timeout (in seconds)
    @Transactional(timeout = 30)
    public void processLargeUpdate() {
        // Long running operation
    }

    // Rollback configuration
    @Transactional(rollbackFor = Exception.class)
    public void createOrder(Order order) {
        // Rollback for any exception
    }

    @Transactional(noRollbackFor = ValidationException.class)
    public void updateUser(User user) {
        // Don't rollback for ValidationException
    }

    // Propagation
    @Transactional(propagation = Propagation.REQUIRED)  // Default
    public void methodWithRequired() {
        // Join existing transaction or create new one
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void methodWithRequiresNew() {
        // Always create new transaction, suspend existing
    }

    @Transactional(propagation = Propagation.MANDATORY)
    public void methodWithMandatory() {
        // Must be called within existing transaction
    }

    @Transactional(propagation = Propagation.SUPPORTS)
    public void methodWithSupports() {
        // Join transaction if exists, otherwise execute non-transactionally
    }

    @Transactional(propagation = Propagation.NOT_SUPPORTED)
    public void methodWithNotSupported() {
        // Execute non-transactionally, suspend existing transaction
    }

    @Transactional(propagation = Propagation.NEVER)
    public void methodWithNever() {
        // Throw exception if transaction exists
    }

    // Isolation levels
    @Transactional(isolation = Isolation.READ_COMMITTED)
    public void methodWithReadCommitted() {
        // Prevents dirty reads
    }

    @Transactional(isolation = Isolation.REPEATABLE_READ)
    public void methodWithRepeatableRead() {
        // Prevents dirty and non-repeatable reads
    }

    @Transactional(isolation = Isolation.SERIALIZABLE)
    public void methodWithSerializable() {
        // Highest isolation, prevents phantom reads
    }

    // Complete example
    @Transactional(
        propagation = Propagation.REQUIRED,
        isolation = Isolation.READ_COMMITTED,
        timeout = 30,
        readOnly = false,
        rollbackFor = {Exception.class},
        noRollbackFor = {ValidationException.class}
    )
    public void complexTransaction() {
        // Transaction configuration
    }
}
```

**Enable Transaction Management:**

```java
@Configuration
@EnableTransactionManagement
public class AppConfig {

    @Bean
    public PlatformTransactionManager transactionManager(DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
}
```

---

## Validation Annotations

### JSR-303/JSR-380 Bean Validation

```java
public class User {

    @NotNull(message = "ID cannot be null")
    private Long id;

    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 50, message = "Name must be between 2 and 50 characters")
    private String name;

    @Email(message = "Invalid email format")
    @NotBlank
    private String email;

    @Min(value = 18, message = "Age must be at least 18")
    @Max(value = 120, message = "Age must be less than 120")
    private Integer age;

    @Pattern(regexp = "^\\+?[1-9]\\d{1,14}$", message = "Invalid phone number")
    private String phoneNumber;

    @Past(message = "Birth date must be in the past")
    private LocalDate birthDate;

    @Future(message = "Expiry date must be in the future")
    private LocalDate expiryDate;

    @DecimalMin(value = "0.0", message = "Balance cannot be negative")
    @DecimalMax(value = "1000000.0", message = "Balance too large")
    private BigDecimal balance;

    @Positive
    private Integer score;

    @PositiveOrZero
    private Integer attempts;

    @Negative
    private Integer deficit;

    @NegativeOrZero
    private Integer penaltyPoints;

    @Size(min = 1, max = 10)
    private List<String> roles;

    @NotEmpty
    private Set<String> permissions;

    @Valid  // Cascade validation
    private Address address;

    // Getters and setters
}

public class Address {

    @NotBlank
    private String street;

    @NotBlank
    private String city;

    @Size(min = 5, max = 10)
    private String zipCode;

    // Getters and setters
}
```

**Using Validation in Controller:**

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @PostMapping
    public ResponseEntity<User> createUser(@Valid @RequestBody User user) {
        // @Valid triggers validation
        // BindingResult captures errors
        return ResponseEntity.ok(user);
    }

    @PostMapping("/with-errors")
    public ResponseEntity<?> createUserWithErrorHandling(
            @Valid @RequestBody User user,
            BindingResult bindingResult) {

        if (bindingResult.hasErrors()) {
            Map<String, String> errors = new HashMap<>();
            bindingResult.getFieldErrors().forEach(error ->
                errors.put(error.getField(), error.getDefaultMessage())
            );
            return ResponseEntity.badRequest().body(errors);
        }

        return ResponseEntity.ok(user);
    }
}
```

**Custom Validation Annotation:**

```java
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = UniqueEmailValidator.class)
public @interface UniqueEmail {
    String message() default "Email already exists";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

// Validator implementation
public class UniqueEmailValidator implements ConstraintValidator<UniqueEmail, String> {

    @Autowired
    private UserRepository userRepository;

    @Override
    public boolean isValid(String email, ConstraintValidatorContext context) {
        if (email == null) {
            return true;
        }
        return !userRepository.existsByEmail(email);
    }
}

// Usage
public class User {

    @UniqueEmail
    @Email
    private String email;
}
```

---

## Spring Web Annotations

### @RequestMapping

Maps HTTP requests to handler methods.

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    // GET /api/users
    @RequestMapping(method = RequestMethod.GET)
    public List<User> getAllUsers() {
        return userService.getAllUsers();
    }

    // GET /api/users/123
    @RequestMapping(value = "/{id}", method = RequestMethod.GET)
    public User getUserById(@PathVariable Long id) {
        return userService.getUserById(id);
    }

    // POST /api/users
    @RequestMapping(method = RequestMethod.POST, consumes = "application/json")
    public User createUser(@RequestBody User user) {
        return userService.createUser(user);
    }

    // Multiple paths
    @RequestMapping(value = {"/find", "/search"}, method = RequestMethod.GET)
    public List<User> searchUsers(@RequestParam String query) {
        return userService.search(query);
    }

    // Multiple methods
    @RequestMapping(value = "/{id}", method = {RequestMethod.PUT, RequestMethod.PATCH})
    public User updateUser(@PathVariable Long id, @RequestBody User user) {
        return userService.updateUser(id, user);
    }

    // Headers
    @RequestMapping(
        value = "/premium",
        method = RequestMethod.GET,
        headers = "X-API-Version=v2"
    )
    public List<User> getPremiumUsers() {
        return userService.getPremiumUsers();
    }

    // Params
    @RequestMapping(
        value = "/active",
        method = RequestMethod.GET,
        params = "status=active"
    )
    public List<User> getActiveUsers() {
        return userService.getActiveUsers();
    }

    // Produces
    @RequestMapping(
        value = "/export",
        method = RequestMethod.GET,
        produces = {"application/json", "application/xml"}
    )
    public List<User> exportUsers() {
        return userService.getAllUsers();
    }
}
```

---

### @GetMapping / @PostMapping / @PutMapping / @DeleteMapping / @PatchMapping

Shortcuts for @RequestMapping with specific HTTP methods.

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {

    @GetMapping
    public List<Product> getAllProducts() {
        return productService.findAll();
    }

    @GetMapping("/{id}")
    public ResponseEntity<Product> getProduct(@PathVariable Long id) {
        return productService.findById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }

    @PostMapping
    public ResponseEntity<Product> createProduct(@RequestBody Product product) {
        Product created = productService.save(product);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }

    @PutMapping("/{id}")
    public ResponseEntity<Product> updateProduct(
            @PathVariable Long id,
            @RequestBody Product product) {
        Product updated = productService.update(id, product);
        return ResponseEntity.ok(updated);
    }

    @PatchMapping("/{id}")
    public ResponseEntity<Product> partialUpdate(
            @PathVariable Long id,
            @RequestBody Map<String, Object> updates) {
        Product updated = productService.partialUpdate(id, updates);
        return ResponseEntity.ok(updated);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteProduct(@PathVariable Long id) {
        productService.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

---

### @PathVariable

Extracts values from URI path.

```java
@RestController
@RequestMapping("/api")
public class ApiController {

    // Simple path variable
    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findById(id);
    }

    // Multiple path variables
    @GetMapping("/users/{userId}/orders/{orderId}")
    public Order getUserOrder(
            @PathVariable Long userId,
            @PathVariable Long orderId) {
        return orderService.findByUserAndId(userId, orderId);
    }

    // Custom name
    @GetMapping("/users/{user-id}")
    public User getUser(@PathVariable("user-id") Long id) {
        return userService.findById(id);
    }

    // Optional path variable
    @GetMapping({"/users", "/users/{id}"})
    public ResponseEntity<?> getUsers(
            @PathVariable(required = false) Long id) {
        if (id != null) {
            return ResponseEntity.ok(userService.findById(id));
        }
        return ResponseEntity.ok(userService.findAll());
    }

    // Map of all path variables
    @GetMapping("/resources/{category}/{subcategory}")
    public String getResource(@PathVariable Map<String, String> pathVars) {
        return "Category: " + pathVars.get("category") +
               ", Subcategory: " + pathVars.get("subcategory");
    }

    // Regular expression constraint
    @GetMapping("/files/{filename:.+}")  // Allows dots in filename
    public Resource getFile(@PathVariable String filename) {
        return resourceService.loadFile(filename);
    }
}
```

---

### @RequestParam

Extracts query parameters.

```java
@RestController
@RequestMapping("/api/search")
public class SearchController {

    // Simple request param
    // GET /api/search?query=spring
    @GetMapping
    public List<Result> search(@RequestParam String query) {
        return searchService.search(query);
    }

    // Multiple params
    // GET /api/search?query=spring&page=1&size=10
    @GetMapping("/advanced")
    public Page<Result> advancedSearch(
            @RequestParam String query,
            @RequestParam int page,
            @RequestParam int size) {
        return searchService.search(query, page, size);
    }

    // Optional param with default value
    @GetMapping("/users")
    public List<User> getUsers(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "20") int size,
            @RequestParam(defaultValue = "name") String sortBy) {
        return userService.findAll(page, size, sortBy);
    }

    // Optional param
    @GetMapping("/products")
    public List<Product> getProducts(
            @RequestParam(required = false) String category) {
        if (category != null) {
            return productService.findByCategory(category);
        }
        return productService.findAll();
    }

    // Custom parameter name
    @GetMapping("/items")
    public List<Item> getItems(
            @RequestParam("cat") String category) {
        return itemService.findByCategory(category);
    }

    // List parameter
    // GET /api/users?ids=1,2,3 or ?ids=1&ids=2&ids=3
    @GetMapping("/users/batch")
    public List<User> getUsersByIds(@RequestParam List<Long> ids) {
        return userService.findByIds(ids);
    }

    // Map of all parameters
    @GetMapping("/filter")
    public List<Product> filterProducts(
            @RequestParam Map<String, String> filters) {
        return productService.filter(filters);
    }
}
```

---

### @RequestBody

Binds HTTP request body to method parameter.

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    // Simple request body
    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody User user) {
        User created = userService.save(user);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }

    // With validation
    @PostMapping("/validated")
    public ResponseEntity<User> createValidatedUser(
            @Valid @RequestBody User user) {
        User created = userService.save(user);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }

    // Optional request body
    @PostMapping("/optional")
    public ResponseEntity<User> createOptionalUser(
            @RequestBody(required = false) User user) {
        if (user == null) {
            user = new User("Default", "default@example.com");
        }
        User created = userService.save(user);
        return ResponseEntity.ok(created);
    }

    // Map as request body
    @PostMapping("/dynamic")
    public ResponseEntity<?> processDynamicData(
            @RequestBody Map<String, Object> data) {
        // Process dynamic JSON data
        return ResponseEntity.ok(data);
    }
}
```

---

### @ResponseBody

Indicates method return value should be written to response body.

```java
@Controller
@RequestMapping("/api")
public class ApiController {

    @GetMapping("/users")
    @ResponseBody  // Serialize list to JSON
    public List<User> getUsers() {
        return userService.findAll();
    }

    @GetMapping("/user/{id}")
    @ResponseBody
    public User getUser(@PathVariable Long id) {
        return userService.findById(id);
    }
}

// @RestController combines @Controller and @ResponseBody
@RestController  // All methods have @ResponseBody by default
@RequestMapping("/api")
public class RestApiController {

    @GetMapping("/users")
    public List<User> getUsers() {  // No need for @ResponseBody
        return userService.findAll();
    }
}
```

---

### @ResponseStatus

Sets HTTP status code for response.

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)  // 201
    public User createUser(@RequestBody User user) {
        return userService.save(user);
    }

    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)  // 204
    public void deleteUser(@PathVariable Long id) {
        userService.delete(id);
    }

    @GetMapping("/count")
    @ResponseStatus(HttpStatus.OK)  // 200 (default)
    public long getUserCount() {
        return userService.count();
    }
}

// Exception with status
@ResponseStatus(HttpStatus.NOT_FOUND)
public class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(String message) {
        super(message);
    }
}
```

---

### @ExceptionHandler

Handles exceptions at controller level.

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findById(id)
            .orElseThrow(() -> new UserNotFoundException("User not found: " + id));
    }

    // Handle specific exception
    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleUserNotFound(UserNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage(),
            System.currentTimeMillis()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    // Handle multiple exceptions
    @ExceptionHandler({ValidationException.class, IllegalArgumentException.class})
    public ResponseEntity<ErrorResponse> handleValidation(Exception ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.BAD_REQUEST.value(),
            ex.getMessage(),
            System.currentTimeMillis()
        );
        return ResponseEntity.badRequest().body(error);
    }

    // Handle all exceptions
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneral(Exception ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "An error occurred: " + ex.getMessage(),
            System.currentTimeMillis()
        );
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}
```

---

### @ControllerAdvice / @RestControllerAdvice

Global exception handling.

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleUserNotFound(UserNotFoundException ex) {
        return new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage(),
            System.currentTimeMillis()
        );
    }

    @ExceptionHandler(ValidationException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidation(ValidationException ex) {
        return new ErrorResponse(
            HttpStatus.BAD_REQUEST.value(),
            ex.getMessage(),
            System.currentTimeMillis()
        );
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public Map<String, String> handleValidationErrors(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error ->
            errors.put(error.getField(), error.getDefaultMessage())
        );
        return errors;
    }

    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleGeneral(Exception ex) {
        return new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "Internal server error",
            System.currentTimeMillis()
        );
    }
}

// Target specific controllers
@ControllerAdvice(assignableTypes = {UserController.class, ProductController.class})
public class SpecificExceptionHandler {
    // Exception handlers
}

// Target specific packages
@ControllerAdvice(basePackages = "com.example.admin")
public class AdminExceptionHandler {
    // Exception handlers
}

// Target annotations
@ControllerAdvice(annotations = RestController.class)
public class RestExceptionHandler {
    // Exception handlers
}
```

---

### @CrossOrigin

Configures CORS for controllers or methods.

```java
// Method level
@RestController
@RequestMapping("/api/users")
public class UserController {

    @CrossOrigin(origins = "http://localhost:3000")
    @GetMapping
    public List<User> getAllUsers() {
        return userService.findAll();
    }
}

// Controller level
@RestController
@RequestMapping("/api/products")
@CrossOrigin(
    origins = {"http://localhost:3000", "https://example.com"},
    methods = {RequestMethod.GET, RequestMethod.POST},
    maxAge = 3600
)
public class ProductController {

    @GetMapping
    public List<Product> getAllProducts() {
        return productService.findAll();
    }

    @PostMapping
    public Product createProduct(@RequestBody Product product) {
        return productService.save(product);
    }
}

// Allow all origins
@CrossOrigin(origins = "*")
@GetMapping("/public")
public String publicEndpoint() {
    return "Public data";
}

// Global CORS configuration
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
            .allowedOrigins("http://localhost:3000")
            .allowedMethods("GET", "POST", "PUT", "DELETE")
            .allowedHeaders("*")
            .allowCredentials(true)
            .maxAge(3600);
    }
}
```

---

## Spring Boot Annotations

### @SpringBootApplication

Combines @Configuration, @EnableAutoConfiguration, and @ComponentScan.

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

// Equivalent to:
@Configuration
@EnableAutoConfiguration
@ComponentScan
public class Application {
    // ...
}

// Exclude auto-configurations
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
public class Application {
    // ...
}

// Custom component scan
@SpringBootApplication(scanBasePackages = "com.example")
public class Application {
    // ...
}
```

---

### @EnableAutoConfiguration

Enables Spring Boot's auto-configuration mechanism.

```java
@Configuration
@EnableAutoConfiguration
public class AppConfig {
    // Spring Boot will auto-configure based on classpath
}

// Exclude specific auto-configurations
@Configuration
@EnableAutoConfiguration(exclude = {
    DataSourceAutoConfiguration.class,
    SecurityAutoConfiguration.class
})
public class AppConfig {
}
```

---

### @ConfigurationPropertiesScan

Scans for @ConfigurationProperties classes.

```java
@SpringBootApplication
@ConfigurationPropertiesScan("com.example.config")
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

---

## Advanced Annotations

### @DependsOn

Specifies bean initialization order.

```java
@Configuration
public class AppConfig {

    @Bean
    public CacheManager cacheManager() {
        return new CacheManager();
    }

    @Bean
    @DependsOn("cacheManager")  // Ensure cacheManager is created first
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

---

### @Lookup

Method injection for getting new instances.

```java
@Component
public abstract class CommandProcessor {

    public void processCommand() {
        Command command = createCommand();  // Gets new instance
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
}
```

---

### @Async

Enables asynchronous method execution.

```java
@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        executor.initialize();
        return executor;
    }
}

@Service
public class EmailService {

    @Async
    public void sendEmail(String to, String subject, String body) {
        // Executes asynchronously
        System.out.println("Sending email on thread: " +
                         Thread.currentThread().getName());
        // Send email logic
    }

    @Async
    public CompletableFuture<String> sendEmailWithResult(String to) {
        // Asynchronous with return value
        String result = "Email sent to " + to;
        return CompletableFuture.completedFuture(result);
    }
}
```

---

### @Scheduled

Schedules method execution.

```java
@Configuration
@EnableScheduling
public class SchedulingConfig {
}

@Component
public class ScheduledTasks {

    // Fixed rate (milliseconds)
    @Scheduled(fixedRate = 5000)
    public void executeEvery5Seconds() {
        System.out.println("Executed every 5 seconds");
    }

    // Fixed delay
    @Scheduled(fixedDelay = 5000)
    public void executeWithDelay() {
        System.out.println("Executed 5 seconds after previous completion");
    }

    // Initial delay
    @Scheduled(initialDelay = 10000, fixedRate = 5000)
    public void executeWithInitialDelay() {
        System.out.println("First execution after 10 seconds");
    }

    // Cron expression
    @Scheduled(cron = "0 0 * * * *")  // Every hour
    public void executeEveryHour() {
        System.out.println("Executed every hour");
    }

    @Scheduled(cron = "0 0 8 * * MON-FRI")  // 8 AM on weekdays
    public void executeWeekdayMorning() {
        System.out.println("Good morning!");
    }

    @Scheduled(cron = "0 0/15 * * * *")  // Every 15 minutes
    public void executeEvery15Minutes() {
        System.out.println("Executed every 15 minutes");
    }

    // From properties
    @Scheduled(cron = "${cron.expression}")
    public void executeFromConfig() {
        System.out.println("Executed based on config");
    }
}
```

---

### @EventListener

Handles application events.

```java
// Custom event
public class UserCreatedEvent extends ApplicationEvent {
    private final User user;

    public UserCreatedEvent(Object source, User user) {
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

    @Autowired
    private ApplicationEventPublisher eventPublisher;

    public User createUser(User user) {
        User saved = userRepository.save(user);

        // Publish event
        eventPublisher.publishEvent(new UserCreatedEvent(this, saved));

        return saved;
    }
}

// Event listener
@Component
public class UserEventListener {

    @EventListener
    public void handleUserCreated(UserCreatedEvent event) {
        User user = event.getUser();
        System.out.println("User created: " + user.getName());
        // Send welcome email, log audit, etc.
    }

    // Conditional event handling
    @EventListener(condition = "#event.user.premium == true")
    public void handlePremiumUserCreated(UserCreatedEvent event) {
        System.out.println("Premium user created!");
    }

    // Multiple events
    @EventListener({UserCreatedEvent.class, UserUpdatedEvent.class})
    public void handleUserEvents(ApplicationEvent event) {
        System.out.println("User event occurred");
    }

    // Async event listener
    @Async
    @EventListener
    public void handleUserCreatedAsync(UserCreatedEvent event) {
        // Processed asynchronously
    }

    // Transaction event
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    public void handleAfterCommit(UserCreatedEvent event) {
        // Executed after transaction commits
    }
}
```

---

## Conclusion

This comprehensive guide covers all major Spring Framework annotations with practical examples. Understanding these annotations is crucial for modern Spring development.

### Key Takeaways

1. **Stereotype Annotations** - @Component, @Service, @Repository, @Controller
2. **Configuration** - @Configuration, @Bean, @ComponentScan
3. **Dependency Injection** - @Autowired, @Qualifier, @Value
4. **Lifecycle** - @PostConstruct, @PreDestroy
5. **Scope** - @Scope, @Lazy
6. **Web** - @RestController, @RequestMapping, @PathVariable
7. **AOP** - @Aspect, @Before, @After, @Around
8. **Transactions** - @Transactional
9. **Validation** - @Valid, @NotNull, @Size
10. **Spring Boot** - @SpringBootApplication, @EnableAutoConfiguration

### Best Practices

- Use constructor injection with @Autowired
- Prefer @RestController over @Controller for REST APIs
- Use @Validated for method-level validation
- Apply @Transactional at service layer
- Use @Async for non-blocking operations
- Leverage @EventListener for loose coupling
- Apply @ControllerAdvice for global exception handling

---

## Additional Resources

- Spring Framework Documentation: https://docs.spring.io/spring-framework/reference/
- Spring Boot Documentation: https://docs.spring.io/spring-boot/docs/current/reference/html/
- Spring Annotations Guide
- JSR-303 Bean Validation
- AspectJ Documentation
