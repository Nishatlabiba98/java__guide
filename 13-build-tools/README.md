# 13 — Build Tools

How Java projects declare dependencies, compile, test, and package — plus a few related tooling odds and ends.

## Maven

Project structure and dependencies declared in `pom.xml`:

```xml
<project>
    <groupId>com.taproot</groupId>
    <artifactId>my-app</artifactId>
    <version>1.0.0</version>

    <dependencies>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.10.0</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

Standard directory layout Maven expects:

```
src/main/java/     ← application code
src/main/resources/ ← config files, non-code resources
src/test/java/      ← test code
```

Common commands:

```
mvn compile     # compile main code
mvn test        # run tests
mvn package     # produce a .jar (or .war)
mvn clean       # delete build output
```

## Gradle

Same job as Maven, different config format (`build.gradle`, Groovy or Kotlin DSL) and generally faster incremental builds:

```groovy
dependencies {
    testImplementation 'org.junit.jupiter:junit-jupiter:5.10.0'
}
```

Common commands: `gradle build`, `gradle test`, `gradle clean`. TapRoot's actual choice between the two will determine which of these you use day to day — the concepts (dependency declaration, standard source layout, lifecycle phases) carry over either way.

## Dependency management

Both tools resolve a dependency tree — your direct dependencies pull in their own dependencies (transitive dependencies) automatically. Version conflicts between two dependencies wanting different versions of the same library are one of the most common build headaches you'll hit; both tools have a way to inspect the resolved tree (`mvn dependency:tree` / `gradle dependencies`) when something's not behaving as expected.

## Javadoc

Structured comments that generate browsable API documentation:

```java
/**
 * Calculates simple interest.
 *
 * @param principal the initial amount
 * @param rate annual interest rate, as a decimal (0.05 for 5%)
 * @param years number of years
 * @return the interest earned
 */
double calculateInterest(double principal, double rate, int years) {
    return principal * rate * years;
}
```

Generate with `mvn javadoc:javadoc` (Maven) or `gradle javadoc` (Gradle). Write Javadoc for public API methods, especially anything other teams will call — a plain inline comment is enough for private implementation details.

## Java Modules (JPMS)

Since Java 9, an optional module system (`module-info.java`) that declares what a module exports and requires, enforced at compile and run time — a stricter alternative to the classic classpath. Most application code (and this guide) doesn't need it; it mostly matters for library authors controlling their public API surface, or very large codebases enforcing internal boundaries. Know it exists; you're unlikely to write a `module-info.java` as an intern.

## Exercise: build the payroll project

Set up a Maven (or Gradle) module for the `Employee`/payroll code from sections 03, 08, and 12: correct source layout, JUnit as a test-scoped dependency, and confirm `mvn test` (or `gradle test`) runs the tests you wrote in section 12 successfully from the command line, not just the IDE.

## Checklist

- [ ] Know the standard Maven/Gradle source layout (`src/main/java`, `src/test/java`)
- [ ] Can add a dependency and explain what "transitive dependency" means
- [ ] Comfortable running compile/test/package from the command line, not just the IDE
- [ ] Know what Javadoc is for and how to generate it
- [ ] Payroll module builds and `mvn test`/`gradle test` passes from the command line
