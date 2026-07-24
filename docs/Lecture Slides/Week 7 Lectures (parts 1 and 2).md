---
share_cop3223c: "true"
site-folder: docs/Lecture Slides
theme: ucf-knights.css
height: "1080"
width: "1920"
---

# Multi-Dimensional Arrays, Pointers to Pointers & Dynamic Memory
## COP 3223C — Lecture 1 of 2
### Arrays of Arrays & The Heap

---

## Where We Are

So far our data has lived in two places:

| Location | Declared With | Lifetime |
|----------|---------------|---------|
| **Stack** | Local variables, fixed arrays | Until function returns |
| **Data segment** | `static`, global vars | Entire program |

Today we add a third:

| **Heap** | `malloc` / `calloc` | Until you call `free` |

---

## Stack vs Heap — The Core Contrast

```c
/* STACK — size known at compile time */
int scores[100];

/* HEAP — size decided at runtime */
int n = get_student_count();
int *scores = malloc(
    n * sizeof(int)
);
```

- Stack allocation is **automatic** — compiler handles it
- Heap allocation is **manual** — you are responsible
- Heap memory **outlives** the function that allocated it

---

## 2-D Arrays — The Concept

A 2-D array is an **array of arrays**.

```
grid[3][4]:

     col0  col1  col2  col3
row0 [  0][  1][  2][  3]
row1 [  4][  5][  6][  7]
row2 [  8][  9][ 10][ 11]
```

Declared as:

```c
/* 3 rows, 4 columns */
int grid[3][4];
```

---

## 2-D Array Memory Layout

Despite looking like a grid, memory is **flat** (row-major order):

```
grid[0][0] grid[0][1] grid[0][2] grid[0][3]
grid[1][0] grid[1][1] grid[1][2] grid[1][3]
grid[2][0] grid[2][1] grid[2][2] grid[2][3]

Address: 100  104  108  112  116  120  124 ...
```

Row 1 starts immediately after row 0 ends.

---

## Declaring & Initializing 2-D Arrays

```c
/* Uninitialized */
int a[3][4];

/* Fully initialized */
int b[3][4] = {
    {1,  2,  3,  4},
    {5,  6,  7,  8},
    {9, 10, 11, 12}
};

/* Partial — rest zeroed */
int c[3][4] = {
    {1, 2},
    {3}
};
```

---

## Accessing 2-D Array Elements

```c
int grid[3][4] = {0};
int row, col;

/* Set every element */
for (row = 0; row < 3; row++) {
    for (col = 0; col < 4; col++) {
        /* row-major index */
        grid[row][col] =
            row * 4 + col;
    }
}

/* Read one element */
printf("%d\n", grid[1][2]);
/* Prints: 6 */
```

---

## 2-D Arrays as Function Parameters

The **column count** must be specified — it's part of the type:

```c
/* Correct — columns fixed */
void print_grid(
    int g[][4],
    int rows)
{
    int r, c;
    for (r = 0; r < rows; r++) {
        for (c = 0; c < 4; c++)
            printf("%3d", g[r][c]);
        printf("\n");
    }
}
```

The row count can be a parameter, but columns must be a compile-time constant.

---

## Why Must Columns Be Fixed?

The compiler uses the column count to calculate element addresses:

```
address of grid[r][c]
  = base
  + r * (COLS * sizeof(int))
  + c * sizeof(int)
```

Without knowing `COLS` at compile time, it cannot generate the right arithmetic.

---

## 3-D and Higher Arrays

The same pattern extends:

```c
/* 2 layers, 3 rows, 4 columns */
int cube[2][3][4];

/* Access */
cube[layer][row][col] = 42;

/* As parameter */
void process(
    int c[][3][4],
    int layers)
{ /* ... */ }
```

Every dimension except the first must be a compile-time constant.

---

## Strings as 2-D Arrays

```c
/* Array of 5 strings,
   each up to 31 chars + '\0' */
char names[5][32];

/* Initialize */
char words[3][16] = {
    "hello",
    "world",
    "C"
};

/* Access */
printf("%s\n", words[1]);
/* Prints: world */
```

---

## Pointers — Quick Review

```c
int  x  = 42;

/* & gives the address of x */
int *p  = &x;

/* * dereferences — gives value */
printf("%d\n", *p);   /* 42 */

/* Writing through pointer */
*p = 99;
printf("%d\n", x);    /* 99 */
```

A pointer stores an **address**. Dereferencing it reads or writes the value at that address.

---

## Pointer to Pointer — `int **`

