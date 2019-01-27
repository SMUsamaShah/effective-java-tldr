
# Chapter 2: Creating and destroying objects

## Item 1: Consider static factory methods instead of constructors {#item-1}

Instead of using constructors to instantiate e.g. `new Canvas(int width, int height)` static methods can be used e.g. `Canvas.Build(...)`

### Pros

* Unlike constructors, factory methods can have meaningful names.
```java
public static Canvas Build(Rectangle r) {
    return new Canvas(r.width, r.height);
}
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```
* Unlike constructors, multiple factory methods can have same signature.
* Unlike constructors, factory methods may not create new object instance each time they are invoked.
  * Useful for immutable classes, \(item 15\)
  * Cache and reuse instances
  * Singleton \(item 3\), Non-instantiable \(item 4\) 
* Unlike constructors, they can return object of any subtype
  * Factory pattern
  * Interface based frameworks \(item 18\)
  * Forces clients to use returned classes using their interface instead of implementation which is a good practice \(item 52\)
* Simplify parameterized constructor
```java
Map<String, List<String>> m = new HashMap<String, List<String>>();
public static <K, V> HashMap<K, V> newInstance() {
    return new HashMap<K, V>();
}
Map<String, List<String>> m = HashMap.newInstance();
```

### Cons

1. Class without public or protected constructor cannot be sub-classed.
   * But this also encourages to use Composition instead of Inheritance \(item 16\)
2. They do not stand-out and get mixed among other static methods.
   * These naming conventions can be followed.
     * `valueOf(TypeToConvert t)`, `of(TypeToConvert t)`, `getInstance(...)`, `newInstance(...)`, `getType(...)`, `newType(...)`

---

## Item 2: Consider builder when faced with many constructor parameters

A constructor with too many parameters is difficult to write and also difficult to read. Multiple constructors are usually created to handle optional parameters in a telescoping pattern like following.

```java
class Pizza {
    public Pizza(){}
    public Pizza(int salt){}
    public Pizza(int salt, int pepper){}
    public Pizza(int salt, int pepper, int oil){}
    public Pizza(int salt, int pepper, int oil, int layers){}
}
```

or Java Beans pattern like following

```java
class Pizza {
    public Pizza(){}
    public void setSalt(int salt){}
    public void setPepper(int pepper){}
    public void setOil(int oil){}
    public void setLayers(int layers){}
}
```

This has a disadvantage that a complete instance is created in multiple steps. Also it is difficult to create immutable object with this technique.

Builder looks like this and solves above problems.

```java
class Pizza {
    private final int salt;
    private final int size;
    private final int pepper;
    private final int oil;

    private Pizza(Builder b){
        this.salt = b.salt;
        this.pepper = b.pepper;
        this.size = b.size;
        this.oil = b. oil;
    }
    public static class Builder {
        private final int size; // required
        private int salt; // optional
        private int pepper;
        private int oil;

        Builder(int size){
            this.size = size;
        }
        public Builder salt(int val) { this.salt = val; return this; }
        public Builder pepper(int val) { this.pepper = val; return this; }
        public Builder oil(int val) { this.oil = val; return this; }
        public Pizza build() {
            return new Pizza(this);
        }
    }
}
```

this is how it is used

```java
Pizza pizza = Pizza.Builder(13).salt(1).pepper(4).build();
```

With builder we can have as many optional parameters as we like with better readability.

Moreover we can create a builder interface and pass it to a method to build objects.

```java
public interface Builder<T> {
    public T build();
}
```

Use builder when constructors or static factories have more than a handful of parameters.

### Pros

1. More readable and manageable than telescoping pattern
2. If more parameters are going to be added later in constructor, it is better to start with a builder first, because using factory methods or constructors will leave obsolete methods in your class.  
3. They are safer than JavaBeans

### Cons

1. A builder must be created first to created an object, can cause problems in performance critical situations.

---

## Item 3: Enforce singleton with a private constructor or Enum type

Singleton is a class which is instantiated only once. Before 1.5 there were two ways to make singleton.

### Method 1: Using a public static final member

