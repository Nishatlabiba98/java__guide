# 12 — Testing

Untested code is code you can't safely change. This section covers JUnit and basic mocking — the tools, not a full testing philosophy course.

## JUnit basics

```java
class CalculatorTest {

    @Test
    void addsTwoNumbers() {
        Calculator calc = new Calculator();
        assertEquals(5, calc.add(2, 3));
    }

    @Test
    void throwsOnDivideByZero() {
        Calculator calc = new Calculator();
        assertThrows(ArithmeticException.class, () -> calc.divide(1, 0));
    }
}
```

- `@Test` marks a method as a test case — JUnit finds and runs every `@Test`-annotated method (recall section 07: this is reflection + annotations at work).
- `@BeforeEach`/`@AfterEach` run before/after every test in the class — use for setup/teardown shared across tests.
- `@BeforeAll`/`@AfterAll` (must be `static`) run once for the whole class.

## Assertions

The common ones from `org.junit.jupiter.api.Assertions`:

```java
assertEquals(expected, actual);
assertTrue(condition);
assertFalse(condition);
assertNull(value);
assertNotNull(value);
assertThrows(SomeException.class, () -> methodThatShouldThrow());
```

A failing assertion throws, which JUnit catches to mark the test failed and reports which assertion failed — write one logical assertion (or a tight related group) per test so a failure tells you exactly what broke.

## Test naming and structure

Name tests for the behavior, not the method: `throwsOnDivideByZero`, not `testDivide2`. Structure each test as **Arrange / Act / Assert**:

```java
@Test
void withdrawalFailsWhenBalanceInsufficient() {
    // Arrange
    Account account = new Account(100);

    // Act + Assert
    assertThrows(InsufficientFundsException.class, () -> account.withdraw(200));
}
```

## Mocking basics

When testing code that depends on something slow, external, or hard to set up (a database, an API call), replace it with a fake that returns canned responses — commonly done with Mockito:

```java
@Test
void sendsWelcomeEmailOnSignup() {
    EmailService emailService = mock(EmailService.class);
    UserService userService = new UserService(emailService);

    userService.signUp("sam@example.com");

    verify(emailService).sendWelcomeEmail("sam@example.com");
}
```

`mock(EmailService.class)` creates a fake implementation that does nothing by default; `verify(...)` checks a method was actually called. You can also stub return values: `when(repository.findById(1)).thenReturn(someUser);`.

Only mock things you don't own or that are genuinely expensive/external (a database, a third-party API) — mocking your own simple classes usually means the test is asserting the implementation instead of the behavior.

## Exercise: test the payroll hierarchy

Write a JUnit test class for the `Employee` hierarchy from [03](../03-oop-fundamentals/README.md#exercise-a-small-class-hierarchy):

1. `SalariedEmployee.calculatePay()` returns exactly `baseSalary`.
2. `CommissionEmployee.calculatePay()` returns `baseSalary + commission`.
3. Setting a negative salary throws (recall the setter validation from section 03).

Then write one test using a mock: assume `PayrollService.runPayroll(List<Employee>)` calls a `PaymentGateway.pay(Employee, double)` for each employee — mock `PaymentGateway` and verify it's called once per employee with the correct amount.

## Checklist

- [ ] Can write a `@Test` method with a meaningful name and Arrange/Act/Assert structure
- [ ] Comfortable with `assertEquals`, `assertThrows`, and `assertTrue`/`assertFalse`
- [ ] Know the difference between `@BeforeEach` and `@BeforeAll`
- [ ] Can explain when mocking a dependency is appropriate vs. overkill
- [ ] Payroll tests pass, including the mocked `PaymentGateway` verification
