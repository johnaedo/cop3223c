---
share_cop3223c: "true"
site-folder: docs/Lecture Slides
theme: ucf-knights.css
height: "1080"
width: "1920"
---

# `const` in C


---

## What `const` Means

`const` tells the compiler that a value **must not be modified** after it is initialized.

```c
const double PI = 3.14159265358979;
// compile error — assignment to a const variable
PI = 3.0;   
```

That's the whole idea. Everything else is a consequence of it.

---

## `const` vs. `#define`

Both are used for named constants. They are not the same thing.

```c
// preprocessor text substitution
#define MAX_SIZE 100        
// typed, scoped variable
const int max_size = 100;   
```
---

|                            | #define                             | const variable              |
| -------------------------- | ----------------------------------- | --------------------------- |
| Has a type                 | No — raw text replacement           | Yes — `int`, `double`, etc. |
| Visible to debugger        | No                                  | Yes                         |
| Obeys scope                | No — file-wide once defined         | Yes — block or file scope   |
| Can be watched in debugger | No                                  | Yes                         |
| Accidentally misused       | `#define SQ(x) x*x` → `SQ(1+2)` = 5 | Not possible                |

> **Prefer `const` variables** over `#define` for numeric and typed constants. Reserve `#define` for include guards and macros.

---

## `const` in Function Parameters

When a function receives a value by copy, `const` on a value parameter is unusual — it just means the function cannot reassign its own local copy. More useful and common is `const` on a **pointer parameter**:

```c
// Without const — caller can't tell if their data is safe
void print_score(int *score);

// With const — compiler guarantees the function 
// won't modify *score
void print_score(const int *score);
```

`const int *score` means: "I receive an address, but I promise only to read through it."

---

## What `const` on a Pointer Parameter Buys You

**For the caller:** safety. You can pass a `const` variable or a literal without a cast.

```c
const int high_score = 9999;
// fine — function promises not to modify it
print_score(&high_score);   
```
---

**For the reader:** documentation. A `const` pointer parameter immediately signals "this is an input, not an output."

```c
// Output params have no const — they will be written to
void compute(const int *input, int *output);
//           ^— read only      ^— will be modified
```

**For the compiler:** enforcement. Any attempt to write through a `const` pointer is a compile error — caught before the program ever runs.

---

## The Two Positions of `const` with Pointers

This is covered in detail in the Week 4 Pointers lecture. As a preview:

```c
const int *p;      // pointer to const int
                   // → cannot change *p (the value)
                   // → can change p  (the address)

int *const p;      // const pointer to int
                   // → can change *p  (the value)
                   // → cannot change p (the address)

const int *const p; // both are fixed
```

The first form — **pointer to `const`** — is what you will use in nearly every function parameter. It means "read-only access through this pointer."

The second form — **`const` pointer** — is rare in practice; it appears mainly in fixed-address hardware registers and similar low-level contexts.

---

## The Practical Rules

1. **Named numeric constants:** use `const` instead of `#define`
   ```c
   const int    MAX_PLAYERS  = 4;
   const double GRAVITY      = 9.81;
   ```

---

2. **String parameters a function only reads:** use `const char *`
   ```c
   void greet(const char *name);
   void log_message(const char *msg);
   ```
---

3. **Any pointer parameter a function only reads:** add `const`
   ```c
   double average(const int arr[], int len);
   int    str_length(const char *s);
   ```

---

4. **Output pointer parameters:** no `const` — the function needs to write through them
   ```c
   void min_max(const int arr[], int len, int *out_min, int *out_max);
   //           ^— read only             ^— will be written
   ```

---

## Quick Reference

```c
// Named constant — use instead of #define
const double PI = 3.14159;

// Read-only pointer parameter 
// — function will not modify what p points to
void f(const int *p);

// Read-only array parameter 
// — same thing, array syntax
void g(const int arr[], int len);

// Read-only string parameter 
// — standard for string functions
void h(const char *s);

// Both pointer and value fixed — rare
const int *const p = &x;
```
---

> **The one-sentence rule:** put `const` on anything that should not change. Let the compiler enforce it.
