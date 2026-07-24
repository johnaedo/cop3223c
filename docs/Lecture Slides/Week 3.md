---
share_cop3223c: "true"
site-folder: docs/Lecture Slides
theme: dracula-custom.css
height: "1080"
width: "1920"
---

# Week 3: Functions & Header Files

---

## Agenda

1. Why functions exist
2. Anatomy of a function
3. Prototypes — what they are and why C needs them
4. Pass-by-value semantics
5. Scope: local variables and why globals are dangerous
6. Header files — the `.h` / `.c` split
7. Include guards
8. Compiling multi-file programs

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 1

## Why Functions Exist

---

## The Problem: Code Without Functions

```c
// Computing damage for three different enemies...
int dmg1 = hero_attack - enemy1_defense;
if (dmg1 < 1) dmg1 = 1;

int dmg2 = hero_attack - enemy2_defense;
if (dmg2 < 1) dmg2 = 1;

int dmg3 = hero_attack - enemy3_defense;
if (dmg3 < 1) dmg3 = 1;
```

What happens when the damage formula changes?  
You find and update every copy. And miss one.

---

## Functions Solve Three Problems

| Problem | Solution |
|---------|----------|
| **Duplication** | Write logic once, call it anywhere |
| **Complexity** | Break a large problem into named, understandable pieces |
| **Testability** | Verify one small unit of behavior in isolation |

A function is a **named, reusable unit of computation**.

---

## The Same Logic, Done Right

```c
int calculate_damage(int attack, int defense) {
    int damage = attack - defense;
    if (damage < 1) damage = 1;
    return damage;
}

int dmg1 = calculate_damage(hero_attack, enemy1_defense);
int dmg2 = calculate_damage(hero_attack, enemy2_defense);
int dmg3 = calculate_damage(hero_attack, enemy3_defense);
```

Change the formula in one place. Every call site benefits automatically.

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 2

## Anatomy of a Function

---

## The Four Parts

```c
int calculate_damage(int attack, int defense) {
    int damage = attack - defense;
    if (damage < 1) damage = 1;
    return damage;
}
```

```
┌──────────────────────────────────────────────┐
│  int              ← return type              │
│  calculate_damage ← name                     │
│  (int attack, int defense) ← parameter list  │
│  { ... }          ← body                     │
└──────────────────────────────────────────────┘
```

---

## Return Types

```c
int    add(int a, int b)     { return a + b;          }
double average(int a, int b) { return (a + b) / 2.0;  }
void   print_banner(void)    { printf("===\n");        }
```

- The return type tells the caller what value to expect back
- `void` means the function produces no value
- A `void` function **can** have a bare `return;` to exit early
- A non-`void` function **must** return a value on every code path

---

## Parameters vs. Arguments

```c
// attack and defense are PARAMETERS — placeholders in the definition
int calculate_damage(int attack, int defense) {
    return attack - defense;
}

// 15 and 3 are ARGUMENTS — actual values supplied at the call site
int dmg = calculate_damage(15, 3);
```

**Parameters** live in the function definition.  
**Arguments** live at the call site.

> The terms are used interchangeably in casual conversation — that's fine. The distinction matters most when reading compiler error messages.

---

## `(void)` vs. `()`

```c
   // correct C: no parameters
void print_banner(void) { ... }
   // legal but means "unspecified" — avoid!
void print_banner()     { ... }
```

In C (unlike C++), an empty `()` does **not** mean "takes no arguments" — it means "parameters unspecified." Always write `(void)` for functions that take nothing.

---

## Early Return

```c
void print_positive(int x) {
	// exit immediately — nothing to print
    if (x <= 0) return;
    printf("%d\n", x);
}
```

```c
int first_negative(int a, int b, int c) {
    if (a < 0) return a;
    if (b < 0) return b;
    if (c < 0) return c;
    return 0;               // none found
}
```

Early returns keep functions flat. Prefer them over deeply nested `if-else` chains.

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 3

## Prototypes

---

## The Single-Pass Compiler

C's compiler reads your source file **top to bottom, once**.

```c
int main(void) {
  // ERROR: what is calculate_damage?
    int dmg = calculate_damage(15, 3);
    return 0;
}
  // seen too late
int calculate_damage(int attack, int defense) {
    return attack - defense;
}
```

