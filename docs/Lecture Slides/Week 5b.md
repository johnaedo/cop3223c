---
share_cop3223c: "true"
site-folder: docs/Lecture Slides
theme: ucf-knights.css
height: "1080"
width: "1920"
---

# Week 5, Lecture 2: Structs in Depth

---

## Agenda

1. Pointers to structs and the `->` operator
2. Passing by pointer vs. by value
3. Arrays of structs
4. Returning structs from functions
5. Nested structs
6. Bit fields — awareness only

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 1

## Pointers to Structs and `->`

---

## Taking the Address of a Struct

```c
Enemy goblin = { .name = "Goblin", .hp = 40, .max_hp = 40 };
Enemy *p     = &goblin;
```

`p` holds the address of `goblin`. Dereferencing gives back the whole struct:

```c
(*p).hp = 30;   // dereference p, then access .hp
```

This works — but the parentheses are required because `.` binds tighter than `*`. Without them:

```c
*p.hp   // parsed as *(p.hp) — wrong, and a compile error
```

---

## The `->` Operator

Because `(*p).field` is clunky to type and read, C provides the **arrow operator** as direct shorthand:

```c
// identical — same operation
p->hp     ==   (*p).hp    
p->name   ==   (*p).name
```

**Arrow means:** "go to the struct this pointer points at, then access the named field."

```c
Enemy *p = &goblin;

p->hp -= 10;
printf("%s HP: %d\n", p->name, p->hp);
```

In practice you will almost always use `->` rather than `(*p).`. It is the universal idiom for pointer-to-struct field access.

---

## `.` vs `->`

The rule is mechanical:

| You have | Use |
|----------|-----|
| A struct variable | `.` |
| A pointer to a struct | `->` |

```c
Enemy  e  = { .hp = 40 };
Enemy *p  = &e;

e.hp      // variable — dot
p->hp     // pointer  — arrow
// pointer  — dereference then dot (equivalent, avoid)
(*p).hp   
```

If you use the wrong one the compiler will tell you immediately. This is one of C's clearer error messages.

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 2

## Passing By Pointer vs. By Value

---

## Recap: Pass-by-Value Copies the Struct

```c
void display(Enemy e) {
	// copy of entire struct          
    printf("%s: %d HP\n", e.name, e.hp);
}
```

For a struct with 6 fields, this copies 6 fields. For a struct with 60 fields, it copies 60. For a struct containing arrays, it copies all the array bytes.

Copying is not free. For large structs, pass-by-pointer is significantly more efficient.

---

## Passing a Pointer to Avoid the Copy

```c
void display(const Enemy *e) {   
	// only 8 bytes copied (the pointer)
    printf("%s: %d HP\n", e->name, e->hp);
}

Enemy goblin = { .name = "Goblin", .hp = 40 };
display(&goblin);
```

The `const` is essential here. It says: "I receive this pointer for read-only access — I will not modify the caller's struct." Without `const`, the reader cannot tell whether the function intends to modify the enemy or not.

---

## The Two Pointer Patterns

**Read-only access** — `const` pointer:

```c
void print_enemy(const Enemy *e) {
    printf("%s: %d/%d HP\n", e->name, e->hp, e->max_hp);
}
```

**Modify in place** — non-`const` pointer:

```c
void apply_damage(Enemy *e, int damage) {
    e->hp -= damage;
    if (e->hp < 0) e->hp = 0;
}
```

The presence or absence of `const` at the call site tells the reader everything about intent:

```c
// read-only — const pointer
print_enemy(&goblin);
// modifies goblin — no const         
apply_damage(&goblin, 10);    
```

---

## When to Pass By Value vs. By Pointer

| Scenario | Recommendation |
|----------|---------------|
| Small struct (1–3 fields), read-only | Pass by value — simple and safe |
| Large struct, read-only | Pass `const Struct *` — avoids expensive copy |
| Function must modify the struct | Pass `Struct *` — required |
| Returning a newly created struct | Return by value — compiler optimizes it |

