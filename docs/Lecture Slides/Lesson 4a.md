---
share_cop3223c: "true"
site-folder: docs/Lecture Slides
theme: dracula-custom.css
height: "1080"
width: "1920"
---

# Week 4, Lecture 1: Pointers

---

## Agenda

1. What memory addresses are
2. The `&` and `*` operators
3. Pointer types and why they matter
4. Pass-by-pointer — fixing `swap()`
5. C has no pass-by-reference
6. `NULL` and pointer initialization
7. `const` with pointers

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 1

## What Memory Addresses Are

---

## Every Variable Has an Address

Every variable your program declares occupies some number of bytes in memory. Each byte has a unique numeric **address** — think of it as a house number on a very long street.

```c
int x = 42;
```

```
Address       Value
──────────────────────────────
0x7ffd1a00    42   ← first byte of x
0x7ffd1a01    00   ← (int is 4 bytes on most platforms)
0x7ffd1a02    00
0x7ffd1a03    00
0x7ffd1a04    ...  ← next variable starts here
```

The address is just a number. The value stored at that address is whatever you put there.

---

## A Simplified View of Program Memory

```
High addresses
┌─────────────────────┐
│   Stack             │ ← local variables, function frames
│   (grows downward)  │
├─────────────────────┤
│                     │
│   (unused gap)      │
│                     │
├─────────────────────┤
│   Heap              │ ← dynamic allocation — Week 4
├─────────────────────┤
│   Global / Static   │ ← global variables
├─────────────────────┤
│   Code (Text)       │ ← your compiled instructions
└─────────────────────┘
Low addresses
```

Local variables live on the **stack**. We will explore this layout in full detail next week. For now: every variable has an address, and that address is a number.

---

## What Is a Pointer?

A **pointer** is a variable whose value is a memory address.

```c
int  x = 42;    // x holds an integer
int *p = &x;    // p holds the ADDRESS of x
```

```
        x                       p
  ┌──────────┐            ┌──────────────────┐
  │    42    │◀───────────│  0x7ffd1a00      │
  └──────────┘            └──────────────────┘
  addr: 0x7ffd1a00        addr: 0x7ffd1a08
```

`p` is not `42`. `p` is the address where `42` lives.

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 2

## The `&` and `*` Operators

---

## `&` — Address-Of

`&variable` produces the memory address of `variable`.

```c
int x = 42;
printf("Value:   %d\n",  x);  // 42
printf("Address: %p\n", &x);  // something like 0x7ffd1a00
```

`%p` is the format specifier for pointers — it prints the address in hexadecimal.

> The exact address varies every run. The OS randomizes stack placement as a security measure (ASLR — Address Space Layout Randomization).

---

## `*` — Dereference

`*pointer` follows the address and gives you the value stored there.

```c
int  x = 42;
int *p = &x;

printf("%d\n", *p); // 42 — follows p to x, reads the val
```

"Go to the address stored in `p` and give me what is there."

---

## Reading and Writing Through a Pointer

```c
int  x = 42;
int *p = &x;

*p = 100;              // write through the pointer
printf("%d\n", x);    // 100 — x was modified via p
printf("%d\n", *p);   // 100 — same location, same value
```

`*p = 100` does not change what address `p` holds. It changes the **value at** that address.

---

## The Two Lives of `*`

```c
int *p = &x;   // DECLARATION — "p is a pointer to int"
*p = 100;      // EXPRESSION — deref p, write through it
int y = *p;    // EXPRESSION — deref p, read through it
int z = a * b; // EXPRESSION — multiply, unrelated
```

| Context | Meaning |
|---------|---------|
| In a type declaration `int *p` | "p is a pointer to int" |
| As unary prefix `*p` | Dereference — follow the pointer |
| Between two values `a * b` | Multiplication |

This overloading is genuinely confusing at first. Read pointer declarations slowly and deliberately.

---

## Declaration Syntax — A Common Trap

```c
// MISLEADING: looks like both are pointers
int* p, q; 
// CLEARER: only p is a pointer; q is a plain int
int *p, q; 
// Both are pointers
int *p, *q;  
```

