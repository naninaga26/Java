# Object-Oriented Programming (OOP) - Complete Guide

## Table of Contents
1. [The Four Pillars of OOP](#the-four-pillars-of-oop)
2. [Encapsulation](#encapsulation)
3. [Inheritance](#inheritance)
4. [Polymorphism](#polymorphism)
5. [Abstraction](#abstraction)
6. [Classes and Objects](#classes-and-objects)
7. [Interfaces vs Abstract Classes](#interfaces-vs-abstract-classes)
8. [Common Pitfalls](#common-pitfalls)
9. [Design Principles](#design-principles)
10. [Interview Questions](#interview-questions)

---

## The Four Pillars of OOP

```
┌──────────────────────────────────────────────────┐
│           Object-Oriented Programming            │
├──────────────────────────────────────────────────┤
│  1. ENCAPSULATION  - Data hiding                 │
│  2. INHERITANCE    - Code reuse                  │
│  3. POLYMORPHISM   - Many forms                  │
│  4. ABSTRACTION    - Hide complexity             │
└──────────────────────────────────────────────────┘
```

---

## Encapsulation

**Definition:** Bundling data (variables) and methods that operate on that data into a single unit (class), and restricting direct access to some components.

### Basic Encapsulation

```java
// ❌ BAD: No encapsulation
public class BankAccount {
    public double balance;  // Directly accessible!
}

// Anyone can do this:
BankAccount account = new BankAccount();
account.balance = -1000;  // Invalid state!

// ✅ GOOD: Proper encapsulation
public class BankAccount {
    private double balance;  // Private field

    // Controlled access through methods
    public double getBalance() {
        return balance;
    }

    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        } else {
            throw new IllegalArgumentException("Amount must be positive");
        }
    }

    public void withdraw(double amount) {
        if (amount > 0 && amount <= balance) {
            balance -= amount;
        } else {
            throw new IllegalArgumentException("Invalid withdrawal amount");
        }
    }
}
```

### Access Modifiers

```java
public class AccessModifiersDemo {
    public int publicVar = 1;        // Accessible everywhere
    protected int protectedVar = 2;  // Accessible in package + subclasses
    int defaultVar = 3;              // Accessible only in package (package-private)
    private int privateVar = 4;      // Accessible only within this class

    public void demonstrateAccess() {
        // All accessible within same class
        System.out.println(publicVar);
        System.out.println(protectedVar);
        System.out.println(defaultVar);
        System.out.println(privateVar);
    }
}

// Different package
class AnotherClass {
    void test() {
        AccessModifiersDemo demo = new AccessModifiersDemo();
        demo.publicVar = 10;      // ✅ OK
        // demo.protectedVar = 20; // ❌ Error (different package, not subclass)
        // demo.defaultVar = 30;   // ❌ Error (different package)
        // demo.privateVar = 40;   // ❌ Error (private)
    }
}

// Subclass in different package
class SubClass extends AccessModifiersDemo {
    void test() {
        publicVar = 10;      // ✅ OK
        protectedVar = 20;   // ✅ OK (subclass)
        // defaultVar = 30;  // ❌ Error (different package)
        // privateVar = 40;  // ❌ Error (private)
    }
}
```

**Common Pitfall #1: Breaking Encapsulation with Mutable Objects**

```java
// ❌ BAD: Returning mutable object breaks encapsulation
public class Company {
    private List<Employee> employees = new ArrayList<>();

    public List<Employee> getEmployees() {
        return employees;  // Exposes internal list!
    }
}

// Client code can modify internal state:
Company company = new Company();
List<Employee> emps = company.getEmployees();
emps.clear();  // Oops! Cleared company's internal list

// ✅ GOOD: Return defensive copy
public class Company {
    private List<Employee> employees = new ArrayList<>();

    public List<Employee> getEmployees() {
        return new ArrayList<>(employees);  // Return copy
    }

    // Or return unmodifiable view
    public List<Employee> getEmployeesReadOnly() {
        return Collections.unmodifiableList(employees);
    }
}
```

**Common Pitfall #2: Mutable Fields**

```java
// ❌ BAD: Mutable Date field
public class Person {
    private Date birthDate;

    public Date getBirthDate() {
        return birthDate;  // Returns reference to mutable object!
    }
}

// Client can modify the date:
Person person = new Person();
Date date = person.getBirthDate();
date.setYear(1900);  // Modified person's birth date!

// ✅ GOOD: Use immutable types or return copy
public class Person {
    private final LocalDate birthDate;  // Immutable (Java 8+)

    public LocalDate getBirthDate() {
        return birthDate;  // LocalDate is immutable, safe to return
    }

    // Or if using Date, return copy:
    public Date getBirthDateCopy() {
        return new Date(birthDate.getTime());
    }
}
```

---

## Inheritance

**Definition:** Mechanism where a new class (subclass/child) inherits properties and methods from an existing class (superclass/parent).

### Basic Inheritance

```java
// Parent class
public class Animal {
    protected String name;
    protected int age;

    public Animal(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public void eat() {
        System.out.println(name + " is eating");
    }

    public void sleep() {
        System.out.println(name + " is sleeping");
    }
}

// Child class
public class Dog extends Animal {
    private String breed;

    public Dog(String name, int age, String breed) {
        super(name, age);  // Call parent constructor
        this.breed = breed;
    }

    // Override parent method
    @Override
    public void eat() {
        System.out.println(name + " the dog is eating dog food");
    }

    // New method specific to Dog
    public void bark() {
        System.out.println(name + " is barking");
    }
}

// Usage
Dog dog = new Dog("Buddy", 5, "Golden Retriever");
dog.eat();    // Calls overridden method in Dog
dog.sleep();  // Inherits from Animal
dog.bark();   // Dog-specific method
```

### Method Overriding Rules

```java
public class Parent {
    // 1. Access modifier can be same or more permissive in child
    protected void method1() { }

    // 2. Return type can be covariant (subtype)
    public Parent method2() {
        return this;
    }

    // 3. Cannot override final methods
    public final void method3() { }

    // 4. Cannot override static methods (hiding, not overriding)
    public static void method4() { }
}

public class Child extends Parent {
    // ✅ Same or more permissive access
    @Override
    public void method1() { }  // protected -> public OK

    // ✅ Covariant return type
    @Override
    public Child method2() {  // Parent -> Child OK
        return this;
    }

    // ❌ Cannot override final method
    // @Override
    // public void method3() { }  // Compilation error!

    // This is METHOD HIDING, not overriding
    public static void method4() { }  // Hides parent's static method
}
```

**Common Pitfall #3: Calling Overridden Methods in Constructor**

```java
// ❌ DANGEROUS: Calling overridable method in constructor
public class Parent {
    private int value;

    public Parent() {
        initialize();  // Calls overridable method!
    }

    public void initialize() {
        value = 10;
    }

    public void printValue() {
        System.out.println("Value: " + value);
    }
}

public class Child extends Parent {
    private String name;

    public Child(String name) {
        super();  // Calls Parent constructor
        this.name = name;
    }

    @Override
    public void initialize() {
        // Child's version called during Parent construction!
        // But 'name' is still null here!
        System.out.println("Initializing for: " + name.length());  // NullPointerException!
    }
}

// Problem:
Child child = new Child("Test");  // NPE during construction!

// ✅ SOLUTION: Use final methods or private methods in constructor
public class Parent {
    private int value;

    public Parent() {
        initializeInternal();  // Private, cannot be overridden
    }

    private void initializeInternal() {
        value = 10;
    }

    // Subclasses can override this if needed
    protected void initialize() {
        // Called explicitly after construction
    }
}
```

**Common Pitfall #4: Inheritance vs Composition**

```java
// ❌ BAD: Inappropriate use of inheritance
class Stack extends ArrayList {
    public void push(Object value) {
        add(value);
    }

    public Object pop() {
        return remove(size() - 1);
    }
}

// Problem: Stack inherits ALL ArrayList methods!
Stack stack = new Stack();
stack.push(1);
stack.push(2);
stack.add(0, 3);  // Breaks stack semantics! (LIFO violated)

// ✅ GOOD: Use composition instead
class Stack {
    private List<Object> elements = new ArrayList<>();

    public void push(Object value) {
        elements.add(value);
    }

    public Object pop() {
        return elements.remove(elements.size() - 1);
    }

    // Only expose stack-specific operations
}
```

### Super Keyword

```java
public class Parent {
    protected String name = "Parent";

    public void display() {
        System.out.println("Parent display");
    }
}

public class Child extends Parent {
    private String name = "Child";

    public void showNames() {
        System.out.println(name);        // Child
        System.out.println(this.name);   // Child
        System.out.println(super.name);  // Parent
    }

    @Override
    public void display() {
        super.display();  // Call parent's version
        System.out.println("Child display");
    }
}
```

---

## Polymorphism

**Definition:** Ability of an object to take many forms. One interface, multiple implementations.

### Types of Polymorphism

```
Polymorphism
├── Compile-time (Static) Polymorphism
│   └── Method Overloading
└── Runtime (Dynamic) Polymorphism
    └── Method Overriding
```

### Method Overloading (Compile-time Polymorphism)

```java
public class Calculator {
    // Same method name, different parameters

    // 1. Different number of parameters
    public int add(int a, int b) {
        return a + b;
    }

    public int add(int a, int b, int c) {
        return a + b + c;
    }

    // 2. Different parameter types
    public double add(double a, double b) {
        return a + b;
    }

    // 3. Different order of parameters
    public String concatenate(String str, int num) {
        return str + num;
    }

    public String concatenate(int num, String str) {
        return num + str;
    }
}

// Usage
Calculator calc = new Calculator();
System.out.println(calc.add(5, 3));        // 8
System.out.println(calc.add(5, 3, 2));     // 10
System.out.println(calc.add(5.5, 3.2));    // 8.7
```

**Common Pitfall #5: Overloading with Autoboxing**

```java
public class OverloadingConfusion {
    public void process(int x) {
        System.out.println("int: " + x);
    }

    public void process(Integer x) {
        System.out.println("Integer: " + x);
    }

    public void process(long x) {
        System.out.println("long: " + x);
    }

    public void test() {
        process(5);       // Which one is called?
        process((int)5);  // int version
        process(5L);      // long version
        process(Integer.valueOf(5));  // Integer version
    }
}

// Output:
// int: 5  (primitive preferred over autoboxing)
```

**Common Pitfall #6: Overloading vs Overriding Confusion**

```java
public class Parent {
    public void process(Object obj) {
        System.out.println("Parent: Object");
    }
}

public class Child extends Parent {
    // This is OVERLOADING, not overriding!
    public void process(String str) {
        System.out.println("Child: String");
    }
}

// Usage
Parent p = new Child();
p.process("Hello");  // Prints: "Parent: Object" (not "Child: String")

// Why? At compile time, compiler sees Parent reference with Object parameter
// Resolution happens at compile time for overloading

// ✅ To override, parameter must match exactly
public class Child extends Parent {
    @Override
    public void process(Object obj) {
        if (obj instanceof String) {
            System.out.println("Child: String");
        } else {
            System.out.println("Child: Object");
        }
    }
}
```

### Method Overriding (Runtime Polymorphism)

```java
// Parent class
public abstract class Shape {
    protected String color;

    public Shape(String color) {
        this.color = color;
    }

    // Abstract method - must be implemented by subclasses
    public abstract double calculateArea();

    // Concrete method - can be overridden
    public void display() {
        System.out.println("Shape color: " + color);
    }
}

// Child class 1
public class Circle extends Shape {
    private double radius;

    public Circle(String color, double radius) {
        super(color);
        this.radius = radius;
    }

    @Override
    public double calculateArea() {
        return Math.PI * radius * radius;
    }

    @Override
    public void display() {
        super.display();
        System.out.println("Circle with radius: " + radius);
    }
}

// Child class 2
public class Rectangle extends Shape {
    private double width;
    private double height;

    public Rectangle(String color, double width, double height) {
        super(color);
        this.width = width;
        this.height = height;
    }

    @Override
    public double calculateArea() {
        return width * height;
    }

    @Override
    public void display() {
        super.display();
        System.out.println("Rectangle: " + width + " x " + height);
    }
}

// Polymorphism in action
public class PolymorphismDemo {
    public static void main(String[] args) {
        // Same reference type, different object types
        Shape shape1 = new Circle("Red", 5);
        Shape shape2 = new Rectangle("Blue", 4, 6);

        // Method called depends on actual object type (runtime)
        System.out.println(shape1.calculateArea());  // Circle's implementation
        System.out.println(shape2.calculateArea());  // Rectangle's implementation

        // Process shapes polymorphically
        List<Shape> shapes = Arrays.asList(shape1, shape2);
        for (Shape shape : shapes) {
            shape.display();
            System.out.println("Area: " + shape.calculateArea());
        }
    }
}
```

**Common Pitfall #7: Static Methods and Polymorphism**

```java
public class Parent {
    public static void staticMethod() {
        System.out.println("Parent static");
    }

    public void instanceMethod() {
        System.out.println("Parent instance");
    }
}

public class Child extends Parent {
    public static void staticMethod() {
        System.out.println("Child static");
    }

    @Override
    public void instanceMethod() {
        System.out.println("Child instance");
    }
}

// Testing
Parent p1 = new Child();
p1.staticMethod();    // Prints: "Parent static" (resolved at compile time!)
p1.instanceMethod();  // Prints: "Child instance" (resolved at runtime)

// Static methods are NOT polymorphic!
// They're resolved based on reference type, not object type
```

---

## Abstraction

**Definition:** Hiding implementation details and showing only essential features.

### Abstract Classes

```java
// Abstract class - cannot be instantiated
public abstract class Vehicle {
    protected String brand;
    protected int year;

    // Constructor in abstract class
    public Vehicle(String brand, int year) {
        this.brand = brand;
        this.year = year;
    }

    // Abstract method - no implementation
    public abstract void start();
    public abstract void stop();

    // Concrete method - has implementation
    public void displayInfo() {
        System.out.println(brand + " - " + year);
    }

    // Can have static methods
    public static void printType() {
        System.out.println("This is a vehicle");
    }
}

// Concrete class
public class Car extends Vehicle {
    private int numberOfDoors;

    public Car(String brand, int year, int doors) {
        super(brand, year);
        this.numberOfDoors = doors;
    }

    @Override
    public void start() {
        System.out.println("Car starting with key");
    }

    @Override
    public void stop() {
        System.out.println("Car stopping with brake");
    }
}

// Usage
// Vehicle v = new Vehicle("Toyota", 2020);  // ❌ Cannot instantiate abstract class
Vehicle v = new Car("Toyota", 2020, 4);  // ✅ OK
v.start();  // Calls Car's implementation
```

### Interfaces

```java
// Interface - 100% abstraction (before Java 8)
public interface Flyable {
    // All methods are public abstract by default
    void fly();
    void land();

    // Constants (public static final by default)
    int MAX_ALTITUDE = 50000;
}

// Interface with default methods (Java 8+)
public interface Drawable {
    // Abstract method
    void draw();

    // Default method - has implementation
    default void erase() {
        System.out.println("Erasing...");
    }

    // Static method
    static void printType() {
        System.out.println("This is drawable");
    }
}

// Class implementing multiple interfaces
public class Bird implements Flyable, Drawable {
    @Override
    public void fly() {
        System.out.println("Bird is flying");
    }

    @Override
    public void land() {
        System.out.println("Bird is landing");
    }

    @Override
    public void draw() {
        System.out.println("Drawing a bird");
    }

    // Can override default method
    @Override
    public void erase() {
        System.out.println("Erasing bird drawing");
    }
}
```

### Abstract Class vs Interface

```java
// ✅ Use Abstract Class when:
// - You have common code to share
// - You need constructors
// - You need protected/private members
// - You want to define non-public methods

public abstract class Employee {
    protected String name;
    protected double salary;

    // Constructor
    public Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }

    // Concrete method with implementation
    public double getAnnualSalary() {
        return salary * 12;
    }

    // Abstract method
    public abstract double calculateBonus();

    // Protected method
    protected void validateSalary() {
        if (salary < 0) throw new IllegalArgumentException();
    }
}

// ✅ Use Interface when:
// - You want to define a contract
// - You need multiple inheritance
// - You want to achieve loose coupling

public interface Payable {
    void processPayment(double amount);

    default void sendPaymentNotification() {
        System.out.println("Payment processed");
    }
}

public interface Taxable {
    double calculateTax();
}

// Class can implement multiple interfaces but extend only one class
public class Manager extends Employee implements Payable, Taxable {
    public Manager(String name, double salary) {
        super(name, salary);
    }

    @Override
    public double calculateBonus() {
        return salary * 0.2;
    }

    @Override
    public void processPayment(double amount) {
        System.out.println("Processing payment: " + amount);
    }

    @Override
    public double calculateTax() {
        return salary * 0.3;
    }
}
```

**Comparison Table:**

| Feature | Abstract Class | Interface |
|---------|---------------|-----------|
| Methods | Abstract + Concrete | Abstract + Default + Static |
| Variables | Any (instance/static) | public static final only |
| Constructor | Yes | No |
| Access Modifiers | All | public only (methods) |
| Multiple Inheritance | No (single inheritance) | Yes (multiple interfaces) |
| Speed | Fast | Slightly slower (requires extra lookup) |
| When to use | IS-A relationship + shared code | CAN-DO relationship / contract |

---

## Classes and Objects

### Constructor Chaining

```java
public class Employee {
    private String name;
    private int id;
    private double salary;

    // Constructor 1
    public Employee() {
        this("Unknown", 0, 0.0);  // Calls constructor 3
    }

    // Constructor 2
    public Employee(String name) {
        this(name, 0, 0.0);  // Calls constructor 3
    }

    // Constructor 3
    public Employee(String name, int id, double salary) {
        this.name = name;
        this.id = id;
        this.salary = salary;
    }
}

// Inheritance constructor chaining
public class Manager extends Employee {
    private String department;

    public Manager(String name, int id, double salary, String dept) {
        super(name, id, salary);  // Must be first line
        this.department = dept;
    }
}
```

**Common Pitfall #8: Constructor Call Order**

```java
public class Parent {
    private int value;

    public Parent() {
        System.out.println("Parent constructor");
        value = 10;
    }
}

public class Child extends Parent {
    private int childValue;

    public Child() {
        // super() is automatically called here even if not written
        System.out.println("Child constructor");
        childValue = 20;
    }
}

// Output when creating Child:
// Parent constructor
// Child constructor

// Order:
// 1. Parent constructor
// 2. Parent instance variables initialized
// 3. Child constructor
// 4. Child instance variables initialized
```

### Static vs Instance Members

```java
public class Counter {
    // Class variable (static) - shared across all instances
    private static int totalCount = 0;

    // Instance variable - unique to each instance
    private int instanceCount = 0;

    // Static block - runs once when class is loaded
    static {
        System.out.println("Class loaded");
        totalCount = 0;
    }

    // Instance block - runs before every constructor
    {
        System.out.println("Creating instance");
    }

    // Constructor
    public Counter() {
        totalCount++;
        instanceCount++;
    }

    // Static method - can access only static members
    public static int getTotalCount() {
        return totalCount;
        // return instanceCount;  // ❌ Error! Cannot access instance variable
    }

    // Instance method - can access both static and instance members
    public int getInstanceCount() {
        return instanceCount;  // ✅ OK
        // Can also access totalCount  // ✅ OK
    }
}

// Usage
Counter c1 = new Counter();
Counter c2 = new Counter();
System.out.println(Counter.getTotalCount());  // 2
System.out.println(c1.getInstanceCount());    // 1
System.out.println(c2.getInstanceCount());    // 1
```

**Common Pitfall #9: Static Method Overriding**

```java
public class Parent {
    public static void staticMethod() {
        System.out.println("Parent static");
    }
}

public class Child extends Parent {
    // This is METHOD HIDING, not overriding!
    public static void staticMethod() {
        System.out.println("Child static");
    }
}

// Test
Parent p = new Child();
p.staticMethod();  // Prints "Parent static" (not polymorphic!)

Child c = new Child();
c.staticMethod();  // Prints "Child static"

// Static methods are resolved at compile time based on reference type
```

### Inner Classes

```java
public class OuterClass {
    private int outerField = 10;
    private static int staticField = 20;

    // 1. Non-static inner class (member inner class)
    class InnerClass {
        public void display() {
            System.out.println("Outer field: " + outerField);  // Can access outer fields
            System.out.println("Static field: " + staticField);
        }
    }

    // 2. Static nested class
    static class StaticNestedClass {
        public void display() {
            // System.out.println(outerField);  // ❌ Cannot access non-static
            System.out.println("Static field: " + staticField);  // ✅ Can access static
        }
    }

    // 3. Local inner class (inside method)
    public void methodWithLocalClass() {
        final int localVar = 30;  // Must be final or effectively final

        class LocalInnerClass {
            public void display() {
                System.out.println("Outer: " + outerField);
                System.out.println("Local: " + localVar);
            }
        }

        LocalInnerClass local = new LocalInnerClass();
        local.display();
    }

    // 4. Anonymous inner class
    public void methodWithAnonymousClass() {
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                System.out.println("Anonymous class");
            }
        };
        runnable.run();
    }
}

// Usage
OuterClass outer = new OuterClass();

// Non-static inner class requires outer instance
OuterClass.InnerClass inner = outer.new InnerClass();
inner.display();

// Static nested class doesn't require outer instance
OuterClass.StaticNestedClass nested = new OuterClass.StaticNestedClass();
nested.display();
```

---

## Common Pitfalls Summary

### 1. Breaking Encapsulation
```java
// ❌ Return mutable reference
public List<String> getList() { return list; }

// ✅ Return defensive copy
public List<String> getList() { return new ArrayList<>(list); }
```

### 2. Inappropriate Inheritance
```java
// ❌ IS-A relationship violation
class Stack extends ArrayList { }  // Stack IS-NOT-A ArrayList

// ✅ Use composition
class Stack {
    private List elements = new ArrayList();
}
```

### 3. Overriding vs Overloading Confusion
```java
// Overriding: Same signature, different class
// Overloading: Different signature, same/different class
```

### 4. Static Methods and Polymorphism
```java
// Static methods are NOT polymorphic
// Resolved at compile time, not runtime
```

### 5. Constructor Chains
```java
// Parent constructor always runs first
// Be careful calling overridable methods in constructor
```

---

## Design Principles (SOLID)

### Single Responsibility Principle
```java
// ❌ BAD: Class has multiple responsibilities
class UserService {
    public void createUser(User user) { }
    public void sendEmail(String email) { }
    public void generateReport() { }
}

// ✅ GOOD: Each class has single responsibility
class UserService {
    public void createUser(User user) { }
}

class EmailService {
    public void sendEmail(String email) { }
}

class ReportService {
    public void generateReport() { }
}
```

### Open/Closed Principle
```java
// Open for extension, closed for modification

// ❌ BAD: Need to modify class to add new shape
class AreaCalculator {
    public double calculateArea(Object shape) {
        if (shape instanceof Circle) {
            // calculate circle area
        } else if (shape instanceof Rectangle) {
            // calculate rectangle area
        }
        // Need to modify this method for new shapes!
    }
}

// ✅ GOOD: Extend through inheritance
interface Shape {
    double calculateArea();
}

class Circle implements Shape {
    public double calculateArea() { /* implementation */ }
}

class Rectangle implements Shape {
    public double calculateArea() { /* implementation */ }
}

// No modification needed for new shapes!
```

### Liskov Substitution Principle
```java
// Subtypes must be substitutable for their base types

// ❌ BAD: Square breaks LSP
class Rectangle {
    protected int width;
    protected int height;

    public void setWidth(int width) {
        this.width = width;
    }

    public void setHeight(int height) {
        this.height = height;
    }
}

class Square extends Rectangle {
    @Override
    public void setWidth(int width) {
        this.width = width;
        this.height = width;  // Breaks expected behavior!
    }
}

// Test breaks with Square:
void testRectangle(Rectangle r) {
    r.setWidth(5);
    r.setHeight(4);
    assert r.getArea() == 20;  // Fails for Square!
}

// ✅ GOOD: Don't use inheritance when LSP is violated
interface Shape {
    double getArea();
}

class Rectangle implements Shape {
    // implementation
}

class Square implements Shape {
    // implementation
}
```

---

## Interview Questions

### Q1: What are the four pillars of OOP?
**Answer:** Encapsulation (data hiding), Inheritance (code reuse), Polymorphism (many forms), Abstraction (hiding complexity).

### Q2: Difference between abstract class and interface?
**Answer:**
- Abstract class: Can have constructors, instance variables, any access modifiers. Single inheritance.
- Interface: No constructors, only constants, public methods. Multiple inheritance.
- Use abstract class for IS-A with shared code, interface for CAN-DO contracts.

### Q3: Can we override static methods?
**Answer:** No, static methods cannot be overridden. They can be hidden (redeclared in subclass), but resolution happens at compile time based on reference type, not runtime based on object type.

### Q4: What is the diamond problem in inheritance?
**Answer:** When a class inherits from two classes that have a common ancestor, creating ambiguity about which version of inherited methods to use. Java solves this by:
- Not allowing multiple class inheritance
- Allowing multiple interface inheritance with default methods (Java 8+), which must be explicitly resolved

### Q5: What is the difference between overloading and overriding?
**Answer:**
- Overloading: Same method name, different parameters, compile-time polymorphism
- Overriding: Same signature, different classes (inheritance), runtime polymorphism

### Q6: Can constructors be inherited?
**Answer:** No, constructors are not inherited. Each subclass must have its own constructors. However, subclass constructor can call parent constructor using `super()`.

### Q7: What is the use of 'super' keyword?
**Answer:** Three uses:
1. Call parent constructor: `super(args)`
2. Access parent methods: `super.method()`
3. Access parent fields: `super.field`

### Q8: Explain IS-A vs HAS-A relationship.
**Answer:**
- IS-A: Inheritance (Dog IS-A Animal) - `extends`
- HAS-A: Composition (Car HAS-A Engine) - instance variable
- Prefer composition over inheritance when possible (more flexible)

---

## Best Practices

1. **Favor composition over inheritance** - More flexible and less coupling
2. **Program to interfaces, not implementations** - Depend on abstractions
3. **Keep classes cohesive** - Single Responsibility Principle
4. **Use access modifiers properly** - Minimize exposure (private by default)
5. **Make classes immutable when possible** - Thread-safe and predictable
6. **Override equals() and hashCode() together** - Contract requirement
7. **Use @Override annotation** - Catches mistakes at compile time
8. **Don't call overridable methods in constructor** - Subclass methods may use uninitialized fields
9. **Return defensive copies** - Don't expose internal mutable state
10. **Follow SOLID principles** - Creates maintainable, extensible code
