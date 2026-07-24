---
share_cop3223c: "true"
site-folder: docs/Lecture Slides
theme: ucf-knights.css
height: "1080"
width: "1920"
---

# Lab: Linked Lists
### COP 3223C — TA-Led Lab (50 minutes)
**Prerequisites:** Both linked list lectures.

---

## Learning Objectives

By the end of this lab, students will be able to:

1. Build and traverse a singly-linked list using `malloc` and pointer manipulation
2. Implement insertion, deletion, and search on a linked list
3. Correctly free an entire linked list without memory errors
4. Recognize and explain common linked-list pointer bugs

---

## Setup (5 minutes)

```bash
mkdir lab_linkedlist && cd lab_linkedlist
```

> **Compile with address sanitizer throughout:**
> ```bash
> gcc -Wall -Wextra -g \
>     -fsanitize=address \
>     -o prog prog.c
> ```

All exercises use this node definition — put it in `node.h` once and include it everywhere:

```c
/* node.h */
#ifndef NODE_H
#define NODE_H

typedef struct Node {
    int          data;
    struct Node *next;
} Node;

/* Allocate a node; returns NULL
   on failure */
Node *node_create(int data);

/* Free a node (not the list) */
void  node_free(Node *n);

#endif
```

```c
/* node.c */
#include "node.h"
#include <stdlib.h>

Node *node_create(int data) {
    Node *n = malloc(sizeof(Node));
    if (!n) return NULL;
    n->data = data;
    n->next = NULL;
    return n;
}

void node_free(Node *n) {
    free(n);
}
```

---

## Part 1: Build and Print (10 minutes)

### Task

Write `list1.c`. Complete the three functions below, then test with `main`.

```c
/* list1.c */
#include <stdio.h>
#include <stdlib.h>
#include "node.h"

/* Insert at front — return
   new head */
Node *prepend(
    Node *head, int data)
{
    /* TODO */
    return head;
}

/* Insert at back — return head */
Node *append(
    Node *head, int data)
{
    /* TODO */
    return head;
}

/* Print: 1 -> 2 -> 3 -> NULL */
void list_print(Node *head) {
    /* TODO */
}

/* Free entire list */
void list_free(Node *head) {
    /* TODO */
}

int main(void) {
    Node *head = NULL;

    /* Build: 10 -> 20 -> 30 */
    head = append(head, 10);
    head = append(head, 20);
    head = append(head, 30);
    list_print(head);

    /* Prepend 5 */
    head = prepend(head, 5);
    list_print(head);

    list_free(head);
    head = NULL;
    return 0;
}
```

### Expected Output

```
10 -> 20 -> 30 -> NULL
5 -> 10 -> 20 -> 30 -> NULL
```

---

## Part 2: Search and Delete (12 minutes)

### Task

Extend your work into `list2.c`. Add the three functions below. Copy your working `prepend`, `append`, `list_print`, and `list_free` from Part 1.

```c
/* Return pointer to first node
   with data == target, or NULL */
Node *list_search(
    Node *head, int target)
{
    /* TODO */
    return NULL;
}

/* Remove first node where
   data == target.
   Return (possibly new) head. */
Node *list_delete(
    Node *head, int target)
{
    /* TODO */
    return head;
}

/* Return number of nodes */
int list_length(Node *head) {
    /* TODO */
    return 0;
}
```

### Test `main`

```c
int main(void) {
    Node *head = NULL;
    int i;

    /* Build 1 -> 2 -> 3 -> 4 -> 5 */
    for (i = 1; i <= 5; i++)
        head = append(head, i);
    list_print(head);

    /* Search */
    Node *found =
        list_search(head, 3);
    if (found)
        printf("Found: %d\n",
               found->data);
    else
        printf("Not found\n");

    /* Delete middle */
    head = list_delete(head, 3);
    list_print(head);

    /* Delete head */
    head = list_delete(head, 1);
    list_print(head);

    /* Delete tail */
    head = list_delete(head, 5);
    list_print(head);

    /* Delete non-existent */
    head = list_delete(head, 99);
    list_print(head);

    printf("Length: %d\n",
           list_length(head));

    list_free(head);
    return 0;
}
```

### Expected Output

```
1 -> 2 -> 3 -> 4 -> 5 -> NULL
Found: 3
1 -> 2 -> 4 -> 5 -> NULL
2 -> 4 -> 5 -> NULL
2 -> 4 -> NULL
2 -> 4 -> NULL
Length: 2
```

---

## Part 3: Sorted Insertion (13 minutes)

### Task

Write `list3.c`. Build a sorted singly-linked list from user input — all integers should be kept in ascending order at all times. No sorting after the fact: insert directly into the correct position.

```c
/* Insert data so the list
   stays sorted ascending.
   Return (possibly new) head. */
Node *insert_sorted(
    Node *head, int data)
{
    /* TODO */
    return head;
}
```

### Test `main`

```c
int main(void) {
    Node *head = NULL;
    int vals[] = {
        5, 2, 8, 1, 9, 3, 7
    };
    int n = 7, i;

    for (i = 0; i < n; i++) {
        head = insert_sorted(
            head, vals[i]
        );
        list_print(head);
    }

    list_free(head);
    return 0;
}
```

