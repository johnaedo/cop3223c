---
share_cop3223c: "true"
site-folder: docs/Lecture Slides
theme: ucf-knights.css
height: "1080"
width: "1920"
---

# Linked Lists
## COP 3223C — Lecture 1 of 2
### Nodes, Pointers, and Singly-Linked Lists

---

## The Problem With Arrays

Both fixed and dynamic arrays share one fundamental limitation:

```c
/* Insert 99 at position 2 */
/* In a 10-element array: */
int i;
for (i = 9; i > 2; i--)
    arr[i] = arr[i-1];
arr[2] = 99;
```

**Shifting is O(n).** The bigger the array, the more elements must move.

> Linked lists solve insertion and deletion at the cost of sequential access.

---

## What Is a Linked List?

A linked list is a chain of **nodes**, where each node holds:

1. **Data** — the value stored
2. **A pointer to the next node** — the link

```
[data|next]──►[data|next]──►[data|next]──►NULL
    node 0        node 1        node 2
```

There is no contiguous block of memory. Nodes live wherever `malloc` puts them.

---

## The Node — Self-Referential Struct

```c
typedef struct Node {
    int          data;
    struct Node *next;
} Node;
```

**Why `struct Node` inside the typedef?**

The type `Node` is not fully defined yet when the struct body is being parsed. We must use the struct tag `struct Node` to refer to the type being defined. The typedef alias `Node` is only available after the closing `}`.

---

## Visualizing Node Memory

```
Node n:

┌──────────────────────┐
│ data  │    4 bytes   │
├──────────────────────┤
│ next  │    8 bytes   │
│       │  (pointer)   │
└──────────────────────┘

sizeof(Node) == 12 or 16
(platform-dependent alignment)
```

`next` is just a pointer — it holds an address, nothing more.

---

## Creating a Single Node

```c
#include <stdlib.h>
#include <stdio.h>

Node *node_create(int data) {
    Node *n = malloc(sizeof(Node));
    if (n == NULL) {
        perror("malloc");
        return NULL;
    }
    n->data = data;
    /* Terminate the chain */
    n->next = NULL;
    return n;
}
```

Always set `next = NULL`. An uninitialized pointer is a time bomb.

---

## Building a List — The Head Pointer

```c
/* head points to the first node.
   An empty list: head == NULL */
Node *head = NULL;

/* Create node with data = 10 */
Node *n = node_create(10);

/* This node IS the list */
head = n;

/* head ──► [10|NULL] */
```

The **head pointer** is the only way into the list. Lose it and the list is gone (leaked).

---

## Prepend — Insert at Front

```c
/* Add new node before current
   head — O(1) */
Node *prepend(Node *head, int data)
{
    Node *n = node_create(data);
    if (n == NULL) return head;

    /* New node points to old head */
    n->next = head;

    /* New node IS the new head */
    return n;
}
```

---

## Prepend — Step by Step
```c
head = prepend(head, 20);
head = prepend(head, 30);
/* 30 ──► 20 ──► 10 ──► NULL */
```
```
Before: head ──► [10|NULL]

Step 1: allocate new node [20|?]
Step 2: new->next = head
        [20|──►]──► [10|NULL]
Step 3: return new (caller sets head)
        head ──► [20|──►]──► [10|NULL]
```

Two pointer assignments. O(1) regardless of list length.

---

## Append — Insert at Back

```c
/* Add new node after last node
   — O(n): must traverse */
Node *append(Node *head, int data)
{
    Node *n = node_create(data);
    if (n == NULL) return head;

    if (head == NULL) return n; /* was empty */

    /* Walk to last node */
    Node *cur = head;
    while (cur->next != NULL)
        cur = cur->next;

    cur->next = n;
    return head;
}
```

---

## Traversal — Visiting Every Node

```c
void list_print(Node *head) {
    Node *cur = head;

    while (cur != NULL) {
        printf("%d", cur->data);
        if (cur->next != NULL)
            printf(" -> ");
        cur = cur->next;
    }
    printf(" -> NULL\n");
}
```

**Pattern:** declare a `cur` pointer, start at `head`, advance with `cur = cur->next`, stop when `cur == NULL`.

---

## Never Lose the Head

```c
/* WRONG — head is now lost! */
while (head != NULL) {
    head = head->next; /* leak */
}

/* CORRECT — use a separate
   traversal pointer */
Node *cur = head;
while (cur != NULL) {
    cur = cur->next;
}
/* head still points to start */
```