By the time the compiler reaches `calculate_damage`'s definition, it has already failed on the call in `main`.

---

## Solution 1: Define Before Use

```c
// defined first
int calculate_damage(int attack, int defense) { 
    return attack - defense;
}
// compiler already knows calculate_damage
int main(void) {
    int dmg = calculate_damage(15, 3);
    return 0;
}
```

Works — but only in simple single-file programs. Collapses when functions call each other mutually, or when code lives in multiple files.

---

## Solution 2: The Prototype

```c
// Prototype: signature only, no body, ends with semicolon
int calculate_damage(int attack, int defense);

int main(void) {
    // compiler: OK, I know this signature
    int dmg = calculate_damage(15, 3);  
    return 0;
}
// full definition comes later
int calculate_damage(int attack, int defense) {  
    return attack - defense;
}
```

The prototype is a **promise to the compiler**: "this function exists, here is its type signature, trust me."

---

## What a Prototype Tells the Compiler

```c
int calculate_damage(int attack, int defense);
```

1. The function returns an `int`
2. It takes exactly two parameters
3. Both parameters are `int`
4. The full definition will appear later

Parameter **names** are optional in prototypes but strongly recommended:

```c
int calculate_damage(int, int);                // legal but cryptic
int calculate_damage(int attack, int defense); // preferred
```

---

## Prototypes Catch Mistakes

```c
  // declared as double
double calculate_damage(int attack, int defense);

int main(void) {
  // compiler warns: narrowing double to int
    int dmg = calculate_damage(15, 3);
    return 0;
}
```

Without the prototype, this call compiles silently and produces garbage results. With it, the compiler catches the mismatch at every call site.

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 4

## Pass-by-Value

---

## C Always Passes by Value

Arguments are **copied** into the parameters. The function works on the copy.

```c
void double_it(int x) {
    x = x * 2;
    printf("inside:  %d\n", x);
}

int main(void) {
    int n = 5;
    double_it(n);
    printf("outside: %d\n", n);   // still 5
    return 0;
}
```

```
inside:  10
outside: 5
```

---
## Function Call State (The Stack)
- Each function, including `main()`, keeps track of its own local variables
- This state is tracked in a structure called a `stack`, which you'll learn more about in CDA 3103 and COP 3402
- As functions are called and exited from, `stack frames` (or elements on the stack) are pushed (added) and popped (removed)
- This idea will be elaborated on further when we talk about dynamic memory allocation and introduce more process structures that are important to the lifecycle of a running program.

---

## Visualizing Pass-by-Value

```
main's stack frame          double_it's stack frame
┌──────────────┐            ┌──────────────┐
│   n  =  5    │ ─copy──▶   │   x  =  5    │
└──────────────┘            │   x  = 10    │  ← local modification
                            └──────────────┘
                            (destroyed on return)
```

`x` is a **separate variable** initialized to the value of `n`. Modifying `x` leaves `n` untouched.

---

## The Broken Swap

```c
void swap(int a, int b) {
    int temp = a;
    a = b;
    b = temp;   // swaps the copies — caller unchanged
}

int main(void) {
    int x = 1, y = 2;
    swap(x, y);
    printf("%d %d\n", x, y);   // still 1 2
    return 0;
}
```

This is not a bug to fix today — it's a preview of why we need pointers next week.

---

## Pass-by-Value Is a Feature

Because functions can't accidentally modify the caller's variables, you can:

- Read a function in isolation and reason about it completely
- Call a function without worrying it will corrupt your data
- Test a function with known inputs and predict its output exactly

> **Coming Soon:** pointers let you pass an *address* instead of a value, enabling intentional modification. The key word is **intentional** — it's explicit in the code.

---

## Arrays: The One Exception

```c
void zero_first(int arr[]) {
   // this DOES affect the caller's array
    arr[0] = 0;
}
```

Arrays behave differently — we'll explain exactly why with pointers later. For now: **scalars (ints, floats, chars) are copied; arrays are not.**

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 5

## Scope

---

## What Scope Means

**Scope** is the region of code where a name is visible and usable.

```c
int calculate_damage(int attack, int defense) {
    int damage = attack - defense;
    return damage;
}

int main(void) {
  // ERROR: 'damage' undeclared here
    printf("%d\n", damage);
    return 0;
}
```