### Expected Output

```
5 -> NULL
2 -> 5 -> NULL
2 -> 5 -> 8 -> NULL
1 -> 2 -> 5 -> 8 -> NULL
1 -> 2 -> 5 -> 8 -> 9 -> NULL
1 -> 2 -> 3 -> 5 -> 8 -> 9 -> NULL
1 -> 2 -> 3 -> 5 -> 7 -> 8 -> 9 -> NULL
```

---

## Part 4: Bug Hunt (10 minutes)

Each snippet has one or more bugs. Identify what goes wrong and how to fix it.

**Snippet A — Traversal that destroys the list:**
```c
Node *head = /* some list */;
while (head != NULL) {
    printf("%d\n", head->data);
    head = head->next;
}
/* What is head now? */
```

**Snippet B — Use-after-free:**
```c
Node *cur = head;
while (cur != NULL) {
    free(cur);
    cur = cur->next;
}
```

**Snippet C — Unlinked insert:**
```c
Node *new_node =
    node_create(42);
new_node->next = head->next;
/* What happened to new_node? */
```

**Snippet D — Leaking delete:**
```c
Node *list_delete(
    Node *head, int val)
{
    Node *cur = head;
    while (cur->next) {
        if (cur->next->data
            == val) {
            cur->next =
                cur->next->next;
            return head;
        }
        cur = cur->next;
    }
    return head;
}
```

**Snippet E — NULL dereference on empty list:**
```c
Node *list_delete(
    Node *head, int val)
{
    if (head->data == val) {
        Node *tmp = head->next;
        free(head);
        return tmp;
    }
    /* ... */
}
```

### Answers (TA — reveal after discussion)

- **A:** `head` is advanced to NULL, destroying the only reference to the list. Use a separate `cur` pointer.
- **B:** `cur->next` is read after `free(cur)` — undefined behavior. Save `next = cur->next` before `free(cur)`, then `cur = next`.
- **C:** `new_node->next` is set, but nothing points to `new_node` — it is immediately leaked. Should be `head->next = new_node` or reassign the predecessor's `next`.
- **D:** The deleted node `cur->next` is never `free()`'d — memory leak.
- **E:** If `head == NULL`, `head->data` is a NULL pointer dereference — segfault. Must guard with `if (!head) return NULL;` first.

---

## Wrap-Up & Discussion (5 minutes)

**TA-led questions (pick 2–3):**

1. In `list_delete`, why do we need to handle the head case separately from the middle/tail case?
2. What would happen if we called `list_free` on a circular linked list with the implementation from Part 1?
3. In Part 3, what is the time complexity of building the entire sorted list by repeated `insert_sorted`? (Hint: think about what each insertion costs.)
4. Could you implement `insert_sorted` without a special case for inserting before the head? Describe how.
5. If you needed O(1) append AND O(1) prepend, what single additional pointer would you add to your list? Where would you maintain it?

---

## Submission

No submission required. TAs should check off completion of Parts 1–3.

---

## TA Reference Solutions

### Part 1

```c
Node *prepend(
    Node *head, int data)
{
    Node *n = node_create(data);
    if (!n) return head;
    n->next = head;
    return n;
}

Node *append(
    Node *head, int data)
{
    Node *n = node_create(data);
    if (!n) return head;
    if (!head) return n;
    Node *cur = head;
    while (cur->next)
        cur = cur->next;
    cur->next = n;
    return head;
}

void list_print(Node *head) {
    Node *cur = head;
    while (cur) {
        printf("%d", cur->data);
        if (cur->next)
            printf(" -> ");
        cur = cur->next;
    }
    printf(" -> NULL\n");
}

void list_free(Node *head) {
    Node *cur = head, *nxt;
    while (cur) {
        nxt = cur->next;
        free(cur);
        cur = nxt;
    }
}
```

### Part 2

```c
Node *list_search(
    Node *head, int target)
{
    Node *cur = head;
    while (cur) {
        if (cur->data == target)
            return cur;
        cur = cur->next;
    }
    return NULL;
}

Node *list_delete(
    Node *head, int target)
{
    if (!head) return NULL;
    if (head->data == target) {
        Node *tmp = head->next;
        free(head);
        return tmp;
    }
    Node *cur = head;
    while (cur->next) {
        if (cur->next->data
            == target) {
            Node *del = cur->next;
            cur->next = del->next;
            free(del);
            return head;
        }
        cur = cur->next;
    }
    return head;
}

int list_length(Node *head) {
    int n = 0;
    Node *cur = head;
    while (cur) { n++; cur=cur->next; }
    return n;
}
```

### Part 3

```c
Node *insert_sorted(
    Node *head, int data)
{
    Node *n = node_create(data);
    if (!n) return head;

    if (!head ||
        data <= head->data) {
        n->next = head;
        return n;
    }

    Node *cur = head;
    while (cur->next &&
           cur->next->data < data)
        cur = cur->next;

    n->next = cur->next;
    cur->next = n;
    return head;
}
```
