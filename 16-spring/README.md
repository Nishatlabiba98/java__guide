# 16 — Spring

Everything before this section was plain Java. Spring (specifically Spring Boot) is the framework that turns that into a real, runnable service — handling wiring, configuration, and web/data plumbing so you're not writing it by hand.

## Dependency injection and the Spring container

Instead of a class constructing its own dependencies (`new EmailService()`), it declares what it needs and Spring provides it — this is dependency injection (DI), and it's the core idea everything else in Spring builds on.

```java
@Service
class OrderService {
    private final EmailService emailService;

    @Autowired   // optional on a single constructor since Spring 4.3, but good to know it's there
    OrderService(EmailService emailService) {
        this.emailService = emailService;
    }
}

@Service
class EmailService { }
```

`@Service` marks a class as something Spring should manage — an instance of it (a "bean") is created once, held in the Spring container, and handed to anything that asks for it via the constructor. This is exactly [07](../07-annotations-and-reflection/README.md)'s annotations + reflection in action: Spring scans for annotated classes at startup and wires them together via reflection, so you never call `new OrderService(...)` yourself.

Prefer constructor injection (as above) over field injection (`@Autowired private EmailService emailService;`) — constructor injection makes dependencies explicit and required, and lets you write plain unit tests (section 12) without needing Spring running at all.

## Spring Boot basics

`@SpringBootApplication` marks the entry point and turns on auto-configuration — Spring inspects what's on the classpath and configures sensible defaults (a web server, a database connection) without you writing that config by hand:

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

Configuration lives in `src/main/resources/application.properties` (or `.yml`):

```properties
server.port=8080
spring.datasource.url=jdbc:postgresql://localhost:5432/mydb
```

## Building a REST API with Spring Web

```java
@RestController
@RequestMapping("/api/orders")
class OrderController {

    private final OrderService orderService;

    OrderController(OrderService orderService) {
        this.orderService = orderService;
    }

    @GetMapping("/{id}")
    OrderDto getOrder(@PathVariable Long id) {
        return orderService.findById(id);
    }

    @PostMapping
    OrderDto createOrder(@RequestBody CreateOrderRequest request) {
        return orderService.create(request);
    }
}
```

- `@RestController` = `@Controller` + automatic JSON serialization of return values (no manual serialization needed — see the JSON note in [09](../09-io-and-files/README.md#serialization)).
- `@GetMapping`/`@PostMapping`/etc. map HTTP methods + paths to a method.
- `@PathVariable` pulls a value from the URL; `@RequestBody` deserializes the JSON request body into an object.
- A record (section 03) is a natural fit for `OrderDto`/`CreateOrderRequest` — immutable data carried across the API boundary.

## Persistence with Spring Data JPA

An entity maps a class to a database table:

```java
@Entity
class Order {
    @Id @GeneratedValue
    Long id;
    String customerName;
    double total;
}
```

A repository interface gives you CRUD operations without writing SQL for the common cases:

```java
interface OrderRepository extends JpaRepository<Order, Long> {
    List<Order> findByCustomerName(String customerName);   // implemented for you, from the method name
}
```

`JpaRepository` already provides `save`, `findById`, `findAll`, `deleteById`. Spring Data can also generate query implementations from method names that follow its naming convention, as `findByCustomerName` does above — for anything more complex, `@Query` lets you write JPQL or native SQL directly.

## Configuration and profiles

Different environments need different config (recall [14 — Environment Setup](../14-environment-setup/README.md)) — Spring profiles let you swap config without changing code:

```
application.properties          # shared defaults
application-dev.properties      # overrides when the "dev" profile is active
application-uat.properties
application-prod.properties
```

Activate one with `spring.profiles.active=dev` (a JVM arg, env var, or in the base properties file). This is how a local DB connection string in dev becomes a real UAT/PROD connection string in those environments, with the application code never changing.

## Stretch: Spring Security basics

`spring-boot-starter-security` locks everything down by default (every endpoint requires authentication) until you configure otherwise. A minimal security config restricts or opens specific paths:

```java
@Bean
SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.authorizeHttpRequests(auth -> auth
        .requestMatchers("/api/public/**").permitAll()
        .anyRequest().authenticated()
    );
    return http.build();
}
```

This is a stretch topic — enough to recognize what's happening in an existing Spring Security config, not a full authentication/authorization course.

## Exercise: a small Order service

Build a Spring Boot app with:

1. An `Order` entity (id, customer name, total) and an `OrderRepository` (extends `JpaRepository`).
2. An `OrderService` that depends on `OrderRepository` via constructor injection.
3. An `OrderController` exposing `GET /api/orders/{id}` and `POST /api/orders`.
4. `application-dev.properties` pointing at an in-memory H2 database, so it runs without needing a real database installed.

Start the app, and use `curl` (or Postman) to create an order via `POST`, then fetch it back via `GET` — confirming the whole request → controller → service → repository → database path actually works end to end.

## Checklist

- [ ] Can explain dependency injection and why constructor injection is preferred over field injection
- [ ] Know what `@SpringBootApplication` and auto-configuration do for you
- [ ] Comfortable writing a `@RestController` with a `GET` and a `POST` endpoint
- [ ] Can write an `@Entity` and a `JpaRepository` interface
- [ ] Can explain how Spring profiles connect to the environments from section 14
- [ ] Order service exercise runs and a full create → fetch round trip works via `curl`/Postman
