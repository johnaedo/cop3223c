---
share_cop3223c: "true"
site-folder: docs/Lecture Slides
theme: css/dracula-custom.css
height: "1080"
width: "1920"
---
# Week 2: Getting Started in C
[View Presentation Online](http://teaching.johnaedo.com/lectures/cop3223c/Lesson%202%20-%20Getting%20Started%20in%20C/)

---

## Agenda

1. Structure of a C Program (review)
2. Variables & Data Types
3. Statements & Expressions
4. Console I/O: `printf` and `fscanf`
5. Conditionals
6. Loops

---

# Part 1

## Structure of a C Program

---

## The Minimal C Program

```c
#include <stdio.h>

int main(void) {
    return 0;
}
```

- `#include <stdio.h>` — preprocessor directive; pulls in the Standard I/O library
- `int main(void)` — program entry point; OS calls this function
- `return 0;` — signals successful execution to the OS

---

## Anatomy: Preprocessor → Compiler → Linker

```
Source (.c)
    │
    ▼  preprocessor (cpp)
Expanded Source
    │
    ▼  compiler (gcc)
Object File (.o)
    │
    ▼  linker (ld)
Executable (a.out)
```

The compiler only sees a single translation unit at a time.  
The linker stitches multiple `.o` files and libraries together.

---

## Your First Compile

```bash
gcc -Wall -Wextra -std=c17 -o hello hello.c
./hello
```

| Flag       | Meaning                |
| ---------- | ---------------------- |
| `-Wall`    | Enable common warnings |
| `-Wextra`  | Enable extra warnings  |
| `-std=c17` | Target C17 standard    |
| `-o hello` | Name the output binary |

> **Treat warnings as errors during development.** Use `-Werror` for zero tolerance.

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 2

## Variables & Data Types

---
## Declaring Variables
- In C, you must tell the compiler about the variables you are going to use *and their type* before you use them.
- Contrast with Python, which is dynamically typed:
```python
# This is valid in Python, but not in C!
myVariable = 3
myVariable = "Hello"
```
```c
int myVariable1 = 3;
char *myVariable2 = "Hello";
```

---

## Fundamental Types

| Type     | Typical Size | Range (signed)                  | Format Specifier |
| -------- | ------------ | ------------------------------- | ---------------- |
| `char`   | 1 byte       | −128 to 127                     | `%c` / `%d`      |
| `int`    | 4 bytes      | −2,147,483,648 to 2,147,483,647 | `%d`             |
| `float`  | 4 bytes      | ~6–7 decimal digits             | `%f`             |
| `double` | 8 bytes      | ~15–16 decimal digits           | `%lf`            |

> Sizes are **implementation-defined**. Use `sizeof` to be sure.

---

## `sizeof` Operator

```c
#include <stdio.h>

int main(void) {
    printf("char:   %zu bytes\n", sizeof(char));
    printf("int:    %zu bytes\n", sizeof(int));
    printf("float:  %zu bytes\n", sizeof(float));
    printf("double: %zu bytes\n", sizeof(double));
    return 0;
}
```

`sizeof` returns a value of type `size_t` — use `%zu` to print it.

---

## `unsigned` and Type Modifiers

```c
unsigned int score = 0;   // 0 to 4,294,967,295
short int   s     = 10;   // typically 2 bytes
long int    big   = 1000000L;
long long   huge  = 9000000000LL;
```

- Dropping the sign bit **doubles** the positive range
- Use `unsigned` for things that can't be negative (sizes, counts)

---

## Integer Division — A Classic Trap

```c
int a = 7, b = 2;
int result = a / b;      // result == 3, NOT 3.5!
```

When **both operands** are integers, `/` performs **integer (truncating) division**.

```c
double result = (double)a / b;   // 3.5 ✓
double wrong  = (double)(a / b); // 3.0 ✗ — cast happens too late
```

---

## Type Mixing & Implicit Conversion

C promotes types automatically in **mixed expressions**:

```
char → int → long → float → double
```

```c
int   i = 5;
float f = 2.0f;
float r = i + f;   // i promoted to float before addition
```

> **Rule of thumb:** the "wider" type wins in arithmetic.

---

## Explicit Type Casting

```c
int   dollars = 7;
int   cents   = 2;
float price   = (float)dollars + cents / 100.0f;
```

Casting syntax: `(type) expression`

Use casts to:
- Force floating-point division from integer operands
- Narrow a wider type (be careful — data loss possible)
- Pass the right type to functions expecting a specific type

---

## Variable Declaration & Initialization

```c
int   health  = 100;       // initialize at declaration — always!
float speed   = 3.14f;     // f suffix → float literal (not double)
char  grade   = 'A';       // single quotes for char literals
double pi     = 3.14159265358979;

int x;                     // uninitialized — undefined behavior to read!
```

> **Best practice:** initialize every variable at the point of declaration.

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 3

## Statements & Expressions

---

## Expressions vs. Statements

| Concept | Definition | Example |
|---------|-----------|---------|
| **Expression** | Produces a value | `3 + 4`, `x * y`, `a > b` |
| **Statement** | Performs an action | `x = 5;`, `return 0;` |
| **Expression statement** | Expression + semicolon | `printf("hi\n");` |

Every expression has a **type** and a **value**.

---

## Arithmetic Operators

| Operator | Operation | Example | Result |
|----------|-----------|---------|--------|
| `+` | Addition | `3 + 4` | `7` |
| `-` | Subtraction | `10 - 3` | `7` |
| `*` | Multiplication | `3 * 4` | `12` |
| `/` | Division | `7 / 2` | `3` |
| `%` | Modulo (remainder) | `7 % 2` | `1` |

---

## Order of Operations (Precedence)

C follows standard mathematical precedence:

```
Highest:  ()         — grouping / function call
          * / %      — multiplicative
          + -        — additive
Lowest:   =          — assignment (right-to-left)
```

```c
int x = 2 + 3 * 4;      // 14, not 20
int y = (2 + 3) * 4;    // 20 — parentheses override
```

> When in doubt, **add parentheses** for clarity.

---

## Compound Assignment & Increment

```c
x += 5;   // x = x + 5
x -= 2;   // x = x - 2
x *= 3;   // x = x * 3
x /= 4;   // x = x / 4
x %= 7;   // x = x % 7

x++;      // post-increment: use x, then add 1
++x;      // pre-increment:  add 1, then use x
x--;      // post-decrement
```

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 4

## Console Input & Output

---

## `printf` — Formatted Output

```c
#include <stdio.h>

int main(void) {
    int    level  = 5;
    double health = 87.5;
    char   grade  = 'B';

    printf("Level: %d\n",         level);
    printf("Health: %.1f%%\n",    health);
    printf("Grade: %c\n",         grade);
    printf("Hex: 0x%X\n",         255);
    return 0;
}
```

---

## Format Specifier Reference

| Specifier | Type | Notes |
|-----------|------|-------|
| `%d` | `int` | Signed decimal integer |
| `%u` | `unsigned int` | Unsigned decimal |
| `%f` | `float` / `double` | Default 6 decimal places |
| `%.2f` | `float` / `double` | Exactly 2 decimal places |
| `%e` | `double` | Scientific notation |
| `%c` | `char` | Single character |
| `%s` | `char *` | String (null-terminated) |
| `%zu` | `size_t` | Result of `sizeof` |
| `%%` | — | Literal `%` character |
| `%p` | pointer | Memory address |

---

## `scanf` — The Problem

```c
scanf("%d", &age);     // Works — but has issues:
```

1. Leaves `\n` in the input buffer
2. Next `scanf` call for a `char` or string reads that leftover newline
3. Leads to confusing, hard-to-debug input behavior
4. Prone to bugs like buffer overflow!

---

## `fscanf` with Newline Stripping

The fix: consume the newline explicitly.

```c
#include <stdio.h>

int main(void) {
    int age;
    char initial;

    fscanf(stdin, "%d ", &age);       // trailing space eats whitespace/newline
    fscanf(stdin, "%c",  &initial);   // now reads the real character

    printf("Age: %d, Initial: %c\n", age, initial);
    return 0;
}
```

> The **space before `%c`** in a format string skips all leading whitespace including `\n`.

---

## Reading Strings Safely

```c
char name[64];
fscanf(stdin, "%63s", name);   // width limiter prevents buffer overflow
```

- `%63s` reads at most 63 characters (leaves room for `\0`)
- Never use `%s` without a width limit on untrusted input

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 5

## Conditionals

---

## What is "True" in C?

C has **no dedicated boolean type** in C89/90.  
(C99 introduced `_Bool` / `<stdbool.h>`)

| Value         | Boolean meaning |
| ------------- | --------------- |
| `0`           | **False**       |
| Anything else | **True**        |

```c
int x = 42;
if (x)          printf("true!\n");   // prints — 42 ≠ 0
if (0)          printf("never\n");   // never prints
if (-1)         printf("true!\n");   // prints — nonzero
```

---

## The `if` Statement

```c
if (condition) {
    // executes when condition is true (non-zero)
}
```

```c
int score = 75;

if (score >= 60) {
    printf("You passed!\n");
}
```

---

## Comparison & Logical Operators

| Operator  | Meaning                      |
| --------- | ---------------------------- |
| `==`      | Equal to                     |
| `!=`      | Not equal to                 |
| `<` `>`   | Less / greater than          |
| `<=` `>=` | Less / greater than or equal |
| `&&`      | Logical AND                  |
| `\|\|`    | Logical OR                   |
| `!`       | Logical NOT                  |

> **Common bug:** `=` (assignment) vs `==` (comparison)

---

## `if` / `else` / `else if`

```c
int score = 82;

if (score >= 90) {
    printf("A\n");
} else if (score >= 80) {
    printf("B\n");
} else if (score >= 70) {
    printf("C\n");
} else if (score >= 60) {
    printf("D\n");
} else {
    printf("F\n");
}
```

Conditions are evaluated **top to bottom**; first match wins.

---

## `switch` / `case`

```c
char grade = 'B';

switch (grade) {
    case 'A':
        printf("Excellent!\n");
        break;
    case 'B':
        printf("Good job!\n");
        break;
    case 'C':
        printf("Passing.\n");
        break;
    default:
        printf("See advisor.\n");
        break;
}
```

- Works on **integer types** and `char` only
- `break` is **required** to prevent fall-through
- `default` handles unmatched values

---

## Fall-Through: Intentional vs. Accidental

```c
// Intentional fall-through (document it!)
// 0: Sunday, 7: Saturday
switch (day) {
    case 6:   /* FALL THROUGH */
    case 0:
        printf("Weekend!\n");
        break;
    default:
        printf("Weekday\n");
        break;
}
```

Forgetting `break` is one of the most common C bugs.

---

<!-- .slide: data-background="#1a1a1a" -->
# Part 6

## Loops

---

## The `for` Loop

```c
for (initialization; condition; update) {
    // body
}
```

```c
for (int i = 0; i < 5; i++) {
    printf("i = %d\n", i);
}
```

Best for: loops with a **known iteration count**.

---

## `for` Loop Mechanics

```
┌─────────────────────────────────────┐
│  int i = 0        ← runs ONCE       │
│      │                              │
│      ▼                              │
│  i < 5 ? ──No──→ exit loop          │
│      │ Yes                          │
│      ▼                              │
│  execute body                       │
│      │                              │
│      ▼                              │
│  i++              ← runs each iter  │
│      │                              │
│      └──────────────────────────────┘
```

---

## The `while` Loop

```c
while (condition) {
    // body
}
```

```c
int n = 1;
while (n <= 100) {
    printf("%d\n", n);
    n *= 2;
}
```

Best for: loops where the **exit condition is dynamic**.  
**May execute zero times** if condition is false initially.

---

## The `do`…`while` Loop

```c
do {
    // body
} while (condition);
```

```c
int choice;
do {
    printf("Enter 1–4: ");
    fscanf(stdin, "%d", &choice);
} while (choice < 1 || choice > 4);
```

**Guaranteed to execute at least once.**  
Perfect for input validation menus.

---

## `break` and `continue`

```c
for (int i = 0; i < 10; i++) {
    if (i == 3) continue;   // skip iteration when i == 3
    if (i == 7) break;      // exit loop entirely when i == 7
    printf("%d ", i);
}
// Output: 0 1 2 4 5 6
```

| Keyword    | Effect                                           |
| ---------- | ------------------------------------------------ |
| `break`    | Immediately exits the innermost loop (or switch) |
| `continue` | Skips the rest of the current iteration          |

---

## Nested Loops

```c
for (int row = 0; row < 3; row++) {
    for (int col = 0; col < 3; col++) {
        printf("[%d,%d] ", row, col);
    }
    printf("\n");
}
```

`break`/`continue` affect **only the innermost loop**.

---

<!-- .slide: data-background="#1a1a1a" -->

# Week 2 Wrap-Up

---

## Key Takeaways

- Every C program starts at `main()`; every statement ends with `;`
- Know your types — **integer division truncates**
- Use `(double)` casts before division when you need precision
- `printf` format specifiers must match the argument type
- Use `fscanf(stdin, "%d ", &x)` with trailing space to strip newlines
- In C, **zero is false**, everything else is true
- Match the loop form to the use case: `for` / `while` / `do-while`

---

## Looking Ahead — Week 2 Lab

You will practice today's concepts hands-on:

- Type sizing exploration with `sizeof`
- Integer vs. float division experiments
- Building a formatted output table with `printf`
- Writing an input-validated menu loop
- Grade calculator using conditionals

See you in lab!

---

## Resources

- *The C Programming Language* — Kernighan & Ritchie (K&R), 2nd Ed.
- *C Programming: A Modern Approach* — K.N. King
- `man 3 printf` — POSIX printf reference
- https://en.cppreference.com/w/c — C language reference
