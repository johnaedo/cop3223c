---
share_cop3223c: "true"
site-folder: docs/Assignments
---

# Week 5 Assignment — Student Roster System
### Assigned: Monday | Due: Sunday 11:59 PM
### Points: 100 | 80 pts automated + 20 pts manual

---

## Overview

You will build a multi-file **student roster system** using structs, enums, and the full header/implementation split. The system tracks students with grades, supports searching and sorting, and prints formatted reports.

This is the first assignment where the design of your types matters as much as the correctness of your logic. A well-chosen struct and enum make the implementation clean; poor choices make it painful.

---

## Required Files

| File | Purpose |
|------|---------|
| `roster.h` | Include guard, all typedefs, all prototypes |
| `roster.c` | All function implementations |
| `main.c` | Interactive driver |

**Compile:** `gcc -Wall -Wextra -std=c17 -o roster main.c roster.c`

Zero warnings required. 5 pts per warning, max 20 pts.

---

## Required Types (define in `roster.h`)

### `Grade` enum

```c
typedef enum {
    GRADE_A,
    GRADE_B,
    GRADE_C,
    GRADE_D,
    GRADE_F,
    GRADE_INCOMPLETE
} Grade;
```

### `Student` struct

```c
typedef struct {
    char   first_name[32];
    char   last_name[32];
    int    student_id;      /* 6-digit positive integer */
    double gpa;             /* 0.00 to 4.00 */
    Grade  standing;        /* current letter grade standing */
} Student;
```

### `Roster` struct

```c
#define MAX_STUDENTS 50

typedef struct {
    Student students[MAX_STUDENTS];
    int     count;
} Roster;
```

---

## Functions to Implement in `roster.c`

### Student construction

**`Student create_student(const char *first, const char *last, int id, double gpa)`**

Returns a fully initialized `Student`. Derive `standing` from `gpa`:

| GPA | Grade |
|-----|-------|
| 3.5 – 4.0 | `GRADE_A` |
| 3.0 – 3.49 | `GRADE_B` |
| 2.0 – 2.99 | `GRADE_C` |
| 1.0 – 1.99 | `GRADE_D` |
| < 1.0 | `GRADE_F` |

### Roster management

**`int roster_add(Roster *r, Student s)`**

Add student `s` to the roster. Return 1 on success, 0 if full or -1 if a student with the same ID already exists.

**`int roster_remove(Roster *r, int student_id)`**
Remove the student with `student_id`. Compact the array. Return 1 if found and removed, 0 otherwise.

