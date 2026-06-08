---
share_cop3223c: "true"
site-folder: docs/Lecture Slides
theme: ucf-knights.css
height: "1080"
width: "1920"
---

# Week 5, Lecture 1: `typedef`, Enums & Structs

---

## Agenda

1. `typedef` — creating type aliases
2. Enums — replacing magic numbers with named constants
3. `typedef enum` — the idiomatic combined form
4. Struct declaration and member access
5. Struct initialization
6. Structs are value types — the copy implication

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 1

## `typedef`

---

## What `typedef` Does

`typedef` creates a new **name** (alias) for an existing type. It does not create a new type — it creates a new way to refer to one.

```c
typedef unsigned long long uint64;
typedef int                PlayerID;
typedef char               Byte;
```

After these declarations:

```c
uint64   big_number = 1234567890123ULL;
PlayerID player     = 42;
Byte     flags      = 0xFF;
```

The aliases and the underlying types are completely interchangeable. The compiler sees no difference.

---

## Why `typedef` Exists

**Readability:** `PlayerID` communicates intent; `int` does not.

**Portability:** Platform-specific types can be hidden behind a single alias.

```c
// On one platform int is 16 bits; on another it's 32.
// Hide the platform detail behind a typedef:
typedef int32_t Score;   // from <stdint.h> — guaranteed 32-bit signed
```

**Simplifying complex types:** Function pointer types and struct types are the canonical use cases — both are dramatically more readable with `typedef`.

---

## `typedef` Syntax

```c
typedef  existing_type  new_name;
//       ─────────────  ────────
//       what it is     what to call it
```

```c
typedef double  Meters;
typedef int     ErrorCode;
// careful — pointer typedefs hide the *
typedef char *  String;    
```

> Avoid `typedef` for pointer types (`char *`, `int *`). Hiding the `*` inside a typedef makes it easy to forget you are dealing with a pointer — a common source of subtle bugs.

---

## `typedef` with Structs and Enums

The two most valuable uses of `typedef` in C are with structs and enums. Both require writing the keyword (`struct`, `enum`) every time you use the type — unless you `typedef` it away.

```c
struct Point { int x; int y; };
// must write "struct" every time without typedef
struct Point p1;   
```

```c
typedef struct { int x; int y; } Point;
// clean — no "struct" keyword needed
Point p1;          
```

We will use this combined form throughout the rest of the course.

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 2

## Enums

---

## The Problem: Magic Numbers

```c
int enemy_type = 2;   // what is 2?

if (enemy_type == 2) {
    // fight a quintesson? an seeker? who knows
}
```

Raw integers for categorical values are opaque. They tell you nothing about meaning, and any integer can be assigned — the compiler cannot help you catch mistakes.

---

## Enums: Named Integer Constants

```c
enum EnemyType {
    ENEMY_DECEPTICON,    // 0
    ENEMY_SEEKER,       // 1
    ENEMY_QUINTESSON,    // 2
    ENEMY_BOSS       // 3
};
```

By default, enum values start at 0 and increment by 1. You can assign explicit values:

```c
enum HttpStatus {
    HTTP_OK        = 200,
    HTTP_NOT_FOUND = 404,
    HTTP_ERROR     = 500
};
```

---

## Using an Enum

```c
enum EnemyType type = ENEMY_SEEKER;

if (type == ENEMY_SEEKER) {
    printf("A seeker!\n");
}

switch (type) {
    case ENEMY_DECEPTICON: printf("Decepticon\n"); break;
    case ENEMY_SEEKER:    printf("Seeker\n");    break;
    case ENEMY_QUINTESSON: printf("Quintesson\n"); break;
    default:           printf("Unknown\n"); break;
}
```

The `switch` over an enum is one of the most readable patterns in C. Many compilers will warn if you omit a case — a free correctness check.

---

## `typedef enum` — The Idiomatic Form

Writing `enum EnemyType` everywhere is verbose. The standard idiom combines the declaration with `typedef`:

```c
typedef enum {
    ENEMY_DECEPTICON,
    ENEMY_SEEKER,
    ENEMY_QUINTESSON,
    ENEMY_BOSS
} EnemyType;
```

Now `EnemyType` is a standalone type name:

```c
// no "enum" keyword needed
EnemyType type = ENEMY_DECEPTICON;   
```

This is the form you will see in virtually all real C codebases.

---

## Enum Naming Conventions

Two conventions are common — pick one and be consistent:

```c
// Convention 1: ALL_CAPS with type prefix
typedef enum { 
	DIR_NORTH, DIR_SOUTH, DIR_EAST, DIR_WEST 
} Direction;

// Convention 2: PascalCase values
typedef enum { North, South, East, West } Direction;
```

The ALL_CAPS-with-prefix convention is more common in systems code and clearly distinguishes enum values from variables. This course uses it.

---

## Enums Are Integers

Under the hood, enum values are `int`. This means:

```c
EnemyType type = ENEMY_SEEKER;
printf("%d\n", type);    // prints 1

// legal but usually wrong
int x = ENEMY_DECEPTICON + ENEMY_QUINTESSON;
// compiles — no range checking!   
EnemyType bad = 999;     
```

C does not enforce that an enum variable holds only declared values. That is your responsibility.

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 3

