# 03 — OOP Fundamentals

Java is fundamentally object-oriented — almost everything you write lives inside a class. This section covers the vocabulary and mechanics you'll use in every file from here on.

## Classes and objects

A **class** is a blueprint; an **object** is an instance of it.

```java
class Employee {
    String name;
    double salary;

    Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }
}

Employee e = new Employee("Sam", 75000);
```

## Access modifiers

Control what other code can see:

| Modifier | Visible from |
|---|---|
| `public` | anywhere |
| `protected` | same package + subclasses |
| *(none / package-private)* | same package only |
| `private` | same class only |

Default to the most restrictive modifier that works, and widen only when something outside the class actually needs it.

## Getters and setters

Fields are usually `private`; controlled access goes through methods:

```java
private double salary;

public double getSalary() { return salary; }
public void setSalary(double salary) {
    if (salary < 0) throw new IllegalArgumentException("salary can't be negative");
    this.salary = salary;
}
```

This is the point of encapsulation — a setter can validate; a bare public field can't.

## Inheritance and `super`

```java
class Manager extends Employee {
    List<Employee> reports;

    Manager(String name, double salary, List<Employee> reports) {
        super(name, salary);     // calls Employee's constructor
        this.reports = reports;
    }
}
```

`super` reaches into the parent class — for its constructor (as above) or to call an overridden method's original implementation (`super.someMethod()`).

## Constructor overloading vs. method overloading vs. method overriding

Three different things that sound similar:

- **Constructor overloading**: multiple constructors, different parameter lists.
- **Method overloading**: same method name, different parameter lists, *same class* — resolved at compile time based on argument types.
- **Method overriding**: a subclass redefines a method it inherited, *same signature* — resolved at runtime based on the actual object type. Mark these `@Override` so the compiler catches typos in the signature.

```java
class Shape {
    double area() { return 0; }
}
class Circle extends Shape {
    double radius;
    Circle(double radius) { this.radius = radius; }

    @Override
    double area() { return Math.PI * radius * radius; }   // overriding
}
```

## Abstract classes vs. interfaces

- **Abstract class**: can't be instantiated directly, can mix implemented and unimplemented (`abstract`) methods, and a class can extend only one.
- **Interface**: a pure contract (though it can have `default` methods with a body since Java 8) — a class can implement many.

```java
abstract class Shape {
    abstract double area();               // no body — subclasses must implement
    void describe() {                     // shared implementation
        System.out.println("Area: " + area());
    }
}

interface Drawable {
    void draw();
}

class Circle extends Shape implements Drawable {
    double radius;
    Circle(double radius) { this.radius = radius; }
    double area() { return Math.PI * radius * radius; }
    public void draw() { System.out.println("Drawing a circle"); }
}
```

Rule of thumb: interface when unrelated classes need to share a capability (`Comparable`, `Runnable`); abstract class when subclasses share actual state or code.

## Anonymous classes

A one-off implementation of an interface or abstract class, declared inline with no name — mostly superseded by lambdas (section 08) when the interface has a single method, but still shows up for multi-method interfaces:

```java
Comparator<String> byLength = new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return a.length() - b.length();
    }
};
```

## Sealed classes

Restrict which classes are allowed to extend/implement yours — the compiler enforces the list, which makes exhaustive `switch` handling possible:

```java
sealed interface Shape permits Circle, Square {}
final class Circle implements Shape { double radius; }
final class Square implements Shape { double side; }
```

## Records

A concise, immutable data carrier — the compiler generates the constructor, getters, `equals()`, `hashCode()`, and `toString()` for you:

```java
record Point(int x, int y) {}

Point p = new Point(3, 4);
p.x();          // 3 — generated accessor (no "get" prefix)
```

Use a record when a type is just a bundle of data with no mutable state or behavior — a DTO, a coordinate pair, a result tuple.

## Inheritance vs. Association vs. Aggregation vs. Composition

Four ways classes relate to each other, in increasing order of "ownership":

- **Inheritance** ("is-a"): `Manager extends Employee` — a Manager *is an* Employee.
- **Association**: two classes reference each other with no ownership implied — a `Teacher` and a `Student` both exist independently.
- **Aggregation** ("has-a", weak): a `Department` has `Employee`s, but the employees exist independently of the department (they'd survive the department being deleted).
- **Composition** ("has-a", strong): a `Car` has an `Engine` it fully owns — the engine's lifecycle is tied to the car's; it doesn't make sense without it.

```java
class Engine {}
class Car {
    private final Engine engine = new Engine();   // composition — Car owns and creates it
}
```

## `final` keyword

Three different uses of the same keyword:

- `final` variable — can't be reassigned after initialization.
- `final` method — can't be overridden by a subclass.
- `final` class — can't be extended at all (e.g. `String`).

## Enums

A fixed set of named constants, but a real class — can have fields, constructors, and methods:

```java
enum Status {
    ACTIVE, INACTIVE, SUSPENDED;
}

enum Day {
    MONDAY(false), SATURDAY(true), SUNDAY(true);

    final boolean isWeekend;
    Day(boolean isWeekend) { this.isWeekend = isWeekend; }
}
```

Prefer an enum over a set of `String`/`int` constants — the compiler catches typos and invalid values that strings can't.

## Inner and nested classes

- **Static nested class**: a class defined inside another, marked `static` — doesn't need an instance of the outer class, mostly just namespacing (e.g. `Map.Entry`).
- **Inner class** (non-static): tied to an instance of the outer class, can access its fields directly — used when the nested class genuinely doesn't make sense without its outer instance.

## Varargs

Lets a method accept a variable number of arguments, treated as an array inside the method:

```java
int sum(int... numbers) {
    int total = 0;
    for (int n : numbers) total += n;
    return total;
}

sum(1, 2, 3);       // fine
sum();               // fine — numbers is an empty array
```

## Wrapper classes and autoboxing

Already covered in [01](../01-types-and-control-flow/README.md) (the `==` vs `.equals()` trap) and [02](../02-data-structures-and-types/README.md) (performance cost in bulk data) — worth a re-read here since this section is where you'll actually start writing classes that mix primitives and their wrapper types in fields and method signatures.

## Exercise: a small class hierarchy

Model a payroll system:

1. An abstract class `Employee` with `name`, `baseSalary` (private, with getters/setters), an abstract `calculatePay()` method, and a concrete `describe()` method that prints name + pay.
2. Two subclasses: `SalariedEmployee` (pay = `baseSalary`) and `CommissionEmployee` (pay = `baseSalary` + `commission`), each overriding `calculatePay()`.
3. An enum `EmployeeType` with values `SALARIED`, `COMMISSION`.
4. A record `PayStub(String name, double amount)` produced by a method that takes an `Employee` and returns its pay stub.

Instantiate a few employees of each type in a `List<Employee>`, loop over them calling `describe()`, and confirm overriding picks the right `calculatePay()` for each without any `instanceof` checks.

## Checklist

- [ ] Can explain the difference between overloading and overriding
- [ ] Know when to reach for an abstract class vs. an interface
- [ ] Can write a record and explain when it's a better fit than a class
- [ ] Can explain composition vs. aggregation with an example
- [ ] Comfortable with enums that carry fields/behavior, not just constants
- [ ] Payroll exercise runs and overriding resolves correctly per employee type