`head` is your only handle to the whole list. Treat it like a bookmark — never advance it during traversal.

---

## List Length

```c
int list_length(Node *head) {
    int count = 0;
    Node *cur = head;

    while (cur != NULL) {
        count++;
        cur = cur->next;
    }
    return count;
}
```

$O(n)$ — there is no shortcut without storing the length separately.

---

## Search — Find a Node

```c
/* Return pointer to first node
   where data == target,
   or NULL if not found */
Node *list_search(
    Node *head, int target)
{
    Node *cur = head;
    while (cur != NULL) {
        if (cur->data == target)
            return cur;
        cur = cur->next;
    }
    return NULL;
}
```

$O(n)$ — no random access like arrays.

---

## Delete — Remove First Node

```c
/* Remove head node; return
   new head */
Node *delete_front(Node *head) {
    if (head == NULL)
        return NULL;

    Node *old = head;
    head = head->next;

    free(old);
    old = NULL;

    return head;
}
```
---
## Delete - Usage

```c
head = delete_front(head);
/* 30 ──► 20 ──► 10  becomes
   20 ──► 10              */
```

---

## Delete — Remove by Value

```c
Node *list_delete(
    Node *head, int target)
{
    /* Empty list */
    if (!head) return NULL;

    /* Target is at head */
    if (head->data == target) {
        Node *tmp = head->next;
        free(head);
        return tmp;
    }
```

---
### Delete - Cont'd
```c
    /* Search from second node */
    Node *cur = head;
    while (cur->next != NULL) {
        if (cur->next->data
            == target) {
            Node *del =
                cur->next;
            cur->next =
                del->next;
            free(del);
            return head;
        }
        cur = cur->next;
    }
    return head; /* not found */
}
```

---

## Delete by Value — Diagram

```
List: 30 ──► 20 ──► 10 ──► NULL
Delete 20:

cur points to 30
cur->next->data == 20  ✓

Step 1: del = cur->next (20 node)
Step 2: cur->next = del->next (10)
Step 3: free(del)

Result: 30 ──► 10 ──► NULL
```

The "trailing pointer" technique — keep `cur` one node behind the target.

---

## Free the Entire List

```c
void list_free(Node *head) {
    Node *cur = head;
    Node *next_node;

    while (cur != NULL) {
        /* Save next BEFORE free */
        next_node = cur->next;
        free(cur);
        cur = next_node;
    }
}
```

**Critical:** save `cur->next` before `free(cur)`. After `free`, reading `cur->next` is undefined behavior.

---

## Insert in Sorted Order

```c
Node *list_insert_sorted(
    Node *head, int data)
{
    Node *n = node_create(data);
    if (!n) return head;

    /* Insert before head */
    if (!head ||
        data <= head->data) {
        n->next = head;
        return n;
    }
```
---
### Sorted Insert cont'd
```c
    Node *cur = head;
    while (cur->next &&
           cur->next->data < data)
    cur = cur->next;
    n->next = cur->next;
    cur->next = n;
    return head;
}
```

---

## Singly-Linked List: Operations Summary

| Operation | Time | Notes |
|-----------|------|-------|
| Prepend | O(1) | Update head |
| Append | O(n) | Must walk to tail |
| Search | O(n) | Sequential scan |
| Delete front | O(1) | Update head |
| Delete by value | O(n) | Trail pointer |
| Length | O(n) | No shortcut |
| Index access | O(n) | No random access |

---

## Linked List vs Array — When to Choose

| | Array | Linked List |
|-|-------|-------------|
| **Access by index** | O(1) | O(n) |
| **Insert at front** | O(n) | O(1) |
| **Insert at back** | O(1) amort | O(n) |
| **Insert at middle** | O(n) | O(1) with pointer |
| **Memory** | Contiguous | Scattered + pointer overhead |
| **Cache performance** | Excellent | Poor |
| **Resize** | `realloc` | Just `malloc` one node |

---

### Common Linked List Bugs

```c
/* Bug 1: forgetting to update head on insert.
	CORRECT:  head = prepend(head, 5) */
prepend(head, 5);

/* Bug 2: dereferencing NULL */
cur->next->data; /* if cur->next is NULL */

/* Bug 3: losing next before free */
free(cur);
cur = cur->next; /* UB! */

/* Bug 4: not returning new head */
Node *list_delete(Node *head, ...) {
    /* do stuff */
    return; /* forgot to return head! */
}
```

---

## Lecture 1 Summary

