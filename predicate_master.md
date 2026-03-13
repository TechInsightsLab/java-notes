# Predicate
In Java, a Predicate is a functional interface introduced in Java 8 within the java.util.function package.\
It represents a boolean-valued function that takes exactly one argument and returns either true or false.

## Core Components
- **The test(T t) Method:** This is the single abstract method (SAM) of the interface. It evaluates the given argument against a specific condition.
- **Functional Nature:** Because it is a functional interface, it can be used as the assignment target for a lambda expression or method reference. 

## Built-in Default & Static Methods 
Predicates allow for complex logic through method chaining without writing nested if-else blocks:
- **and(Predicate other):** Performs a logical AND between two predicates.
- **or(Predicate other):** Performs a logical OR between two predicates.
- **negate()/not():** Returns a predicate that is the logical NOT of the original.
- **isEqual(Object targetRef):** A static method that returns a predicate testing if two objects are equal based on Objects.equals().

## Common Use Cases
- **Filtering Streams:** The most common use is in the Java Stream API, specifically with the .filter() method to extract elements that meet certain criteria.
- **Collection Removal:** Used with methods like Collection.removeIf(Predicate filter) to remove elements from a list or set that satisfy a condition.
- **Data Validation:** Encapsulating validation rules (e.g., checking if a username is valid or if a number is even) into reusable objects.

## Related Specialized Interfaces
To avoid the overhead of auto-boxing primitive types, Java provides specialized versions:
- **IntPredicate:** Takes an int and returns a boolean.
- **LongPredicate:** Takes a long and returns a boolean.
- **DoublePredicate:** Takes a double and returns a boolean.
- **BiPredicate<T, U>:** Takes two arguments of different types and returns a boolean. 

## Default Methods (Logic Chaining)
Default methods allow you to combine multiple predicates into a single, complex rule without deeply nested if statements.

```
import java.util.function.Predicate;

public class DefaultMethodExample {
    public static void main(String[] args) {
        Predicate<String> startsWithA = s -> s.startsWith("A");
        Predicate<String> endsWithZ = s -> s.endsWith("Z");

        // AND: Both conditions must be true
        Predicate<String> andChain = startsWithA.and(endsWithZ);
        System.out.println(andChain.test("ALANIZ")); // true

        // OR: At least one condition must be true
        Predicate<String> orChain = startsWithA.or(s -> s.length() > 5);
        System.out.println(orChain.test("BOBBY_LONG")); // true

        // NEGATE: Inverts the result (NOT)
        Predicate<String> isNotA = startsWithA.negate();
        System.out.println(isNotA.test("APPLE")); // false
    }
}
```
## Static Methods (Factory Helpers)
Static methods provide pre-built logic for common tasks like equality or inversion.
- **isEqual(Object target):** Returns a predicate that tests if an input is equal to the target using Objects.equals().
- **not(Predicate target):** (Added in Java 11) A static alternative to .negate() that is often cleaner when used in method references.

```
// Check if a string is exactly "Secret"
Predicate<String> isSecret = Predicate.isEqual("Secret");
System.out.println(isSecret.test("Secret")); // true

// Java 11+ Static Negation
Predicate<String> isNotEmpty = Predicate.not(String::isEmpty);
```
## Special Predicate Interfaces
Java provides specialized interfaces to handle two arguments or to avoid the performance cost of autoboxing (converting primitives like int to objects like Integer).
- **BiPredicate<T, U>**	Two (can be different types)	boolean	Comparing two different objects.
- **IntPredicate**	One (int)	boolean	Performance-critical integer math.
- **LongPredicate**	One (long)	boolean	Working with large numeric IDs or timestamps.
- **DoublePredicate**	One (double)	boolean	Precision-based math or measurements.

**Example of _BiPredicate_ and _IntPredicate_:**

```
import java.util.function.*;

public class SpecialInterfaceExample {
    public static void main(String[] args) {
        // BiPredicate: Checking if a string's length matches a number
        BiPredicate<String, Integer> checkLength = (str, len) -> str.length() == len;
        System.out.println(checkLength.test("Java", 4)); // true

        // IntPredicate: High-performance primitive check
        IntPredicate isEven = n -> n % 2 == 0;
        System.out.println(isEven.test(10)); // true
    }
}

```
## Complex Object Filtering
can define multiple predicates for a custom object and combine them to create clean, readable filters.

