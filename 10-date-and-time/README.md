# 10 — Date and Time

Java's date/time API (`java.time`, since Java 8) — use this, not the old `java.util.Date`/`Calendar`, which are mutable, not thread-safe, and generally considered a design mistake even by Java's own docs.

## The core types

| Type | Represents |
|---|---|
| `LocalDate` | a date, no time or timezone (`2026-03-05`) |
| `LocalTime` | a time, no date or timezone (`14:30:00`) |
| `LocalDateTime` | date + time, no timezone |
| `ZonedDateTime` | date + time + timezone |
| `Instant` | a single point on the timeline (UTC), typically for timestamps/logging |
| `Duration` | an amount of time (hours, minutes, seconds) — for measuring elapsed time |
| `Period` | an amount of time in date terms (years, months, days) — for calendar arithmetic |

```java
LocalDate today = LocalDate.now();
LocalDate deadline = LocalDate.of(2026, 3, 5);

LocalDateTime meeting = LocalDateTime.of(2026, 3, 5, 14, 30);

ZonedDateTime nyTime = ZonedDateTime.now(ZoneId.of("America/New_York"));

Instant timestamp = Instant.now();   // for logging/storage — always UTC
```

## Arithmetic and comparison

Every type here is **immutable** — every "modifying" method returns a new instance, same as `String`:

```java
LocalDate nextWeek = today.plusWeeks(1);
LocalDate lastMonth = today.minusMonths(1);

boolean isBefore = deadline.isBefore(today);

Period age = Period.between(LocalDate.of(1995, 6, 1), today);
System.out.println(age.getYears() + " years");

Duration elapsed = Duration.between(startInstant, Instant.now());
System.out.println(elapsed.toSeconds() + "s");
```

## Formatting and parsing

```java
DateTimeFormatter fmt = DateTimeFormatter.ofPattern("yyyy-MM-dd");
String formatted = today.format(fmt);            // "2026-03-05"
LocalDate parsed = LocalDate.parse("2026-03-05", fmt);
```

## Timezones

`LocalDateTime` has no timezone — don't use it for anything that gets compared across timezones (scheduling, timestamps shared between services). Use `ZonedDateTime` or `Instant` for that; convert explicitly when you need to:

```java
ZonedDateTime utc = nyTime.withZoneSameInstant(ZoneId.of("UTC"));
```

A common bug: storing `LocalDateTime` in a database column meant for a single global timestamp — it silently loses timezone information, and "2:30 PM" means something different depending on who's reading it.

## Exercise: meeting scheduler

Write a method `boolean canSchedule(ZonedDateTime meetingStart, Duration meetingLength, List<ZonedDateTime[]> existingMeetings)` where each element of `existingMeetings` is a `[start, end]` pair, that returns `false` if the proposed meeting overlaps any existing one, `true` otherwise. Test it across at least two different timezones to confirm your overlap check works on the underlying instant, not the local wall-clock time.

## Checklist

- [ ] Know why `java.time` is preferred over `Date`/`Calendar`
- [ ] Can explain the difference between `LocalDateTime` and `ZonedDateTime`/`Instant`
- [ ] Comfortable with basic arithmetic (`plusDays`, `Period.between`, `Duration.between`)
- [ ] Can format and parse a date with `DateTimeFormatter`
- [ ] Meeting scheduler correctly detects overlap across different timezones