The `*` binds to the **variable name**, not the type. `int*` is not a standalone type that distributes across a declaration. The community convention is `int *p` — star with the variable.

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 3

## Pointer Types

---

## Why Type Matters

All pointers on a 64-bit system are 8 bytes — they all hold addresses. So why isn't there just one pointer type?

```c
int    x  = 42;
double d  = 3.14;

int    *pi = &x;
double *pd = &d;
```

The type tells the compiler **how many bytes to read** on dereference and **how to interpret** those bytes.

---

## Same Address, Different Type, Different Read

```c
int   x  = 305419896;   // 0x12345678
int  *pi = &x;
char *pc = (char *)&x;

// reads 4 bytes → 305419896
printf("%d\n", *pi); 
// reads 1 byte  → 120 (0x78, lowest byte on x86)
printf("%d\n", *pc); 
```

Dereferencing through the wrong type reads the wrong number of bytes and interprets them incorrectly. This is undefined behavior and one of C's most dangerous failure modes.

---

## Pointer Arithmetic

Adding an integer to a pointer advances it by **that many elements**, not that many bytes:

```c
int arr[4] = {10, 20, 30, 40};
int *p = &arr[0];

printf("%d\n", *p);       // 10
// 20 p advanced by sizeof(int) = 4 bytes
printf("%d\n", *(p + 1)); 
printf("%d\n", *(p + 2)); // 30
printf("%d\n", *(p + 3)); // 40
```

`p + 1` adds `sizeof(int)` bytes to the address. The type is what makes arithmetic meaningful.

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 4

## Pass-by-Pointer

---

## Recall: The Broken Swap (Week 2)

```c
void swap(int a, int b) {
    int temp = a;
    a = b;
    b = temp;   // modifies copies — caller unchanged
}

int x = 1, y = 2;
swap(x, y);
printf("%d %d\n", x, y);   // still: 1 2
```

`swap` receives copies of `x` and `y`. The originals are never touched.

---

## The Fix: Pass the Addresses

```c
void swap(int *a, int *b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}

int x = 1, y = 2;
swap(&x, &y);
printf("%d %d\n", x, y);   // 2 1 ✓
```

`swap` now receives the **addresses** of `x` and `y`. Those addresses are copied by value — but dereferencing them reaches the originals.

---

## Visualizing Pass-by-Pointer

```
main's stack frame          swap's stack frame
┌─────────────┐             ┌──────────────────────┐
│   x  =  1   │◀── *a ────  │   a  =  &x           │
│   y  =  2   │◀── *b ────  │   b  =  &y           │
└─────────────┘             │   temp = *a  (= 1)   │
                            │   *a = *b   (x → 2)  │
                            │   *b = temp (y → 1)  │
                            └──────────────────────┘
```

The pointer values (`a` and `b`) are copied. The variables they point to (`x` and `y`) are modified through those copies.

---

## The Pattern: Output Parameters

Pass-by-pointer is also how functions "return" multiple values:

```c
void min_max(int arr[], int len, int *out_min, int *out_max) {
    *out_min = arr[0];
    *out_max = arr[0];
    for (int i = 1; i < len; i++) {
        if (arr[i] < *out_min) *out_min = arr[i];
        if (arr[i] > *out_max) *out_max = arr[i];
    }
}

int lo, hi;
min_max(data, 5, &lo, &hi);
printf("min: %d, max: %d\n", lo, hi);
```

Output parameters are a standard C idiom. The `&` at the call site signals that these variables will be written to.

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 5

## C Has No Pass-by-Reference

---

## C Makes the Indirection Explicit

```c
// C — all indirection is visible in the source
void swap(int *a, int *b) { ... }   // * in signature
swap(&x, &y);                       // & at call site
*a = *b;                            // * in body
```

| | C++ reference parameter | C pointer parameter |
|--|------------------------|---------------------|
| Signature | `void f(int &a)` | `void f(int *a)` |
| Call site | `f(x)` | `f(&x)` |
| Body | `a = 5` | `*a = 5` |
| Caller can tell `x` may change? | **No** | **Yes** — `&` is the signal |