`damage` only exists inside `calculate_damage`. It's created when the function is called and destroyed when it returns.

---

## Block Scope

```c
void demonstrate(void) {
    int x = 10;

    if (x > 5) {
	    // y exists only inside this if block
        int y = 20;             
        printf("%d %d\n", x, y);
    }

    printf("%d\n", x);          // fine — x still in scope
    // printf("%d\n", y);       // ERROR: y is gone
}
```

A variable's scope starts at its declaration and ends at the `}` that closes the block containing it.

---

## Why Local Scope Is Good

- A bug in one function **cannot corrupt another function's variables**
- You can reuse names like `i`, `temp`, `result` freely in every function
- To understand a function, you only need to read that function
- The compiler reclaims memory the moment a variable goes out of scope

This is the primary reason functions make programs easier to debug.

---

## Global Variables

```c
// global — visible to every function in this file
int score = 0;     

void add_points(int pts) { score += pts; }
void reset(void)         { score = 0;    }

int main(void) {
    add_points(10);
    add_points(5);
    printf("Score: %d\n", score);   // 15
    return 0;
}
```

This works. But it is almost always the wrong design.

---

## Why Globals Cause Problems

```c
int player_hp = 100;   // global

void apply_poison(void) { player_hp -= 5;            }
void heal(int amount)   { player_hp += amount;        }
  // silently truncates!
void level_up(void)     { player_hp = player_hp * 1.1; }
```

- **Any** function anywhere can modify `player_hp` — intentionally or by mistake
- When `player_hp` is wrong, every function in the program is a suspect
- Adding a second player breaks everything — a global can only hold one value

**Rule:** pass data as parameters, return results as return values.  
Reach for globals only when you can articulate a specific reason.

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 6

## Header Files

---

## The Multi-File Problem

```
main.c                       combat.c
─────────────────────        ──────────────────────────────
#include <stdio.h>           int calculate_damage(int a,
                                                  int d) {
int main(void) {                 int dmg = a - d;
    int d = calculate_           if (dmg < 1) dmg = 1;
        damage(15, 3);           return dmg;
    ...                      }
}
```

`main.c` calls `calculate_damage` but has never seen its prototype. The compiler generates a warning or error; even if it doesn't, the linker may produce wrong results.

---

## What `#include` Actually Does

`#include` is processed by the **preprocessor**, before compilation begins.

```c
#include <stdio.h>
```

The preprocessor **copies and pastes** the entire contents of `stdio.h` into your file at that exact line. The compiler never sees the `#include` directive — only the expanded result.

```
Source ──▶ preprocessor (text expansion) ──▶ expanded source ──▶ compiler
```

---

## The Solution: A Header File

Put the prototype in `combat.h`, then include it everywhere it's needed.

**`combat.h`**
```c
#ifndef COMBAT_H
#define COMBAT_H

int calculate_damage(int attack, int defense);

#endif /* COMBAT_H */
```

**`combat.c`**
```c
// include our own header — compiler checks our signature
#include "combat.h"    

int calculate_damage(int attack, int defense) {
    int damage = attack - defense;
    if (damage < 1) damage = 1;
    return damage;
}
```
---

**`main.c`**
```c
#include <stdio.h>
#include "combat.h"    // paste in the prototype

int main(void) {
	// compiler: signature known ✓
    int dmg = calculate_damage(15, 3);   
    printf("%d\n", dmg);
    return 0;
}
```

---

## The `.h` / `.c` Contract

| File | Contains | Think of it as |
|------|----------|---------------|
| `combat.h` | Prototypes, type definitions | The **menu** — what's available |
| `combat.c` | Function bodies | The **kitchen** — how it's made |
| `main.c` | `#include "combat.h"` + calls | The **customer** — uses the menu |

The `.h` file is the **public interface**.  
The `.c` file is the **private implementation**.

---

## What Belongs in a Header

| Put in `.h` | Keep in `.c` |
|-------------|-------------|
| Function prototypes | Function definitions (bodies) |
| `typedef` and `struct` definitions | Local variables |
| `#define` constants used by callers | Implementation helpers |
| `#include`s the header itself needs | `#include`s only the `.c` needs |

