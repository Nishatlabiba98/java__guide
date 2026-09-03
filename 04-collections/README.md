# 04 — Collections

The JDK's ready-made implementations of the data structures from [02](../02-data-structures-and-types/README.md). See that section first if `O(1)`/`O(n)`/`O(log n)` isn't familiar yet — this section leans on it constantly.

## List — `ArrayList` vs. `LinkedList`

Both implement `List` (ordered, allows duplicates, indexable), backed by different structures:

```java
List<String> names = new ArrayList<>();   // array-backed: fast get(i), slow insert in the middle
List<String> queue = new LinkedList<>();  // node-backed: slow get(i), fast insert/remove at the ends
names.add("Sam");
names.get(0);          // O(1) for ArrayList, O(n) for LinkedList
```

Default to `ArrayList` unless you specifically need cheap insertion/removal at both ends — `LinkedList` also implements `Deque`, so it can act as a stack/queue too, but `ArrayDeque` (section 02) is usually the better choice for that.

## Set — `HashSet`, `LinkedHashSet`, `TreeSet`

All three forbid duplicates (via `equals()`/`hashCode()`); they differ in what order you get back on iteration:

```java
Set<String> hash = new HashSet<>();        // no guaranteed order
Set<String> linked = new LinkedHashSet<>(); // insertion order
Set<String> sorted = new TreeSet<>();       // sorted order (natural or via Comparator)
```

`HashSet` is fastest (`O(1)` add/contains) when you don't care about order. `TreeSet` costs `O(log n)` for the same operations but keeps things sorted for free.

## Map — `HashMap`, `LinkedHashMap`, `TreeMap`

Same pattern as `Set` (a `Map`'s keys behave like a `Set`):

```java
Map<String, Integer> ages = new HashMap<>();
ages.put("Sam", 29);
ages.get("Sam");            // 29
ages.getOrDefault("Alex", 0); // 0 — no exception on a missing key
```

- `HashMap` — no order guarantee, fastest.
- `LinkedHashMap` — insertion order preserved.
- `TreeMap` — sorted by key.

## `PriorityQueue`

Not sorted overall — but `poll()` always returns the smallest element first (or by a custom `Comparator`):

```java
PriorityQueue<Integer> pq = new PriorityQueue<>();
pq.add(5); pq.add(1); pq.add(3);
pq.poll();   // 1
pq.poll();   // 3
```

Backed by a binary heap — `O(log n)` insert/remove, `O(1)` peek at the smallest. Useful whenever you need "process the smallest/highest-priority item next" without keeping everything fully sorted.

## `Stack`

`java.util.Stack` exists but is a legacy class (synchronized, extends `Vector`) — use `Deque`/`ArrayDeque` as a stack instead, as covered in [02](../02-data-structures-and-types/README.md#stacks-and-queues). It's listed here because you'll see it in older code and should recognize what it is.

## Choosing a collection

| Need | Reach for |
|---|---|
| Ordered list, fast random access | `ArrayList` |
| Fast insert/remove at both ends | `ArrayDeque` |
| Unique elements, don't care about order | `HashSet` |
| Unique elements, sorted | `TreeSet` |
| Key → value lookup, don't care about order | `HashMap` |
| Key → value lookup, sorted by key | `TreeMap` |
| Always process the smallest/highest-priority item next | `PriorityQueue` |

## The `Collections` and `Arrays` helper classes

Static utility methods that work on any `Collection`/array:

```java
Collections.sort(names);
Collections.reverse(names);
List<String> readOnly = Collections.unmodifiableList(names);

int[] nums = {5, 1, 3};
Arrays.sort(nums);
```

## Exercise: word frequency counter

Given a `List<String>` of words, produce a `Map<String, Integer>` of word → count, then print the top 3 most frequent words. Use a `HashMap` to count, then a `PriorityQueue` (or sort a `List<Map.Entry<String, Integer>>`) to find the top 3 — this exercises a `Map`, a `List`, and either a `PriorityQueue` or sorting in one small problem.

## Checklist

- [ ] Can explain when `ArrayList` beats `LinkedList` and vice versa
- [ ] Know the ordering guarantee (or lack of one) for `HashSet`/`LinkedHashSet`/`TreeSet`
- [ ] Comfortable with `Map.getOrDefault` instead of manually checking `containsKey`
- [ ] Can explain what `PriorityQueue.poll()` returns and why
- [ ] Know why `java.util.Stack` should be avoided in new code
- [ ] Word frequency counter correctly finds the top 3 words
