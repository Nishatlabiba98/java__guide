# 08 — Streams and Lambdas

Java's functional-programming toolkit: passing behavior around as a value instead of only as a named method.

## Functional interfaces

An interface with exactly one abstract method — a lambda can implement it inline. The JDK ships the common shapes in `java.util.function`:

| Interface | Signature | Use |
|---|---|---|
| `Function<T, R>` | `R apply(T t)` | transform a value |
| `Predicate<T>` | `boolean test(T t)` | true/false check |
| `Consumer<T>` | `void accept(T t)` | do something with a value, return nothing |
| `Supplier<T>` | `T get()` | produce a value, no input |
| `Comparator<T>` | `int compare(T a, T b)` | ordering |

```java
Function<Integer, Integer> square = x -> x * x;
Predicate<String> isEmpty = s -> s.isEmpty();
Consumer<String> print = s -> System.out.println(s);
Supplier<Double> random = () -> Math.random();

square.apply(5);        // 25
```

## Lambda expressions

Anonymous function syntax — a lighter-weight version of the anonymous classes from [03](../03-oop-fundamentals/README.md#anonymous-classes):

```java
// anonymous class
Comparator<String> byLength = new Comparator<String>() {
    public int compare(String a, String b) { return a.length() - b.length(); }
};

// same thing as a lambda
Comparator<String> byLengthLambda = (a, b) -> a.length() - b.length();
```

A lambda can only implement a functional interface (one abstract method) — that's the whole trade for the shorter syntax.

## Method references

When a lambda just calls one existing method, reference it directly instead:

```java
names.forEach(n -> System.out.println(n));   // lambda
names.forEach(System.out::println);           // method reference — same thing

list.stream().map(x -> x.toString());
list.stream().map(Object::toString);           // method reference
```

## The Stream API

A pipeline over a source of data — `source -> zero or more intermediate operations -> one terminal operation`. Streams don't hold data themselves and don't modify the source; each intermediate operation returns a new stream, and nothing actually runs until the terminal operation.

```java
List<String> names = List.of("Sam", "Alex", "Jo", "Morgan");

List<String> result = names.stream()
    .filter(n -> n.length() > 2)      // intermediate — keep matching elements
    .map(String::toUpperCase)          // intermediate — transform each element
    .sorted()                          // intermediate
    .toList();                         // terminal — triggers execution, produces a List

long count = names.stream().filter(n -> n.startsWith("A")).count();  // terminal

Optional<String> longest = names.stream()
    .max(Comparator.comparingInt(String::length));                    // terminal
```

Common intermediate ops: `filter`, `map`, `sorted`, `distinct`, `limit`. Common terminal ops: `toList`/`collect`, `count`, `forEach`, `reduce`, `anyMatch`/`allMatch`, `min`/`max`.

`reduce` combines everything into one value:

```java
int sum = List.of(1, 2, 3, 4).stream().reduce(0, Integer::sum);   // 10
```

## `Optional`

A container that either holds a value or doesn't — an explicit alternative to returning (or checking for) `null`, which the compiler can help you handle:

```java
Optional<String> found = names.stream().filter(n -> n.equals("Sam")).findFirst();

found.isPresent();               // true/false
found.orElse("not found");       // value, or the fallback
found.ifPresent(System.out::println);
found.map(String::toUpperCase).orElse("N/A");
```

Use `Optional` as a return type when "no result" is a normal, expected outcome — don't use it for fields or method parameters, and don't call `.get()` without checking `isPresent()` first (it throws if empty, which defeats the point).

## Exercise: employee stream pipeline

Reusing the `Employee` hierarchy from [03](../03-oop-fundamentals/README.md#exercise-a-small-class-hierarchy): given a `List<Employee>`, use a stream pipeline (no manual loops) to:

1. Find the names of all employees earning over $60,000, sorted alphabetically.
2. Compute the total pay across all employees using `reduce` or `mapToDouble(...).sum()`.
3. Find the highest-paid employee as an `Optional<Employee>`, and print their name or `"none"` if the list is empty.

## Checklist

- [ ] Can explain the difference between an intermediate and a terminal stream operation
- [ ] Comfortable writing lambdas and converting simple ones to method references
- [ ] Know at least `filter`, `map`, `sorted`, `reduce`, `collect`/`toList`
- [ ] Can explain why streams don't modify their source
- [ ] Know when `Optional` is appropriate (return type) vs. not (fields, parameters)
- [ ] Employee pipeline exercise produces correct results using only streams, no manual loops
