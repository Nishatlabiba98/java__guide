# 02 — Data Structures and Data Types

Builds on [01 — Types and Control Flow](../01-types-and-control-flow/README.md): now that you know the primitive/reference distinction, this section covers how data is actually organized and stored in memory, and the structures you'll reach for before the Collections API (covered in [04 — Collections](../04-collections/README.md)) does the work for you.

- Arrays: fixed-size, contiguous, why `ArrayIndexOutOfBoundsException` happens
- Stacks and queues (LIFO vs FIFO) — and where the JDK already gives you these (`Deque`)
- Linked lists — singly vs. doubly linked, why they trade random access for cheap insertion/removal
- Trees — basic terminology (node, root, leaf, depth), binary trees vs. binary search trees
- Big-O basics — enough to reason about why you'd pick one structure over another (array lookup vs. linked-list insertion vs. tree search)
- Boxed vs. unboxed types (`int` vs `Integer`) and why it matters for these structures