```java
// Singleton with public final field
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    // ...
}
```

This guarantees only one instance but private constructor can still be accessed using reflection \(item 53\)

### Method 2: Using factory-method

```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance(){return INSTANCE;}
    // ...
}
```

This is is more flexible than public field method as `getInstance()` can be changed in future to return multiple instances e.g. unique instance for every thread. But public field method is more simple, clear and readable and often useful.

To make it _serializable_ \(chapter 11\) add `readResolve` \(item 77\) method to this class and make all instance fields `transient`.

```java
public class Elvis implements Serializable {
    private static transient Elvis INSTANCE = null;
    private Elvis() { ... }
    public static Elvis getInstance(){
        if(INSTANCE == null)
            INSTANCE = new Elvis();

        return INSTANCE;
    }

    private Object readResolve() throws ObjectStreamException{
        return INSTANCE; 
    }
}
```

### Method 3: Finally the best way to implement singleton since 1.5 is with `Enum`

```java
public Enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() { ... }
}
```

Its more concise, provides serialization and guarantee against multiple instances.

### Cons

1. Singleton is difficult to test

---

## Item 4: Enforce non-instantiability with a private constructor

Pros/Uses:

1. Group related methods on primitive values or arrays like `java.util.Arrays`
2. Group static methods and [factory-methods](#item-1) like `java.util.Collections`
3. Group methods on a final class instead of extending the class

**Do not use abstract class** to make it non-instantiable

* It can be sub-classed and then instantiated.
* It fools user to think it has to be sub-classed \(item 17\)

Make a class non-instantiable using a private constructor

```java
public class UtilityClass {
    // suppress default constructor for non-instantiability
    private UtilityClass() {
        // ensure that constructor is not accidently invoked from within class by throwing exception
        throw new AssertionError(); 
    }
    // ...
}
```

The idiom to throw exception is somewhat counter intuitive because a constructor is explicitly provided. A comment should be added to explain the reason. But it also prevents this class from being inherited.

---

## Item 5: Avoid creating unnecessary objects

Reuse the object instead of creating new equivalent object. An object can always be reused if it is _**immutable**_ \(item 15\).

```java
String s = new String("pizza" /*another string instance*/); // DON'T DO THIS
```

The argument itself is a new String instance.

```java
String s = "pizza"; // DO THIS INSTEAD
```

This way it is guaranteed that same "pizza" will be reused in same JVM elsewhere in some other running code.

[Static factory-methods](#item-1) can also be used on immutable classes to avoid object creation e.g.

```java
Boolean b = Boolean("true");         // DON'T DO THIS
Boolean b = Boolean.valueOf("true"); // DO THIS, static factory method
```

Reuse _**Mutable**_ objects if they are not going to be modified.

```java
public class Person { 
    private final Date birthDate;
    // ...

    // tell if Person is born between 1964 and 1965
    public boolean isBabyBoomer() { 
        // DON'T DO THIS!
        // Unnecessary allocation of expensive object
        Calendar gmtCal = Calendar.getInstance(TimeZone.getTimeZone("GMT")); 

        gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0); 
        Date boomStart = gmtCal.getTime(); 

        gmtCal.set(1965, Calendar.JANUARY, 1, 0, 0, 0); 
        Date boomEnd = gmtCal.getTime(); 

        return birthDate.compareTo(boomStart) >= 0 && birthDate.compareTo(boomEnd) < 0;
    }
}
```

On every call of `isBabyBoomer()` Calendar, Date and TimeZone objects are created uselessly. Do the following instead.

```java
class Person {
    private final Date birthDate;
    // ...

    private static final Date BOOM_START;
    private static final Date BOOM_END;
    static {
        Calendar gmtCal = Calendar.getInstance(TimeZone.getTimeZone("GMT"));
        gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0);
        BOOM_START = gmtCal.getTime();
        gmtCal.set(1965, Calendar.JANUARY, 1, 0, 0, 0);
        BOOM_END = gmtCal.getTime();
    }
    public boolean isBabyBoomer() {
    return birthDate.compareTo(BOOM_START) >= 0 && birthDate.compareTo(BOOM_END) < 0;
    }
}
```

Now they are created only once improving performance. Also making Date objects `static final` clearly tells they are constants. These objects can also be lazy initialized \(item 71\) but that is often an unnecessary optimization \(item 55\).

**Autoboxing** can create unnecessary objects \(item 49\).

```java
// Hideously slow program! Can you spot the object creation?
public static void main(String[] args) {
    Long sum = 0L;
    for (long i = 0; i < Integer.MAX_VALUE; i++) {
        sum += i;
    }
    System.out.println(sum);
}
```

`Long` instead of `long` will create new `Long` instance on every iteration. Therefore **prefer primitives to  
 boxed primitives, and watch out for unintentional autoboxing.**

**Note:**

* Do not avoid creation of objects if they are small \(less work in constructor\), enhance readability, clarity and power of the program
* Maintaining _object pool_ to avoid object creation is a bad idea unless those objects are extremely heavy weight
  * Database connection is heavy object, because of cost of connection, and justify object pool
  * Generally maintaining object pools clutter code, increase memory footprint and harm performance. Modern JVM's garbage collector outperforms small object pools.
* Counter rule is item 39 "Don’t reuse an existing object when you should create a new one.". Penalty of incorrectly reusing objects is far greater \(bugs, security holes\) than creating objects \(performance, style\).

---

## Item 6: Eliminate obsolete object references

Switching to managed language \(from e.g. C++\) gives a false impression that no memory management is required.

```java
// Can you spot the "memory leak"?
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }
    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }
    /**
    * Ensure space for at least one more element, roughly
    * doubling the capacity each time the array needs to grow.
    */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

The problem lies at return `elements[--size];`. On pop, references to objects are not removed from array leaving _obsolete references_. Therefore garbage collector will completely ignore them including any objects referenced by those objects causing a "memory leak". Such memory leaks are called _unintentional object retentions_.

```java
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; // Eliminate obsolete reference
    return result;
}
```

Null reference will throw exception, otherwise an obsolete reference could be dereferenced mistakenly and quietly.

**Note:**

* Best way is to let the variable with reference go out of scope. This can be done naturally by defining variables in narrowest possible scope \(item 45\)

**Rules:**

* Nulling out object reference should be the exception rather than the norm.
* Generally speaking, whenever a class manages its own memory, the programmer
   should be alert for memory leaks
* Second common source of memory leaks is caches.
  * If lifetime of entry is determined by external references to the key use `WeakHashMap`
  * A background thread can be used to clean unused entries from a cache. Or it can be done while adding entries \(e.g. using `removeEldestEntry()`  of `LinkedHashMap`\)
* Third common source of memory leaks is listeners and other callbacks
  * Callback which are registered and never deregistered will accumulate until some action is taken.
  * Best way to ensure they are garbage collected is by using _weak references_ e.g. by storing them as keys in `WeakHashMap`

Memory leaks are difficult to spot and it is better to learn problems like these. Careful code inspection and \_heap profiler \_is used to find them.

---

## Item 7: Avoid finalizers

Avoid `finalize()`. _Deprecated in Java 9._

* **Finalizers are unpredictable, often dangerous and generally unnecessary.**
  * Can cause erratic behavior, poor performance and portability problems.
* It depends on garbage collector and JVM implementation that when to execute finalizer on object. 
* **Never do anything time-critical in finalizer **e.g. close a file
* **Never depend on a finalizer to update critical persistent state **e.g. to release a persistent lock on a shared resource
* `System.gc` and `System.runFinalization` do not guarantee finalizer execution. `System.runFinalizersOnExit` and `System.runFinalizersOnExit` guarantee this but they have fatal flaws and therefore deprecated.
* Uncaught exception in finalizer is ignored terminating finalization of that object and leaving it in corrupt state.
* **There is a severe performance penalty for using finalizers.**

Solution is to **provide an explicit termination method**

* Require clients to invoke this method it is no longer needed.
* Keep track in the instance if it has been terminated in a private field. 
* Throw an `IllegalStateException` if a method of this instance is invoked after termination.
* Examples: `close` method in `InputStream`, `OutputStream`, and `java.sql.Connection`. `cancel` in `java.util.Timer`. `Graphics.dispose` and `Window.dispose` or `Image.flush` in `java.awt`.

**Explicit termination methods are typically used in combination with the try-finally construct to ensure termination** even if an exception is occurred.

```java
// try-finally block guarantees execution of termination method
Foo foo = new Foo(...);
try {
    // Do what must be done with foo
    ...
} finally {
    foo.terminate(); // Explicit termination method
}
```

Where to use finalizers?

1. Think long and hard then consider using them as a "safety-net" to call explicit termination method in case owner of object forgets to call it. Also log a warning that the resource has not been terminated to indicate a bug that needs to be fixed. `FileInputStream`, `FileOutputStream`, `Timer`, and `Connection` have finalizers as safety-net.
2. Use with native peers \(the native part of the code to which can be accessed by normal objects\) if it does not hold a critical resource. Otherwise it should also provide an explicit termination method.

How to use finalizers properly?

Finalizer chaining is not performed automatically meaning that overridden finalizer in a subclass does not call parent class finalizer automatically, except for `Object`. Overridden finalizer should call parent finalizer in finally block in try catch to make sure it gets called even on exception.

```java
// Manual finalizer chaining
@Override protected void finalize() throws Throwable {
    try {
        ... // Finalize subclass state
    } finally {
        super.finalize();
    }
}
```

Another way is to use "finalizer guardian" pattern.

```java
// Finalizer Guardian idiom
public class Foo {
    // Sole purpose of this object is to finalize outer Foo object
    private final Object finalizerGuardian = new Object() {
        @Override protected void finalize() throws Throwable {
            ... // Finalize outer Foo object
        }
    };
    ... // Remainder omitted

    // note: no finalize() in Foo itself
}

public class Child extends Foo {
    // super.finalize is not called but super class is still finalized because of inherited finalizer guardian
    @Override protected void finalize() throws Throwable {
        // cleanup code
    }
}
```

Consider this technique for every non-final public class that has a finalizer.

Use finalizers only

* as a safety net and log the invalid usage from the finalizer 
* or to terminate noncritical native resources. 

When you do

* remember to invoke super.finalize
* and for public, non-final class, consider using a finalizer guardian, so finalization can take place even if a subclass finalizer fails to invoke super.finalize.

---

# Chapter 3: Methods common to all objects

## Item 8: Obey the general contracts when overriding `equals`

Do not override equals if

* Each instance of the class in inherently unique e.g. `Thread`
* Logical equality test is not needed e.g. `java.util.Random`
* Superclass has already overridden equals and the superclass behavior is appropriate for this class e.g. `AbstractSet` for `Set`, `AbstractList` for `List` and `AbstractMap` for `Map`
* The class is private or package-private and you are sure that its equals method will never be invoked.
* The class is singleton

Override equals when

* Superclass has not has not already overridden equals for desired behaviour
* A class has notion of _logical equality_. e.g. in _value classes_

Follow this contract \(**equivalence relation**\) when overriding `equals`

* **Non-nullity:** For any non-null reference value x, `x.equals(null); // false`
* **Reflexive:**   `x.equals(x); // true`
* **Symmetric:**   `x.equals(y) == y.equals(x); // true`
* **Transitive:**  `x.equals(y) == y.equals(z) == x.equals(z); // true`
* **Consistent:**  `x.equals(y) == x.equals(y) == x.equals(y) == x.equals(y); // true`

Many classes including all collection classes depend on the objects passed to them obeying this contract. Following are some examples for each rule.

**Symmetry:**  
This is easy to break

```java
// Broken - violates symmetry!
public final class CaseInsensitiveString {
    private final String s;
    public CaseInsensitiveString(String s) {
        if (s == null)
            throw new NullPointerException();
        this.s = s;
    }
    // Broken - violates symmetry!
    @Override public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
        if (o instanceof String) // One-way interoperability!
            return s.equalsIgnoreCase((String) o);
        return false;
    }
    ... // Remainder omitted
}
```

It breaks symmetry by comparing with String which doesn't know about this class.

```java
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
String s = "polish";

cis.equals(s); // true
s.equals(cis); // false 

List<CaseInsensitiveString> list = new ArrayList<CaseInsensitiveString>();
list.add(cis);
list.contains(s); // returns false in current implementation, may return true in another implementation
```

This can be fixed by simply removing comparison with `String`

```java
@Override public boolean equals(Object o) {
    return o instanceof CaseInsensitiveString &&
        ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
```

**Transitivity:**

It's easy to break when a subclass adds a new _value component_.

```java
public class Point {
    private final int x;
    private final int y;
    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
    @Override public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;
        Point p = (Point)o;
        return p.x == x && p.y == y;
    }
    // ... 
}
```

Its subclass adds a `color` component

```java
public class ColorPoint extends Point {
    private final Color color;
    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }
    // ignores comparison with Point and therefore breaks Symmetry
    @Override public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;
        return super.equals(o) && ((ColorPoint) o).color == color;
    }
    // ... 
}
```

Broken _symmetry_

```java
Point p = new Point(1, 2);
ColorPoint cp1 = new ColorPoint(1, 2, Color.RED);
ColorPoint cp2 = new ColorPoint(1, 2, Color.BLACK);

p.equals(cp1); // true
cp1.equals(p); // false
```

In `ColorPoint`, ignore `Color` when comparing with `Point`.

```java
public class ColorPoint extends Point {
    private final Color color;
    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }
    // ignores comparison with Point and therefore breaks Symmetry
    @Override public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;

        if (!(o instanceof ColorPoint))
            return false;
        return super.equals(o) && ((ColorPoint) o).color == color;
    }
}
```

This makes it *symmetric*, but its not *transitive*

```java
p.equals(cp1);   // true
cp1.equals(p);   // true
cp1.equals(cp2); // false
```

**Method 1 to fix transitivity:** Match runtime class only

```java
public class Point {
    private final int x;
    private final int y;
    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
    // Broken - violates Liskov substitution principle
    @Override public boolean equals(Object o) {
        // if (!(o instanceof Point))
        if (o == null || o.getClass() != getClass()) // <-- 
            return false;
        Point p = (Point) o;
        return p.x == x && p.y == y;
    }
}
```

This approach will always return false for any comparison between superclass and subclass.

```java
p.equals(cp1);   // false
cp1.equals(p);   // false
cp1.equals(cp2); // false
```

This will fail even if subclass does not add any value component e.g. A subclass which counts number of instances.

```java
public class CounterPoint extends Point {
    private static final AtomicInteger counter = new AtomicInteger();
    public CounterPoint(int x, int y) {
        super(x, y);
        counter.incrementAndGet();
    }
    public int numberCreated() { return counter.get(); }
}

// example
p.equals(new CounterPoint(1, 2)); // false

// also fails for anonymous class
Point pAnon = new Point(1, 1) {
    @Override public int getY() {
        return 2;
    }
};
p.equals(pAnon); // false
```

This also violates *Liskov substitution principle* which says that any important property of a type should also hold for its subtypes, so that any method written for the type should work equally well on its subtypes.

**Method 2:** [Using canEqual](http://www.artima.com/lejava/articles/equality.html) *(not from Effective Java)*


```java
public class Point {
    private final int x;
    private final int y;
    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;

        if (!o.canEqual(this)) // <---
            return false;

        Point p = (Point) o;
        return p.x == x && p.y == y;
    }

    // override this
    public boolean canEqual(Object o) {
        return (o instanceof Point);
    }
}

public class ColorPoint extends Point {
    private final Color color;
    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }

    @Override public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;

        return ((ColorPoint) o).canEqual(this) && super.equals(o) && ((ColorPoint) o).color == color;
    }

    @Override public boolean canEqual(Object o) {
        return (o instanceof ColoredPoint);
    }
}
```

**Method 3:** Use composition instead of inheritance (item 16)

**There is no way to extend an instantiable class and add a value component while preserving the equals contract**, unless you are willing to forgo the benefits of object-oriented abstraction.

```java
// Adds a value component without violating the equals contract 
public class ColorPoint { 
    private final Point point; 
    private final Color color;
    public ColorPoint(int x, int y, Color color) { 
        if (color == null) 
            throw new NullPointerException(); 
        point = new Point(x, y); 
        this.color = color; 
    }
    /** * Returns the point-view of this color point. */ 
    public Point asPoint() { return point; }
    @Override public boolean equals(Object o) { 
        if (!(o instanceof ColorPoint)) return false; 
        ColorPoint cp = (ColorPoint) o; 
        return cp.point.equals(point) && cp.color.equals(color); 
    }
    // ...
}
```

**Java**: `java.sql.Timestamp` extends `java.util.Date` adding a `nanoseconds` field. equals implementation for `Timestamp` violates _symmetry_ and can cause problems if `Timestamp` and `Date` are used together in same collection.

Value component can be added to a subclass of an abstract class, this is important for class hierarchies (item 20). 

Consistency:

When you write a class, **think hard about whether it should be immutable (Item 15)**, if it should, make sure that your equals method enforces the restriction that equal objects remain equal and unequal objects remain unequal for all time.

Do not write an equals method that depends on unreliable sources. 

**Java**: `equals` method `java.net.URL` relies on comparison of IP which requires network access and can change.

**Non-nullity**:

Object must be unequal to `null`. Doing a null check like following is unnecessary when `instanceof` is used because it returns `false` when first operand is `null`.

```java
@Override public boolean equals(Object o) {
    if (o == null)
        return false;
    ...
}

// doing this is sufficient
@Override public boolean equals(Object o) {
    if (!(o instanceof MyType))
        return false;
    MyType mt = (MyType) o;
    ...
}
```

Recipe for high quality `equals`

1. Use the `==` operator to check if the argument is a reference to this object. If so, return true. This is just a performance optimization, but one that is worth doing if the comparison is potentially expensive. For primitive fields whose type is not `float` or `double`, use the `==` operator for comparisons; for object reference fields, invoke the `equals` method recursively; for `float` fields, use the `Float.compare` method; and for `double` fields, use `Double.compare`.
2. Use the `instanceof` operator to check if the argument has the correct type.
  If not, return false.
  If type is an interface, you must access the argument’s fields via interface methods; if the type is a class, you may be able to access the fields directly
3. Cast the argument to the correct type.
4. For each “significant” field in the class, check if that field of the argument matches the corresponding field of this object.

Return `true` if all of these succeed.

Some object reference fields may legitimately contain `null`. To avoid the possibility
of a `NullPointerException`, use this idiom to compare such fields:

    (field == null ? o.field == null : field.equals(o.field))

This alternative may be faster if `field` and `o.field` are often identical:

    (field == o.field || (field != null && field.equals(o.field)))
Performance of `equals` is effected by order of comparison, therefore compare less expensive and more likely to differ fields first.  Comparison of *redundant/calculated* fields is not necessary but it can improve performance where calculate field is enough to differentiate two objects correctly.

5.  After writing `equals`, check that is it symmetric? Is it transitive? Is it consistent? Write unit tests to ensure.



*   Always override `hashCode`  when overriding `equals`

*   Don't be too clever. E.g. `File` class should not equate symbolic links of same files.

*   Don't use another type in equals declaration like following

    ```java
    public boolean equals(MyObject obj) // Note 'MyObject' here instead of 'Object'
    {
      ...
    }
    ```

    Use `@Override` notation (item 36) to avoid this mistake.

---

## Item 9: Always override `hashCode` when  you override `equals`

The equal objects must have equal hash codes.

*   `hashCode` must return same integer on multiple invocations of within same execution of application if information used for `equals` is not changed.
*   If two objects are equal according to `equals`, than `hashCode` must also return same integer for both.
*   For unequal objects, hashCode is not required to be unequal. However, it can improve performance of hash tables.
*   ​