**Never** put a function body in a header. If two `.c` files include it, the linker sees the same function defined twice and errors out.

---

## Angle Brackets vs. Quotes

```c
// search the system include directories
#include <stdio.h> 
// search the current directory first, then system    
#include "combat.h"    
```

`< >` — standard library and installed third-party headers  
`" "` — your own project's headers

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 7

## Include Guards

---

## The Double-Include Problem

```
main.c
 ├── #include "combat.h"    ← combat.h processed here
 └── #include "game.h"
       └── #include "combat.h"  ← duplicate prototypes!
```

Duplicate declarations are a compiler error. And as projects grow, this happens constantly.

---

## Include Guards

```c
#ifndef COMBAT_H      // if this macro is not yet defined
#define COMBAT_H      // ...define it now

int calculate_damage(int attack, int defense);
void print_combat_result(const char *attacker,int damage);

#endif /* COMBAT_H */
```

**First include:** `COMBAT_H` not defined → content is processed → `COMBAT_H` defined.  
**Every subsequent include:** `COMBAT_H` already defined → entire block skipped.

---

## Guard Naming Convention

Use `FILENAME_H` in all caps, replacing dots and path separators with underscores.

| File | Guard macro |
|------|-------------|
| `combat.h` | `COMBAT_H` |
| `player_stats.h` | `PLAYER_STATS_H` |
| `utils/math.h` | `UTILS_MATH_H` |

A typo in the guard name (e.g., `COMBA_TH`) breaks it silently — double-check.

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 8

## Compiling Multi-File Programs

---

## Separate Compilation

Each `.c` file compiles independently to an **object file**:

```bash
gcc -Wall -Wextra -std=c17 -c combat.c    # → combat.o
gcc -Wall -Wextra -std=c17 -c main.c      # → main.o
```

The **linker** combines object files into an executable:

```bash
gcc combat.o main.o -o game
```

Or compile and link in one step:

```bash
gcc -Wall -Wextra -std=c17 -o game main.c combat.c
```

---

## Why Separate Compilation Matters

```
Edit one function in combat.c
          │
          ▼
Recompile only combat.c   (fast — one file)
          │
          ▼
Re-link everything         (fast — just symbol resolution)
```

In large projects with hundreds of source files, recompiling only what changed is the difference between a 2-second and a 20-minute build. This is the foundation of `make` and every modern build system.

---

## The Linker's Job

The compiler resolves names **within** one translation unit.  
The linker resolves names **across** translation units.

```
combat.o  defines:    calculate_damage ✓
main.o    references: calculate_damage (unresolved)
          │
          ▼  linker
game      all references resolved ✓
```

If a function is prototyped but never defined, compilation succeeds but linking fails: `undefined reference to 'calculate_damage'`.

---

## Common Multi-File Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `implicit declaration of function 'foo'` | Called with no prototype in scope | Add `#include "foo.h"` |
| `undefined reference to 'foo'` | Prototype present, definition missing from link | Add the `.c` file to the compile command |
| `multiple definition of 'foo'` | Function body in a header, included by two `.c` files | Move body to `.c`, keep only prototype in `.h` |

---

# Week 3 Wrap-Up

---

## Key Takeaways

- Functions exist for **abstraction, reuse, and testability**
- The **prototype** is a promise to the compiler; the **definition** keeps it
- C is **pass-by-value** — functions work on copies of scalars and cannot modify the caller's variables (arrays are the exception — more next week)
- **Local scope** keeps functions self-contained; **globals** undermine this
- A **header file** is a collection of prototypes pasted in by `#include`
- **Include guards** prevent duplicate declarations when a header is included multiple times
- Each `.c` file compiles to an **object file**; the **linker** combines them

---

## Looking Ahead

- **Pointers:** addresses, `*`, `&`, pointer arithmetic
- **Why pass-by-value isn't the whole story:** modifying caller variables, returning multiple values
- **Strings:** `char` arrays, the null terminator, `string.h`
- **RPG:** hero and enemy passed by pointer into combat functions; string-based item names

---

## Resources

- K&R Ch. 4 — Functions and Program Structure
- https://en.cppreference.com/w/c/language/functions
- https://en.wikibooks.org/wiki/C_Programming/Procedures_and_functions
