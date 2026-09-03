# 17 — Capstone

A single project that pulls together most of what sections 00–16 covered. This isn't a new topic — it's proof the earlier sections actually connected.

## The project: a small task-tracking service

Build a Spring Boot REST API for tracking tasks, with real persistence, tests, and a couple of deliberately-included wrinkles that force you to use specific earlier sections instead of just the easy parts.

### Requirements

1. **Domain model** ([03](../03-oop-fundamentals/README.md), [06](../06-generics/README.md)) — a `Task` entity: id, title, `TaskStatus` enum (`TODO`, `IN_PROGRESS`, `DONE`), `dueDate` (`LocalDate`, section 10), `priority` (int). Use a record for any read-only response DTOs.
2. **Persistence** ([16](../16-spring/README.md)) — a `TaskRepository` (`JpaRepository`), backed by an in-memory H2 database via a `dev` profile ([14](../14-environment-setup/README.md)).
3. **REST API** ([16](../16-spring/README.md)) — endpoints for create, get-by-id, list-all, update-status, and delete. Request/response bodies as records.
4. **Business rule requiring a custom exception** ([05](../05-exceptions/README.md)) — deleting or updating a task that doesn't exist should return a clean 404, not a stack trace; throw a custom `TaskNotFoundException` and handle it with a Spring `@ExceptionHandler` (or `@ControllerAdvice`) that maps it to an HTTP 404.
5. **A stream-based query** ([08](../08-streams-and-lambdas/README.md)) — an endpoint `GET /api/tasks/overdue` that returns tasks where `dueDate` is before today and status isn't `DONE`, computed with a stream pipeline over the results (not a manual loop, and not a generated Spring Data query method — write this one by hand to prove the streams skill, not just the framework skill).
6. **Concurrency-safe stats** ([11](../11-concurrency-basics/README.md)) — an in-memory counter (`AtomicInteger` or `ConcurrentHashMap`) tracking how many requests have hit the API since startup, exposed at `GET /api/stats`. Doesn't need to be persisted — the point is demonstrating a thread-safe counter under concurrent requests, not building real analytics.
7. **Tests** ([12](../12-testing/README.md)) — unit tests for the overdue-task logic and the exception-to-404 mapping; at least one test using a mock for a dependency.
8. **Runs from the command line** ([13](../13-build-tools/README.md)) — `mvn test` (or `gradle test`) passes, and the app starts via `mvn spring-boot:run` (or the Gradle equivalent), not just from the IDE.

### What "done" looks like

- Every endpoint works when exercised with `curl`/Postman: create a few tasks (some overdue, some not, some `DONE`), confirm `GET /api/tasks/overdue` returns exactly the right ones, confirm deleting a nonexistent task returns 404 with a sensible error body instead of a 500.
- `mvn test` is green.
- The code is on a branch, opened as a PR against `main` ([15](../15-git-and-code-review/README.md)), with a description explaining what it does — treat this as a real review, not a formality.

## Stretch goals

If the core is solid and you want to push further:

- Add a `@RestController` endpoint that accepts a CSV of tasks and bulk-creates them, using NIO ([09](../09-io-and-files/README.md)) to read the uploaded content.
- Add a scheduled job ([11](../11-concurrency-basics/README.md)'s timers) that logs a summary of overdue tasks every hour.
- Add basic Spring Security ([16](../16-spring/README.md)'s stretch section) so only authenticated requests can delete a task.

## Checklist

- [ ] All required endpoints work end to end against a running instance
- [ ] Deleting/updating a nonexistent task returns 404 via the custom exception, not a stack trace
- [ ] `/api/tasks/overdue` is computed with a stream pipeline, not a loop
- [ ] Request counter is thread-safe and correct under concurrent requests
- [ ] Unit tests pass, including at least one mocked dependency
- [ ] App builds and runs from the command line, not just the IDE
- [ ] PR opened with a real description, self-reviewed before requesting review
