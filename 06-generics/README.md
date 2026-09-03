# 06 — Generics

You've been using generics since section 04 (`List<String>`) without necessarily naming them. This section covers writing your own.

## Why generics exist

Without them, a collection would have to hold `Object`, and every read needs an unsafe cast:

```java
List rawList = new ArrayList();      // pre-generics style — avoid
rawList.add("hello");
String s = (String) rawList.get(0);  // manual cast, can fail at runtime

List<String> list = new ArrayList<>();  // generics — compiler enforces the type
list.add("hello");
String s2 = list.get(0);                // no cast needed, and add("wrong type") won't compile
```

Generics move type errors from runtime (`ClassCastException`) to compile time.

## Type parameters

Write a generic class or method with a placeholder type, conventionally a single uppercase letter (`T` for type, `E` for element, `K`/`V` for key/value):

```java
class Box<T> {
    private T value;
    void set(T value) { this.value = value; }
    T get() { return value; }
}

Box<String> box = new Box<>();
box.set("hello");
```

```java
<T> T firstNonNull(T a, T b) {
    return a != null ? a : b;
}
```

## Bounded types

Restrict what a type parameter can be — `T extends Number` means "any subclass of `Number`" (works for interfaces too, despite the `extends` keyword):

```java
<T extends Comparable<T>> T max(List<T> list) {
    T best = list.get(0);
    for (T item : list) {
        if (item.compareTo(best) > 0) best = item;
    }
    return best;
}
```

Without the bound, the compiler wouldn't know `T` has a `compareTo` method at all.

## Wildcards

`?` when you need a generic type but don't care (or can't say) exactly which:

```java
void printAll(List<?> list) {              // unknown type — read-only
    for (Object o : list) System.out.println(o);
}

void addNumbers(List<? super Integer> list) {  // ? super — can accept Integers into it
    list.add(1);
}

double sum(List<? extends Number> list) {      // ? extends — can read as Number out of it
    double total = 0;
    for (Number n : list) total += n.doubleValue();
    return total;
}
```

Rule of thumb (PECS — **P**roducer **E**xtends, **C**onsumer **S**uper): use `? extends T` when you're only reading from the structure, `? super T` when you're only writing into it.

## Exercise: a generic pair

Write a generic class `Pair<A, B>` holding two values of possibly different types, with `getFirst()`/`getSecond()` and a `swap()` method returning a new `Pair<B, A>`. Then write a generic method `<T extends Comparable<T>> Pair<T, T> minMax(List<T> list)` that returns the smallest and largest elements as a `Pair`.

## Checklist

- [ ] Can explain what problem generics solve vs. raw `Object` types
- [ ] Comfortable writing a generic class and a generic method
- [ ] Can explain a bounded type parameter (`T extends Comparable<T>`)
- [ ] Can explain PECS and when to use `? extends` vs `? super`
- [ ] `Pair` and `minMax` compile and work for at least `Integer` and `String`