```
import java.util.*;
import java.util.function.Predicate;
import java.util.stream.Collectors;

class Employee {
    String name;
    int age;
    String department;

    // Constructor, Getters, and toString...
}

public class StreamFilterExample {
    public static void main(String[] args) {
        List<Employee> employees = Arrays.asList(
            new Employee("Alice", 30, "IT"),
            new Employee("Bob", 25, "HR"),
            new Employee("Charlie", 35, "IT"),
            new Employee("David", 22, "IT")
        );

        // Define reusable Predicates
        Predicate<Employee> isIT = e -> "IT".equals(e.department);
        Predicate<Employee> isSenior = e -> e.age > 28;

        // Apply multiple predicates using .and()
        List<Employee> seniorIT = employees.stream()
            .filter(isIT.and(isSenior)) 
            .collect(Collectors.toList());

        System.out.println(seniorIT); // Alice and Charlie
    }
}
```

## Advanced Default & Static Methods
- **Predicate.isEqual(Object):** Useful for finding specific objects within a stream.
- **Predicate.not(Predicate):** (Java 11+) Inverts a condition. This is often more readable than using .negate() or !.

```
// Filtering for employees who are NOT in IT
List<Employee> nonIT = employees.stream()
    .filter(Predicate.not(isIT))
    .collect(Collectors.toList());
```
## Specialized Predicates for Performance
When working with primitive streams (like IntStream or DoubleStream), always use the specialized interfaces to avoid the performance cost of autoboxing.
```
import java.util.stream.IntStream;
import java.util.function.IntPredicate;

// Efficiently filtering primitive ints
IntPredicate isPrime = n -> {
    if (n < 2) return false;
    for (int i = 2; i <= Math.sqrt(n); i++) {
        if (n % i == 0) return false;
    }
    return true;
};

IntStream.range(1, 100)
    .filter(isPrime)
    .forEach(System.out::println);
```

## Summary of Common Usage
- **.filter(predicate)**	The primary way to apply a condition to each element.
- **.anyMatch(predicate)**	Returns true if any element matches (short-circuits).
- **.allMatch(predicate)**	Returns true only if all elements match.
- **.noneMatch(predicate)**	Returns true if no elements match the condition.

