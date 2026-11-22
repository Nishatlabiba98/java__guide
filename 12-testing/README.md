# 12 — Testing

Untested code is code you can't safely change. This section covers JUnit, mocking with Mockito, and simulating HTTP APIs with Hoverfly — the tools, not a full testing philosophy course.

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

## Mockito features worth knowing

### Annotations instead of `mock(...)` calls

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock EmailService emailService;        // fake, does nothing by default
    @Mock UserRepository repository;
    @InjectMocks UserService userService;   // real object, mocks injected via constructor

    @Test
    void sendsWelcomeEmailOnSignup() {
        userService.signUp("sam@example.com");
        verify(emailService).sendWelcomeEmail("sam@example.com");
    }
}
```

`MockitoExtension` creates fresh mocks before each test, so no state leaks between tests.

### Stubbing

```java
when(repository.findById(1L)).thenReturn(Optional.of(user));
when(repository.findById(99L)).thenReturn(Optional.empty());
when(repository.save(any())).thenThrow(new DataAccessException("db down"));

// Consecutive calls: first call returns a, second returns b
when(clock.now()).thenReturn(t1, t2);

// Compute the answer from the arguments
when(repository.save(any(User.class))).thenAnswer(inv -> inv.getArgument(0));

// void methods (and spies) use the do...when form
doThrow(new IllegalStateException()).when(emailService).sendWelcomeEmail(anyString());
```

Unstubbed methods return "empty" defaults: `null`, `0`, `false`, empty collections, `Optional.empty()`.

### Argument matchers

`any()`, `anyString()`, `eq(...)`, `argThat(x -> ...)`. **If you use a matcher for one argument, every argument must be a matcher**:

```java
verify(gateway).pay(eq(employee), anyDouble());   // OK
verify(gateway).pay(employee, anyDouble());       // throws InvalidUseOfMatchersException
```

### Verification

```java
verify(gateway, times(3)).pay(any(), anyDouble());
verify(emailService, never()).sendWelcomeEmail(any());
verify(audit, atLeastOnce()).log(any());
verifyNoMoreInteractions(gateway);   // nothing else was called on it

InOrder inOrder = inOrder(repository, emailService);
inOrder.verify(repository).save(any());
inOrder.verify(emailService).sendWelcomeEmail(any());
```

### Capturing arguments

When you need to assert on an object your code built internally:

```java
@Captor ArgumentCaptor<Payment> paymentCaptor;

@Test
void paysNetAmount() {
    payrollService.runPayroll(List.of(employee));

    verify(gateway).submit(paymentCaptor.capture());
    assertEquals(4200.0, paymentCaptor.getValue().amount());
}
```

### Spies

`@Spy` wraps a **real** object — real methods run unless you stub them. Use `doReturn(...).when(spy).method()` to stub a spy; `when(spy.method())` calls the real method first. Reach for spies rarely: needing one often means the class is doing too much.

### Static methods and final classes

Since Mockito 5 the inline mock maker is the default, so final classes and methods can be mocked out of the box. Static methods need a scoped mock that must be closed:

```java
try (MockedStatic<UUID> uuid = mockStatic(UUID.class)) {
    uuid.when(UUID::randomUUID).thenReturn(fixedId);
    // ... code under test ...
}   // static behaviour restored here
```

Prefer injecting a `Clock` or an ID supplier over mocking statics — it's simpler and doesn't need special machinery.

### BDD style

`BDDMockito` gives the same features with Given/When/Then naming, which some teams prefer:

```java
given(repository.findById(1L)).willReturn(Optional.of(user));
// when
userService.deactivate(1L);
// then
then(repository).should().save(argThat(u -> !u.isActive()));
```

## Hoverfly: simulating HTTP APIs

Mockito replaces a Java object. **Hoverfly** replaces a whole HTTP service: it runs a lightweight proxy that your app's HTTP traffic goes through, and returns simulated responses. That means your real HTTP client code — URLs, headers, JSON serialization, error handling — actually executes in the test.

Dependency (test scope): `io.specto:hoverfly-java-junit5`.

### Simulating with the Java DSL

```java
@ExtendWith(HoverflyExtension.class)
class PaymentClientTest {

    @Test
    void returnsChargeIdOnSuccess(Hoverfly hoverfly) {
        hoverfly.simulate(dsl(
            service("api.payments.example.com")
                .post("/charges")
                .body(equalsToJson("{\"amount\":100}"))
                .willReturn(success("{\"id\":\"ch_1\",\"status\":\"ok\"}", "application/json"))
        ));

        PaymentClient client = new PaymentClient("https://api.payments.example.com");

        assertEquals("ch_1", client.charge(100).id());
    }

