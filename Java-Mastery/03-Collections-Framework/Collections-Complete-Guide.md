# Java Collections Framework - Complete Guide

## Table of Contents
1. [Collections Hierarchy](#collections-hierarchy)
2. [List Interface](#list-interface)
3. [Set Interface](#set-interface)
4. [Map Interface](#map-interface)
5. [Queue and Deque](#queue-and-deque)
6. [Sorting and Comparators](#sorting-and-comparators)
7. [Collections Utility Class](#collections-utility-class)
8. [Performance Analysis](#performance-analysis)
9. [Common Pitfalls](#common-pitfalls)
10. [Interview Questions](#interview-questions)

---

## Collections Hierarchy

```
Collection (I)
├── List (I)
│   ├── ArrayList (C)
│   ├── LinkedList (C)
│   ├── Vector (C)
│   └── Stack (C)
├── Set (I)
│   ├── HashSet (C)
│   ├── LinkedHashSet (C)
│   └── SortedSet (I)
│       └── TreeSet (C)
└── Queue (I)
    ├── PriorityQueue (C)
    ├── Deque (I)
    │   ├── ArrayDeque (C)
    │   └── LinkedList (C)

Map (I) - Not part of Collection
├── HashMap (C)
├── LinkedHashMap (C)
├── Hashtable (C)
├── SortedMap (I)
│   └── TreeMap (C)
└── ConcurrentHashMap (C)

(I) = Interface
(C) = Class
```

---

## List Interface

List is an **ordered** collection that **allows duplicates** and provides **index-based access**.

### ArrayList

**Internal Structure:** Dynamic array (resizable array)

```java
// Creation
List<String> list = new ArrayList<>();  // Default capacity: 10
List<String> list2 = new ArrayList<>(100);  // Initial capacity
List<String> list3 = new ArrayList<>(Arrays.asList("A", "B", "C"));

// Basic operations
list.add("Apple");           // O(1) amortized
list.add(0, "Banana");       // O(n) - shifts elements
list.set(1, "Cherry");       // O(1) - replace element
String fruit = list.get(0);  // O(1) - random access
list.remove(0);              // O(n) - shifts elements
list.remove("Apple");        // O(n) - search + shift
int size = list.size();      // O(1)
boolean empty = list.isEmpty();  // O(1)
list.clear();                // O(n)
```

**How ArrayList Works Internally:**

```java
public class CustomArrayList<E> {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_CAPACITY = 10;

    public CustomArrayList() {
        elements = new Object[DEFAULT_CAPACITY];
    }

    public void add(E element) {
        if (size == elements.length) {
            grow();  // Increase capacity
        }
        elements[size++] = element;
    }

    private void grow() {
        int newCapacity = elements.length * 3 / 2 + 1;  // Grow by ~50%
        elements = Arrays.copyOf(elements, newCapacity);
    }

    public E get(int index) {
        if (index >= size || index < 0) {
            throw new IndexOutOfBoundsException();
        }
        return (E) elements[index];
    }

    public E remove(int index) {
        if (index >= size || index < 0) {
            throw new IndexOutOfBoundsException();
        }

        E oldValue = (E) elements[index];

        // Shift elements left
        int numMoved = size - index - 1;
        if (numMoved > 0) {
            System.arraycopy(elements, index + 1, elements, index, numMoved);
        }

        elements[--size] = null;  // Clear reference
        return oldValue;
    }
}
```

**Common Pitfall #1: ConcurrentModificationException**

```java
// ❌ WRONG: Modifying list while iterating
List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C", "D"));

for (String item : list) {
    if (item.equals("B")) {
        list.remove(item);  // ConcurrentModificationException!
    }
}

// ✅ CORRECT: Use Iterator
Iterator<String> iterator = list.iterator();
while (iterator.hasNext()) {
    String item = iterator.next();
    if (item.equals("B")) {
        iterator.remove();  // Safe removal
    }
}

// ✅ CORRECT: Use removeIf (Java 8+)
list.removeIf(item -> item.equals("B"));

// ✅ CORRECT: Collect items to remove, then remove
List<String> toRemove = new ArrayList<>();
for (String item : list) {
    if (item.equals("B")) {
        toRemove.add(item);
    }
}
list.removeAll(toRemove);
```

**Common Pitfall #2: ArrayList vs Array**

```java
// ❌ WRONG: Using array when size changes
String[] array = new String[10];
// Cannot easily add/remove elements

// ✅ CORRECT: Use ArrayList for dynamic size
List<String> list = new ArrayList<>();
list.add("Item");  // Easy addition

// Converting between ArrayList and Array
// ArrayList to Array
String[] arr = list.toArray(new String[0]);

// Array to ArrayList
List<String> fromArray = Arrays.asList(arr);  // Fixed-size list!
List<String> mutableList = new ArrayList<>(Arrays.asList(arr));  // Mutable
```

### LinkedList

**Internal Structure:** Doubly-linked list

```java
// Creation
List<String> linkedList = new LinkedList<>();

// List operations (same as ArrayList interface)
linkedList.add("Apple");
linkedList.get(0);
linkedList.remove(0);

// Deque operations (LinkedList implements Deque)
linkedList.addFirst("First");
linkedList.addLast("Last");
linkedList.removeFirst();
linkedList.removeLast();
```

**How LinkedList Works:**

```java
public class CustomLinkedList<E> {
    private Node<E> first;
    private Node<E> last;
    private int size = 0;

    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }

    public void add(E element) {
        Node<E> l = last;
        Node<E> newNode = new Node<>(l, element, null);
        last = newNode;

        if (l == null) {
            first = newNode;
        } else {
            l.next = newNode;
        }
        size++;
    }

    public E get(int index) {
        if (index < 0 || index >= size) {
            throw new IndexOutOfBoundsException();
        }

        Node<E> node;
        if (index < (size >> 1)) {  // Search from front
            node = first;
            for (int i = 0; i < index; i++) {
                node = node.next;
            }
        } else {  // Search from back
            node = last;
            for (int i = size - 1; i > index; i--) {
                node = node.prev;
            }
        }
        return node.item;
    }
}
```

### ArrayList vs LinkedList

```java
/**
 * Performance Comparison
 */

// ArrayList: Fast random access, slow insert/delete in middle
List<Integer> arrayList = new ArrayList<>();

// O(1) - add at end (amortized)
arrayList.add(1);

// O(1) - get by index
int value = arrayList.get(0);

// O(n) - add/remove in middle (shift elements)
arrayList.add(0, 1);
arrayList.remove(0);

// LinkedList: Slow random access, fast insert/delete at ends
List<Integer> linkedList = new LinkedList<>();

// O(1) - add at end
linkedList.add(1);

// O(n) - get by index (traverse nodes)
int val = linkedList.get(0);

// O(1) - add/remove at ends
((LinkedList<Integer>)linkedList).addFirst(1);
((LinkedList<Integer>)linkedList).removeFirst();

// O(n) - add/remove in middle (find + O(1) link change)
linkedList.add(5, 1);
```

**When to use what:**

| Operation | ArrayList | LinkedList |
|-----------|-----------|------------|
| Get by index | O(1) ✅ | O(n) ❌ |
| Add at end | O(1) ✅ | O(1) ✅ |
| Add at beginning | O(n) ❌ | O(1) ✅ |
| Add in middle | O(n) ❌ | O(n) ❌ |
| Remove from end | O(1) ✅ | O(1) ✅ |
| Remove from beginning | O(n) ❌ | O(1) ✅ |
| Memory overhead | Low ✅ | High (2 pointers per node) ❌ |
| Cache locality | Good ✅ | Poor ❌ |

**Rule of Thumb:**
- Use **ArrayList** for most cases (default choice)
- Use **LinkedList** only when frequent insertions/deletions at beginning

---

## Set Interface

Set is an **unordered** collection that **does NOT allow duplicates**.

### HashSet

**Internal Structure:** HashMap (values are dummy objects)

```java
// Creation
Set<String> set = new HashSet<>();
Set<String> set2 = new HashSet<>(100);  // Initial capacity
Set<String> set3 = new HashSet<>(Arrays.asList("A", "B", "C"));

// Operations - O(1) average
set.add("Apple");        // Returns false if already exists
set.add("Apple");        // Returns false, no duplicate
set.remove("Apple");     // Returns true if existed
boolean contains = set.contains("Apple");  // O(1)
int size = set.size();
set.clear();

// Iteration (order not guaranteed)
for (String item : set) {
    System.out.println(item);
}
```

**How HashSet Works:**

```java
public class CustomHashSet<E> {
    private HashMap<E, Object> map;
    private static final Object PRESENT = new Object();

    public CustomHashSet() {
        map = new HashMap<>();
    }

    public boolean add(E element) {
        return map.put(element, PRESENT) == null;
    }

    public boolean remove(E element) {
        return map.remove(element) == PRESENT;
    }

    public boolean contains(E element) {
        return map.containsKey(element);
    }

    public int size() {
        return map.size();
    }
}
```

**Common Pitfall #3: HashSet with Mutable Objects**

```java
class Person {
    String name;
    int age;

    // ... constructor

    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (!(obj instanceof Person)) return false;
        Person other = (Person) obj;
        return age == other.age && Objects.equals(name, other.name);
    }
}

// ❌ WRONG: Modifying object after adding to HashSet
Set<Person> set = new HashSet<>();
Person person = new Person("John", 30);
set.add(person);

person.age = 31;  // Modifies hashCode!

set.contains(person);  // May return false! (wrong bucket)
set.remove(person);    // May fail!

// ✅ CORRECT: Make objects immutable
class Person {
    private final String name;
    private final int age;

    // No setters - immutable
}
```

### LinkedHashSet

Maintains **insertion order** (uses doubly-linked list)

```java
Set<String> set = new LinkedHashSet<>();
set.add("C");
set.add("A");
set.add("B");

// Prints: C, A, B (insertion order maintained)
for (String item : set) {
    System.out.println(item);
}
```

### TreeSet

**Sorted** set (uses Red-Black Tree). Elements must be **Comparable** or provide **Comparator**.

```java
// Natural ordering (elements must implement Comparable)
Set<Integer> treeSet = new TreeSet<>();
treeSet.add(5);
treeSet.add(2);
treeSet.add(8);
treeSet.add(1);

// Prints: 1, 2, 5, 8 (sorted order)
for (int num : treeSet) {
    System.out.println(num);
}

// Custom ordering with Comparator
Set<String> customOrder = new TreeSet<>(Comparator.reverseOrder());
customOrder.add("Apple");
customOrder.add("Banana");
customOrder.add("Cherry");

// Prints: Cherry, Banana, Apple (reverse order)

// TreeSet specific methods - O(log n)
TreeSet<Integer> ts = new TreeSet<>();
ts.add(10); ts.add(20); ts.add(30);

ts.first();         // 10 (smallest)
ts.last();          // 30 (largest)
ts.lower(20);       // 10 (greatest < 20)
ts.higher(20);      // 30 (smallest > 20)
ts.floor(25);       // 20 (greatest <= 25)
ts.ceiling(25);     // 30 (smallest >= 25)
ts.subSet(10, 30);  // [10, 20)
```

### Set Comparison

| Feature | HashSet | LinkedHashSet | TreeSet |
|---------|---------|---------------|---------|
| Ordering | None | Insertion order | Sorted |
| Duplicates | No | No | No |
| Null elements | One null | One null | No nulls (NPE) |
| Add/Remove/Contains | O(1) | O(1) | O(log n) |
| Memory | Low | Medium | Medium |
| Use case | No order needed | Order matters | Sorted data |

---

## Map Interface

Map stores **key-value pairs**. Keys are unique.

### HashMap

**Internal Structure:** Array of buckets (linked lists/trees)

```java
// Creation
Map<String, Integer> map = new HashMap<>();
Map<String, Integer> map2 = new HashMap<>(100);  // Initial capacity

// Operations - O(1) average
map.put("Apple", 10);      // Add/Update
map.put("Apple", 20);      // Updates value
Integer value = map.get("Apple");  // 20
Integer def = map.getOrDefault("Banana", 0);  // 0 (not found)
map.remove("Apple");
boolean exists = map.containsKey("Apple");
boolean hasValue = map.containsValue(20);
int size = map.size();

// Java 8+ methods
map.putIfAbsent("Cherry", 30);  // Only if key doesn't exist
map.compute("Apple", (k, v) -> (v == null) ? 1 : v + 1);  // Increment
map.merge("Apple", 1, Integer::sum);  // Increment by 1

// Iteration
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println(entry.getKey() + " = " + entry.getValue());
}

// Java 8+ forEach
map.forEach((key, value) -> System.out.println(key + " = " + value));

// Keys and values
Set<String> keys = map.keySet();
Collection<Integer> values = map.values();
```

**How HashMap Works:**

```java
public class CustomHashMap<K, V> {
    private static final int DEFAULT_CAPACITY = 16;
    private static final float LOAD_FACTOR = 0.75f;

    private Node<K, V>[] table;
    private int size = 0;

    static class Node<K, V> {
        final K key;
        V value;
        final int hash;
        Node<K, V> next;  // For collision handling

        Node(int hash, K key, V value, Node<K, V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }
    }

    public CustomHashMap() {
        table = new Node[DEFAULT_CAPACITY];
    }

    private int hash(K key) {
        return (key == null) ? 0 : key.hashCode() ^ (key.hashCode() >>> 16);
    }

    private int indexFor(int hash, int length) {
        return hash & (length - 1);  // Equivalent to hash % length (for power of 2)
    }

    public V put(K key, V value) {
        int hash = hash(key);
        int index = indexFor(hash, table.length);

        // Check if key exists
        for (Node<K, V> node = table[index]; node != null; node = node.next) {
            if (node.hash == hash && Objects.equals(node.key, key)) {
                V oldValue = node.value;
                node.value = value;
                return oldValue;  // Update existing
            }
        }

        // Add new node
        Node<K, V> newNode = new Node<>(hash, key, value, table[index]);
        table[index] = newNode;
        size++;

        // Resize if needed
        if (size > table.length * LOAD_FACTOR) {
            resize();
        }

        return null;
    }

    public V get(K key) {
        int hash = hash(key);
        int index = indexFor(hash, table.length);

        for (Node<K, V> node = table[index]; node != null; node = node.next) {
            if (node.hash == hash && Objects.equals(node.key, key)) {
                return node.value;
            }
        }

        return null;  // Not found
    }

    private void resize() {
        Node<K, V>[] oldTable = table;
        table = new Node[oldTable.length * 2];
        size = 0;

        // Rehash all entries
        for (Node<K, V> node : oldTable) {
            while (node != null) {
                put(node.key, node.value);
                node = node.next;
            }
        }
    }
}
```

**HashMap Collision Handling (Java 8+):**

```
Bucket (Array Index)
    │
    ├─ If collisions < 8: Linked List
    │   Node1 -> Node2 -> Node3 -> ...
    │
    └─ If collisions >= 8: Red-Black Tree
        Tree for better O(log n) search
        (instead of O(n) with linked list)
```

**Common Pitfall #4: equals() and hashCode() Contract**

```java
// ❌ WRONG: Override equals() but not hashCode()
class Key {
    private String value;

    public Key(String value) {
        this.value = value;
    }

    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof Key)) return false;
        return this.value.equals(((Key)obj).value);
    }

    // Missing hashCode()!
}

Map<Key, String> map = new HashMap<>();
Key key1 = new Key("test");
Key key2 = new Key("test");

map.put(key1, "Value");
System.out.println(map.get(key2));  // null! (different hashCode)

// ✅ CORRECT: Override both equals() and hashCode()
class Key {
    private String value;

    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof Key)) return false;
        return this.value.equals(((Key)obj).value);
    }

    @Override
    public int hashCode() {
        return Objects.hash(value);
    }
}

// Now works correctly
map.put(key1, "Value");
System.out.println(map.get(key2));  // "Value"
```

**equals() and hashCode() Contract Rules:**

```java
// RULE 1: If equals() returns true, hashCode() MUST be same
obj1.equals(obj2) == true  →  obj1.hashCode() == obj2.hashCode()

// RULE 2: If hashCode() is same, equals() may or may not be true
obj1.hashCode() == obj2.hashCode()  →  obj1.equals(obj2) maybe true/false

// RULE 3: If equals() returns false, hashCode() SHOULD be different (but not required)
obj1.equals(obj2) == false  →  obj1.hashCode() != obj2.hashCode() (preferred)
```

### LinkedHashMap

Maintains **insertion order** or **access order**.

```java
// Insertion order (default)
Map<String, Integer> map = new LinkedHashMap<>();
map.put("C", 3);
map.put("A", 1);
map.put("B", 2);

// Iteration order: C, A, B
for (String key : map.keySet()) {
    System.out.println(key);
}

// Access order mode (for LRU cache)
Map<String, Integer> lruMap = new LinkedHashMap<>(16, 0.75f, true);
lruMap.put("A", 1);
lruMap.put("B", 2);
lruMap.put("C", 3);

lruMap.get("A");  // Accesses A, moves to end

// Order now: B, C, A
```

**Building LRU Cache with LinkedHashMap:**

```java
class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;

    public LRUCache(int capacity) {
        super(capacity, 0.75f, true);  // access-order mode
        this.capacity = capacity;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > capacity;  // Remove oldest when exceeds capacity
    }
}

// Usage
LRUCache<String, Integer> cache = new LRUCache<>(3);
cache.put("A", 1);
cache.put("B", 2);
cache.put("C", 3);
cache.get("A");  // Access A
cache.put("D", 4);  // Evicts B (least recently used)

System.out.println(cache);  // {C=3, A=1, D=4}
```

### TreeMap

**Sorted** map (Red-Black Tree). Keys must be **Comparable** or provide **Comparator**.

```java
// Natural ordering
Map<Integer, String> treeMap = new TreeMap<>();
treeMap.put(3, "C");
treeMap.put(1, "A");
treeMap.put(2, "B");

// Iteration in sorted key order: 1, 2, 3
for (Integer key : treeMap.keySet()) {
    System.out.println(key + " = " + treeMap.get(key));
}

// TreeMap specific methods
TreeMap<Integer, String> tm = new TreeMap<>();
tm.put(10, "Ten");
tm.put(20, "Twenty");
tm.put(30, "Thirty");

tm.firstKey();          // 10
tm.lastKey();           // 30
tm.lowerKey(20);        // 10
tm.higherKey(20);       // 30
tm.subMap(10, 30);      // {10=Ten, 20=Twenty}
```

### Map Comparison

| Feature | HashMap | LinkedHashMap | TreeMap | Hashtable |
|---------|---------|---------------|---------|-----------|
| Ordering | None | Insertion/Access | Sorted | None |
| Null key | One | One | No (NPE) | No (NPE) |
| Null values | Yes | Yes | Yes | No (NPE) |
| Synchronized | No | No | No | Yes (legacy) |
| Performance | O(1) | O(1) | O(log n) | O(1) |
| Use case | General | LRU cache | Sorted keys | Legacy (use ConcurrentHashMap) |

---

## Queue and Deque

### Queue Interface

**FIFO** (First-In-First-Out) structure.

```java
Queue<Integer> queue = new LinkedList<>();

// Offer vs Add
queue.offer(1);  // Adds element, returns false if fails
queue.add(2);    // Adds element, throws exception if fails

// Poll vs Remove
Integer item1 = queue.poll();    // Removes head, returns null if empty
Integer item2 = queue.remove();  // Removes head, throws exception if empty

// Peek vs Element
Integer peek1 = queue.peek();    // Returns head, null if empty
Integer peek2 = queue.element(); // Returns head, throws exception if empty
```

### PriorityQueue

**Heap-based** priority queue (min-heap by default).

```java
// Min heap (natural ordering)
Queue<Integer> minHeap = new PriorityQueue<>();
minHeap.offer(5);
minHeap.offer(2);
minHeap.offer(8);
minHeap.offer(1);

System.out.println(minHeap.poll());  // 1 (smallest)
System.out.println(minHeap.poll());  // 2
System.out.println(minHeap.poll());  // 5

// Max heap (reverse ordering)
Queue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
maxHeap.offer(5);
maxHeap.offer(2);
maxHeap.offer(8);

System.out.println(maxHeap.poll());  // 8 (largest)

// Custom comparator
Queue<String> customQueue = new PriorityQueue<>(
    (a, b) -> Integer.compare(a.length(), b.length())
);
customQueue.offer("Apple");
customQueue.offer("Hi");
customQueue.offer("Banana");

System.out.println(customQueue.poll());  // "Hi" (shortest)
```

### Deque Interface

**Double-Ended Queue** - can add/remove from both ends.

```java
Deque<Integer> deque = new ArrayDeque<>();

// Add to front
deque.addFirst(1);
deque.offerFirst(2);

// Add to back
deque.addLast(3);
deque.offerLast(4);

// Remove from front
deque.removeFirst();
deque.pollFirst();

// Remove from back
deque.removeLast();
deque.pollLast();

// Peek
deque.peekFirst();
deque.peekLast();

// Use as Stack (LIFO)
deque.push(1);    // addFirst
deque.pop();      // removeFirst
deque.peek();     // peekFirst
```

**ArrayDeque vs LinkedList as Deque:**

| Feature | ArrayDeque | LinkedList |
|---------|------------|------------|
| Structure | Circular array | Doubly-linked list |
| Memory | Less overhead | More overhead (2 pointers/node) |
| Cache locality | Better | Worse |
| Random access | Not supported | O(n) |
| Use case | **Preferred for Stack/Deque** | When need List operations too |

---

## Sorting and Comparators

### Comparable vs Comparator

```java
// Comparable: Natural ordering (one way to compare)
class Employee implements Comparable<Employee> {
    private String name;
    private int salary;

    @Override
    public int compareTo(Employee other) {
        return Integer.compare(this.salary, other.salary);  // Sort by salary
    }
}

// Sort using natural ordering
List<Employee> employees = new ArrayList<>();
Collections.sort(employees);  // Uses compareTo()

// Comparator: Custom ordering (multiple ways to compare)
Comparator<Employee> byName = (e1, e2) -> e1.name.compareTo(e2.name);
Comparator<Employee> bySalary = (e1, e2) -> Integer.compare(e1.salary, e2.salary);
Comparator<Employee> byNameDesc = (e1, e2) -> e2.name.compareTo(e1.name);

// Sort using custom comparator
Collections.sort(employees, byName);
Collections.sort(employees, bySalary);

// Java 8+ Comparator methods
Comparator<Employee> comparing = Comparator
    .comparing(Employee::getName)
    .thenComparing(Employee::getSalary)
    .reversed();

employees.sort(comparing);
```

**Common Pitfall #5: compareTo() Subtraction Anti-pattern**

```java
// ❌ WRONG: Subtraction can cause integer overflow
class Person implements Comparable<Person> {
    private int age;

    @Override
    public int compareTo(Person other) {
        return this.age - other.age;  // DANGEROUS!
    }
}

// Problem:
Person p1 = new Person(Integer.MAX_VALUE);
Person p2 = new Person(-1);
p1.compareTo(p2);  // Integer overflow! Returns negative (wrong!)

// ✅ CORRECT: Use Integer.compare()
@Override
public int compareTo(Person other) {
    return Integer.compare(this.age, other.age);
}

// Or for other types:
return Long.compare(this.value, other.value);
return Double.compare(this.value, other.value);
return this.string.compareTo(other.string);
```

---

## Collections Utility Class

```java
List<Integer> list = new ArrayList<>(Arrays.asList(3, 1, 4, 1, 5));

// Sorting
Collections.sort(list);  // [1, 1, 3, 4, 5]
Collections.sort(list, Comparator.reverseOrder());  // [5, 4, 3, 1, 1]

// Searching (list must be sorted!)
int index = Collections.binarySearch(list, 3);  // Returns index or -(insertion point) - 1

// Reversing
Collections.reverse(list);

// Shuffling
Collections.shuffle(list);

// Min/Max
int min = Collections.min(list);
int max = Collections.max(list);

// Frequency
int freq = Collections.frequency(list, 1);  // Count occurrences

// Fill
Collections.fill(list, 0);  // Replace all elements with 0

// Copy
List<Integer> dest = new ArrayList<>(Arrays.asList(0, 0, 0, 0, 0));
Collections.copy(dest, list);  // dest must be at least as large

// Unmodifiable views
List<Integer> unmodifiable = Collections.unmodifiableList(list);
// unmodifiable.add(1);  // Throws UnsupportedOperationException

// Synchronized wrappers
List<Integer> syncList = Collections.synchronizedList(new ArrayList<>());
synchronized(syncList) {
    for (Integer i : syncList) {  // Must synchronize iteration!
        System.out.println(i);
    }
}

// Singleton collections
Set<String> singleton = Collections.singleton("Only");
List<String> singletonList = Collections.singletonList("Only");
Map<String, Integer> singletonMap = Collections.singletonMap("Key", 1);

// Empty collections
List<String> empty = Collections.emptyList();
Set<String> emptySet = Collections.emptySet();
Map<String, Integer> emptyMap = Collections.emptyMap();
```

---

## Performance Analysis

### Time Complexity Cheat Sheet

| Operation | ArrayList | LinkedList | HashSet | TreeSet | HashMap | TreeMap |
|-----------|-----------|------------|---------|---------|---------|---------|
| Add | O(1)* | O(1) | O(1) | O(log n) | O(1) | O(log n) |
| Remove | O(n) | O(n)** | O(1) | O(log n) | O(1) | O(log n) |
| Get | O(1) | O(n) | N/A | N/A | O(1) | O(log n) |
| Contains | O(n) | O(n) | O(1) | O(log n) | O(1) | O(log n) |
| Iterate | O(n) | O(n) | O(n) | O(n) | O(n) | O(n) |

*Amortized O(1), worst case O(n) when resizing
**O(1) if removing first/last, O(n) for arbitrary position

### Space Complexity

| Collection | Space Overhead |
|------------|----------------|
| ArrayList | Low (just array) |
| LinkedList | High (2 pointers + data per node) |
| HashSet | Medium (HashMap underneath) |
| TreeSet | Medium (tree nodes) |
| HashMap | Medium (array + linked lists/trees) |

---

## Common Pitfalls Summary

### 1. ConcurrentModificationException
```java
// ❌ Modify while iterating with foreach
// ✅ Use Iterator.remove() or removeIf()
```

### 2. Arrays.asList() Returns Fixed-Size List
```java
List<String> list = Arrays.asList("A", "B");
// list.add("C");  // ❌ UnsupportedOperationException
List<String> mutable = new ArrayList<>(Arrays.asList("A", "B"));  // ✅
```

### 3. HashMap with Mutable Keys
```java
// ❌ Modifying key after insertion breaks HashMap
// ✅ Use immutable keys
```

### 4. Missing equals()/hashCode()
```java
// ❌ Override equals() without hashCode()
// ✅ Always override both together
```

### 5. compareTo() Integer Overflow
```java
// ❌ return this.value - other.value;
// ✅ return Integer.compare(this.value, other.value);
```

---

## Interview Questions

### Q1: What is the difference between ArrayList and LinkedList?
**Answer:** ArrayList uses dynamic array (fast random access O(1), slow insertions/deletions in middle O(n)). LinkedList uses doubly-linked list (slow random access O(n), fast insertions/deletions at ends O(1)). Use ArrayList by default.

### Q2: How does HashMap work internally?
**Answer:** HashMap uses array of buckets. Key's hashCode() determines bucket index. Collisions handled by linked list (or tree if many collisions). put() and get() are O(1) average. When load factor exceeded, array is resized and all entries are rehashed.

### Q3: What is the equals() and hashCode() contract?
**Answer:**
1. If equals() returns true, hashCode() must be same
2. If hashCode() is same, equals() may be true or false
3. Consistent: multiple calls should return same value if object unchanged

### Q4: Difference between Comparable and Comparator?
**Answer:** Comparable defines natural ordering (one way), implemented by the class itself (compareTo()). Comparator defines custom ordering (multiple ways), separate class (compare()). Use Comparable for default sorting, Comparator for alternative sortings.

### Q5: When would you use TreeMap over HashMap?
**Answer:** Use TreeMap when you need sorted keys. TreeMap maintains keys in sorted order (O(log n) operations). Use HashMap for better performance (O(1) operations) when order doesn't matter.

### Q6: What is fail-fast vs fail-safe iterator?
**Answer:**
- **Fail-fast**: Throws ConcurrentModificationException if collection modified during iteration (ArrayList, HashMap)
- **Fail-safe**: Works on copy, doesn't throw exception (ConcurrentHashMap, CopyOnWriteArrayList)

### Q7: Why is String commonly used as HashMap key?
**Answer:** Strings are:
- Immutable (hashCode doesn't change)
- Properly implement equals() and hashCode()
- Widely used and optimized
- Good hash distribution

### Q8: What is the load factor in HashMap?
**Answer:** Load factor (default 0.75) determines when HashMap should resize. When size > capacity * load factor, HashMap doubles capacity and rehashes all entries. Lower load factor = less collisions but more memory. Higher = more memory efficient but more collisions.

---

## Best Practices

1. **Use interfaces as variable types**: `List<String> list = new ArrayList<>();`
2. **Prefer ArrayList over LinkedList**: Unless frequent insertions at beginning
3. **Use HashMap over Hashtable**: Hashtable is synchronized (slower) and legacy
4. **Implement equals() and hashCode() together**: For custom objects in collections
5. **Use generics**: Avoid raw types for type safety
6. **Initialize with capacity**: When size is known: `new ArrayList<>(1000)`
7. **Use EnumSet for enums**: Most efficient set implementation for enums
8. **Prefer forEach over for loop**: More readable with lambdas (Java 8+)
9. **Use Collections.unmodifiableX()**: To return read-only views
10. **Use isEmpty() instead of size() == 0**: More readable and sometimes faster

---

Would you like me to continue creating guides for the remaining topics (Multithreading, Functional Programming, JVM Internals, etc.)?
