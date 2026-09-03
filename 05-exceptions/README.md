# 05 — Exceptions

Java forces you to acknowledge failure paths rather than let them fail silently. This section covers how, and how not to abuse that.

## Checked vs. unchecked exceptions

- **Checked** (`Exception` and subclasses, excluding `RuntimeException`) — the compiler forces you to either `catch` or declare (`throws`) them. Used for recoverable conditions the caller should be aware of (`IOException`).
- **Unchecked** (`RuntimeException` and subclasses) — no compiler enforcement. Used for programming errors that shouldn't normally be caught and handled (`NullPointerException`, `IllegalArgumentException`).
- **`Error`** — serious problems outside your control (`OutOfMemoryError`). Don't catch these.

```java
void readFile(String path) throws IOException {   // checked — caller must handle or propagate
    Files.readString(Path.of(path));
}

void divide(int a, int b) {
    if (b == 0) throw new IllegalArgumentException("b can't be zero");  // unchecked
}
```

## `try` / `catch` / `finally`

```java
try {
    riskyOperation();
} catch (IOException e) {
    log.error("failed to read file", e);
} finally {
    cleanup();   // always runs, even if catch rethrows or try returns
}
```

Catch the most specific exception type you can actually do something about. Catching bare `Exception` (or worse, `Throwable`) hides bugs you didn't mean to swallow.

## try-with-resources

Anything implementing `AutoCloseable` (streams, readers, connections) gets closed automatically, even if an exception is thrown — no `finally` block needed:

```java
try (BufferedReader reader = Files.newBufferedReader(path)) {
    return reader.readLine();
}   // reader.close() called automatically here
```

Prefer this over manual `finally { resource.close(); }` — it's shorter and doesn't have the bug where an exception in `close()` masks the original exception.

## Custom exceptions

Extend `Exception` (checked) or `RuntimeException` (unchecked) to give callers a meaningful, specific type instead of throwing generic ones:

```java
class InsufficientFundsException extends RuntimeException {
    InsufficientFundsException(String message) {
        super(message);
    }
}

void withdraw(double amount) {
    if (amount > balance) {
        throw new InsufficientFundsException("requested " + amount + ", balance " + balance);
    }
}
```

## Common mistakes

- **Swallowing exceptions**: `catch (Exception e) {}` — the failure vanishes silently and someone debugs a symptom three layers away. At minimum, log it.
- **Catching to log then rethrow without adding context**: fine if you're adding something (a request ID, a retry decision); pointless if you're just re-logging the same trace.
- **Using exceptions for normal control flow**: exceptions are for exceptional conditions, not expected outcomes — checking `list.isEmpty()` beats catching an exception from `list.get(0)`.

## Exercise: validate and report

Write a method `void processOrder(Order order)` that throws:

- `InvalidOrderException` (custom, unchecked) if the order has no items
- `InsufficientFundsException` (custom, unchecked) if `order.total() > account.balance()`

Then write a caller that processes a `List<Order>`, catches each exception type separately, and prints a per-order success/failure report — one bad order shouldn't stop the rest from processing.

## Checklist

- [ ] Can explain checked vs. unchecked and give an example of each
- [ ] Know why try-with-resources is preferred over manual `finally { close(); }`
- [ ] Can write a custom exception with a meaningful message
- [ ] Can explain why swallowing exceptions silently is a problem
- [ ] Order-processing exercise handles multiple orders without one failure stopping the rest