```c
int   x  = 42;
int  *p  = &x;   /* points to x */

/* pp points to p */
int **pp = &p;

/* Three ways to get x: */
printf("%d\n", x);    /* 42 */
printf("%d\n", *p);   /* 42 */
printf("%d\n", **pp); /* 42 */
```

Each `*` in the type adds one level of indirection.

---

## Visualizing `int **`

```
pp          p           x
[addr of p]─►[addr of x]─►[42]

*pp  == p        (the pointer)
**pp == x        (the int)
&pp  == address of pp itself
```

Think of it as a pointer that points to a pointer that points to data.

---

## Why Do We Need `int **`?

**Use case 1: modifying a pointer inside a function**

```c
void set_ptr(int **pp,
             int  *target) {
    /* Without **, changes
       would be local only */
    *pp = target;
}

int x = 5, y = 10;
int *p = &x;

/* Now p points to y */
set_ptr(&p, &y);
printf("%d\n", *p); /* 10 */
```

---

## `int **` — Use Case 2: 2-D Dynamic Arrays

An `int **` can point to a **jagged array** — an array of `int *` pointers, each pointing to a row:

```
pp ──► [ ptr ]──► [1][2][3][4]
       [ ptr ]──► [5][6][7]
       [ ptr ]──► [8][9]
```

- Rows can have **different lengths** (jagged)
- All allocated on the heap at runtime
- This is the dynamic 2-D array pattern

---

## `malloc` — Memory Allocation

```c
#include <stdlib.h>

/* Allocate n bytes on the heap.
   Returns void *, or NULL. */
void *malloc(size_t n);
```

```c
/* Allocate space for 10 ints */
int *arr = malloc(
    10 * sizeof(int)
);
if (arr == NULL) {
    perror("malloc");
    exit(EXIT_FAILURE);
}
```

**Always check for NULL.** `malloc` can fail.

---

## `sizeof` With Types and Variables

```c
/* Type — parentheses required */
malloc(10 * sizeof(int));
malloc(sizeof(MyStruct));

/* Variable — no parens needed */
int x;
malloc(sizeof x);

/* Array — gives total bytes */
int a[10];
printf("%zu\n", sizeof a);
/* Prints: 40 (on most systems) */
```

Prefer `sizeof(type)` for `malloc` arguments.

---

## `free` — Releasing Memory

```c
int *arr = malloc(
    10 * sizeof(int)
);
if (!arr) { /* handle error */ }

/* ... use arr ... */

/* Release the memory */
free(arr);

/* Good practice: null the pointer
   so it can't be used again */
arr = NULL;
```

Every `malloc` must have exactly one matching `free`.

---

## `calloc` — Zeroed Allocation

```c
/* calloc(count, size)
   allocates count*size bytes
   AND zeros all of them */
int *arr = calloc(10, sizeof(int));
if (!arr) { /* handle error */ }

/* All elements start at 0 */
printf("%d\n", arr[0]); /* 0 */

free(arr);
```

Use `calloc` when you need guaranteed zero-initialization. `malloc` leaves memory uninitialized.

---

## `realloc` — Resizing Allocations

```c
/* Resize an existing allocation */
void *realloc(
    // we'll talk about void pointers later!
    void  *ptr,
    size_t new_size
);
```
---

![800](Realloc%20-%20creating%20a%20new%20slab.md)

---

```c
int *arr = malloc(5 * sizeof(int));
/* ... fill arr[0..4] ... */

/* Grow to 10 ints */
int *tmp = realloc(
    arr, 10 * sizeof(int)
);
if (!tmp) {
    free(arr);
    return -1;
}
arr = tmp; /* tmp may differ */
```

---

## The `realloc` Pattern — Critical Points

```c
/* WRONG — if realloc fails,
   arr is lost (memory leak) */
arr = realloc(arr, new_size);

/* CORRECT — use a temp pointer */
int *tmp = realloc(arr, new_size);
if (tmp == NULL) {
    /* arr still valid — clean up */
    free(arr);
    return ERROR;
}
arr = tmp;
```

`realloc` may move the block to a new address. The old pointer is invalid after a successful `realloc`.

---

### Dynamic Array Pattern

```c
int  cap  = 4;
int  len  = 0;
int *data = malloc(
    cap * sizeof(int)
);
```

---
### Dynamic Array Pattern

```c
/* Add element — grow if needed */
void push(int val) {
    if (len == cap) {
        cap *= 2;
        int *t = realloc(
            data,
            cap * sizeof(int)
        );
        if (!t) { /* handle */ }
        data = t;
    }
    data[len++] = val;
}
```