## Dynamic Chaining with reduce()
The Stream `_reduce()_` method can fold a collection of predicates into a single composite predicate. 
- **Logic AND:** Start with an identity that always returns true (so it doesn't change the first real condition).
- **Logic OR:** Start with an identity that always returns false.
```
import java.util.*;
import java.util.function.Predicate;

public class DynamicPredicateExample {
    public static void main(String[] args) {
        List<Predicate<String>> filters = new ArrayList<>();
        
        // Simulating dynamic user input
        filters.add(s -> s.length() > 3);
        filters.add(s -> s.contains("a"));
        filters.add(s -> s.endsWith("e"));

        // Combine all using AND
        Predicate<String> combinedAnd = filters.stream()
            .reduce(x -> true, Predicate::and); // Identity is true for AND

        // Combine all using OR
        Predicate<String> combinedOr = filters.stream()
            .reduce(x -> false, Predicate::or); // Identity is false for OR

        List<String> names = List.of("Apple", "Ace", "Banana", "Blue");
        
        System.out.println("Matches ALL: " + names.stream().filter(combinedAnd).toList()); 
        // Result: [Apple]
        
        System.out.println("Matches ANY: " + names.stream().filter(combinedOr).toList()); 
        // Result: [Apple, Ace, Banana]
    }
}
```
## Performance Tip: orElse vs. Identity
Using an identity like `x -> true` is simple but adds one extra function call (the identity itself). For massive streams, you can use the `Optional` version of reduce to avoid this overhead:
```
Predicate<String> optimizedAnd = filters.stream()
    .reduce(Predicate::and)          // Returns Optional<Predicate>
    .orElse(x -> true);              // Default if list is empty
```
## Why use this?
- **Scalability:** Handles 0, 1, or 100 filters without changing code structure.
- **Clean Streams:** Keeps your Stream .filter() call readable by passing a single object instead of a long chain of logic.
- **Validation Engines:** Allows you to build rulesets at runtime based on configuration files or database entries.

## Dynamic BiPredicate Combination
to compare two different types of data—like matching a User against a SecurityPolicy—you use BiPredicate.\
Combining these dynamically follows the same logic as standard Predicates but handles two input types.\

Because BiPredicate does not have an easy static "identity" method like Predicate.isEqual, you define your own "always true" or "always false" starting point.
```
import java.util.*;
import java.util.function.BiPredicate;

public class DynamicBiPredicate {
    public static void main(String[] args) {
        // Example: Checking if a User is authorized for a Resource
        List<BiPredicate<String, String>> rules = new ArrayList<>();

        // Dynamic rule 1: Username must be longer than 3 chars
        rules.add((user, res) -> user.length() > 3);
        // Dynamic rule 2: Resource must start with "ADMIN_"
        rules.add((user, res) -> res.startsWith("ADMIN_"));

        // Combine all rules using AND
        BiPredicate<String, String> combinedRules = rules.stream()
            .reduce((u, r) -> true, BiPredicate::and);

        System.out.println(combinedRules.test("Alice", "ADMIN_DASHBOARD")); // true
        System.out.println(combinedRules.test("Bob", "GUEST_PAGE"));        // false
    }
}
```

## The "Cross-List" Scenario
A common real-world use case for BiPredicate is checking if elements in List A satisfy a condition relative to List B.
```
List<String> users = List.of("Alice", "Bob", "Charlie");
List<String> resources = List.of("ADMIN_PANEL", "USER_PROFILE");

BiPredicate<String, String> canAccess = (user, res) -> 
    user.equals("Alice") || !res.contains("ADMIN");

// Find all valid User-Resource pairs
users.forEach(u -> 
    resources.stream()
        .filter(r -> canAccess.test(u, r))
        .forEach(r -> System.out.println(u + " can access " + r))
);
```

## Key Considerations
- **Complexity**: While Predicate has negate(), BiPredicate also has and(), or(), and negate() as default methods, allowing for equally complex logic chains.
- **Stateful Rules:** If your dynamic rules depend on external state (like a database), ensure your BiPredicate remains thread-safe if used in a parallelStream().

## Custom functional interface
When two arguments aren't enough, you can create a TriPredicate. Since Java only provides up to BiPredicate out of the box,\
you must define your own functional interface and annotate it with @FunctionalInterface.

### Defining the TriPredicate
To make it as powerful as the built-in versions, you should manually implement and, or, and negate as default methods.

```
@FunctionalInterface
public interface TriPredicate<T, U, V> {
    // The Single Abstract Method (SAM)
    boolean test(T t, U u, V v);

    // Default 'and' method for chaining
    default TriPredicate<T, U, V> and(TriPredicate<? super T, ? super U, ? super V> other) {
        Objects.requireNonNull(other);
        return (t, u, v) -> test(t, u, v) && other.test(t, u, v);
    }

    // Default 'negate' method
    default TriPredicate<T, U, V> negate() {
        return (t, u, v) -> !test(t, u, v);
    }
}
```
### Practical Implementation
Imagine a banking system where a transaction requires a User, an Amount, and a Location.
```
public class CustomInterfaceExample {
    public static void main(String[] args) {
        // Implementation via Lambda
        TriPredicate<String, Double, String> isFlagged = (user, amount, location) -> 
            amount > 10000 && !location.equals("HOME_COUNTRY");

        // Usage
        boolean result = isFlagged.test("JohnDoe", 15000.0, "Foreign_Land");
        System.out.println("Is Suspicious: " + result); // true
        
        // Chaining with negate
        TriPredicate<String, Double, String> isSafe = isFlagged.negate();
        System.out.println("Is Safe: " + isSafe.test("JohnDoe", 50.0, "HOME_COUNTRY")); // true
    }
}
```
### Why Use This Pattern?
- **Type Safety:** Avoids using generic Object[] arrays to pass multiple parameters.
- **Consistency:** Keeps your custom logic behaving exactly like the Java Functional Interfaces developers already know.
- **Clean API:** It allows you to pass logic as a single unit into methods that handle complex validation rules.

  
