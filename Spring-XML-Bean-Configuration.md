# Spring Framework XML-Based Bean Configuration - In-Depth Guide

## Table of Contents
1. [Introduction to XML Configuration](#introduction-to-xml-configuration)
2. [Bean Definition Basics](#bean-definition-basics)
3. [Dependency Injection](#dependency-injection)
4. [Constructor Injection](#constructor-injection)
5. [Setter Injection](#setter-injection)
6. [Bean Scopes](#bean-scopes)
7. [Auto-wiring](#auto-wiring)
8. [Bean Lifecycle](#bean-lifecycle)
9. [Advanced Configuration](#advanced-configuration)

---

## Introduction to XML Configuration

Spring Framework's XML-based configuration is the traditional way of defining beans and their dependencies. Although annotation-based and Java-based configurations are more popular now, understanding XML configuration is crucial for maintaining legacy applications and understanding Spring's core concepts.

### Basic XML Configuration Structure

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- Bean definitions go here -->

</beans>
```

---

## Bean Definition Basics

### Simple Bean Definition

A bean is an object managed by the Spring IoC (Inversion of Control) container.

```xml
<bean id="userService" class="com.example.service.UserService"/>
```

**Components:**
- `id`: Unique identifier for the bean
- `class`: Fully qualified class name

### Bean with Alias

```xml
<bean id="userService" name="userSvc,userManager" class="com.example.service.UserService"/>
```

Or using separate alias tags:

```xml
<bean id="userService" class="com.example.service.UserService"/>
<alias name="userService" alias="userManager"/>
```

---

## Dependency Injection

Dependency Injection (DI) is a design pattern where objects receive their dependencies from external sources rather than creating them internally.

### How DI Works in Spring

1. **Container Initialization**: Spring IoC container reads the XML configuration
2. **Bean Instantiation**: Container creates bean instances based on definitions
3. **Dependency Resolution**: Container identifies dependencies between beans
4. **Dependency Injection**: Container injects dependencies into beans
5. **Bean Ready**: Fully configured beans are ready for use

---

## Constructor Injection

Constructor injection provides dependencies through class constructors.

### Example Classes

```java
public class UserRepository {
    private String databaseUrl;

    public UserRepository(String databaseUrl) {
        this.databaseUrl = databaseUrl;
    }

    public void save(User user) {
        System.out.println("Saving user to: " + databaseUrl);
    }
}

public class UserService {
    private UserRepository userRepository;
    private String serviceName;

    // Constructor with multiple parameters
    public UserService(UserRepository userRepository, String serviceName) {
        this.userRepository = userRepository;
        this.serviceName = serviceName;
    }

    public void createUser(User user) {
        userRepository.save(user);
    }
}
```

### XML Configuration - Constructor Injection

#### Method 1: Using constructor-arg with ref

```xml
<bean id="userRepository" class="com.example.repository.UserRepository">
    <constructor-arg value="jdbc:mysql://localhost:3306/mydb"/>
</bean>

<bean id="userService" class="com.example.service.UserService">
    <constructor-arg ref="userRepository"/>
    <constructor-arg value="UserManagementService"/>
</bean>
```

#### Method 2: Using index attribute

```xml
<bean id="userService" class="com.example.service.UserService">
    <constructor-arg index="0" ref="userRepository"/>
    <constructor-arg index="1" value="UserManagementService"/>
</bean>
```

#### Method 3: Using name attribute

```xml
<bean id="userService" class="com.example.service.UserService">
    <constructor-arg name="userRepository" ref="userRepository"/>
    <constructor-arg name="serviceName" value="UserManagementService"/>
</bean>
```

#### Method 4: Using type attribute

```xml
<bean id="userService" class="com.example.service.UserService">
    <constructor-arg type="com.example.repository.UserRepository" ref="userRepository"/>
    <constructor-arg type="java.lang.String" value="UserManagementService"/>
</bean>
```

### Constructor Injection with Collections

```java
public class EmailService {
    private List<String> recipients;
    private Map<String, String> config;

    public EmailService(List<String> recipients, Map<String, String> config) {
        this.recipients = recipients;
        this.config = config;
    }
}
```

```xml
<bean id="emailService" class="com.example.service.EmailService">
    <constructor-arg>
        <list>
            <value>admin@example.com</value>
            <value>support@example.com</value>
            <value>info@example.com</value>
        </list>
    </constructor-arg>
    <constructor-arg>
        <map>
            <entry key="host" value="smtp.gmail.com"/>
            <entry key="port" value="587"/>
            <entry key="username" value="myapp@gmail.com"/>
        </map>
    </constructor-arg>
</bean>
```

---

## Setter Injection

Setter injection provides dependencies through setter methods.

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

### XML Configuration - Setter Injection

```xml
<bean id="paymentService" class="com.example.service.PaymentService"/>

<bean id="notificationService" class="com.example.service.NotificationService"/>

<bean id="orderService" class="com.example.service.OrderService">
    <property name="paymentService" ref="paymentService"/>
    <property name="notificationService" ref="notificationService"/>
    <property name="taxRate" value="0.08"/>
</bean>
```

### Setter Injection with Inner Beans

```xml
<bean id="orderService" class="com.example.service.OrderService">
    <property name="paymentService">
        <bean class="com.example.service.PaymentService">
            <property name="provider" value="Stripe"/>
        </bean>
    </property>
    <property name="taxRate" value="0.08"/>
</bean>
```

### Setter Injection with Collections

```xml
<bean id="productService" class="com.example.service.ProductService">
    <!-- List injection -->
    <property name="categories">
        <list>
            <value>Electronics</value>
            <value>Books</value>
            <value>Clothing</value>
        </list>
    </property>

    <!-- Set injection -->
    <property name="tags">
        <set>
            <value>featured</value>
            <value>popular</value>
            <value>new</value>
        </set>
    </property>

    <!-- Map injection -->
    <property name="metadata">
        <map>
            <entry key="version" value="1.0"/>
            <entry key="author" value="Admin"/>
        </map>
    </property>

    <!-- Properties injection -->
    <property name="configuration">
        <props>
            <prop key="cache.enabled">true</prop>
            <prop key="cache.timeout">3600</prop>
        </props>
    </property>
</bean>
```

---

## Bean Scopes

Bean scope defines the lifecycle and visibility of beans within the container.

### Available Scopes

| Scope | Description |
|-------|-------------|
| **singleton** | Single instance per Spring IoC container (default) |
| **prototype** | New instance created each time bean is requested |
| **request** | Single instance per HTTP request (web-aware) |
| **session** | Single instance per HTTP session (web-aware) |
| **application** | Single instance per ServletContext (web-aware) |
| **websocket** | Single instance per WebSocket (web-aware) |

### 1. Singleton Scope (Default)

```xml
<bean id="userService" class="com.example.service.UserService" scope="singleton"/>
<!-- OR simply -->
<bean id="userService" class="com.example.service.UserService"/>
```

**Characteristics:**
- Only one instance per container
- Same instance returned for all requests
- Created at container startup (eager initialization)
- Stateless beans work best

**Example:**

```java
ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
UserService service1 = context.getBean("userService", UserService.class);
UserService service2 = context.getBean("userService", UserService.class);
System.out.println(service1 == service2); // true
```

### 2. Prototype Scope

```xml
<bean id="shoppingCart" class="com.example.model.ShoppingCart" scope="prototype"/>
```

**Characteristics:**
- New instance created for each request
- Container doesn't manage complete lifecycle
- Destruction callbacks not called
- Stateful beans work best

**Example:**

```java
ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
ShoppingCart cart1 = context.getBean("shoppingCart", ShoppingCart.class);
ShoppingCart cart2 = context.getBean("shoppingCart", ShoppingCart.class);
System.out.println(cart1 == cart2); // false
```

### 3. Request Scope (Web Application)

```xml
<bean id="loginAction" class="com.example.web.LoginAction" scope="request"/>
```

**Characteristics:**
- New instance per HTTP request
- Destroyed after request completes
- Requires web-aware Spring ApplicationContext

### 4. Session Scope (Web Application)

```xml
<bean id="userPreferences" class="com.example.model.UserPreferences" scope="session"/>
```

**Characteristics:**
- Single instance per HTTP session
- Destroyed when session invalidates
- Requires web-aware Spring ApplicationContext

### Lazy Initialization

```xml
<!-- Lazy initialization for singleton -->
<bean id="expensiveService" class="com.example.service.ExpensiveService"
      lazy-init="true"/>

<!-- Global lazy initialization -->
<beans default-lazy-init="true">
    <bean id="service1" class="com.example.Service1"/>
    <bean id="service2" class="com.example.Service2"/>
</beans>
```

---

## Auto-wiring

Auto-wiring allows Spring to automatically resolve dependencies without explicit configuration.

### Auto-wiring Modes

| Mode | Description |
|------|-------------|
| **no** | No auto-wiring (default) |
| **byName** | Auto-wire by property name |
| **byType** | Auto-wire by property type |
| **constructor** | Auto-wire by constructor arguments |

### 1. No Auto-wiring (Default)

```xml
<bean id="orderService" class="com.example.service.OrderService">
    <property name="paymentService" ref="paymentService"/>
    <property name="inventoryService" ref="inventoryService"/>
</bean>
```

### 2. Auto-wire by Name

```xml
<bean id="paymentService" class="com.example.service.PaymentService"/>
<bean id="inventoryService" class="com.example.service.InventoryService"/>

<bean id="orderService" class="com.example.service.OrderService" autowire="byName"/>
```

**How it works:**
- Spring looks for beans with IDs matching property names
- Property `paymentService` matches bean `paymentService`
- Property `inventoryService` matches bean `inventoryService`

```java
public class OrderService {
    private PaymentService paymentService;      // matches bean id "paymentService"
    private InventoryService inventoryService;  // matches bean id "inventoryService"

    // Setters required
    public void setPaymentService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    public void setInventoryService(InventoryService inventoryService) {
        this.inventoryService = inventoryService;
    }
}
```

### 3. Auto-wire by Type

```xml
<bean id="payment" class="com.example.service.PaymentService"/>
<bean id="inventory" class="com.example.service.InventoryService"/>

<bean id="orderService" class="com.example.service.OrderService" autowire="byType"/>
```

**How it works:**
- Spring looks for beans matching property types
- Property of type `PaymentService` matches bean of class `PaymentService`
- **Limitation:** Only one bean of each type should exist

**Example with ambiguity:**

```xml
<!-- This will cause an error - two beans of same type -->
<bean id="payment1" class="com.example.service.PaymentService"/>
<bean id="payment2" class="com.example.service.PaymentService"/>

<bean id="orderService" class="com.example.service.OrderService" autowire="byType"/>
<!-- Error: No qualifying bean of type 'PaymentService' available: expected single matching bean but found 2 -->
```

**Solution - Mark primary bean:**

```xml
<bean id="payment1" class="com.example.service.PaymentService" primary="true"/>
<bean id="payment2" class="com.example.service.PaymentService"/>

<bean id="orderService" class="com.example.service.OrderService" autowire="byType"/>
```

### 4. Auto-wire by Constructor

```xml
<bean id="paymentService" class="com.example.service.PaymentService"/>
<bean id="inventoryService" class="com.example.service.InventoryService"/>

<bean id="orderService" class="com.example.service.OrderService" autowire="constructor"/>
```

**How it works:**
- Similar to `byType` but applies to constructor arguments
- Spring matches constructor parameter types with available beans

```java
public class OrderService {
    private PaymentService paymentService;
    private InventoryService inventoryService;

    // Constructor - Spring will auto-wire these parameters
    public OrderService(PaymentService paymentService, InventoryService inventoryService) {
        this.paymentService = paymentService;
        this.inventoryService = inventoryService;
    }
}
```

### Global Auto-wiring

```xml
<beans default-autowire="byType">
    <bean id="paymentService" class="com.example.service.PaymentService"/>
    <bean id="orderService" class="com.example.service.OrderService"/>
    <!-- All beans will use byType autowiring by default -->
</beans>
```

### Excluding Beans from Auto-wiring

```xml
<bean id="paymentService" class="com.example.service.PaymentService"
      autowire-candidate="false"/>
```

---

## Bean Lifecycle

Understanding the bean lifecycle is crucial for proper initialization and cleanup.

### Lifecycle Phases

1. **Bean Instantiation**
2. **Property Population**
3. **Bean Name Awareness** (if implements BeanNameAware)
4. **Bean Factory Awareness** (if implements BeanFactoryAware)
5. **Pre-Initialization** (BeanPostProcessor)
6. **Initialization** (InitializingBean or init-method)
7. **Post-Initialization** (BeanPostProcessor)
8. **Bean Ready for Use**
9. **Destruction** (DisposableBean or destroy-method)

### Initialization and Destruction Methods

#### Using init-method and destroy-method

```java
public class DatabaseConnection {
    private String url;

    public void setUrl(String url) {
        this.url = url;
    }

    // Custom initialization method
    public void connect() {
        System.out.println("Connecting to database: " + url);
        // Establish connection
    }

    // Custom destruction method
    public void disconnect() {
        System.out.println("Disconnecting from database");
        // Close connection
    }
}
```

```xml
<bean id="dbConnection" class="com.example.DatabaseConnection"
      init-method="connect"
      destroy-method="disconnect">
    <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
</bean>
```

#### Using default-init-method and default-destroy-method

```xml
<beans default-init-method="init" default-destroy-method="cleanup">
    <bean id="service1" class="com.example.Service1"/>
    <bean id="service2" class="com.example.Service2"/>
    <!-- All beans will call init() and cleanup() if they exist -->
</beans>
```

### Lifecycle Interfaces

#### InitializingBean and DisposableBean

```java
import org.springframework.beans.factory.InitializingBean;
import org.springframework.beans.factory.DisposableBean;

public class CacheService implements InitializingBean, DisposableBean {

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("Initializing cache...");
        // Initialization logic
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("Clearing cache...");
        // Cleanup logic
    }
}
```

```xml
<bean id="cacheService" class="com.example.service.CacheService"/>
```

---

## Advanced Configuration

### 1. Bean Inheritance

```xml
<bean id="baseService" class="com.example.service.BaseService" abstract="true">
    <property name="timeout" value="30"/>
    <property name="retryCount" value="3"/>
</bean>

<bean id="userService" class="com.example.service.UserService" parent="baseService">
    <property name="maxUsers" value="1000"/>
    <!-- Inherits timeout and retryCount from baseService -->
</bean>

<bean id="orderService" class="com.example.service.OrderService" parent="baseService">
    <property name="maxOrders" value="5000"/>
    <!-- Inherits timeout and retryCount from baseService -->
</bean>
```

### 2. Factory Beans

#### Static Factory Method

```java
public class ServiceFactory {
    public static UserService createUserService() {
        return new UserService("admin");
    }
}
```

```xml
<bean id="userService"
      class="com.example.factory.ServiceFactory"
      factory-method="createUserService"/>
```

#### Instance Factory Method

```java
public class ServiceFactory {
    public UserService createUserService() {
        return new UserService("admin");
    }
}
```

```xml
<bean id="serviceFactory" class="com.example.factory.ServiceFactory"/>

<bean id="userService"
      factory-bean="serviceFactory"
      factory-method="createUserService"/>
```

### 3. Method Injection (Lookup Method)

Useful when singleton bean needs prototype bean.

```java
public abstract class CommandManager {
    public void process() {
        Command command = createCommand();  // Gets new instance each time
        command.execute();
    }

    protected abstract Command createCommand();  // Lookup method
}
```

```xml
<bean id="command" class="com.example.Command" scope="prototype"/>

<bean id="commandManager" class="com.example.CommandManager">
    <lookup-method name="createCommand" bean="command"/>
</bean>
```

### 4. Depends-on

```xml
<!-- Ensure cacheManager is initialized before userService -->
<bean id="cacheManager" class="com.example.CacheManager"/>

<bean id="userService" class="com.example.service.UserService"
      depends-on="cacheManager"/>
```

### 5. Property Placeholder

#### Configuration File (application.properties)

```properties
db.url=jdbc:mysql://localhost:3306/mydb
db.username=root
db.password=secret
mail.host=smtp.gmail.com
mail.port=587
```

#### XML Configuration

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context.xsd">

    <context:property-placeholder location="classpath:application.properties"/>

    <bean id="dataSource" class="com.example.datasource.DataSource">
        <property name="url" value="${db.url}"/>
        <property name="username" value="${db.username}"/>
        <property name="password" value="${db.password}"/>
    </bean>

    <bean id="mailService" class="com.example.service.MailService">
        <property name="host" value="${mail.host}"/>
        <property name="port" value="${mail.port}"/>
    </bean>
</beans>
```

### 6. Importing Other Configuration Files

```xml
<beans>
    <import resource="classpath:service-config.xml"/>
    <import resource="classpath:dao-config.xml"/>
    <import resource="classpath:security-config.xml"/>

    <!-- Local bean definitions -->
    <bean id="mainController" class="com.example.MainController"/>
</beans>
```

### 7. Profiles

```xml
<beans profile="development">
    <bean id="dataSource" class="com.example.DevDataSource">
        <property name="url" value="jdbc:h2:mem:testdb"/>
    </bean>
</beans>

<beans profile="production">
    <bean id="dataSource" class="com.example.ProdDataSource">
        <property name="url" value="jdbc:mysql://prod-server:3306/proddb"/>
    </bean>
</beans>
```

Activate profile:

```java
System.setProperty("spring.profiles.active", "development");
// OR
context.getEnvironment().setActiveProfiles("development");
```

---

## Complete Example

### Java Classes

```java
// Repository Layer
public class UserRepository {
    private String databaseUrl;

    public UserRepository(String databaseUrl) {
        this.databaseUrl = databaseUrl;
    }

    public void save(User user) {
        System.out.println("Saving " + user.getName() + " to " + databaseUrl);
    }

    public void init() {
        System.out.println("UserRepository initialized");
    }

    public void cleanup() {
        System.out.println("UserRepository cleanup");
    }
}

// Service Layer
public class UserService {
    private UserRepository userRepository;
    private EmailService emailService;
    private String serviceName;
    private int maxUsers;

    // Constructor injection
    public UserService(UserRepository userRepository, String serviceName) {
        this.userRepository = userRepository;
        this.serviceName = serviceName;
    }

    // Setter injection
    public void setEmailService(EmailService emailService) {
        this.emailService = emailService;
    }

    public void setMaxUsers(int maxUsers) {
        this.maxUsers = maxUsers;
    }

    public void registerUser(User user) {
        userRepository.save(user);
        emailService.sendWelcomeEmail(user);
    }
}

// Email Service
public class EmailService {
    private String smtpHost;
    private int smtpPort;

    public void setSmtpHost(String smtpHost) {
        this.smtpHost = smtpHost;
    }

    public void setSmtpPort(int smtpPort) {
        this.smtpPort = smtpPort;
    }

    public void sendWelcomeEmail(User user) {
        System.out.println("Sending welcome email to " + user.getEmail());
    }
}
```

### Complete XML Configuration

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- Property placeholder -->
    <context:property-placeholder location="classpath:application.properties"/>

    <!-- Repository bean with constructor injection and lifecycle methods -->
    <bean id="userRepository"
          class="com.example.repository.UserRepository"
          init-method="init"
          destroy-method="cleanup">
        <constructor-arg value="${db.url}"/>
    </bean>

    <!-- Email service bean with setter injection -->
    <bean id="emailService" class="com.example.service.EmailService">
        <property name="smtpHost" value="${mail.host}"/>
        <property name="smtpPort" value="${mail.port}"/>
    </bean>

    <!-- User service with mixed injection (constructor + setter) -->
    <bean id="userService"
          class="com.example.service.UserService"
          scope="singleton">
        <!-- Constructor injection -->
        <constructor-arg ref="userRepository"/>
        <constructor-arg value="UserManagementService"/>
        <!-- Setter injection -->
        <property name="emailService" ref="emailService"/>
        <property name="maxUsers" value="10000"/>
    </bean>

    <!-- Prototype bean for shopping cart -->
    <bean id="shoppingCart"
          class="com.example.model.ShoppingCart"
          scope="prototype"/>

    <!-- Auto-wired bean example -->
    <bean id="orderService"
          class="com.example.service.OrderService"
          autowire="byType"/>
</beans>
```

### Application Startup

```java
public class Application {
    public static void main(String[] args) {
        // Load XML configuration
        ApplicationContext context =
            new ClassPathXmlApplicationContext("applicationContext.xml");

        // Get bean from container
        UserService userService = context.getBean("userService", UserService.class);

        // Use the bean
        User user = new User("John Doe", "john@example.com");
        userService.registerUser(user);

        // Close context (triggers destroy methods)
        ((ClassPathXmlApplicationContext) context).close();
    }
}
```

---

## Best Practices

1. **Use Constructor Injection for Mandatory Dependencies**
   - Ensures object is fully initialized
   - Makes dependencies explicit
   - Supports immutability

2. **Use Setter Injection for Optional Dependencies**
   - More flexible
   - Allows reconfiguration

3. **Prefer Explicit Wiring Over Auto-wiring**
   - More maintainable
   - Easier to understand
   - Better for large applications

4. **Use Appropriate Scopes**
   - Singleton for stateless beans
   - Prototype for stateful beans

5. **Implement Lifecycle Methods for Resource Management**
   - Use init methods to set up resources
   - Use destroy methods to clean up

6. **Externalize Configuration**
   - Use property placeholders
   - Separate environment-specific configs

7. **Use Profiles for Environment-Specific Beans**
   - Development, testing, production profiles

8. **Organize Configuration Files**
   - Split large configs into modules
   - Use import to combine them

---

## Common Pitfalls

1. **Circular Dependencies**
   ```xml
   <!-- Avoid this -->
   <bean id="beanA" class="com.example.BeanA">
       <property name="beanB" ref="beanB"/>
   </bean>
   <bean id="beanB" class="com.example.BeanB">
       <property name="beanA" ref="beanA"/>
   </bean>
   ```

2. **Singleton with Prototype Dependency**
   - Singleton beans cache prototype dependencies
   - Use method injection or Provider interface

3. **Type Ambiguity in Auto-wiring**
   - Multiple beans of same type cause errors
   - Use primary or qualifier

4. **Forgetting Setter Methods**
   - Setter injection requires setter methods
   - Property names must match

5. **Not Closing Context**
   - Resources may not be cleaned up properly
   - Use try-with-resources or registerShutdownHook()

---

## Conclusion

XML-based configuration provides a comprehensive way to manage Spring beans and their dependencies. While annotation and Java-based configurations are more modern, understanding XML configuration helps in:

- Maintaining legacy applications
- Understanding Spring's core concepts
- Debugging configuration issues
- Making informed decisions about configuration styles

Key takeaways:
- **Dependency Injection** decouples object creation from usage
- **Constructor Injection** for mandatory, immutable dependencies
- **Setter Injection** for optional, reconfigurable dependencies
- **Bean Scopes** control object lifecycle and sharing
- **Auto-wiring** reduces configuration but use carefully
- **Lifecycle Methods** manage resources properly

---

## Additional Resources

- Spring Framework Documentation: https://docs.spring.io/spring-framework/docs/current/reference/html/core.html
- Spring IoC Container Reference
- XML Schema Reference
- Bean Definition Profiles
- Environment Abstraction
