# Data Types and Variables in Java

## Table of Contents
1. [Primitive Data Types](#primitive-data-types)
2. [Reference Types](#reference-types)
3. [Type Conversion and Casting](#type-conversion-and-casting)
4. [Variables and Scope](#variables-and-scope)
5. [Common Pitfalls](#common-pitfalls)
6. [Interview Questions](#interview-questions)

---

## Primitive Data Types

Java has 8 primitive data types that hold actual values in memory.

### Integer Types

```java
// byte: 8-bit signed integer (-128 to 127)
byte age = 25;
byte minValue = -128;
byte maxValue = 127;

// short: 16-bit signed integer (-32,768 to 32,767)
short year = 2025;
short minShort = -32768;
short maxShort = 32767;

// int: 32-bit signed integer (-2^31 to 2^31-1)
int population = 1_000_000;  // Underscores for readability (Java 7+)
int minInt = Integer.MIN_VALUE;  // -2,147,483,648
int maxInt = Integer.MAX_VALUE;  //  2,147,483,647

// long: 64-bit signed integer
long distance = 9_223_372_036_854_775_807L;  // Note the 'L' suffix
long worldPopulation = 8_000_000_000L;
```

**Common Pitfall #1: Integer Overflow**

```java
// ❌ WRONG: Integer overflow
int maxInt = Integer.MAX_VALUE;
int overflow = maxInt + 1;
System.out.println(overflow);  // -2147483648 (wraps around!)

// ✅ CORRECT: Use long or check for overflow
long safeValue = (long)maxInt + 1;
System.out.println(safeValue);  // 2147483648

// Check for overflow before operation
public static int addSafe(int a, int b) {
    if (a > 0 && b > Integer.MAX_VALUE - a) {
        throw new ArithmeticException("Integer overflow");
    }
    if (a < 0 && b < Integer.MIN_VALUE - a) {
        throw new ArithmeticException("Integer underflow");
    }
    return a + b;
}
```

### Floating-Point Types

```java
// float: 32-bit IEEE 754 floating point
float price = 19.99f;  // Note the 'f' suffix
float pi = 3.14159f;

// double: 64-bit IEEE 754 floating point (default for decimals)
double precisePrice = 19.99;
double scientificNotation = 1.23e-4;  // 0.000123
```

**Common Pitfall #2: Floating-Point Precision**

```java
// ❌ WRONG: Floating-point arithmetic is not precise
double a = 0.1;
double b = 0.2;
double result = a + b;
System.out.println(result);  // 0.30000000000000004 (NOT 0.3!)

// ❌ WRONG: Comparing doubles with ==
if (result == 0.3) {  // This will be FALSE!
    System.out.println("Equal");
}

// ✅ CORRECT: Use BigDecimal for precise calculations
import java.math.BigDecimal;

BigDecimal x = new BigDecimal("0.1");
BigDecimal y = new BigDecimal("0.2");
BigDecimal sum = x.add(y);
System.out.println(sum);  // 0.3 (exact!)

// ✅ CORRECT: Compare with epsilon for doubles
double epsilon = 0.0001;
if (Math.abs(result - 0.3) < epsilon) {
    System.out.println("Equal within tolerance");
}
```

**Common Pitfall #3: Financial Calculations**

```java
// ❌ WRONG: Never use float/double for money
double money = 1.20;
double discount = 0.10;
double finalPrice = money - discount;
System.out.println(finalPrice);  // 1.0999999999999999

// ✅ CORRECT: Use BigDecimal for money
BigDecimal amount = new BigDecimal("1.20");
BigDecimal discountAmount = new BigDecimal("0.10");
BigDecimal final = amount.subtract(discountAmount);
System.out.println(final);  // 1.10

// ✅ CORRECT: Or use cents (int)
int priceInCents = 120;  // $1.20
int discountInCents = 10;  // $0.10
int finalInCents = priceInCents - discountInCents;  // 110 ($1.10)
```

### Character Type

```java
// char: 16-bit Unicode character
char letter = 'A';
char unicode = '\u0041';  // 'A' in Unicode
char newline = '\n';
char tab = '\t';

// Characters are actually numbers
char a = 'A';
System.out.println((int)a);  // 65
char nextChar = (char)(a + 1);
System.out.println(nextChar);  // 'B'
```

**Common Pitfall #4: Char vs String**

```java
// ❌ WRONG: Mixing char and String
char c = "A";  // Compilation error! "A" is a String
String s = 'A';  // Compilation error! 'A' is a char

// ✅ CORRECT
char c = 'A';
String s = "A";

// Converting between char and String
char toChar = s.charAt(0);
String fromChar = String.valueOf(c);
String fromChar2 = Character.toString(c);
```

### Boolean Type

```java
// boolean: true or false
boolean isActive = true;
boolean isValid = false;

// Boolean expressions
boolean canVote = age >= 18;
boolean isEligible = isActive && canVote;
```

**Common Pitfall #5: Boolean Evaluation**

```java
// ❌ WRONG: Verbose boolean check
if (isActive == true) {  // Redundant
    // do something
}

// ✅ CORRECT
if (isActive) {
    // do something
}

// ❌ WRONG: Comparing to boolean literal
boolean result = (x > 5) == true;

// ✅ CORRECT
boolean result = x > 5;

// ❌ WRONG: Ternary with boolean literals
boolean isAdult = age >= 18 ? true : false;

// ✅ CORRECT
boolean isAdult = age >= 18;
```

---

## Reference Types

Reference types store references (memory addresses) to objects, not the actual values.

### Strings

```java
// String is a reference type (immutable)
String name = "John";
String greeting = "Hello";
String full = greeting + " " + name;  // Creates new String object

// String literal vs new String()
String s1 = "Hello";  // Stored in String pool
String s2 = "Hello";  // Points to same object in pool
String s3 = new String("Hello");  // New object in heap

System.out.println(s1 == s2);  // true (same reference)
System.out.println(s1 == s3);  // false (different references)
System.out.println(s1.equals(s3));  // true (same content)
```

**Common Pitfall #6: String Comparison**

```java
// ❌ WRONG: Using == for String comparison
String input = scanner.nextLine();  // User enters "hello"
if (input == "hello") {  // DON'T DO THIS! May be false
    System.out.println("Match");
}

// ✅ CORRECT: Use .equals()
if (input.equals("hello")) {
    System.out.println("Match");
}

// ✅ For case-insensitive comparison
if (input.equalsIgnoreCase("hello")) {
    System.out.println("Match");
}

// ✅ Check for null first
if (input != null && input.equals("hello")) {
    System.out.println("Match");
}

// Or use Java 7+ Objects utility
if (Objects.equals(input, "hello")) {  // Null-safe
    System.out.println("Match");
}
```

**Common Pitfall #7: String Immutability**

```java
// ❌ WRONG: Inefficient string concatenation in loop
String result = "";
for (int i = 0; i < 1000; i++) {
    result += i;  // Creates 1000 new String objects!
}

// ✅ CORRECT: Use StringBuilder
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append(i);
}
String result = sb.toString();

// When to use what:
// String - Immutable, thread-safe, use when content won't change
// StringBuilder - Mutable, NOT thread-safe, use in single-threaded scenarios
// StringBuffer - Mutable, thread-safe, use in multi-threaded scenarios
```

### Arrays

```java
// Array declaration and initialization
int[] numbers = new int[5];  // All elements initialized to 0
int[] primes = {2, 3, 5, 7, 11};  // Array literal

// Multi-dimensional arrays
int[][] matrix = new int[3][3];
int[][] grid = {
    {1, 2, 3},
    {4, 5, 6},
    {7, 8, 9}
};

// Array length
System.out.println(primes.length);  // 5
```

**Common Pitfall #8: Array Index Out of Bounds**

```java
// ❌ WRONG: Off-by-one error
int[] arr = {1, 2, 3, 4, 5};
for (int i = 0; i <= arr.length; i++) {  // <= causes error!
    System.out.println(arr[i]);  // ArrayIndexOutOfBoundsException
}

// ✅ CORRECT: Use < instead of <=
for (int i = 0; i < arr.length; i++) {
    System.out.println(arr[i]);
}

// ✅ BETTER: Use enhanced for loop
for (int num : arr) {
    System.out.println(num);
}

// ✅ BEST: Use streams (Java 8+)
Arrays.stream(arr).forEach(System.out::println);
```

**Common Pitfall #9: Array vs ArrayList**

```java
// Arrays: Fixed size, can hold primitives
int[] arr = new int[5];
// arr.add(10);  // ❌ No add method!

// ArrayList: Dynamic size, only objects (no primitives)
ArrayList<Integer> list = new ArrayList<>();
list.add(10);  // ✅ Works fine
list.add(20);

// Converting between array and list
Integer[] array = {1, 2, 3};
List<Integer> listFromArray = Arrays.asList(array);

List<Integer> list2 = new ArrayList<>();
list2.add(1); list2.add(2);
Integer[] arrayFromList = list2.toArray(new Integer[0]);
```

---

## Type Conversion and Casting

### Implicit Casting (Widening)

Automatic conversion from smaller to larger type.

```java
// Widening: byte -> short -> int -> long -> float -> double
byte b = 10;
int i = b;  // Automatic widening
long l = i;  // Automatic widening
double d = l;  // Automatic widening

System.out.println(d);  // 10.0
```

### Explicit Casting (Narrowing)

Manual conversion from larger to smaller type.

```java
// Narrowing requires explicit cast
double d = 9.99;
int i = (int)d;  // Explicit cast, truncates decimal
System.out.println(i);  // 9 (loses precision)

long l = 1000L;
int i2 = (int)l;  // May lose data if l > Integer.MAX_VALUE
```

**Common Pitfall #10: Data Loss in Casting**

```java
// ❌ WRONG: Casting without checking bounds
long bigNumber = 3_000_000_000L;  // Larger than int max
int smallNumber = (int)bigNumber;  // Data loss!
System.out.println(smallNumber);  // -1294967296 (wrong!)

// ✅ CORRECT: Check bounds before casting
if (bigNumber >= Integer.MIN_VALUE && bigNumber <= Integer.MAX_VALUE) {
    int safeNumber = (int)bigNumber;
} else {
    throw new IllegalArgumentException("Value out of int range");
}

// ✅ CORRECT: Use wrapper class methods
int safeNumber = Math.toIntExact(bigNumber);  // Throws ArithmeticException if overflow
```

### Autoboxing and Unboxing

```java
// Autoboxing: primitive -> wrapper object
int primitive = 42;
Integer object = primitive;  // Autoboxing

// Unboxing: wrapper object -> primitive
Integer obj = 100;
int prim = obj;  // Unboxing

// Works in collections
List<Integer> numbers = new ArrayList<>();
numbers.add(10);  // Autoboxing int -> Integer
int value = numbers.get(0);  // Unboxing Integer -> int
```

**Common Pitfall #11: NullPointerException with Unboxing**

```java
// ❌ WRONG: Unboxing null causes NPE
Integer number = null;
int value = number;  // NullPointerException!

// ✅ CORRECT: Check for null
Integer number = null;
int value = (number != null) ? number : 0;

// ✅ Better: Use Optional (Java 8+)
Optional<Integer> optional = Optional.ofNullable(number);
int value = optional.orElse(0);
```

**Common Pitfall #12: Performance with Autoboxing**

```java
// ❌ WRONG: Unnecessary autoboxing in loop
Long sum = 0L;  // Wrapper type
for (long i = 0; i < 1000000; i++) {
    sum += i;  // Boxing and unboxing on every iteration!
}

// ✅ CORRECT: Use primitive type
long sum = 0L;  // Primitive
for (long i = 0; i < 1000000; i++) {
    sum += i;  // No boxing/unboxing
}

// Performance difference can be significant!
```

---

## Variables and Scope

### Types of Variables

```java
public class VariablesDemo {
    // 1. Instance variables (non-static fields)
    private int instanceVar = 10;

    // 2. Class variables (static fields)
    private static int classVar = 20;

    // 3. Constants (final variables)
    private static final int CONSTANT = 100;

    public void method(int parameter) {  // 4. Parameters
        // 5. Local variables
        int localVar = 30;

        System.out.println(instanceVar);  // Accessible
        System.out.println(classVar);     // Accessible
        System.out.println(CONSTANT);     // Accessible
        System.out.println(parameter);    // Accessible
        System.out.println(localVar);     // Accessible
    }
}
```

### Variable Scope

```java
public class ScopeDemo {
    private int x = 10;  // Instance scope

    public void method() {
        int y = 20;  // Method scope

        if (true) {
            int z = 30;  // Block scope
            System.out.println(x);  // ✅ Accessible
            System.out.println(y);  // ✅ Accessible
            System.out.println(z);  // ✅ Accessible
        }

        // System.out.println(z);  // ❌ z out of scope!
    }
}
```

**Common Pitfall #13: Variable Shadowing**

```java
public class Shadowing {
    private int value = 10;

    public void method(int value) {  // Parameter shadows instance variable
        System.out.println(value);      // Prints parameter value
        System.out.println(this.value); // Prints instance variable

        value = 20;  // Modifies parameter, NOT instance variable
    }

    // ❌ WRONG: Confusing shadowing
    public void confusing() {
        int value = 5;  // Local variable shadows instance variable
        System.out.println(value);  // Which value? (local = 5)
    }

    // ✅ CORRECT: Avoid shadowing by using different names
    public void clear(int inputValue) {
        System.out.println(inputValue);  // Clear intent
        System.out.println(this.value);  // Clear intent
    }
}
```

### Final Variables

```java
// Final variables can be assigned only once
final int CONSTANT = 100;
// CONSTANT = 200;  // ❌ Compilation error!

// Final instance variable must be initialized
public class FinalDemo {
    private final int x = 10;  // Initialized at declaration

    private final int y;
    public FinalDemo(int y) {
        this.y = y;  // Initialized in constructor
    }

    private final int z;
    {
        z = 30;  // Initialized in instance initializer block
    }
}

// Final reference vs final object
final StringBuilder sb = new StringBuilder("Hello");
sb.append(" World");  // ✅ Can modify object
// sb = new StringBuilder();  // ❌ Cannot reassign reference
```

---

## Common Pitfalls Summary

### 1. Integer Overflow/Underflow
```java
// Always check bounds or use larger type
int result = Integer.MAX_VALUE + 1;  // ❌ Overflow
long result = (long)Integer.MAX_VALUE + 1;  // ✅ Safe
```

### 2. Floating-Point Precision
```java
// Use BigDecimal for precision
double result = 0.1 + 0.2;  // ❌ 0.30000000000000004
BigDecimal result = new BigDecimal("0.1").add(new BigDecimal("0.2"));  // ✅ 0.3
```

### 3. String Comparison
```java
// Use .equals(), not ==
if (s1 == s2) { }  // ❌ Compares references
if (s1.equals(s2)) { }  // ✅ Compares content
```

### 4. String Concatenation in Loops
```java
// Use StringBuilder for multiple concatenations
String s = "";
for (int i = 0; i < 1000; i++) s += i;  // ❌ Slow
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) sb.append(i);  // ✅ Fast
```

### 5. Array Index Bounds
```java
// Use < instead of <=
for (int i = 0; i <= arr.length; i++) { }  // ❌ IndexOutOfBounds
for (int i = 0; i < arr.length; i++) { }  // ✅ Correct
```

### 6. NullPointerException with Unboxing
```java
// Check for null before unboxing
Integer obj = null;
int value = obj;  // ❌ NullPointerException
int value = (obj != null) ? obj : 0;  // ✅ Safe
```

### 7. Data Loss in Casting
```java
// Check bounds before narrowing cast
long l = 3_000_000_000L;
int i = (int)l;  // ❌ Data loss
int i = Math.toIntExact(l);  // ✅ Throws exception if overflow
```

---

## Interview Questions

### Q1: What is the difference between int and Integer?
**Answer:** `int` is a primitive type that stores actual values. `Integer` is a wrapper class (reference type) that wraps an int value in an object. Integer provides utility methods and can be null, while int cannot.

### Q2: Why can't we use == to compare Strings?
**Answer:** `==` compares references (memory addresses), not content. Two String objects with the same content may have different references. Use `.equals()` to compare content.

### Q3: What is the output of: `System.out.println(0.1 + 0.2);`?
**Answer:** `0.30000000000000004` - This is due to floating-point precision limitations. Use BigDecimal for precise decimal calculations.

### Q4: Explain autoboxing and unboxing.
**Answer:** Autoboxing is automatic conversion of primitives to wrapper objects (int → Integer). Unboxing is the reverse (Integer → int). Introduced in Java 5 for convenience.

### Q5: What happens when you cast a long to int if the long value exceeds int range?
**Answer:** Data loss occurs. The bits are truncated, resulting in incorrect value. Use `Math.toIntExact()` to throw exception on overflow.

### Q6: Why is String immutable in Java?
**Answer:**
- Security: Strings are used in class loading, prevents malicious code
- Thread-safety: Multiple threads can access without synchronization
- String pool: Enables sharing of String literals
- Hashing: String hashcode is cached, efficient for HashMap keys

### Q7: What is the difference between `final`, `finally`, and `finalize`?
**Answer:**
- `final`: Keyword for constants, prevents inheritance/overriding
- `finally`: Block that executes after try-catch, used for cleanup
- `finalize()`: Method called by GC before object destruction (deprecated)

### Q8: Explain the String pool.
**Answer:** String pool is a special memory region in heap where String literals are stored. When you create a String literal, JVM checks if it exists in pool. If yes, returns reference; if no, creates new String and adds to pool. Saves memory by sharing common Strings.

```java
String s1 = "Hello";  // Added to pool
String s2 = "Hello";  // Returns reference from pool
System.out.println(s1 == s2);  // true

String s3 = new String("Hello");  // Creates new object in heap
System.out.println(s1 == s3);  // false
```

---

## Best Practices

1. **Use appropriate data types**: Don't use `int` for everything; choose the smallest type that fits your needs
2. **Avoid magic numbers**: Use named constants instead of literal values
3. **Use BigDecimal for money**: Never use float/double for financial calculations
4. **Prefer primitives over wrappers**: For performance-critical code
5. **Check for null**: Before unboxing or accessing wrapper objects
6. **Use meaningful variable names**: `customerAge` not `x`
7. **Initialize variables**: Don't rely on default values
8. **Use final when possible**: Makes code more predictable and thread-safe
9. **Avoid variable shadowing**: Use different names to prevent confusion
10. **Use StringBuilder for multiple concatenations**: Not String +=
