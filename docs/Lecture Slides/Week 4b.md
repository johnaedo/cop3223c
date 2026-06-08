---
share_cop3223c: "true"
site-folder: docs/Lecture Slides
theme: ucf-knights.css
height: "1080"
width: "1920"
---

# Week 4, Lecture 2: 
## Arrays & Strings

---

## Agenda

1. Arrays — declaration, indexing, iteration
2. Array-to-pointer decay
3. `sizeof` on arrays vs. pointers — the critical trap
4. Passing arrays to functions
5. Strings — `char` arrays and the null terminator
6. `string.h` — the essential functions
7. String safety

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 1

## Arrays

---

## Array Declaration and Initialization

```c
// uninitialized — do not read yet
int scores[5];
// fully initialized                          
int scores[5] = {95, 82, 78, 90, 65};  
// size inferred from initializer (5)
int scores[]  = {95, 82, 78, 90, 65};
// all elements set to 0  
int scores[5] = {0};                   
```

- Size must be a compile-time constant
- Array elements are **contiguous in memory** — no gaps between them
- Elements are accessed via **zero-based indexing**: `scores[0]` through `scores[4]`

---

## Indexing and Iteration

```c
int scores[5] = {95, 82, 78, 90, 65};

// Read individual elements
printf("First: %d\n", scores[0]);   // 95
printf("Last:  %d\n", scores[4]);   // 65

// Iterate with a for loop
int total = 0;
for (int i = 0; i < 5; i++) {
    total += scores[i];
}
printf("Total: %d\n", total);   // 410
```

---

## Out-of-Bounds Access — Undefined Behavior

```c
int scores[5] = {95, 82, 78, 90, 65};
// index 5 does not exist — undefined behavior
scores[5]  = 100;
// negative index — undefined behavior   
scores[-1] = 100;   
```

C performs **no bounds checking**. An out-of-bounds write silently corrupts whatever happens to live at that memory location — another variable, the return address of the current function, anything. This is the source of the most serious class of security vulnerabilities in systems software.

> [!IMPORTANT]
> Bounds checking is your responsibility. The compiler will not save you.

---

## Multi-Dimensional Arrays

```c
int grid[3][4];   // 3 rows, 4 columns

// Initialize
int matrix[2][3] = {
    {1, 2, 3},
    {4, 5, 6}
};

// Access
// 6 — row 1, column 2
printf("%d\n", matrix[1][2]);   

// Iterate
for (int r = 0; r < 2; r++) {
    for (int c = 0; c < 3; c++) {
        printf("%d ", matrix[r][c]);
    }
    printf("\n");
}
```

Multi-dimensional arrays are stored in **row-major order** — all of row 0, then all of row 1, etc.

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 2

## Array-to-Pointer Decay

---

## Arrays Are Not Pointers — But They Decay to Them

```c
int scores[5] = {10, 20, 30, 40, 50};
// no & needed — the array name decays to &scores[0]
int *p = scores;   
```

---

In most expressions, an array name is automatically converted to a **pointer to its first element**. This is called **decay**.

```c
scores        →   &scores[0]    (pointer to first element)
scores + 1    →   &scores[1]
scores + 2    →   &scores[2]
```

This is why array indexing and pointer arithmetic are interchangeable:

```c
// always true, by definition
scores[2]   ==   *(scores + 2)   
```

---

## Decay Is One-Way and Lossy

When an array decays to a pointer, **size information is lost**.

```c
int scores[5] = {10, 20, 30, 40, 50};
int *p = scores;   // p points to scores[0]

// p knows nothing about 5 — it is just an address
// You CANNOT recover the array size from p alone
```

This is a critical asymmetry. The array knows its size; the pointer does not.

---

## The `sizeof` Trap — The Most Important Slide This Week

```c
int scores[5] = {10, 20, 30, 40, 50};
int *p = scores;

// 20 — 5 elements × 4 bytes each ✓
printf("%zu\n", sizeof(scores));
// 8  — size of a pointer on 64-bit ✗  
printf("%zu\n", sizeof(p));       
```

`sizeof` on an **array** gives the total size in bytes.  
`sizeof` on a **pointer** gives the size of the pointer itself — always 4 or 8 bytes.

---

The standard idiom for element count:

```c
// 20 / 4 = 5 ✓
int len = sizeof(scores) / sizeof(scores[0]);   
```

> [!WARNING]
**This only works in the scope where the array was declared.** Once it decays to a pointer (e.g., in a function parameter), `sizeof` gives the pointer size — not the array size.

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 3

## Passing Arrays to Functions

---

## Arrays Decay at Function Calls

When you pass an array to a function, it **immediately decays** to a pointer to its first element. These three signatures are identical:

```c
// looks like array
void print_scores(int scores[], int len);
// pointer — same thing     
void print_scores(int *scores, int len);
// size hint — ignored by compiler      
void print_scores(int scores[5], int len);    
```

The `[]` in a parameter list is purely documentary. The compiler treats them all as `int *`. The function **cannot** determine the length from the parameter alone.

---

## Always Pass the Length Separately

```c
void print_scores(int arr[], int len) {
    for (int i = 0; i < len; i++) {
        printf("Score %d: %d\n", i + 1, arr[i]);
    }
}

int scores[5] = {95, 82, 78, 90, 65};
print_scores(scores, 5);

// Or use the sizeof idiom — 
// but only where the array is declared:
print_scores(scores, sizeof(scores) / sizeof(scores[0]));
```

> [!TIP]
Passing the length as a separate parameter is the standard C pattern. It is not optional — without it, the function has no safe way to know when to stop iterating.

---

## Functions Can Modify Array Elements

Because the pointer reaches the original array, modifications through the parameter affect the caller's data:

```c
void zero_out(int arr[], int len) {
    for (int i = 0; i < len; i++) {
        arr[i] = 0;   // modifies the CALLER's array
    }
}

int data[4] = {1, 2, 3, 4};
zero_out(data, 4);
// data is now {0, 0, 0, 0}
```

This is not pass-by-reference — it is a consequence of passing a pointer to the first element. The function and the caller share access to the same memory.

---

## Preventing Modification: `const`

```c
// This function promises not to modify the array
double average(const int arr[], int len) {
    int total = 0;
    for (int i = 0; i < len; i++) total += arr[i];
    return (double)total / len;
}
```

`const int arr[]` is equivalent to `const int *arr` — the function cannot write through the pointer. The compiler enforces this.

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 4

## Strings

---

## Strings in C: A `char` Array with a Sentinel

C has no built-in string type. A string is a **`char` array** with a **null terminator** (`'\0'`, value 0) marking the end.

```c
char greeting[6] = {'H', 'e', 'l', 'l', 'o', '\0'};
```

```
Index:  0    1    2    3    4    5
Value: 'H'  'e'  'l'  'l'  'o'  '\0'
```

String functions find the end by scanning for `'\0'` — there is no stored length.

---

## String Literals and Initialization

```c
// compiler adds '\0' automatically, size = 6
char greeting[] = "Hello";
// p points to a string literal (read-only!)   
char *p = "Hello";           
```

---

**Critical distinction:**

```c
// buf is a char array — writable, on the stack
char buf[] = "Hello";
// p points to a string literal — read-only!    
char *p    = "Hello";    

// fine — modifying the array copy
buf[0] = 'h';
// undefined behavior — string literals are immutable   
p[0]   = 'h';   
```
> [!TIP]
Always use `char buf[]` when you need a modifiable string.

---

## String Literals Are Read-Only

String literals like `"Hello"` are stored in the read-only data segment of your program. Modifying them through a `char *` pointer is undefined behavior — typically a segmentation fault.

```c
char *p = "Hello";
// UB — may crash, may corrupt, may silently work
p[0] = 'h';   

// correct — enforces read-only access
const char *p = "Hello";   
```
> [!TIP]
Declare pointers to string literals as `const char *` to let the compiler catch accidental modifications.

---

## String I/O

```c
char name[64];

// Read a single word (stops at whitespace)
// width limit prevents overflow
fscanf(stdin, "%63s", name);  

// Print a string
printf("Hello, %s!\n", name);

// Safer multi-word input (reads a whole line)
fgets(name, sizeof(name), stdin);
// fgets includes the trailing '\n' — strip it if needed:
name[strcspn(name, "\n")] = '\0';
```

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 5

## `string.h`

---

## Essential String Functions

```c
#include <string.h>
```

| Function | Signature | Does |
|----------|-----------|------|
| `strlen` | `size_t strlen(const char *s)` | Length excluding `'\0'` |
| `strcpy` | `char *strcpy(char *dst, const char *src)` | Copy src → dst |
| `strncpy` | `char *strncpy(char *dst, const char *src, size_t n)` | Copy at most n bytes |
| `strcat` | `char *strcat(char *dst, const char *src)` | Append src to dst |
| `strncat` | `char *strncat(char *dst, const char *src, size_t n)` | Append at most n bytes |
| `strcmp` | `int strcmp(const char *a, const char *b)` | Compare lexicographically |
| `strncmp` | `int strncmp(const char *a, const char *b, size_t n)` | Compare at most n bytes |
| `strchr` | `char *strchr(const char *s, int c)` | Find first occurrence of c |

---

## `strlen` — The Length of a String

```c
char name[] = "Alice";
size_t len = strlen(name);   // 5 — does NOT count '\0'

printf("%zu characters\n", len);
```

`strlen` scans from the start until it finds `'\0'` and counts bytes. It does **not** include the null terminator in the count.