## Structs

---

## The Problem: Related Data in Separate Variables

Representing an enemy with individual variables:

```c
char  enemy_name[32];
int   enemy_hp;
int   enemy_max_hp;
int   enemy_attack;
int   enemy_defense;
EnemyType enemy_type;
```

Every function that works with an enemy needs all six parameters. Adding a new field means updating every function signature. There is no way to pass "an enemy" — only a collection of loosely related values.

---

## Structs: Grouping Related Data

```c
typedef struct {
    char      name[32];
    int       hp;
    int       max_hp;
    int       attack;
    int       defense;
    EnemyType type;
} Enemy;
```

`Enemy` is now a single type that carries all related fields together. A function that needs an enemy takes one `Enemy` parameter instead of six.

---

#### Declaring Struct Variables
```c
// Declare and initialize later
Enemy decepticon;

// Declare and initialize at once (positional)
Enemy decepticon = {"Decepticon", 40, 40, 8, 2, 
	ENEMY_DECEPTICON};

// Designated initializers (C99) — 
// preferred, order-independent
Enemy decepticon = {
    .name    = "Decepticon",
    .hp      = 40,
    .max_hp  = 40,
    .attack  = 8,
    .defense = 2,
    .type    = ENEMY_DECEPTICON
};
```
---
> [!TIP]
> Designated initializers are safer — they are not sensitive to field order and clearly document which field receives which value.

---

#### Member Access with `.`

```c
Enemy decepticon = {
    .name    = "Decepticon",
    .hp      = 40,
    .max_hp  = 40,
    .attack  = 8,
    .defense = 2,
    .type    = ENEMY_DECEPTICON
};

printf("Name:   %s\n",  decepticon.name);
printf("HP:     %d/%d\n", decepticon.hp, 
	decepticon.max_hp);
printf("Attack: %d\n",  decepticon.attack);

decepticon.hp -= 10;   // modify a field
```

The dot operator gives direct access to any field.

---

## Struct Initialization: Uninitialized Fields

With designated initializers, any fields you omit are **zero-initialized**:

```c
typedef struct {
    char name[32];
    int  hp;
    int  max_hp;
    int  attack;
    int  defense;
} Hero;

Hero h = { .name = "Aria", .hp = 100, .max_hp = 100 };
// h.attack == 0, h.defense == 0  (zero-initialized)
```

This is safe and predictable. Positional initialization without all fields is more dangerous — easy to miscount.

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 4

## Structs Are Value Types

---

## Assignment Copies the Entire Struct

```c
Enemy a = { .name = "Decepticon", .hp = 40, .attack = 8 };
// b is a complete independent copy of a
Enemy b = a;    
// modifying b does NOT affect a
b.hp = 0;       

printf("a.hp = %d\n", a.hp);   // still 40
printf("b.hp = %d\n", b.hp);   // 0
```

Struct assignment copies every field — the same way `int x = y` copies an integer. `a` and `b` are completely independent after the assignment.

---

## Pass-by-Value Copies the Whole Struct

```c
void display_enemy(Enemy e) {
    printf("%s: %d/%d HP\n", e.name, e.hp, e.max_hp);
}

Enemy decepticon = { .name = "Decepticon", .hp = 40, .max_hp = 40 };
// decepticon is COPIED into e
display_enemy(decepticon);   
```

---

```
main's stack frame         display_enemy's frame
┌───────────────┐          ┌───────────────┐
│ decepticon    │─copy──▶  │ e             │
│  .name="dec"  │          │  .name="Dec"  │
│  .hp=40       │          │  .hp=40       │
│  .max_hp=40   │          │  .max_hp=40   │
└───────────────┘          └───────────────┘
```

For large structs, this copy is expensive. For read-only access, `const` pointer is the solution — covered in Lecture 2.

---

## The Implication: Functions Cannot Modify the Caller's Struct

```c
void reset_hp(Enemy e) {
// modifies the local copy only
    e.hp = e.max_hp;    
}

Enemy decepticon = { .hp = 10, .max_hp = 40 };
reset_hp(decepticon);
// still 10 — unchanged
printf("%d\n", decepticon.hp);   
```

This is the same pass-by-value behavior we saw with scalars. The solution is the same: pass a pointer. That is the subject of Lecture 2.

---

# Lecture 1 Wrap-Up

---

## Key Takeaways

- **`typedef`** creates a name alias for a type — makes code more readable and intent-explicit
- **`typedef enum`** is the idiomatic way to define a set of named integer constants
- Enum values are integers — the compiler does not range-check assignments
- **`typedef struct`** groups related fields into a single named type
- Use **designated initializers** (`.field = value`) — they are order-independent and self-documenting
- Structs are **value types** — assignment and function calls copy the entire struct
- A function receiving a struct by value **cannot modify the caller's struct**

---

## Wednesday

- Pointers to structs and the `->` operator
- Passing by pointer vs. by value — when to use each
- Arrays of structs
- Returning structs from functions
- Nested structs
- Bit fields — one slide, awareness only

---

## Resources

- K&R Ch. 6 — Structures
- K.N. King Ch. 16 — Structures, Unions, and Enumerations
- https://en.cppreference.com/w/c/language/struct
- https://en.cppreference.com/w/c/language/enum