To compact the array, you may reference this sample code:  [Compacting an Array (naive)](https://teaching.johnaedo.com/cop3223c/Examples/Compacting%20an%20Array%20%28naive%29/)

**`Student *roster_find_by_id(Roster *r, int student_id)`**
Return a pointer to the matching student, or `NULL` if not found.

**`Student *roster_find_by_name(Roster *r, const char *last_name)`**
Return a pointer to the first student with matching `last_name`, or `NULL` otherwise.

### Sorting

**`void roster_sort_by_name(Roster *r)`**
Sort students alphabetically by last name (then first name as a tiebreaker). Use [bubble sort](https://en.wikipedia.org/wiki/Bubble_sort) or [selection sort](https://en.wikipedia.org/wiki/Selection_sort) — swap entire `Student` structs.

**`void roster_sort_by_gpa(Roster *r)`**
Sort students descending by GPA.

### Display

**`void print_student(const Student *s)`**
Print one student on one line:

```
[123456] Smith, John          GPA: 3.75  Standing: A
```
If `s` is NULL, print `No Student to Print` and return


**`void print_roster(const Roster *r)`**
Print a header, then all students using `print_student`, then a summary line:

```
╔══════════════════════════════════════════════════╗
║  Student Roster (12 students)                    ║
╠══════════════════════════════════════════════════╣
  [100001] Adams, Alice         GPA: 3.92  Standing: A
  [100002] Baker, Bob           GPA: 2.45  Standing: C
  ...
╠══════════════════════════════════════════════════╣
║  Class average GPA: 3.14                         ║
╚══════════════════════════════════════════════════╝
```

**`const char *grade_to_string(Grade g)`**
Return `"A"`, `"B"`, `"C"`, `"D"`, `"F"`, or `"Incomplete"` for the given grade.

**`double roster_average_gpa(const Roster *r)`**
Return the mean GPA of all students, or `0.0` if roster is empty.

---

## The Driver: `main.c`

Present a menu that loops until Quit:

```
=== Student Roster System ===
1. Add student
2. Remove student (by ID)
3. Find student (by ID)
4. Find student (by last name)
5. Sort by name
6. Sort by GPA
7. Print roster
8. Quit
Enter choice (1-8):
```

**Input for option 1:**
```
First name: Alice
Last name:  Adams
Student ID: 100001
GPA:        3.92
Student added.
```
If `roster_add` returns a 0, print `Roster Full, Student Not Added`
If `roster_add` returns a -1, print `ID Already Exists, Student Not Added`

**Input for option 2:**
```
Enter student ID: 100001
Student removed.
```
If `roster_remove` returns 0, print  `Student Not Found`

**Input for option 3/4:**
Print the student using `print_student` if found, or `Student not found.`

---

## Grading Rubric

### Automated (80 points)

| Test | Pts | Description                                                                 |
| ---- | --- | --------------------------------------------------------------------------- |
| T01  | 5   | Compiles, zero warnings                                                     |
| T02  | 8   | `create_student` — correct fields, correct `standing` derived from GPA      |
| T03  | 8   | `roster_add` — success, duplicate ID rejected, full roster rejected         |
| T04  | 6   | `roster_remove` — found, not found, array compacted correctly               |
| T05  | 8   | `roster_find_by_id` — found returns correct pointer, not found returns NULL |
| T06  | 6   | `roster_find_by_name` — found, not found                                    |
| T07  | 8   | `roster_sort_by_name` — alphabetical by last name                           |
| T08  | 8   | `roster_sort_by_gpa` — descending GPA                                       |
| T09  | 8   | `print_roster` — correct format, correct average GPA                        |
| T10  | 5   | `grade_to_string` — all six grades correct                                  |
| T11  | 5   | Menu loops; Quit exits; invalid choice re-prompts                           |
| T12  | 5   | `roster.h` has include guard; types defined there; no bodies                |

### Manual Review (20 points)

| Criterion                                                                    | Pts |
| ---------------------------------------------------------------------------- | --- |
| `Student` and `Roster` defined exactly as specified in `roster.h`            | 5   |
| `roster_find_*` return pointers (not copies); caller can modify through them | 5   |
| `roster_sort_*` swap entire `Student` structs (not pointers or indices)      | 5   |
| No global variables; `const` used correctly on read-only pointer params      | 5   |

---

## Starter Scaffold

**`roster.h`**
```c
#ifndef ROSTER_H
#define ROSTER_H

#define MAX_STUDENTS 50

typedef enum {
    GRADE_A, GRADE_B, GRADE_C, GRADE_D, GRADE_F, GRADE_INCOMPLETE
} Grade;

typedef struct {
    char   first_name[32];
    char   last_name[32];
    int    student_id;
    double gpa;
    Grade  standing;
} Student;

typedef struct {
    Student students[MAX_STUDENTS];
    int     count;
} Roster;

/* Prototypes */
Student  create_student(const char *first, const char *last,
                        int id, double gpa);
int      roster_add(Roster *r, Student s);
int      roster_remove(Roster *r, int student_id);
Student *roster_find_by_id(Roster *r, int student_id);
Student *roster_find_by_name(Roster *r, const char *last_name);
void     roster_sort_by_name(Roster *r);
void     roster_sort_by_gpa(Roster *r);
void     print_student(const Student *s);
void     print_roster(const Roster *r);
const char *grade_to_string(Grade g);
double   roster_average_gpa(const Roster *r);

#endif /* ROSTER_H */
```

---

## Submission

Submit exactly: `roster.h`, `roster.c`, `main.c`
Up to 2 resubmissions; highest score counts.