> [!WARNING]
> If the string is missing its null terminator, `strlen` will scan past the end of the buffer — undefined behavior. Always ensure strings are properly terminated.

---

## `strcpy` and `strncpy`

```c
char src[] = "Hello";
char dst[10];

strcpy(dst, src);          // copies "Hello\0" into dst
printf("%s\n", dst);       // Hello

// strcpy is dangerous if dst is too small — 
//      no bounds checking
// Safer alternative:
strncpy(dst, src, sizeof(dst) - 1);
// strncpy may not null-terminate if src is too long
dst[sizeof(dst) - 1] = '\0';   
```

`strncpy` copies at most `n` characters but **does not guarantee null termination** if `src` is longer than `n`. Always manually null-terminate after `strncpy`.

---

## `strcmp` — String Comparison

```c
strcmp(a, b)
```

| Return value | Meaning                                |
| ------------ | -------------------------------------- |
| `< 0`        | `a` comes before `b` lexicographically |
| `0`          | `a` and `b` are identical              |
| `> 0`        | `a` comes after `b` lexicographically  |

---

```c
char *s1 = "apple";
char *s2 = "banana";

if (strcmp(s1, s2) == 0) {
    printf("same\n");
} else if (strcmp(s1, s2) < 0) {
    printf("apple comes first\n");   // prints this
}
```

> [!WARNING]
**Never compare strings with `==`** — that compares addresses, not content.

---

## `strcat` — String Concatenation

```c
char greeting[20] = "Hello, ";
char name[]       = "Alice";

strncat(greeting, name, 
	sizeof(greeting) - strlen(greeting) - 1);
printf("%s\n", greeting);   // Hello, Alice
```

`strcat` appends `src` to the end of `dst`. `dst` must be large enough to hold the combined result plus `'\0'`. `strncat` is safer — it limits how many characters are appended.

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 6

## String Safety

---

## The Buffer Overflow

```c
char buf[8];
// reads unlimited input — NEVER use gets()
gets(buf);
// no size check — 
//    src may be longer than 8 bytes          
strcpy(buf, src);   
```

If input is longer than the buffer, adjacent memory is overwritten. This corrupts data, crashes programs, and is the basis of a large class of security exploits.

---

**Functions to avoid:**

| Dangerous | Safe alternative |
|-----------|-----------------|
| `gets()` | `fgets()` |
| `strcpy()` | `strncpy()` + manual null termination |
| `strcat()` | `strncat()` |
| `sprintf()` | `snprintf()` |
| `scanf("%s")` | `scanf("%63s")` with width limit |

---

## The Safe String Pattern

```c
#define BUFSIZE 64

char name[BUFSIZE];

// Read — always width-limited
//    one less than BUFSIZE for '\0'
fscanf(stdin, "%63s", name);          

// Copy — always size-limited
strncpy(dest, name, sizeof(dest) - 1);
// guarantee null termination
dest[sizeof(dest) - 1] = '\0';       

// Concatenate — always room-checked
if (strlen(dest) + strlen(name) + 1 <= sizeof(dest)) {
    strncat(dest, name, sizeof(dest) - strlen(dest) - 1);
}
```

---

## Strings vs. Characters

```c
'A'    // a char   — single character, value 65
"A"    // a string — char array {'A', '\0'}, two bytes
```

```c
char c = 'A';    // one byte
char s[] = "A";  // two bytes: 'A' and '\0'
char *p = "A";   // pointer to a read-only two-byte array

printf("%c\n", c);     // A
printf("%s\n", s);     // A
printf("%d\n", c);     // 65 (ASCII value)
```
> [!WARNING]
Mixing them up is a common source of bugs. `'A'` is an integer; `"A"` is a pointer.

---

# Lecture 2 Wrap-Up

---

## Key Takeaways

- Arrays are zero-indexed, contiguous, and have **no bounds checking**
- An array name **decays** to a pointer to its first element in most expressions
- `sizeof(array)` gives total bytes; `sizeof(pointer)` gives pointer width — **not** the same
- Functions receive a pointer when passed an array — always pass **length separately**
- A C string is a **`char` array with a `'\0'` terminator** — no built-in string type
- `char buf[] = "text"` is writable; `char *p = "text"` points to read-only memory
- **Never compare strings with `==`** — use `strcmp`
- Always use **width-limited** variants: `strncpy`, `strncat`, `snprintf`, `%63s`

---

## Resources

- K&R Ch. 5 — Pointers and Arrays; Ch. 6 Appendix — String Functions  
- K.N. King Ch. 12 — Pointers and Arrays; Ch. 13 — Strings  
- `man 3 strcpy`, `man 3 strncpy`, `man 3 strcmp`  
- https://en.cppreference.com/w/c/string/byte