In practice: **almost always pass structs by pointer.** The only common exception is returning a new struct from a constructor-style function.

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 3

## Arrays of Structs

---

## Declaring an Array of Structs

```c
#define MAX_ENEMIES 10

Enemy dungeon[MAX_ENEMIES];
```

Each element is a complete `Enemy` struct. Access combines array indexing with member access:

```c
dungeon[0].hp      = 40;
dungeon[0].attack  = 8;
```

Or with a pointer to an element:

```c
Enemy *e = &dungeon[0];
e->hp = 40;
```

---

### Initializing an Array of Structs

```c
Enemy dungeon[3] = {
	{   .name = "Goblin", 
	    .hp = 40, .max_hp = 40, 
	    .attack =  8, .defense = 2 
	},
    {   .name = "Orc",    
	    .hp = 70, .max_hp = 70, 
	    .attack = 14, .defense = 6 
	},
    {   .name = "Dragon", 
	    .hp = 200, .max_hp = 200, 
	    .attack = 25, .defense = 12
	}
};
```

Each element uses the same designated initializer syntax as a standalone struct declaration.

---

## Iterating Over an Array of Structs

```c
int count = 3;

for (int i = 0; i < count; i++) {
    printf("%-10s %3d/%3d HP  ATK:%2d  DEF:%2d\n",
           dungeon[i].name,
           dungeon[i].hp,
           dungeon[i].max_hp,
           dungeon[i].attack,
           dungeon[i].defense);
}
```

```
Goblin      40/ 40 HP  ATK: 8  DEF: 2
Orc         70/ 70 HP  ATK:14  DEF: 6
Dragon     200/200 HP  ATK:25  DEF:12
```

---

## Passing an Array of Structs to a Function

Arrays of structs decay to a pointer to the first element — exactly the same as arrays of `int`:

```c
void print_all(const Enemy enemies[], int count) {
    for (int i = 0; i < count; i++) {
        printf("%s: %d HP\n", 
	        enemies[i].name, enemies[i].hp);
    }
}

print_all(dungeon, 3);
```

The `const` qualifier on the array parameter prevents the function from modifying any of the enemies. Always pass the count separately — size information is lost on decay.

---

## Finding an Element in an Array of Structs

```c
// Returns pointer to the first matching enemy, or NULL
Enemy *find_enemy(Enemy enemies[], int count, const char *name) {
    for (int i = 0; i < count; i++) {
        if (strcmp(enemies[i].name, name) == 0) {
            return &enemies[i];
        }
    }
    return NULL;
}

Enemy *target = find_enemy(dungeon, 3, "Orc");
if (target != NULL) {
    printf("Found: %s\n", target->name);
    apply_damage(target, 15);   // modifies the array element in place
}
```

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 4

## Returning Structs from Functions

---

#### Functions Can Return Structs

```c
Enemy create_enemy(const char *name, int hp, 
	int attack, int defense) {
    Enemy e = {
        .hp      = hp,
        .max_hp  = hp,
        .attack  = attack,
        .defense = defense
    };
    strncpy(e.name, name, sizeof(e.name) - 1);
    e.name[sizeof(e.name) - 1] = '\0';
    return e;   // returns a copy of the local struct
}

Enemy goblin = create_enemy("Goblin", 40, 8, 2);
Enemy orc    = create_enemy("Orc",    70, 14, 6);
```

This is the **constructor pattern** — a function whose job is to build and return a properly initialized struct.

---

## Is Returning by Value Expensive?

Returning a struct by value looks like it should copy the entire struct. Modern compilers apply **Return Value Optimization (RVO)** — the struct is constructed directly in the caller's stack frame, with no copy at all.

You can return structs by value confidently for this use case. The compiler handles the efficiency.

Do not return a **pointer to a local struct** — the local variable is destroyed when the function returns:

```c
Enemy *bad(void) {
    Enemy e = { .hp = 40 };
    // UNDEFINED BEHAVIOR — e is destroyed on return
    // because it lives in the fn's stack frame!
    return &e;   
}
```

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 5

## Nested Structs

---

## A Struct Can Contain Other Structs

```c
typedef struct {
    int x;
    int y;
} Position;

typedef struct {
    char     name[32];
    int      hp;
    int      attack;
    Position pos;     // struct nested inside struct
} Hero;
```

The nested struct is a full member — initialized and accessed like any other field.

---

## Accessing Nested Fields

```c
Hero hero = {
    .name   = "Aria",
    .hp     = 100,
    .attack = 15,
    .pos    = { .x = 0, .y = 0 }
};

// Chained dot access
printf("Position: (%d, %d)\n", hero.pos.x, hero.pos.y);

hero.pos.x += 1;   // move right

// Through a pointer
Hero *h = &hero;
// move down — arrow to outer, dot to inner
h->pos.y += 1;     
```

---

## When to Use Nested Structs

Nest a struct when the inner fields form a coherent concept that stands on its own:

```c
typedef struct { int r; int g; int b; } Color;
typedef struct { int width; int height; } Size;

typedef struct {
    char  name[32];
    // Color is meaningful on its own
    Color primary_color;
    // Size is meaningful on its own   
    Size  dimensions;      
} Sprite;
```

Do not nest just to avoid typing — only when the inner type has independent meaning and reuse potential.

---

<!-- .slide: data-background="#1a1a1a" -->

# Part 6

## Bit Fields — Awareness Only

---

## What Bit Fields Are

A bit field allows struct members to occupy a specified number of **bits** rather than whole bytes:

```c
typedef struct {
    unsigned int is_alive  : 1;   // 1 bit  — 0 or 1
    unsigned int level     : 4;   // 4 bits — 0 to 15
    unsigned int team      : 2;   // 2 bits — 0 to 3
} EntityFlags;
```

This packs all three fields into a single integer word rather than three separate `int` fields.

---

## When Bit Fields Are Used

- **Hardware registers** — a memory-mapped hardware register may have specific bits meaning specific things; bit fields map directly to that layout
- **Network protocol headers** — compact binary formats where byte layout is specified by a standard (TCP headers, Ethernet frames)
- **Embedded systems** — when RAM is measured in kilobytes and every byte matters

**You will not write bit fields in this course.** You will encounter them in the wild — particularly in systems headers and network code — and now you know what you are looking at.

---

## Bit Field Caveats

Bit field behavior is **implementation-defined** in several ways:
- Bit ordering within a word (big-endian vs. little-endian machines differ)
- Whether `int` bit fields are signed or unsigned
- How padding is inserted between fields

This makes bit fields non-portable across compilers and architectures — another reason to avoid them in general-purpose code.

---

# Lecture 2 Wrap-Up

---

## Key Takeaways

- Use `->` when you have a pointer to a struct; use `.` when you have the struct itself
- Pass structs by **`const Struct *`** for read-only access; by **`Struct *`** to modify; by value only for small structs or return values
- Arrays of structs decay to a pointer to the first element — **always pass count separately**
- The **constructor pattern** — a function that builds and returns a struct by value — is clean and compiler-optimized
- **Never return a pointer to a local struct** — the local is destroyed when the function returns
- Nested structs are appropriate when the inner type has independent meaning
- Bit fields exist for hardware and embedded use cases — you will see them, but not write them here

---

## Looking Ahead — Week 6

- **File I/O:** `fopen`, `fclose`, `fprintf`, `fscanf`, `fgets`
- **Command-line arguments:** `argc` and `argv`
- **RPG:** save and load game state; accept hero name from the command line

---

## Resources

- K&R Ch. 6 — Structures
- K.N. King Ch. 16 — Structures, Unions, and Enumerations
- https://en.cppreference.com/w/c/language/struct