---

## The Rule to Memorize

> **In C, the only thing that ever crosses a function boundary is a value. Sometimes that value happens to be an address.**

"Pass-by-reference" in the context of C is informal shorthand for "pass-by-value of a pointer." When you hear it, mentally translate it. They are not the same mechanism.

The explicit `&` and `*` are not a limitation — they are a readability feature. Any function that can modify your variable will announce it clearly at the call site.

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 6

## NULL and Pointer Initialization

---

## Uninitialized Pointers Are Dangerous

```c
// p contains whatever bytes happened to be in that memory
int *p;
// writes 42 to a random address — undefined behavior        
*p = 42;       
```

An uninitialized pointer is one of C's most dangerous constructs. It may:
- Crash immediately (segmentation fault)
- Silently corrupt other data
- Appear to work until a demo or deployment

**Always initialize every pointer at the point of declaration.**

---

## NULL — The "Points to Nothing" Value

```c
int *p = NULL;   // safe — p explicitly points to nothing
```

`NULL` is a macro (from `<stddef.h>` and others) that expands to a null pointer constant — guaranteed to compare unequal to any valid address.

```c
if (p != NULL) {
    *p = 42;      // safe to dereference
}
```

---

## Returning NULL as a Sentinel

```c
// Returns a pointer to the first negative element, 
// or NULL if none found
int *find_negative(int arr[], int len) {
    for (int i = 0; i < len; i++) {
        if (arr[i] < 0) return &arr[i];
    }
    return NULL;
}

int data[] = {5, -3, 8, 2};
int *result = find_negative(data, 4);

if (result != NULL) {
    printf("Found: %d\n", *result);
}
```

Returning `NULL` to signal "not found" or "failure" is a fundamental C idiom. **Always check the return value before dereferencing.**

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 7

## `const` with Pointers

---

## Two Things Can Be `const`

With a pointer, `const` can qualify the **pointed-to value** or the **pointer itself**. Position determines which.

```c
// pointer to const int — *p is read-only; p can change
const int *p;
// const pointer to int — p is fixed; *p can change         
int *const p;         
const int *const p;   // both are const
```

**Reading right-to-left:** "p is a [const] pointer to [const] int."

---

## Pointer-to-`const` — The Common Case

```c
void print_name(const char *name) {
    printf("%s\n", name);
    // compile error — read-only through this pointer
    // name[0] = 'X';   
}
```

`const char *name` means: "I promise not to modify what `name` points to."

This allows the caller to safely pass string literals and `const` data without a cast, and tells the reader that this function only reads the string.

---

## Using `const` in Function Design

```c
// Reads src, writes into dest — const signals intent
void string_copy(char *dest, const char *src);
//               ^               ^
//               writable        read-only

// Reads both — can accept literals safely
int string_compare(const char *a, const char *b);
```

`const` in parameter lists is documentation the compiler enforces. Use it whenever a pointer parameter should not be used to modify the pointed-to data.

---

# Lecture 1 Wrap-Up

---

## Key Takeaways

- A **pointer** is a variable holding a memory **address**
- `&x` gives the address of `x`; `*p` follows the address in `p`
- Pointer **type** determines read width and arithmetic scaling
- Pass-by-pointer is **still pass-by-value** — the address is the value being copied
- **C has no pass-by-reference** — the `&` at every call site is a deliberate, visible signal
- Always **initialize** pointers; use **`NULL`** for "no valid address"; always **check before dereferencing**
- `const char *` means "read-only through this pointer" — use it for string parameters you won't modify

---

## Wednesday

- Arrays and pointer decay
- `sizeof` on arrays vs. pointers
- Strings as `char` arrays, the null terminator
- `string.h` — safe string operations
- The RPG inventory system

---

## Resources

- K&R Ch. 5 — Pointers and Arrays
- K.N. King Ch. 11 — Pointers
- https://en.cppreference.com/w/c/language/pointer