Doubling capacity keeps amortized cost O(1) per insertion.

---

### Dynamic 1-D Array - Full Example

```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    int n = 5;

    int *arr = calloc(n, sizeof(int));
    if (!arr) {
        perror("calloc");
        return 1;
    }

```
---
### Dynamic 1-D Array - Cont'd

```c
    int i;
    for (i = 0; i < n; i++)
        arr[i] = i * i;

    for (i = 0; i < n; i++)
        printf("%d ", arr[i]);
    printf("\n");

    free(arr);
    return 0;
}
```

---

## Dynamic 2-D Array — Allocation

```c
int rows = 3, cols = 4;

/* Step 1: allocate array of
           row pointers */
int **grid = malloc(
    rows * sizeof(int *)
);
if (!grid) { /* handle */ }

/* Step 2: allocate each row */
int r;
for (r = 0; r < rows; r++) {
    grid[r] = malloc(
        cols * sizeof(int)
    );
    if (!grid[r]) { /* handle */ }
}
```

---

## Dynamic 2-D Array — Use & Free

```c
/* Use exactly like a 2-D array */
int r, c;
for (r = 0; r < rows; r++)
    for (c = 0; c < cols; c++)
        grid[r][c] = r * cols + c;

/* Free in reverse order:
   rows first, then the array */
for (r = 0; r < rows; r++)
    free(grid[r]);
free(grid);
grid = NULL;
```

**Free every row before freeing the pointer array.** Reversing the order leaks memory.

---

## Common Memory Errors

| Error | Description |
|-------|-------------|
| **Memory leak** | `malloc` without matching `free` |
| **Double free** | `free` called twice on same pointer |
| **Use after free** | Dereferencing after `free` |
| **Buffer overflow** | Writing past the end of allocation |
| **Uninitialized read** | Using `malloc`'d memory before writing |
| **Freeing stack memory** | `free` on a non-heap pointer |

---

## Memory Leak Example

```c
void leak(int n) {
    /* Allocated but never freed */
    int *p = malloc(
        n * sizeof(int)
    );

    /* Returning without free()
       — n ints are now leaked */
    return;
}

/* Every call to leak() loses
   memory until the OS reclaims
   it on process exit */
```

Leaks accumulate in long-running programs and can exhaust memory.

---

## Null the Pointer After Free

```c
int *p = malloc(sizeof(int));
*p = 42;
free(p);

/* p is now a DANGLING pointer */
/* This is undefined behavior: */
printf("%d\n", *p);

/* Set NULL to make bugs obvious */
p = NULL;

/* Now this crashes immediately
   rather than silently corrupting */
*p = 99;  /* segfault — good! */
```

A crash is better than silent data corruption.

---

## Valgrind — Finding Memory Errors

```bash
gcc -g -o myprog myprog.c
valgrind --leak-check=full \
         ./myprog
```

Valgrind reports:
- **Heap summary** — bytes allocated/freed
- **Leak summary** — what was never freed
- **Invalid reads/writes** — out-of-bounds access
- **Use of uninitialized values**

Compile with `-g` to get line numbers in reports.

---

## Lecture 1 Summary

| Concept | Key Point |
|---------|-----------|
| 2-D array | Array of arrays; row-major in memory |
| Column count | Must be fixed in function parameters |
| `int **` | Pointer to pointer; one extra level |
| `malloc` | Allocate heap memory; returns `void *` |
| `calloc` | Like `malloc` but zeroed |
| `realloc` | Resize; always use temp pointer |
| `free` | Release; null the pointer after |
| Valgrind | Tool for detecting memory errors |

**Next lecture:** Jagged arrays, `argv` as `char **`, and putting it all together

---

# Multi-Dimensional Arrays, Pointers to Pointers & Dynamic Memory
## COP 3223C — Lecture 2 of 2
### Jagged Arrays, `char **`, and Patterns

---

## Review: The Two 2-D Array Models

| | Fixed 2-D Array | Dynamic `int **` |
|-|-----------------|-----------------|
| **Syntax** | `int a[R][C]` | `int **a` |
| **Column count** | Compile-time constant | Varies per row |
| **Memory** | One contiguous block | Scattered allocations |
| **Passed to function** | `f(int a[][C], int r)` | `f(int **a, int r, int c)` |
| **Row lengths** | All equal | Can differ (jagged) |

---

## Jagged Arrays

When rows have different lengths:

