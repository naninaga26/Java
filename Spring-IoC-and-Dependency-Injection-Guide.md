# Spring IoC and Dependency Injection - Complete Guide

## Table of Contents
1. [Introduction to IoC](#introduction-to-ioc)
2. [Understanding Dependency Injection](#understanding-dependency-injection)
3. [Spring IoC Container](#spring-ioc-container)
4. [Types of Dependency Injection](#types-of-dependency-injection)
5. [IoC Container Types](#ioc-container-types)
6. [Bean Lifecycle in IoC Container](#bean-lifecycle-in-ioc-container)
7. [Circular Dependencies](#circular-dependencies)
8. [Real-World Examples](#real-world-examples)
9. [Best Practices](#best-practices)
10. [IoC vs Service Locator Pattern](#ioc-vs-service-locator-pattern)

---

## Introduction to IoC

### What is Inversion of Control (IoC)?

**Inversion of Control (IoC)** is a design principle where the control of object creation and management is transferred from the application code to a framework or container.

### Traditional Approach (Without IoC)

```java
public class UserService {

    // Tightly coupled - UserService creates its own dependencies
    private UserRepository userRepository = new UserRepository();
    private EmailService emailService = new EmailService();

    public void registerUser(User user) {
        userRepository.save(user);
        emailService.sendWelcomeEmail(user);
    }
}

public class Application {
    public static void main(String[] args) {
        // Application controls object creation
        UserService userService = new UserService();
        User user = new User("John", "john@example.com");
        userService.registerUser(user);
    }
}
```

**Problems:**
- **Tight Coupling**: UserService is tightly coupled to specific implementations
- **Hard to Test**: Cannot easily mock dependencies for testing
- **Difficult to Change**: Changing dependencies requires code modification
- **No Flexibility**: Cannot switch implementations at runtime

### IoC Approach (With Spring)

```java
public class UserService {

    // Dependencies are injected by Spring IoC container
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
    public UserService userService(UserRepository userRepository,
                                   EmailService emailService) {
        return new UserService(userRepository, emailService);
    }
}

public class Application {
    public static void main(String[] args) {
        // Spring IoC container controls object creation
        ApplicationContext context =
            new AnnotationConfigApplicationContext(AppConfig.class);

        UserService userService = context.getBean(UserService.class);
        User user = new User("John", "john@example.com");
        userService.registerUser(user);
    }
}
```

**Benefits:**
- **Loose Coupling**: Dependencies are injected, not created
- **Testability**: Easy to inject mock objects for testing
- **Flexibility**: Can switch implementations via configuration
- **Maintainability**: Changes don't affect dependent classes

### The Inversion

```
Traditional Flow:
    Application → Creates Objects → Uses Objects

IoC Flow:
    Application → Requests Objects → IoC Container → Creates & Injects Objects
```

**What Gets Inverted?**
- **Object Creation**: From application code to IoC container
- **Dependency Management**: From manual to automatic
- **Lifecycle Management**: From developer to framework
- **Configuration**: From code to external configuration

---

## Understanding Dependency Injection

### What is Dependency Injection?

**Dependency Injection (DI)** is a design pattern that implements IoC, where dependencies are "injected" into an object rather than the object creating them.

### Without Dependency Injection

```java
public class OrderService {

    // Creating dependencies internally
    private PaymentGateway paymentGateway;
    private InventoryService inventoryService;
    private NotificationService notificationService;

    public OrderService() {
        // Tight coupling to specific implementations
        this.paymentGateway = new StripePaymentGateway();
        this.inventoryService = new InventoryService();
        this.notificationService = new EmailNotificationService();
    }

    public void placeOrder(Order order) {
        if (inventoryService.checkAvailability(order)) {
            paymentGateway.charge(order.getTotal());
            notificationService.sendConfirmation(order);
        }
    }
}

// Testing is difficult
public class OrderServiceTest {
    @Test
    public void testPlaceOrder() {
        OrderService service = new OrderService();
        // Cannot mock dependencies - will use real implementations
        // This means:
        // - Real payment will be charged
        // - Real inventory will be checked
        // - Real email will be sent
    }
}
```

### With Dependency Injection

```java
public class OrderService {

    // Dependencies declared, not created
    private final PaymentGateway paymentGateway;
    private final InventoryService inventoryService;
    private final NotificationService notificationService;

    // Dependencies injected via constructor
    public OrderService(PaymentGateway paymentGateway,
                       InventoryService inventoryService,
                       NotificationService notificationService) {
        this.paymentGateway = paymentGateway;
        this.inventoryService = inventoryService;
        this.notificationService = notificationService;
    }

    public void placeOrder(Order order) {
        if (inventoryService.checkAvailability(order)) {
            paymentGateway.charge(order.getTotal());
            notificationService.sendConfirmation(order);
        }
    }
}

// Testing is easy
public class OrderServiceTest {
    @Test
    public void testPlaceOrder() {
        // Create mock dependencies
        PaymentGateway mockPayment = mock(PaymentGateway.class);
        InventoryService mockInventory = mock(InventoryService.class);
        NotificationService mockNotification = mock(NotificationService.class);

        // Inject mocks
        OrderService service = new OrderService(
            mockPayment, mockInventory, mockNotification
        );

        // Test with mocks - no real charges or emails
        when(mockInventory.checkAvailability(any())).thenReturn(true);

        Order order = new Order();
        service.placeOrder(order);

        verify(mockPayment).charge(order.getTotal());
    }
}
```

### DI Principles

1. **Dependency**: Object A needs Object B to function
2. **Injection**: Object B is provided (injected) to Object A
3. **Inversion**: Control of creating B is inverted from A to external source

---

## Spring IoC Container

The Spring IoC container is the core of the Spring Framework. It creates objects, wires them together, configures them, and manages their complete lifecycle.

### Container Responsibilities

```java
/*
 * Spring IoC Container Responsibilities:
 *
 * 1. Bean Instantiation
 *    - Creates objects based on configuration
 *    - Manages object lifecycle
 *
 * 2. Dependency Resolution
 *    - Identifies dependencies between beans
 *    - Resolves dependency graph
 *
 * 3. Dependency Injection
 *    - Injects dependencies into beans
 *    - Handles constructor, setter, and field injection
 *
 * 4. Configuration Management
 *    - Reads configuration (XML, Java, Annotations)
 *    - Applies configuration to beans
 *
 * 5. Lifecycle Management
 *    - Calls initialization methods
 *    - Manages bean scope
 *    - Calls destruction methods
 *
 * 6. AOP Proxy Creation
 *    - Creates proxies for AOP
 *    - Applies cross-cutting concerns
 */
```

### How IoC Container Works

```java
/**
 * Step-by-step: How Spring IoC Container Works
 */

// Step 1: Define Beans
@Component
public class UserRepository {
    public void save(User user) {
        System.out.println("Saving user: " + user.getName());
    }
}

@Component
public class EmailService {
    public void sendEmail(String to, String subject) {
        System.out.println("Sending email to: " + to);
    }
}

@Component
public class UserService {
    private final UserRepository userRepository;
    private final EmailService emailService;

    @Autowired
    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
        System.out.println("UserService created");
    }

    public void registerUser(User user) {
        userRepository.save(user);
        emailService.sendEmail(user.getEmail(), "Welcome!");
    }
}

// Step 2: Configure Component Scanning
@Configuration
@ComponentScan(basePackages = "com.example")
public class AppConfig {
}

// Step 3: Bootstrap IoC Container
public class Application {
    public static void main(String[] args) {

        // Phase 1: Container Initialization
        System.out.println("--- Phase 1: Initializing Container ---");
        ApplicationContext context =
            new AnnotationConfigApplicationContext(AppConfig.class);

        /*
         * What happens internally:
         *
         * 1. Container scans for @Component, @Service, @Repository
         * 2. Registers bean definitions
         * 3. Analyzes dependencies (UserService needs UserRepository, EmailService)
         * 4. Creates beans in correct order:
         *    - UserRepository (no dependencies)
         *    - EmailService (no dependencies)
         *    - UserService (depends on above two)
         * 5. Injects dependencies
         * 6. Calls initialization methods
         */

        // Phase 2: Using Beans
        System.out.println("\n--- Phase 2: Using Beans ---");
        UserService userService = context.getBean(UserService.class);

        User user = new User("John Doe", "john@example.com");
        userService.registerUser(user);

        // Phase 3: Container Shutdown
        System.out.println("\n--- Phase 3: Shutting Down ---");
        ((AnnotationConfigApplicationContext) context).close();

        /*
         * Output:
         * --- Phase 1: Initializing Container ---
         * UserService created
         *
         * --- Phase 2: Using Beans ---
         * Saving user: John Doe
         * Sending email to: john@example.com
         *
         * --- Phase 3: Shutting Down ---
         */
    }
}
```

### Container Configuration Metadata

Spring IoC container can be configured in three ways:

#### 1. XML Configuration

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="userRepository" class="com.example.UserRepository"/>

    <bean id="emailService" class="com.example.EmailService"/>

    <bean id="userService" class="com.example.UserService">
        <constructor-arg ref="userRepository"/>
        <constructor-arg ref="emailService"/>
    </bean>
</beans>
```

#### 2. Java Configuration

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
    public UserService userService() {
        return new UserService(userRepository(), emailService());
    }
}
```

#### 3. Annotation-Based Configuration

```java
@Component
public class UserRepository { }

@Component
public class EmailService { }

@Component
public class UserService {

    @Autowired
    public UserService(UserRepository userRepository, EmailService emailService) {
        // Dependencies automatically injected
    }
}

@Configuration
@ComponentScan(basePackages = "com.example")
public class AppConfig { }
```

---

## Types of Dependency Injection

Spring supports three types of dependency injection:

### 1. Constructor Injection (Recommended)

Dependencies are provided through class constructor.

```java
/**
 * Constructor Injection Example
 */
@Service
public class OrderService {

    private final PaymentService paymentService;
    private final InventoryService inventoryService;
    private final EmailService emailService;

    // Single constructor - @Autowired is optional (Spring 4.3+)
    @Autowired
    public OrderService(PaymentService paymentService,
                       InventoryService inventoryService,
                       EmailService emailService) {
        this.paymentService = paymentService;
        this.inventoryService = inventoryService;
        this.emailService = emailService;
    }

    public void placeOrder(Order order) {
        if (inventoryService.checkStock(order)) {
            paymentService.processPayment(order);
            emailService.sendConfirmation(order);
        }
    }
}

// Configuration
@Configuration
@ComponentScan(basePackages = "com.example")
public class AppConfig {
}

// Dependencies
@Service
public class PaymentService {
    public void processPayment(Order order) {
        System.out.println("Processing payment: $" + order.getTotal());
    }
}

@Service
public class InventoryService {
    public boolean checkStock(Order order) {
        System.out.println("Checking inventory");
        return true;
    }
}

@Service
public class EmailService {
    public void sendConfirmation(Order order) {
        System.out.println("Sending confirmation email");
    }
}
```

**Advantages:**
- **Immutability**: Dependencies can be final
- **Mandatory Dependencies**: Constructor ensures all dependencies are provided
- **Testability**: Easy to test - just call constructor with mocks
- **Clear Dependencies**: All dependencies visible in constructor
- **Thread Safety**: Immutable objects are thread-safe

**Best For:**
- Required dependencies
- Immutable objects
- Clear dependency declaration

### 2. Setter Injection

Dependencies are provided through setter methods.

```java
/**
 * Setter Injection Example
 */
@Service
public class ReportService {

    private DataSource dataSource;
    private TemplateEngine templateEngine;
    private EmailService emailService; // Optional dependency

    // Required dependency
    @Autowired
    public void setDataSource(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    // Required dependency
    @Autowired
    public void setTemplateEngine(TemplateEngine templateEngine) {
        this.templateEngine = templateEngine;
    }

    // Optional dependency
    @Autowired(required = false)
    public void setEmailService(EmailService emailService) {
        this.emailService = emailService;
    }

    public Report generateReport(String type) {
        Data data = dataSource.fetchData(type);
        Report report = templateEngine.generate(data);

        // Use optional dependency if available
        if (emailService != null) {
            emailService.sendReport(report);
        }

        return report;
    }
}

// Java Config example
@Configuration
public class AppConfig {

    @Bean
    public ReportService reportService() {
        ReportService service = new ReportService();
        service.setDataSource(dataSource());
        service.setTemplateEngine(templateEngine());
        // emailService is optional
        return service;
    }

    @Bean
    public DataSource dataSource() {
        return new DataSource();
    }

    @Bean
    public TemplateEngine templateEngine() {
        return new TemplateEngine();
    }
}
```

**Advantages:**
- **Optional Dependencies**: Can mark dependencies as optional
- **Reconfiguration**: Can change dependencies after object creation
- **Circular Dependencies**: Can resolve some circular dependency issues
- **Default Values**: Can provide defaults in the class

**Disadvantages:**
- **Mutability**: Object state can change
- **Incomplete Objects**: Object may be in invalid state if setters not called
- **Less Clear**: Dependencies not obvious from class declaration

**Best For:**
- Optional dependencies
- Reconfigurable objects
- Legacy code integration

### 3. Field Injection

Dependencies are injected directly into fields.

```java
/**
 * Field Injection Example
 */
@Service
public class NotificationService {

    @Autowired
    private EmailSender emailSender;

    @Autowired
    private SmsSender smsSender;

    @Autowired
    private PushNotificationSender pushSender;

    public void sendNotification(User user, String message) {
        emailSender.send(user.getEmail(), message);
        smsSender.send(user.getPhone(), message);
        pushSender.send(user.getDeviceId(), message);
    }
}

// With @Qualifier
@Service
public class PaymentService {

    @Autowired
    @Qualifier("stripeGateway")
    private PaymentGateway paymentGateway;

    public void processPayment(Payment payment) {
        paymentGateway.charge(payment);
    }
}
```

**Advantages:**
- **Concise**: Less boilerplate code
- **Convenience**: Quick to write

**Disadvantages:**
- **Cannot be Final**: Fields cannot be final
- **Hard to Test**: Cannot easily inject mocks (need reflection or Spring test context)
- **Hidden Dependencies**: Dependencies not visible in class API
- **Framework Coupling**: Tied to Spring framework
- **Immutability Issues**: Cannot create immutable objects

**Not Recommended** except for:
- Quick prototypes
- Testing (with @SpringBootTest)
- Simple applications

### Comparison

```java
/**
 * Comparing Injection Types
 */

// 1. CONSTRUCTOR INJECTION (RECOMMENDED)
@Service
public class UserService {
    private final UserRepository userRepository; // Can be final
    private final EmailService emailService;     // Can be final

    @Autowired
    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }
}

// Easy to test
@Test
public void testUserService() {
    UserRepository mockRepo = mock(UserRepository.class);
    EmailService mockEmail = mock(EmailService.class);
    UserService service = new UserService(mockRepo, mockEmail);
    // Test service
}

// 2. SETTER INJECTION
@Service
public class ProductService {
    private ProductRepository productRepository; // Cannot be final
    private CacheService cacheService;           // Cannot be final

    @Autowired
    public void setProductRepository(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }

    @Autowired(required = false)
    public void setCacheService(CacheService cacheService) {
        this.cacheService = cacheService;
    }
}

// Testing requires setter calls
@Test
public void testProductService() {
    ProductRepository mockRepo = mock(ProductRepository.class);
    ProductService service = new ProductService();
    service.setProductRepository(mockRepo);
    // Test service
}

// 3. FIELD INJECTION (NOT RECOMMENDED)
@Service
public class OrderService {
    @Autowired
    private OrderRepository orderRepository; // Cannot be final

    @Autowired
    private PaymentService paymentService;   // Cannot be final
}

// Testing is difficult
@Test
public void testOrderService() {
    OrderRepository mockRepo = mock(OrderRepository.class);
    OrderService service = new OrderService();
    // Cannot inject mock without reflection or Spring context
    // ReflectionTestUtils.setField(service, "orderRepository", mockRepo);
}
```

---

## IoC Container Types

Spring provides two types of IoC containers:

### 1. BeanFactory

Basic IoC container with fundamental features.

```java
/**
 * BeanFactory - Lazy initialization
 */
public class BeanFactoryExample {

    public static void main(String[] args) {

        // Create BeanFactory
        Resource resource = new ClassPathResource("beans.xml");
        BeanFactory factory = new XmlBeanFactory(resource);

        System.out.println("BeanFactory created");
        // Beans are NOT created yet (lazy initialization)

        // Get bean - created on first request
        UserService userService = (UserService) factory.getBean("userService");
        System.out.println("Bean retrieved");

        // Get same bean again - returns same instance (singleton)
        UserService userService2 = (UserService) factory.getBean("userService");
        System.out.println(userService == userService2); // true
    }
}
```

**Characteristics:**
- **Lazy Initialization**: Beans created on first request
- **Basic Features**: Core DI functionality
- **Lightweight**: Minimal memory footprint
- **Manual Configuration**: Requires explicit configuration

**Methods:**
```java
public interface BeanFactory {

    Object getBean(String name);

    <T> T getBean(String name, Class<T> requiredType);

    <T> T getBean(Class<T> requiredType);

    boolean containsBean(String name);

    boolean isSingleton(String name);

    boolean isPrototype(String name);

    Class<?> getType(String name);

    String[] getAliases(String name);
}
```

### 2. ApplicationContext

Advanced IoC container with additional enterprise features.

```java
/**
 * ApplicationContext - Eager initialization
 */
public class ApplicationContextExample {

    public static void main(String[] args) {

        // Create ApplicationContext
        System.out.println("Creating ApplicationContext...");
        ApplicationContext context =
            new ClassPathXmlApplicationContext("beans.xml");

        System.out.println("ApplicationContext created");
        // All singleton beans are ALREADY created (eager initialization)

        // Get bean - already instantiated
        UserService userService = context.getBean(UserService.class);
        System.out.println("Bean retrieved");
    }
}
```

**Characteristics:**
- **Eager Initialization**: All singleton beans created at startup
- **Rich Features**: I18n, event propagation, AOP support
- **Enterprise Ready**: Full enterprise application features
- **Multiple Implementations**: Various context types available

#### ApplicationContext Implementations

```java
/**
 * Different ApplicationContext Implementations
 */

// 1. ClassPathXmlApplicationContext - Load XML from classpath
ApplicationContext context =
    new ClassPathXmlApplicationContext("applicationContext.xml");

// 2. FileSystemXmlApplicationContext - Load XML from file system
ApplicationContext context =
    new FileSystemXmlApplicationContext("/path/to/applicationContext.xml");

// 3. AnnotationConfigApplicationContext - Java configuration
ApplicationContext context =
    new AnnotationConfigApplicationContext(AppConfig.class);

// 4. AnnotationConfigWebApplicationContext - Web applications
AnnotationConfigWebApplicationContext context =
    new AnnotationConfigWebApplicationContext();
context.register(WebConfig.class);
context.setServletContext(servletContext);
context.refresh();

// 5. GenericApplicationContext - Programmatic bean registration
GenericApplicationContext context = new GenericApplicationContext();
context.registerBean(UserService.class);
context.registerBean("customBean", CustomService.class);
context.refresh();
```

#### ApplicationContext Features

```java
/**
 * ApplicationContext Additional Features
 */

public class ApplicationContextFeatures {

    public static void main(String[] args) {
        ApplicationContext context =
            new AnnotationConfigApplicationContext(AppConfig.class);

        // 1. Bean Lifecycle Management
        UserService userService = context.getBean(UserService.class);

        // 2. Internationalization (i18n)
        String message = context.getMessage(
            "greeting.message",
            new Object[]{"John"},
            Locale.US
        );
        System.out.println(message);

        // 3. Event Publishing
        context.publishEvent(new CustomEvent(this, "Event data"));

        // 4. Environment Access
        Environment env = context.getEnvironment();
        String property = env.getProperty("app.name");
        String[] activeProfiles = env.getActiveProfiles();

        // 5. Resource Loading
        Resource resource = context.getResource("classpath:data.txt");

        // 6. Parent Context
        ApplicationContext parentContext = context.getParent();

        // 7. Bean Definition Names
        String[] beanNames = context.getBeanDefinitionNames();
        for (String name : beanNames) {
            System.out.println("Bean: " + name);
        }

        // 8. Shutdown Hook
        ((ConfigurableApplicationContext) context).registerShutdownHook();
    }
}
```

### BeanFactory vs ApplicationContext

| Feature | BeanFactory | ApplicationContext |
|---------|-------------|-------------------|
| **Initialization** | Lazy | Eager |
| **Memory** | Lightweight | Heavier |
| **I18n** | No | Yes |
| **Event Publishing** | No | Yes |
| **AOP** | Manual | Automatic |
| **Bean Post Processing** | Manual | Automatic |
| **Use Case** | Resource-constrained | Enterprise apps |

```java
/**
 * When to Use Which?
 */

// Use BeanFactory when:
// - Mobile applications (memory constrained)
// - Embedded systems
// - Simple applications with few beans

// Use ApplicationContext when:
// - Web applications
// - Enterprise applications
// - Need AOP, events, i18n
// - Most real-world applications

// Recommendation: Always use ApplicationContext
// unless you have specific memory constraints
```

---

## Bean Lifecycle in IoC Container

Understanding the bean lifecycle is crucial for proper initialization and cleanup.

### Complete Bean Lifecycle

```java
/**
 * Complete Bean Lifecycle Demonstration
 */

@Component
public class LifecycleBean implements
    BeanNameAware,
    BeanFactoryAware,
    ApplicationContextAware,
    InitializingBean,
    DisposableBean {

    private String beanName;
    private BeanFactory beanFactory;
    private ApplicationContext applicationContext;

    // 1. Constructor
    public LifecycleBean() {
        System.out.println("1. Constructor called");
    }

    // 2. Setter Injection
    @Autowired
    public void setDependency(SomeDependency dependency) {
        System.out.println("2. Dependency injected via setter");
    }

    // 3. BeanNameAware
    @Override
    public void setBeanName(String name) {
        this.beanName = name;
        System.out.println("3. BeanNameAware: setBeanName(" + name + ")");
    }

    // 4. BeanFactoryAware
    @Override
    public void setBeanFactory(BeanFactory beanFactory) {
        this.beanFactory = beanFactory;
        System.out.println("4. BeanFactoryAware: setBeanFactory()");
    }

    // 5. ApplicationContextAware
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) {
        this.applicationContext = applicationContext;
        System.out.println("5. ApplicationContextAware: setApplicationContext()");
    }

    // 6. @PostConstruct
    @PostConstruct
    public void postConstruct() {
        System.out.println("6. @PostConstruct method called");
    }

    // 7. InitializingBean
    @Override
    public void afterPropertiesSet() {
        System.out.println("7. InitializingBean: afterPropertiesSet()");
    }

    // 8. Custom init method (defined in @Bean or XML)
    public void customInit() {
        System.out.println("8. Custom init method called");
    }

    // Bean is now ready for use
    public void doWork() {
        System.out.println(">>> Bean is working <<<");
    }

    // 9. @PreDestroy
    @PreDestroy
    public void preDestroy() {
        System.out.println("9. @PreDestroy method called");
    }

    // 10. DisposableBean
    @Override
    public void destroy() {
        System.out.println("10. DisposableBean: destroy()");
    }

    // 11. Custom destroy method
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

public class LifecycleDemo {
    public static void main(String[] args) {
        System.out.println("=== Starting Application ===");

        ConfigurableApplicationContext context =
            new AnnotationConfigApplicationContext(AppConfig.class);

        System.out.println("\n=== Using Bean ===");
        LifecycleBean bean = context.getBean(LifecycleBean.class);
        bean.doWork();

        System.out.println("\n=== Shutting Down ===");
        context.close();
    }
}

/*
 * Output:
 * === Starting Application ===
 * 1. Constructor called
 * 2. Dependency injected via setter
 * 3. BeanNameAware: setBeanName(lifecycleBean)
 * 4. BeanFactoryAware: setBeanFactory()
 * 5. ApplicationContextAware: setApplicationContext()
 * 6. @PostConstruct method called
 * 7. InitializingBean: afterPropertiesSet()
 * 8. Custom init method called
 *
 * === Using Bean ===
 * >>> Bean is working <<<
 *
 * === Shutting Down ===
 * 9. @PreDestroy method called
 * 10. DisposableBean: destroy()
 * 11. Custom destroy method called
 */
```

### BeanPostProcessor

Allows custom modification of beans during initialization.

```java
/**
 * BeanPostProcessor - Intercept Bean Initialization
 */

@Component
public class CustomBeanPostProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) {
        System.out.println("BeanPostProcessor: BEFORE initialization of " + beanName);

        // Can modify bean or return different instance
        if (bean instanceof UserService) {
            System.out.println("  -> Found UserService, applying custom logic");
        }

        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) {
        System.out.println("BeanPostProcessor: AFTER initialization of " + beanName);

        // Can wrap bean in proxy
        if (bean instanceof PaymentService) {
            return createPaymentProxy((PaymentService) bean);
        }

        return bean;
    }

    private PaymentService createPaymentProxy(PaymentService target) {
        return (PaymentService) Proxy.newProxyInstance(
            target.getClass().getClassLoader(),
            target.getClass().getInterfaces(),
            (proxy, method, args) -> {
                System.out.println("Proxy: Before " + method.getName());
                Object result = method.invoke(target, args);
                System.out.println("Proxy: After " + method.getName());
                return result;
            }
        );
    }
}

// Updated lifecycle with BeanPostProcessor:
// 1. Constructor
// 2. Dependency Injection
// 3. BeanNameAware
// 4. BeanFactoryAware
// 5. ApplicationContextAware
// 6. BeanPostProcessor: postProcessBeforeInitialization
// 7. @PostConstruct
// 8. InitializingBean: afterPropertiesSet
// 9. Custom init method
// 10. BeanPostProcessor: postProcessAfterInitialization
// >>> Bean Ready <<<
```

### Practical Lifecycle Example

```java
/**
 * Practical Example: Database Connection Pool
 */

@Component
public class DatabaseConnectionPool implements InitializingBean, DisposableBean {

    @Value("${db.url}")
    private String url;

    @Value("${db.username}")
    private String username;

    @Value("${db.password}")
    private String password;

    @Value("${db.pool.size:10}")
    private int poolSize;

    private HikariDataSource dataSource;

    // Constructor
    public DatabaseConnectionPool() {
        System.out.println("DatabaseConnectionPool: Constructor");
    }

    // @PostConstruct - Initialize connection pool
    @PostConstruct
    public void init() {
        System.out.println("DatabaseConnectionPool: Initializing...");

        HikariConfig config = new HikariConfig();
        config.setJdbcUrl(url);
        config.setUsername(username);
        config.setPassword(password);
        config.setMaximumPoolSize(poolSize);

        dataSource = new HikariDataSource(config);

        System.out.println("DatabaseConnectionPool: Initialized with " +
                         poolSize + " connections");
    }

    // InitializingBean - Additional initialization
    @Override
    public void afterPropertiesSet() {
        System.out.println("DatabaseConnectionPool: afterPropertiesSet");
        // Verify connection
        try (Connection conn = dataSource.getConnection()) {
            System.out.println("DatabaseConnectionPool: Connection test successful");
        } catch (Exception e) {
            throw new RuntimeException("Failed to connect to database", e);
        }
    }

    // Get connection from pool
    public Connection getConnection() throws SQLException {
        return dataSource.getConnection();
    }

    // @PreDestroy - Start cleanup
    @PreDestroy
    public void cleanup() {
        System.out.println("DatabaseConnectionPool: Starting cleanup...");
    }

    // DisposableBean - Close pool
    @Override
    public void destroy() {
        System.out.println("DatabaseConnectionPool: Closing connection pool");
        if (dataSource != null && !dataSource.isClosed()) {
            dataSource.close();
        }
        System.out.println("DatabaseConnectionPool: Closed successfully");
    }
}
```

---

## Circular Dependencies

Circular dependencies occur when two or more beans depend on each other.

### Problem: Circular Dependencies

```java
/**
 * Circular Dependency Problem
 */

// BeanA depends on BeanB
@Component
public class ServiceA {

    private final ServiceB serviceB;

    @Autowired
    public ServiceA(ServiceB serviceB) {
        this.serviceB = serviceB;
    }

    public void methodA() {
        System.out.println("ServiceA.methodA()");
        serviceB.methodB();
    }
}

// BeanB depends on BeanA
@Component
public class ServiceB {

    private final ServiceA serviceA;

    @Autowired
    public ServiceB(ServiceA serviceA) {
        this.serviceA = serviceA;
    }

    public void methodB() {
        System.out.println("ServiceB.methodB()");
        serviceA.methodA();
    }
}

/*
 * Error:
 * The dependencies of some of the beans in the application context form a cycle:
 *
 * ┌─────┐
 * | serviceA defined in file [ServiceA.class]
 * ↑     ↓
 * | serviceB defined in file [ServiceB.class]
 * └─────┘
 */
```

### Solution 1: Setter Injection

```java
/**
 * Solution 1: Use Setter Injection
 */

@Component
public class ServiceA {

    private ServiceB serviceB;

    // Use setter injection instead of constructor
    @Autowired
    public void setServiceB(ServiceB serviceB) {
        this.serviceB = serviceB;
    }

    public void methodA() {
        System.out.println("ServiceA.methodA()");
        serviceB.methodB();
    }
}

@Component
public class ServiceB {

    private ServiceA serviceA;

    @Autowired
    public void setServiceA(ServiceA serviceA) {
        this.serviceA = serviceA;
    }

    public void methodB() {
        System.out.println("ServiceB.methodB()");
        serviceA.methodA();
    }
}

// Spring can now resolve this:
// 1. Create ServiceA instance (without dependencies)
// 2. Create ServiceB instance (without dependencies)
// 3. Inject ServiceB into ServiceA via setter
// 4. Inject ServiceA into ServiceB via setter
```

### Solution 2: @Lazy Annotation

```java
/**
 * Solution 2: Use @Lazy
 */

@Component
public class ServiceA {

    private final ServiceB serviceB;

    // Inject lazy proxy of ServiceB
    @Autowired
    public ServiceA(@Lazy ServiceB serviceB) {
        this.serviceB = serviceB;
    }

    public void methodA() {
        System.out.println("ServiceA.methodA()");
        serviceB.methodB(); // Actual ServiceB created on first use
    }
}

@Component
public class ServiceB {

    private final ServiceA serviceA;

    @Autowired
    public ServiceB(ServiceA serviceA) {
        this.serviceA = serviceA;
    }

    public void methodB() {
        System.out.println("ServiceB.methodB()");
        serviceA.methodA();
    }
}

// Spring injects a lazy proxy for ServiceB into ServiceA
// The actual ServiceB instance is created when first accessed
```

### Solution 3: Refactor Design

```java
/**
 * Solution 3: Refactor to Remove Circular Dependency (BEST)
 */

// Extract common functionality to a new service
@Component
public class SharedService {

    public void sharedMethod() {
        System.out.println("SharedService.sharedMethod()");
    }
}

@Component
public class ServiceA {

    private final SharedService sharedService;

    @Autowired
    public ServiceA(SharedService sharedService) {
        this.sharedService = sharedService;
    }

    public void methodA() {
        System.out.println("ServiceA.methodA()");
        sharedService.sharedMethod();
    }
}

@Component
public class ServiceB {

    private final SharedService sharedService;

    @Autowired
    public ServiceB(SharedService sharedService) {
        this.sharedService = sharedService;
    }

    public void methodB() {
        System.out.println("ServiceB.methodB()");
        sharedService.sharedMethod();
    }
}

// No circular dependency:
// SharedService has no dependencies
// ServiceA depends on SharedService
// ServiceB depends on SharedService
```

### Solution 4: ApplicationContextAware

```java
/**
 * Solution 4: Use ApplicationContextAware (Not Recommended)
 */

@Component
public class ServiceA implements ApplicationContextAware {

    private ApplicationContext context;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) {
        this.context = applicationContext;
    }

    public void methodA() {
        System.out.println("ServiceA.methodA()");
        // Get ServiceB from context when needed
        ServiceB serviceB = context.getBean(ServiceB.class);
        serviceB.methodB();
    }
}

@Component
public class ServiceB {

    private final ServiceA serviceA;

    @Autowired
    public ServiceB(ServiceA serviceA) {
        this.serviceA = serviceA;
    }

    public void methodB() {
        System.out.println("ServiceB.methodB()");
        serviceA.methodA();
    }
}

// Not recommended because:
// - Breaks IoC principle
// - Tight coupling to Spring framework
// - Makes testing difficult
```

---

## Real-World Examples

### Example 1: E-Commerce Order Processing

```java
/**
 * Real-World Example: E-Commerce Order System
 */

// Domain Models
public class Order {
    private Long id;
    private List<OrderItem> items;
    private BigDecimal total;
    private String customerEmail;
    // Getters and setters
}

public class OrderItem {
    private Product product;
    private int quantity;
    private BigDecimal price;
    // Getters and setters
}

// Repository Layer
@Repository
public class OrderRepository {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    public void save(Order order) {
        String sql = "INSERT INTO orders (customer_email, total) VALUES (?, ?)";
        jdbcTemplate.update(sql, order.getCustomerEmail(), order.getTotal());
        System.out.println("Order saved to database");
    }

    public Order findById(Long id) {
        String sql = "SELECT * FROM orders WHERE id = ?";
        return jdbcTemplate.queryForObject(sql, new OrderRowMapper(), id);
    }
}

@Repository
public class InventoryRepository {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    public boolean checkStock(Long productId, int quantity) {
        String sql = "SELECT stock FROM inventory WHERE product_id = ?";
        Integer stock = jdbcTemplate.queryForObject(sql, Integer.class, productId);
        return stock != null && stock >= quantity;
    }

    public void reduceStock(Long productId, int quantity) {
        String sql = "UPDATE inventory SET stock = stock - ? WHERE product_id = ?";
        jdbcTemplate.update(sql, quantity, productId);
        System.out.println("Stock reduced for product: " + productId);
    }
}

// Service Layer
@Service
@Transactional
public class OrderService {

    private final OrderRepository orderRepository;
    private final InventoryService inventoryService;
    private final PaymentService paymentService;
    private final NotificationService notificationService;

    // Constructor Injection
    @Autowired
    public OrderService(OrderRepository orderRepository,
                       InventoryService inventoryService,
                       PaymentService paymentService,
                       NotificationService notificationService) {
        this.orderRepository = orderRepository;
        this.inventoryService = inventoryService;
        this.paymentService = paymentService;
        this.notificationService = notificationService;
    }

    public Order placeOrder(Order order) {
        // 1. Validate inventory
        for (OrderItem item : order.getItems()) {
            if (!inventoryService.checkAvailability(item.getProduct().getId(),
                                                   item.getQuantity())) {
                throw new InsufficientStockException("Not enough stock for: " +
                                                    item.getProduct().getName());
            }
        }

        // 2. Process payment
        boolean paymentSuccess = paymentService.processPayment(order);
        if (!paymentSuccess) {
            throw new PaymentFailedException("Payment failed for order");
        }

        // 3. Reduce inventory
        for (OrderItem item : order.getItems()) {
            inventoryService.reduceStock(item.getProduct().getId(),
                                        item.getQuantity());
        }

        // 4. Save order
        orderRepository.save(order);

        // 5. Send notification
        notificationService.sendOrderConfirmation(order);

        return order;
    }
}

@Service
public class InventoryService {

    private final InventoryRepository inventoryRepository;

    @Autowired
    public InventoryService(InventoryRepository inventoryRepository) {
        this.inventoryRepository = inventoryRepository;
    }

    public boolean checkAvailability(Long productId, int quantity) {
        return inventoryRepository.checkStock(productId, quantity);
    }

    public void reduceStock(Long productId, int quantity) {
        inventoryRepository.reduceStock(productId, quantity);
    }
}

@Service
public class PaymentService {

    private final PaymentGateway paymentGateway;

    @Autowired
    public PaymentService(@Qualifier("stripeGateway") PaymentGateway paymentGateway) {
        this.paymentGateway = paymentGateway;
    }

    public boolean processPayment(Order order) {
        return paymentGateway.charge(order.getTotal(), order.getCustomerEmail());
    }
}

@Service
public class NotificationService {

    private final EmailService emailService;
    private final SmsService smsService;

    @Autowired
    public NotificationService(EmailService emailService,
                              @Autowired(required = false) SmsService smsService) {
        this.emailService = emailService;
        this.smsService = smsService;
    }

    public void sendOrderConfirmation(Order order) {
        // Send email (required)
        emailService.send(order.getCustomerEmail(),
                         "Order Confirmation",
                         "Your order has been placed successfully");

        // Send SMS (optional)
        if (smsService != null) {
            smsService.send(order.getCustomerEmail(),
                           "Order confirmed: #" + order.getId());
        }
    }
}

// Configuration
@Configuration
@ComponentScan(basePackages = "com.example.ecommerce")
@EnableTransactionManagement
public class AppConfig {

    @Bean
    public DataSource dataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:mysql://localhost:3306/ecommerce");
        config.setUsername("root");
        config.setPassword("password");
        return new HikariDataSource(config);
    }

    @Bean
    public JdbcTemplate jdbcTemplate(DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }

    @Bean
    public PlatformTransactionManager transactionManager(DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean
    @Qualifier("stripeGateway")
    public PaymentGateway stripePaymentGateway() {
        return new StripePaymentGateway();
    }
}

// Usage
public class ECommerceApplication {

    public static void main(String[] args) {
        ApplicationContext context =
            new AnnotationConfigApplicationContext(AppConfig.class);

        OrderService orderService = context.getBean(OrderService.class);

        // Create order
        Order order = new Order();
        order.setCustomerEmail("customer@example.com");
        // Add items...

        try {
            Order placedOrder = orderService.placeOrder(order);
            System.out.println("Order placed successfully: " + placedOrder.getId());
        } catch (Exception e) {
            System.err.println("Failed to place order: " + e.getMessage());
        }
    }
}
```

### Example 2: Multi-Layered Application

```java
/**
 * Real-World Example: User Management System
 */

// Entity
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String username;
    private String email;
    private String password;
    private boolean active;

    // Getters and setters
}

// Repository Layer - Data Access
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
    Optional<User> findByEmail(String email);
    List<User> findByActiveTrue();
}

// Service Layer - Business Logic
@Service
@Transactional
public class UserService {

    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    private final EmailService emailService;
    private final AuditService auditService;

    // Constructor injection - all dependencies are required
    @Autowired
    public UserService(UserRepository userRepository,
                      PasswordEncoder passwordEncoder,
                      EmailService emailService,
                      AuditService auditService) {
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
        this.emailService = emailService;
        this.auditService = auditService;
    }

    public User registerUser(UserRegistrationDTO dto) {
        // Validate
        if (userRepository.findByUsername(dto.getUsername()).isPresent()) {
            throw new UsernameAlreadyExistsException();
        }

        if (userRepository.findByEmail(dto.getEmail()).isPresent()) {
            throw new EmailAlreadyExistsException();
        }

        // Create user
        User user = new User();
        user.setUsername(dto.getUsername());
        user.setEmail(dto.getEmail());
        user.setPassword(passwordEncoder.encode(dto.getPassword()));
        user.setActive(true);

        // Save
        User savedUser = userRepository.save(user);

        // Send welcome email
        emailService.sendWelcomeEmail(savedUser);

        // Audit
        auditService.logUserCreated(savedUser);

        return savedUser;
    }

    @Transactional(readOnly = true)
    public Optional<User> getUserById(Long id) {
        return userRepository.findById(id);
    }

    @Transactional(readOnly = true)
    public List<User> getActiveUsers() {
        return userRepository.findByActiveTrue();
    }

    public void deleteUser(Long id) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id));

        user.setActive(false);
        userRepository.save(user);

        auditService.logUserDeleted(user);
    }
}

// Supporting Services
@Service
public class EmailService {

    @Value("${mail.smtp.host}")
    private String smtpHost;

    @Value("${mail.smtp.port}")
    private int smtpPort;

    @Value("${mail.from}")
    private String fromAddress;

    private JavaMailSender mailSender;

    @Autowired
    public void setMailSender(JavaMailSender mailSender) {
        this.mailSender = mailSender;
    }

    public void sendWelcomeEmail(User user) {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setFrom(fromAddress);
        message.setTo(user.getEmail());
        message.setSubject("Welcome to Our Platform!");
        message.setText("Hello " + user.getUsername() +
                       ",\n\nWelcome to our platform!");

        mailSender.send(message);
        System.out.println("Welcome email sent to: " + user.getEmail());
    }
}

@Service
public class AuditService {

    private final AuditRepository auditRepository;

    @Autowired
    public AuditService(AuditRepository auditRepository) {
        this.auditRepository = auditRepository;
    }

    public void logUserCreated(User user) {
        AuditLog log = new AuditLog();
        log.setAction("USER_CREATED");
        log.setEntityId(user.getId());
        log.setTimestamp(LocalDateTime.now());
        auditRepository.save(log);
    }

    public void logUserDeleted(User user) {
        AuditLog log = new AuditLog();
        log.setAction("USER_DELETED");
        log.setEntityId(user.getId());
        log.setTimestamp(LocalDateTime.now());
        auditRepository.save(log);
    }
}

// Controller Layer - Web Interface
@RestController
@RequestMapping("/api/users")
public class UserController {

    private final UserService userService;

    // Constructor injection
    @Autowired
    public UserController(UserService userService) {
        this.userService = userService;
    }

    @PostMapping
    public ResponseEntity<User> registerUser(@Valid @RequestBody UserRegistrationDTO dto) {
        User user = userService.registerUser(dto);
        return ResponseEntity.status(HttpStatus.CREATED).body(user);
    }

    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        return userService.getUserById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }

    @GetMapping
    public ResponseEntity<List<User>> getActiveUsers() {
        List<User> users = userService.getActiveUsers();
        return ResponseEntity.ok(users);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.noContent().build();
    }
}

// Configuration
@Configuration
@EnableJpaRepositories(basePackages = "com.example.repository")
@EnableTransactionManagement
@ComponentScan(basePackages = "com.example")
public class AppConfig {

    @Bean
    public DataSource dataSource() {
        // Configure datasource
        return new HikariDataSource();
    }

    @Bean
    public LocalContainerEntityManagerFactoryBean entityManagerFactory(
            DataSource dataSource) {
        LocalContainerEntityManagerFactoryBean em =
            new LocalContainerEntityManagerFactoryBean();
        em.setDataSource(dataSource);
        em.setPackagesToScan("com.example.entity");
        em.setJpaVendorAdapter(new HibernateJpaVendorAdapter());
        return em;
    }

    @Bean
    public PlatformTransactionManager transactionManager(
            EntityManagerFactory entityManagerFactory) {
        return new JpaTransactionManager(entityManagerFactory);
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public JavaMailSender mailSender() {
        JavaMailSenderImpl mailSender = new JavaMailSenderImpl();
        mailSender.setHost("smtp.gmail.com");
        mailSender.setPort(587);
        return mailSender;
    }
}
```

---

## Best Practices

### 1. Prefer Constructor Injection

```java
// GOOD - Constructor injection
@Service
public class UserService {

    private final UserRepository userRepository;
    private final EmailService emailService;

    @Autowired  // Optional for single constructor
    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }
}

// BAD - Field injection
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private EmailService emailService;
}
```

**Reasons:**
- Immutability (final fields)
- Testability (easy to mock)
- Clear dependencies
- Prevents NullPointerException

### 2. Use Interfaces for Dependencies

```java
// GOOD - Depend on interface
@Service
public class OrderService {

    private final PaymentGateway paymentGateway;

    @Autowired
    public OrderService(PaymentGateway paymentGateway) {
        this.paymentGateway = paymentGateway;
    }
}

// Interface
public interface PaymentGateway {
    boolean processPayment(Order order);
}

// Implementations
@Service("stripeGateway")
public class StripePaymentGateway implements PaymentGateway {
    public boolean processPayment(Order order) {
        // Stripe implementation
    }
}

@Service("paypalGateway")
public class PaypalPaymentGateway implements PaymentGateway {
    public boolean processPayment(Order order) {
        // PayPal implementation
    }
}
```

### 3. Avoid Circular Dependencies

```java
// BAD - Circular dependency
@Service
public class ServiceA {
    @Autowired
    private ServiceB serviceB;
}

@Service
public class ServiceB {
    @Autowired
    private ServiceA serviceA;
}

// GOOD - Refactored
@Service
public class ServiceA {
    @Autowired
    private SharedService sharedService;
}

@Service
public class ServiceB {
    @Autowired
    private SharedService sharedService;
}

@Service
public class SharedService {
    // Common functionality
}
```

### 4. Use @Lazy for Expensive Beans

```java
@Configuration
public class AppConfig {

    @Bean
    @Lazy  // Created only when needed
    public ExpensiveService expensiveService() {
        return new ExpensiveService();
    }
}
```

### 5. Organize Configuration

```java
// Separate configurations by concern
@Configuration
@ComponentScan(basePackages = "com.example")
public class AppConfig {
}

@Configuration
public class DatabaseConfig {
    @Bean
    public DataSource dataSource() { /* ... */ }
}

@Configuration
public class SecurityConfig {
    @Bean
    public SecurityManager securityManager() { /* ... */ }
}

// Import all configurations
@Configuration
@Import({DatabaseConfig.class, SecurityConfig.class})
public class MainConfig {
}
```

### 6. Use Profiles for Environment-Specific Beans

```java
@Configuration
@Profile("development")
public class DevConfig {
    @Bean
    public DataSource dataSource() {
        return new H2DataSource(); // In-memory
    }
}

@Configuration
@Profile("production")
public class ProdConfig {
    @Bean
    public DataSource dataSource() {
        return new MysqlDataSource(); // Production DB
    }
}
```

### 7. Proper Exception Handling

```java
@Service
public class UserService {

    private final UserRepository userRepository;

    @Autowired
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public User getUserById(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException("User not found: " + id));
    }
}
```

---

## IoC vs Service Locator Pattern

### Service Locator Pattern (Anti-Pattern in Spring)

```java
// Service Locator - NOT RECOMMENDED
public class ServiceLocator {

    private static ApplicationContext context;

    public static void setApplicationContext(ApplicationContext ctx) {
        context = ctx;
    }

    public static <T> T getService(Class<T> serviceClass) {
        return context.getBean(serviceClass);
    }
}

// Usage - BAD
public class UserController {

    public void createUser(User user) {
        // Manually looking up dependencies
        UserService userService = ServiceLocator.getService(UserService.class);
        userService.create(user);
    }
}
```

**Problems:**
- Hides dependencies
- Tight coupling to ServiceLocator
- Hard to test
- Defeats purpose of IoC

### IoC/DI Pattern (Recommended)

```java
// IoC/DI - RECOMMENDED
@RestController
public class UserController {

    private final UserService userService;

    // Dependencies injected by Spring
    @Autowired
    public UserController(UserService userService) {
        this.userService = userService;
    }

    @PostMapping("/users")
    public ResponseEntity<User> createUser(@RequestBody User user) {
        User created = userService.create(user);
        return ResponseEntity.ok(created);
    }
}
```

**Benefits:**
- Clear dependencies
- Loose coupling
- Easy to test
- True IoC

---

## Conclusion

### Key Takeaways

1. **IoC (Inversion of Control)**
   - Framework controls object creation
   - Dependencies are injected, not created
   - Reduces coupling and increases testability

2. **Dependency Injection**
   - Constructor injection (recommended)
   - Setter injection (for optional dependencies)
   - Avoid field injection

3. **Spring IoC Container**
   - BeanFactory: Basic container
   - ApplicationContext: Enterprise container (recommended)
   - Manages complete bean lifecycle

4. **Bean Lifecycle**
   - Instantiation → DI → Initialization → Use → Destruction
   - Use @PostConstruct and @PreDestroy for lifecycle hooks

5. **Best Practices**
   - Constructor injection with final fields
   - Depend on interfaces, not implementations
   - Avoid circular dependencies
   - Use profiles for environment-specific configuration

### Benefits of IoC and DI

- **Loose Coupling**: Components are independent
- **Testability**: Easy to inject mocks
- **Maintainability**: Changes isolated to configuration
- **Flexibility**: Easy to swap implementations
- **Reusability**: Components can be reused
- **Separation of Concerns**: Business logic separated from wiring

### When to Use

**Use IoC/DI when:**
- Building enterprise applications
- Need to test components
- Want to swap implementations
- Working with complex dependencies
- Need flexibility and maintainability

**Avoid over-engineering:**
- Simple scripts or utilities
- Performance-critical code with minimal dependencies
- Embedded systems with memory constraints

---

## Additional Resources

- Spring Framework Documentation: https://docs.spring.io/spring-framework/reference/core.html
- Spring IoC Container Guide
- Dependency Injection Patterns
- Martin Fowler's Inversion of Control Containers
- Spring Bean Lifecycle
- Effective Java by Joshua Bloch (Item 1: Consider static factory methods)
