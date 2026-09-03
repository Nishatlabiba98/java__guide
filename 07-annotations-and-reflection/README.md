# 07 — Annotations and Reflection

Metadata about code, and code that inspects itself at runtime. You'll use annotations constantly (especially once you hit Spring in section 16); reflection you'll rarely write yourself, but the frameworks you use lean on it heavily, so it's worth knowing what's happening underneath.

## Annotations

Metadata attached to code that doesn't change what the code does by itself — it's read by the compiler, a tool, or a framework at runtime.

You've already seen one: `@Override` (section 03) — the compiler checks it matches a real overridden method and errors if not.

```java
@FunctionalInterface
interface Calculator {
    int apply(int a, int b);
}

@Deprecated
void oldMethod() { }

@SuppressWarnings("unchecked")
void legacyCode() { }
```

### Writing your own

```java
@Retention(RetentionPolicy.RUNTIME)   // keep it available at runtime (vs. SOURCE or CLASS)
@Target(ElementType.METHOD)           // only valid on methods
@interface Loggable {
    String value() default "";
}

class Service {
    @Loggable("important operation")
    void doWork() { }
}
```

`@Retention(RUNTIME)` is what makes an annotation readable via reflection — without it, the annotation is gone after compilation and nothing at runtime can see it. This is exactly how Spring's `@RestController`, `@Autowired`, and `@Service` (section 16) work: they're just annotations a framework scans for and reacts to at startup.

## Reflection

The `java.lang.reflect` API lets code inspect classes, methods, and fields at runtime — including ones it doesn't know about at compile time.

```java
Class<?> clazz = Service.class;

for (Method method : clazz.getDeclaredMethods()) {
    if (method.isAnnotationPresent(Loggable.class)) {
        Loggable ann = method.getAnnotation(Loggable.class);
        System.out.println(method.getName() + " -> " + ann.value());
    }
}
```

Reflection can also invoke methods and read/write fields dynamically (`method.invoke(instance)`, `field.set(instance, value)`), even `private` ones — this is how dependency injection frameworks construct and wire objects without your code calling `new` directly, and how test frameworks call your `@Test` methods.

**Costs**: reflection is slower than direct calls, bypasses compile-time type checking, and can break encapsulation. Don't reach for it in application code — this section exists so that when you see `@Autowired` seemingly work "by magic," you know it's not magic, it's reflection reading annotations.

## Exercise: a tiny annotation scanner

Write a `@Test` annotation (`@Retention(RUNTIME)`, `@Target(METHOD)`), put it on a few no-argument methods in a class, then write a runner that uses reflection to find every method annotated `@Test` and invoke each one, printing pass/fail (catch any exception thrown as a failure). This is, in miniature, what JUnit (section 12) actually does.

## Checklist

- [ ] Can explain what `@Retention(RUNTIME)` controls and why it matters
- [ ] Can write a custom annotation with a `@Target`
- [ ] Can use `getDeclaredMethods()`/`isAnnotationPresent()` to find annotated methods
- [ ] Can explain, in one sentence, how a framework like Spring uses annotations + reflection together
- [ ] Tiny test runner finds and invokes all `@Test`-annotated methods