```c
/* Row 0: 3 elements
   Row 1: 5 elements
   Row 2: 2 elements */
int *jagged[3];
jagged[0] = malloc(3 * sizeof(int));
jagged[1] = malloc(5 * sizeof(int));
jagged[2] = malloc(2 * sizeof(int));

/* Must track each row's length
   separately */
int lens[3] = {3, 5, 2};
```

---

## Jagged Array — Triangle Pattern

```c
/* Allocate a lower triangle:
   row i has (i+1) elements */
int rows = 5;
int **tri = malloc(
    rows * sizeof(int *)
);
int r;
for (r = 0; r < rows; r++) {
    /* Row r has r+1 columns */
    tri[r] = malloc(
        (r + 1) * sizeof(int)
    );
    int c;
    for (c = 0; c <= r; c++)
        tri[r][c] = r + c;
}
```

---

## Freeing a Jagged Array

```c
int r;
for (r = 0; r < rows; r++) {
    free(tri[r]);
    tri[r] = NULL;
}
free(tri);
tri = NULL;
```

The rule is the same: **free the leaves before the root**. Each row was allocated separately, so each must be freed separately.

---

## `char **` — Array of Strings

`char **` is how C represents an array of strings on the heap:

```c
/* Allocate space for 3 strings */
char **words = malloc(
    3 * sizeof(char *)
);

words[0] = malloc(6);
strcpy(words[0], "hello");

words[1] = malloc(6);
strcpy(words[1], "world");

words[2] = malloc(2);
strcpy(words[2], "!");
```

---

## `argv` Is a `char **`

```c
int main(int argc, char *argv[])
/* is identical to: */
int main(int argc, char **argv)
```

`argv` is a pointer to the first element of an array of `char *` strings. Now you understand exactly what it is:

```
argv ──► [ char * ]──► "program\0"
         [ char * ]──► "arg1\0"
         [ char * ]──► "arg2\0"
         [ NULL   ]
```

---

## Copying `argv` onto the Heap

```c
/* Make a heap copy of argv
   (useful if you need to
    sort or modify arguments) */
char **args = malloc(
    argc * sizeof(char *)
);
int i;
for (i = 0; i < argc; i++) {
    /* +1 for null terminator */
    int len = strlen(argv[i]) + 1;
    args[i] = malloc(len);
    strcpy(args[i], argv[i]);
}
```

Use `strdup` if available: `args[i] = strdup(argv[i]);`

---

## `strdup` — Duplicate a String

```c
#include <string.h>

/* Allocates exactly strlen(s)+1
   bytes and copies s into them */
char *strdup(const char *s);
```

```c
char *copy = strdup("hello");
/* Equivalent to:
   malloc(6) + strcpy */

if (!copy) { /* handle error */ }

/* Must free it when done */
free(copy);
```

`strdup` is in POSIX but not C99 standard — confirm availability.

---

## Passing `int **` to Functions

```c
/* rows and cols are needed
   since ** loses dimension info */
void fill(
    int **grid,
    int   rows,
    int   cols,
    int   val)
{
    int r, c;
    for (r = 0; r < rows; r++)
        for (c = 0; c < cols; c++)
            grid[r][c] = val;
}
```

Unlike fixed 2-D arrays, `int **` functions need explicit row AND column counts.

---

## Returning Heap Memory From Functions

```c
/* Caller is responsible for
   freeing the returned array */
int *make_range(int n) {
    int *a = malloc(
        n * sizeof(int)
    );
    if (!a) return NULL;

    int i;
    for (i = 0; i < n; i++)
        a[i] = i;
    return a;
}

int *r = make_range(5);
/* use r ... */
free(r);
```

Document ownership clearly — who must call `free`?

---

## Returning a 2-D Dynamic Array

```c
int **make_grid(
    int rows, int cols)
{
    int **g = malloc(
        rows * sizeof(int *)
    );
    if (!g) return NULL;

    int r;
    for (r = 0; r < rows; r++) {
        g[r] = calloc(
            cols, sizeof(int)
        );
        if (!g[r]) {
            /* Partial cleanup */
            while (--r >= 0)
                free(g[r]);
            free(g);
            return NULL;
        }
    }
    return g;
}
```

---

## Partial Cleanup on Error

When allocating in a loop, an error mid-way means cleaning up what succeeded:

```c
/* If row 2 of 5 fails: */
int r;
for (r = 0; r < rows; r++) {
    g[r] = malloc(
        cols * sizeof(int)
    );
    if (!g[r]) {
        /* Free rows 0..(r-1) */
        while (--r >= 0)
            free(g[r]);
        free(g);
        return NULL;
    }
}
```

Leaving partially-allocated arrays is a memory leak.