| Concept                 | Key Points                             |
| ----------------------- | -------------------------------------- |
| Node                    | `data` + `next` pointer                |
| Self-referential struct | Use `struct Tag` inside body           |
| Head pointer            | Only entry to the list                 |
| Traversal               | `cur = head; while(cur) cur=cur->next` |
| Prepend                 | O(1) — rewire head                     |
| Append                  | O(n) — walk to tail                    |
| Delete                  | Save next, free, relink                |
| Free list               | Save next before free                  |

**Next lecture:** Doubly-linked lists, tail pointers, and common applications

---

# Linked Lists
## COP 3223C — Lecture 2 of 2
### Doubly-Linked Lists, Tail Pointers & Applications

---

## Singly-Linked List Limitations

What can't we do efficiently with a singly-linked list?

- **Delete the tail** — O(n): must walk to find the node before it
- **Walk backwards** — impossible; there's no `prev` pointer
- **Insert before a known node** — O(n): need the predecessor

Solution: add a `prev` pointer to each node.

---

## The Doubly-Linked Node

```c
typedef struct DNode {
    int          data;
    struct DNode *prev;
    struct DNode *next;
} DNode;
```

```
NULL ◄──[prev|data|next]──►[prev|data|next]──► NULL
              node 0              node 1
```

Every node knows both its predecessor and successor.

---

## The List Header Struct

Instead of a bare pointer, wrap the list in a struct:

```c
typedef struct {
    DNode *head;
    DNode *tail;
    int    size;
} List;
```

```c
void list_init(List *L) {
    L->head = NULL;
    L->tail = NULL;
    L->size = 0;
}
```

Benefits: O(1) tail access, O(1) size, cleaner API.

---

## DLL Prepend — Insert at Front

```c
void dll_prepend(
    List *L, int data)
{
    DNode *n = malloc(
        sizeof(DNode)
    );
    if (!n) return;
    n->data = data;
    n->prev = NULL;
    n->next = L->head;

    if (L->head)
        L->head->prev = n;
    else
        L->tail = n; /* first node */

    L->head = n;
    L->size++;
}
```

---

## DLL Prepend — Diagram

```
Before: head ──► [NULL|10|NULL] ◄── tail

Insert 20:
  new node: [NULL|20|?]

Step 1: new->next = head        (20 ──► 10)
Step 2: head->prev = new        (20 ◄── 10)
Step 3: head = new

After: head ──► [NULL|20|──►][◄──|10|NULL] ◄── tail
```

---

## DLL Append — Insert at Back

```c
void dll_append(
    List *L, int data)
{
    DNode *n = malloc(sizeof(DNode));
    if (!n) return;
    n->data = data;
    n->next = NULL;
    n->prev = L->tail;

    if (L->tail)
        L->tail->next = n;
    else
        L->head = n; /* first node */

    L->tail = n;
    L->size++;
}
```

$O(1)$ because we keep a `tail` pointer.

---

## DLL Delete — Remove a Known Node

With a `prev` pointer, deleting a node you already have a pointer to is O(1):

```c
void dll_delete_node(
    List *L, DNode *n)
{
    if (n->prev)
        n->prev->next = n->next;
    else
        L->head = n->next;

    if (n->next)
        n->next->prev = n->prev;
    else
        L->tail = n->prev;

    free(n);
    L->size--;
}
```

---

## DLL Delete — Diagram

```
List: 30 ◄──► 20 ◄──► 10
Delete node 20:

Before:
  30->next = 20,  20->prev = 30
  20->next = 10,  10->prev = 20

Step 1: 20->prev->next = 20->next
        → 30->next = 10
Step 2: 20->next->prev = 20->prev
        → 10->prev = 30
Step 3: free(20)

After: 30 ◄──► 10
```

---

## DLL Traversal — Forward and Backward

```c
/* Forward */
void dll_print_fwd(List *L) {
    DNode *cur = L->head;
    while (cur) {
        printf("%d ", cur->data);
        cur = cur->next;
    }
    printf("\n");
}

/* Backward — use tail */
void dll_print_rev(List *L) {
    DNode *cur = L->tail;
    while (cur) {
        printf("%d ", cur->data);
        cur = cur->prev;
    }
    printf("\n");
}
```

---

## Freeing a Doubly-Linked List

```c
void dll_free(List *L) {
    DNode *cur = L->head;
    DNode *nxt;

    while (cur) {
        nxt = cur->next;
        free(cur);
        cur = nxt;
    }

    /* Reset the struct */
    L->head = NULL;
    L->tail = NULL;
    L->size = 0;
}
```

