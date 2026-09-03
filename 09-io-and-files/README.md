# 09 — I/O and Files

Reading and writing data — from the console, from files, and across the network boundary via serialization.

## Console I/O

```java
Scanner scanner = new Scanner(System.in);
System.out.print("Enter your name: ");
String name = scanner.nextLine();

System.out.println("Hello, " + name);
System.out.printf("Pi is approximately %.2f%n", Math.PI);
```

`printf` uses format specifiers: `%s` (string), `%d` (integer), `%f` (float, `%.2f` for 2 decimal places), `%n` (platform-correct newline). Prefer it over string concatenation when formatting numbers for display.

## Classic I/O (`java.io`)

Stream-based, blocking. The classes you'll actually reach for:

```java
// reading a whole text file
try (BufferedReader reader = new BufferedReader(new FileReader("data.txt"))) {
    String line;
    while ((line = reader.readLine()) != null) {
        System.out.println(line);
    }
}

// writing to a text file
try (BufferedWriter writer = new BufferedWriter(new FileWriter("out.txt"))) {
    writer.write("hello");
    writer.newLine();
}
```

`Buffered*` wraps the raw reader/writer to batch disk access — always prefer it over the raw `FileReader`/`FileWriter` for anything beyond trivial reads.

## Java NIO (`java.nio.file`)

The modern, generally preferred API for file operations — more concise for common cases:

```java
Path path = Path.of("data.txt");

List<String> lines = Files.readAllLines(path);
String content = Files.readString(path);

Files.writeString(path, "hello\n", StandardOpenOption.CREATE, StandardOpenOption.APPEND);

boolean exists = Files.exists(path);
Files.copy(path, Path.of("backup.txt"));
Files.delete(path);
```

For anything that's "read/write a whole file," reach for `Files.readString`/`Files.writeString`/`Files.readAllLines` before classic I/O — it's less boilerplate and handles resource closing for you.

## Serialization

Converting an object to a byte stream (to save to disk or send over a network) and back. Java's built-in mechanism:

```java
class User implements Serializable {
    String name;
    int age;
}

// write
try (ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("user.ser"))) {
    out.writeObject(new User("Sam", 29));
}

// read
try (ObjectInputStream in = new ObjectInputStream(new FileInputStream("user.ser"))) {
    User u = (User) in.readObject();
}
```

In practice, most real systems use JSON (via a library like Jackson — this is what Spring's REST layer in section 16 uses under the hood) rather than raw Java serialization, which is version-fragile and a known security liability when deserializing untrusted input. Know that `Serializable` exists and what it's for; don't reach for it by default.

## Exercise: log file summarizer

Given a text file where each line looks like `2026-01-15 ERROR Connection timed out`, write a program that:

1. Reads the file using NIO (`Files.readAllLines`).
2. Counts how many lines are `ERROR`, `WARN`, and `INFO` (use a `Map<String, Integer>` from section 04, and streams from section 08 if you want).
3. Writes a summary to a new file: one line per level, e.g. `ERROR: 3`.

## Checklist

- [ ] Comfortable reading input with `Scanner` and formatting output with `printf`
- [ ] Know when to use classic I/O (`Buffered*`) vs. NIO (`Files.*`)
- [ ] Can explain what try-with-resources buys you here (recap from section 05)
- [ ] Know what serialization is and why JSON is more commonly used than Java's built-in `Serializable` in practice
- [ ] Log summarizer reads the file, counts levels correctly, and writes the summary