    @Test
    void throwsWhenProviderIsDown(Hoverfly hoverfly) {
        hoverfly.simulate(dsl(
            service("api.payments.example.com")
                .post("/charges")
                .anyBody()
                .willReturn(serverError())
        ));

        PaymentClient client = new PaymentClient("https://api.payments.example.com");

        assertThrows(PaymentFailedException.class, () -> client.charge(100));
    }
}
```

Other useful response creators: `badRequest()`, `notFound()`, `unauthorised()`, and `.withDelay(2, TimeUnit.SECONDS)` to test timeouts.

### Verifying requests were made

```java
hoverfly.verify(service("api.payments.example.com").post("/charges"), times(1));
```

Like Mockito's `verify`, but at the HTTP level.

### Modes

| Mode | What it does | Use for |
|---|---|---|
| **Simulate** (default) | Returns responses from the simulation; unmatched requests fail | Normal tests |
| **Capture** | Proxies to the real API and records request/response pairs to a JSON file | Bootstrapping a simulation from a real (test) environment |
| **Spy** | Returns simulated responses when matched, otherwise forwards to the real API | Partially simulated environments |
| **Diff** | Forwards to the real API and reports differences from the simulation | Detecting when a real API has drifted from your simulation |

Capture once, commit the JSON, then load it in simulate mode:

```java
@HoverflyCapture(path = "src/test/resources/hoverfly", filename = "payments.json")   // record
@HoverflySimulate(source = @HoverflySimulate.Source(value = "payments.json",
        type = HoverflySimulate.SourceType.DEFAULT_PATH))                          // replay
```

Scrub tokens, card numbers, and personal data from captured files before committing them.

## Notes and learnings

- **Mockito vs. Hoverfly — pick the level you're testing.** Testing business logic in `PayrollService`? Mock the `PaymentGateway` interface with Mockito. Testing the `HttpPaymentGateway` that actually talks HTTP? Use Hoverfly — a Mockito mock of the HTTP client would skip exactly the code you need to test (serialization, status-code handling, headers).
- **Don't mock types you don't own** (e.g. `RestTemplate`, `HttpClient`). Stubbing their call chains is brittle and proves little; wrap them in your own interface and mock that, or test the wrapper with Hoverfly.
- **Strict stubs catch dead setup.** `MockitoExtension` fails a test with `UnnecessaryStubbingException` if you stub something that's never used. Delete the stub rather than switching to `lenient()` — it's usually leftover setup that misleads readers.
- **Stub only what the test needs, verify only what matters.** Over-verifying (`verifyNoMoreInteractions` everywhere) couples tests to implementation and breaks them on harmless refactors.
- **`when(spy.foo())` runs the real `foo()`.** Use `doReturn(...).when(spy).foo()` for spies.
- **Hoverfly works as a proxy, so your HTTP client must use it.** It sets the JVM proxy system properties and trusts its own certificate for HTTPS. `HttpURLConnection` (and `RestTemplate` on top of it) picks this up automatically; Java 11's `HttpClient` needs `.proxy(ProxySelector.getDefault())`, and some clients (e.g. Reactor Netty/`WebClient`) need explicit proxy configuration.
- **Unmatched request = failure.** If a Hoverfly test gets a 502 with a "no match" message, the request differs from the simulation (path, query param, header, or body). The Hoverfly log printed in the test output shows the closest miss.
- **Simulations go stale.** A simulation encodes your *assumption* about the API. Re-capture or run diff mode periodically against a real test environment, and complement with contract tests where available.
- **Neither replaces an integration test.** Mocks and simulations make tests fast and deterministic; still have a small number of tests against real dependencies (e.g. via Testcontainers or a shared test environment).

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
- [ ] Can use `@Mock`/`@InjectMocks`, stub with `when(...)`/`doThrow(...)`, and capture arguments with `ArgumentCaptor`
- [ ] Know the matcher all-or-nothing rule and why `UnnecessaryStubbingException` happens
- [ ] Can simulate an HTTP API with Hoverfly's DSL and explain simulate vs. capture mode
- [ ] Can explain when to use Mockito vs. Hoverfly for a given class
- [ ] Payroll tests pass, including the mocked `PaymentGateway` verification