Same pattern as singly-linked: save `next`, then free.

---

## Circular Linked List

A variation where the last node points back to the first:

```
head ──► [A|next]──►[B|next]──►[C|next]
              ▲__________________________|
```

```c
/* Make a singly-linked list
   circular */
Node *tail = /* walk to end */;
tail->next = head; /* close loop */
```

**Traversal requires a stop condition** — you can't rely on `cur == NULL`:

```c
Node *cur = head;
do {
    process(cur);
    cur = cur->next;
} while (cur != head);
```

---

## Sentinel / Dummy Nodes

A **sentinel** is a placeholder node at the head (and/or tail) that is never deleted and never holds real data:

```c
/* Create list with dummy head */
Node *sentinel = node_create(0);
sentinel->next = NULL;
/* Real data starts at sentinel->next */
```

Benefits:
- Eliminates the `if (head == NULL)` special case in insert/delete
- All insertions look the same — always "insert after some node"

---

## Stack Using a Linked List

```c
/* Push: prepend O(1) */
void stack_push(
    Node **top, int val)
{
    *top = prepend(*top, val);
}

/* Pop: delete front O(1) */
int stack_pop(Node **top) {
    if (!*top) return -1;
    int val = (*top)->data;
    *top = delete_front(*top);
    return val;
}
```

Linked-list stacks never need resizing.

---

## Queue Using a Linked List

```c
typedef struct {
    Node *front; /* dequeue here */
    Node *back;  /* enqueue here */
} Queue;

/* Enqueue: append to back O(1)
   (keep tail pointer) */

/* Dequeue: remove from front O(1) */
int dequeue(Queue *q) {
    if (!q->front) return -1;
    int val = q->front->data;
    Node *old = q->front;
    q->front = q->front->next;
    if (!q->front) q->back = NULL;
    free(old);
    return val;
}
```

---

## Linked List Applications in Practice

| Use Case            | List Type | Why                       |
| ------------------- | --------- | ------------------------- |
| Undo history        | Singly    | Push/pop at one end       |
| Browser history     | Doubly    | Forward + back navigation |
| OS process queue    | Circular  | Round-robin scheduling    |
| Text editor         | Doubly    | Insert/delete at cursor   |
| Hash table chaining | Singly    | Collision resolution      |
| Memory allocator    | Singly    | Free-block tracking       |

---

## Recursion and Linked Lists

Many list operations have elegant recursive forms:

```c
/* Print list recursively */
void list_print_r(Node *n) {
    if (n == NULL) return;
    printf("%d ", n->data);
    list_print_r(n->next);
}

/* Print in reverse — recursive */
void list_print_rev_r(Node *n) {
    if (n == NULL) return;
    /* Recurse first, then print */
    list_print_rev_r(n->next);
    printf("%d ", n->data);
}
```

---

## Recursive List Length

```c
int list_len_r(Node *n) {
    if (n == NULL) return 0;
    return 1 + list_len_r(n->next);
}
```

Each call adds 1 for the current node plus the length of the rest. Base case: empty list has length 0.

**Warning:** large lists cause deep recursion → stack overflow. Iterative is safer for production code.

---

## Lecture 2 Summary

| Concept            | Key Points                               |
| ------------------ | ---------------------------------------- |
| Doubly-linked      | `prev` + `next`; backward traversal      |
| List header struct | `head`, `tail`, `size` for O(1) access   |
| DLL append         | O(1) with tail pointer                   |
| DLL delete         | O(1) for known-pointer deletion          |
| Circular list      | `tail->next = head`; need stop condition |
| Sentinel node      | Eliminates NULL edge cases               |
| Stack / Queue      | Natural linked-list applications         |
| Recursion          | Elegant but risky for large lists        |

---

## The Linked List Mental Checklist

```
Before writing any linked list code:

□ Is head updated on insert/delete?
□ Is tail updated (if DLL)?
□ Is size updated (if tracked)?
□ Is next saved before free()?
□ Is next = NULL set on new nodes?
□ Is prev set correctly (DLL)?
□ Are both head and tail handled
  for empty-list edge case?
□ Is the whole list freed?
```

---

## Looking Ahead

**Next week:** Recursion & Searching
- Binary search on arrays
- Recursive tree traversal
- Merge sort (uses linked lists naturally)

The dungeon's room connections will become a proper graph — rooms pointing to rooms via linked exit lists.