---

## `void *` — The Generic Pointer

`malloc` returns `void *`:

```c
/* void * can be assigned to
   any pointer type — no cast
   needed in C (unlike C++) */
int    *pi = malloc(sizeof(int));
double *pd = malloc(sizeof(double));
char   *pc = malloc(sizeof(char));

/* All valid — no explicit cast */
```

Do **not** cast `malloc` in C — it can hide the missing `#include <stdlib.h>` bug.

---

## Pointer Arithmetic on Heap Memory

```c
int *arr = malloc(
    5 * sizeof(int)
);

/* Pointer arithmetic works
   the same as with arrays */
int *p = arr;
*p = 10;  p++;   /* arr[0] = 10 */
*p = 20;  p++;   /* arr[1] = 20 */

/* arr still points to start */
printf("%d\n", arr[0]); /* 10 */

free(arr); /* NOT free(p)! */
```

Always keep the original pointer — you need it to call `free`.

---

## Sizing Checklist

```
Allocating n items of type T:

malloc(n * sizeof(T))
calloc(n, sizeof(T))
realloc(ptr, new_n * sizeof(T))

For a 2-D grid (rows × cols):

/* Row pointer array */
malloc(rows * sizeof(T *))

/* Each row */
malloc(cols * sizeof(T))
```

Getting `sizeof` arguments right is the most common source of allocation bugs.

---

## Reading Into Dynamically Allocated Strings

```c
/* Read n lines from fp;
   returns array of strings */
char **read_lines(FILE *fp, int n) {
    char **lines = malloc(
        n * sizeof(char *)
    );
    if (!lines) return NULL;

    char buf[1024];
    int i;
    for (i = 0; i < n; i++) {
        if (!fgets(buf,
                   sizeof(buf),
                   fp)) break;
        /* Strip newline */
        buf[strcspn(buf, "\n")] = 0;
        lines[i] = strdup(buf);
    }
    return lines;
}
```

---

## `strcspn` for Stripping Newlines

```c
/* strcspn returns the length of
   the initial segment of s1
   that has no chars from s2 */
char line[128];
fgets(line, sizeof(line), fp);

/* Find position of '\n' or '\0'
   and overwrite with '\0' */
line[strcspn(line, "\n")] = '\0';
```

Cleaner than a manual `strlen` + index check.

---

## Memory Layout: Stack vs Heap

```
High address
┌──────────────────┐
│   Stack          │ grows down ↓
│   local vars     │
│   function calls │
├──────────────────┤
│        ...       │
├──────────────────┤
│   Heap           │ grows up ↑
│   malloc/free    │
├──────────────────┤
│   BSS (zeroed)   │
│   static vars    │
├──────────────────┤
│   Data segment   │
│   global vars    │
├──────────────────┤
│   Text segment   │
│   code           │
└──────────────────┘
Low address
```

---

## Fixed 2-D vs Dynamic 2-D — Choosing

**Use fixed 2-D `int a[R][C]` when:**
- Dimensions known at compile time
- Array is small enough for the stack
- Passing to functions with fixed columns

**Use dynamic `int **` when:**
- Dimensions not known until runtime
- Array is large (avoid stack overflow)
- Rows may have different lengths
- Array must outlive the allocating function

---

## Lecture 2 Summary

| Concept | Key Point |
|---------|-----------|
| Jagged array | `int **`; rows allocated individually |
| `char **` | Array of strings; same as `argv` type |
| `strdup` | Allocates + copies a string |
| `void *` | Generic pointer; no cast needed in C |
| Returning heap | Caller must `free`; document ownership |
| Partial cleanup | On error, free what succeeded |
| Choosing model | Fixed if known; dynamic if runtime |

---

## Week Summary: The Memory Picture

```c
/* 1. Fixed 2-D — stack */
int grid[3][4];

/* 2. Dynamic 1-D — heap */
int *row = malloc(n * sizeof(int));

/* 3. Dynamic 2-D — heap */
int **g = malloc(r * sizeof(int*));
for (i=0; i<r; i++)
    g[i] = malloc(c * sizeof(int));

/* 4. Array of strings — heap */
char **strs = malloc(
    n * sizeof(char *)
);
for (i=0; i<n; i++)
    strs[i] = strdup(words[i]);
```

---

## Looking Ahead

**Next week:** Linked Lists
- Nodes allocated with `malloc`
- Connecting nodes with pointers
- Traversal, insertion, deletion
- Everything we learned about heap memory applies directly

The dungeon's inventory, enemy list, and room connections will all become linked lists.
