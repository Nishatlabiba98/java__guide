# 02 — Data Structures and Data Types

Builds on [01 — Types and Control Flow](../01-types-and-control-flow/README.md): now that you know the primitive/reference distinction, this section covers how data is actually organized and stored, and the structures you'll reach for before the Collections API (covered in [04 — Collections](../04-collections/README.md)) does the work for you. Knowing how these work underneath is what makes the Collections API make sense instead of feeling like a grab bag of classes.

## Arrays

Fixed-size, contiguous block of memory — every element is the same type and sits right next to the next one, which is what makes indexed access fast (`O(1)`, covered below).

```java
int[] scores = new int[5];       // all zeros, size fixed at 5 forever
scores[0] = 90;
scores[5] = 10;                  // throws ArrayIndexOutOfBoundsException — no index 5 in a length-5 array
```

Because the size is fixed at creation, "growing" an array actually means allocating a new, bigger array and copying everything over — which is exactly what `ArrayList` does for you internally (more in section 04).

## Stacks and queues

Two access patterns, not really "structures" of their own — you can implement either with an array or a linked list:

- **Stack**: LIFO (last in, first out). Think: undo history, call stack, matching brackets.
- **Queue**: FIFO (first in, first out). Think: a task queue, print jobs.

The JDK already gives you both via `Deque` (double-ended queue) — don't write your own for real code, `java.util.Stack` is legacy and best avoided:

```java
Deque<Integer> stack = new ArrayDeque<>();
stack.push(1);
stack.push(2);
stack.pop();          // 2 — LIFO

Deque<Integer> queue = new ArrayDeque<>();
queue.offer(1);
queue.offer(2);
queue.poll();         // 1 — FIFO
```

## Linked lists

A chain of nodes, each holding a value and a reference to the next (and, for a *doubly* linked list, the previous) node — no contiguous memory required.

```java
class Node {
    int value;
    Node next;
}
```

The trade-off against arrays: linked lists give up fast indexed access (`O(n)` — you have to walk from the head) in exchange for cheap insertion/removal at a known position (`O(1)`, no shifting elements). `java.util.LinkedList` implements both `List` and `Deque`.

## Trees

Terminology first: a tree has a **root** (top node), **leaves** (nodes with no children), and each node has a **depth** (distance from the root).

A **binary tree** caps each node at two children. A **binary search tree (BST)** adds an ordering rule: everything in the left subtree is smaller than the node, everything in the right subtree is larger — which is what makes searching one fast.

```java
class TreeNode {
    int value;
    TreeNode left;
    TreeNode right;
}
```

## Big-O basics

Enough to reason about *why* you'd pick one structure over another — not a full course in algorithmic complexity:

| Operation | Array | Linked List | BST (balanced) |
|---|---|---|---|
| Access by index | O(1) | O(n) | — |
| Search by value | O(n) | O(n) | O(log n) |
| Insert/remove at known position | O(n) (shift elements) | O(1) | O(log n) |

`O(1)` = constant, doesn't get slower as data grows. `O(log n)` = grows slowly (doubling the data adds one more step). `O(n)` = grows linearly with the data. This is the vocabulary you need to explain *why* a `HashMap` lookup is fast or why inserting into the middle of an `ArrayList` is expensive — both come up in section 04.

## Boxed vs. unboxed types, revisited

Section 01 covered `==` vs `.equals()` on boxed types. The other place this bites: an `int[]` stores raw primitives (compact, fast), while a data structure built for objects — a `LinkedList<Integer>`, a `List<Integer>` backed by an array — stores boxed `Integer`s, each a separate heap object. That's why `int[]` outperforms `List<Integer>` for large numeric data, and why autoboxing inside a tight loop is a common, easy-to-miss performance mistake.

## Exercise: balanced parentheses

Write a method `boolean isBalanced(String input)` that checks whether every `(`, `[`, and `{` in a string has a matching, correctly-nested closing bracket. Examples:

```java
isBalanced("({[]})")   // true
isBalanced("([)]")     // false — crossed, not nested
isBalanced("(()")      // false — unclosed
```

Implement it using a `Deque` as a stack (push opening brackets, pop and check when you hit a closing one) — this is the canonical use case for a stack, and you'll recognize the pattern again in real parsing/validation code.

## Checklist

- [ ] Can explain why array access is O(1) but linked-list access is O(n)
- [ ] Know the LIFO/FIFO distinction and can use `Deque` as both a stack and a queue
- [ ] Can explain the BST ordering property (left < node < right)
- [ ] Comfortable reading O(1) / O(log n) / O(n) and saying which is faster
- [ ] `isBalanced` passes on nested, crossed, and unclosed bracket inputs
