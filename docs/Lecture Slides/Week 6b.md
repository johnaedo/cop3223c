---
share_cop3223c: "true"
site-folder: docs/Lecture Slides
theme: ucf-knights.css
height: "1080"
width: "1920"
---

# File I/O and Command-Line Arguments
## COP 3223C — Lecture 2 of 2
### Command-Line Arguments & Integration

---

## What Are Command-Line Arguments?

```
$ ./myprogram input.txt output.txt
```

- Everything after the program name is an **argument**
- The shell splits on whitespace and passes to `main`
- Arguments arrive as strings — always

---

## The `main` Signature With Arguments

```c
int main(int argc, char *argv[])
```

| Parameter | Meaning |
|-----------|---------|
| `argc` | Argument **count** (always ≥ 1) |
| `argv` | Argument **vector** (array of strings) |

```
argc = 3
argv[0] = "./myprogram"
argv[1] = "input.txt"
argv[2] = "output.txt"
argv[3] = NULL   /* sentinel */
```

---

## `argv` in Memory

```
argv ──► [ ptr ]──► "./myprogram\0"
         [ ptr ]──► "input.txt\0"
         [ ptr ]──► "output.txt\0"
         [ NULL ]
```

- `argv[0]` is the program name (path)
- `argv[argc]` is always `NULL`
- Each `argv[i]` is a `char *` string

---

## Accessing Arguments

```c
#include <stdio.h>

int main(int argc, char *argv[]) {
    int i;

    printf("argc = %d\n", argc);

    for (i = 0; i < argc; i++) {
        printf(
            "argv[%d] = \"%s\"\n",
            i, argv[i]
        );
    }
    return 0;
}
```

---

## Checking Argument Count

```c
int main(int argc, char *argv[]) {
    /* Program name counts as 1 */
    if (argc != 3) {
        fprintf(
            stderr,
            "Usage: %s <in> <out>\n",
            argv[0]
        );
        return 1;
    }

    /* Safe to use argv[1], argv[2] */
    return 0;
}
```

Always print usage to `stderr`, not `stdout`.

---

## Converting Arguments to Numbers

```c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr,
            "Usage: %s <n>\n",
            argv[0]);
        return 1;
    }

    /* atoi: string → int */
    int n = atoi(argv[1]);
    printf("n = %d\n", n);
    return 0;
}
```

---

## `atoi` vs `strtol` — Know the Difference

```c
#include <stdlib.h>

/* atoi: simple, no error detection */
int n = atoi("42abc");  /* n = 42 */
int m = atoi("xyz");    /* m = 0 */

/* strtol: detects errors */
char *end;
long v = strtol("42abc", &end, 10);
/* end points to "abc" */
if (*end != '\0') {
    fprintf(stderr,
        "Invalid number\n");
}
```

Prefer `strtol` when correctness matters.

---

## `strtod` for Floating Point

```c
#include <stdlib.h>

char *end;
double d = strtod("3.14xyz", &end);

if (*end != '\0') {
    fprintf(stderr,
        "Not a valid float\n");
    return 1;
}

printf("d = %.2f\n", d);
```

---

## Flags and Options

Programs often use flags:

```
$ ./sort -r -n input.txt
```

Simple manual parsing:

```c
int reverse = 0, numeric = 0;
int i;

for (i = 1; i < argc; i++) {
    if (argv[i][0] == '-') {
        if (argv[i][1] == 'r')
            reverse = 1;
        else if (argv[i][1] == 'n')
            numeric = 1;
    }
}
```

---

## `getopt()` — Standard Option Parsing

```c
#include <unistd.h>

int main(int argc, char *argv[]) {
    int opt;

    /* "rn:" means: -r (no arg),
                    -n (no arg) */
    while ((opt = getopt(
                argc, argv, "rn"))
           != -1) {
        switch (opt) {
        case 'r': reverse = 1; break;
        case 'n': numeric = 1; break;
        default:
            /* '?' for unknown */
            return 1;
        }
    }
}
```

---

## `getopt()` With Required Arguments

```c
/* "o:" — -o requires an argument */
while ((opt = getopt(
            argc, argv, "o:"))
       != -1) {
    switch (opt) {
    case 'o':
        /* optarg points to value */
        outfile = optarg;
        break;
    default:
        return 1;
    }
}
/* Non-option args start at
   argv[optind] */
```

---

## Putting It Together: File Copier

```c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[]) {
    FILE *src, *dst;
    int c;

    if (argc != 3) {
        fprintf(stderr,
            "Usage: %s src dst\n",
            argv[0]);
        return 1;
    }

    src = fopen(argv[1], "r");
    dst = fopen(argv[2], "w");
```
---
```c
    if (!src || !dst) {
        perror("fopen");
        return 1;
    }

    while ((c = fgetc(src)) != EOF)
        fputc(c, dst);

    fclose(src);
    fclose(dst);
    return 0;
}
```

---

## Word Count Tool — Skeleton

```c
#include <stdio.h>
#include <ctype.h>

int count_words(FILE *fp) {
    int c, in_word = 0, count = 0;

    while ((c = fgetc(fp)) != EOF) {
        if (isspace(c)) {
            in_word = 0;
        } else if (!in_word) {
            in_word = 1;
            count++;
        }
    }
    return count;
}
```

---

## Word Count — `main`

```c
int main(int argc, char *argv[]) {
    FILE *fp;
    int words;

    if (argc != 2) {
        fprintf(stderr,
            "Usage: %s file\n",
            argv[0]);
        return 1;
    }

    fp = fopen(argv[1], "r");
    if (!fp) {
        perror(argv[1]);
        return 1;
    }

    words = count_words(fp);
    fclose(fp);

    printf("%d %s\n",
           words, argv[1]);
    return 0;
}
```

---

## CSV Reader Example

```c
/* grades.csv:
   Alice,95
   Bob,87
   Carol,91
*/
char name[64];
int  score;

while (fgets(line,
             sizeof(line), fp)) {
    /* sscanf with comma delimiter */
    if (sscanf(line,
               "%63[^,],%d",
               name, &score) == 2) {
        printf("%-10s %3d\n",
               name, score);
    }
}
```

`%63[^,]` reads up to 63 chars stopping at a comma.

---

## Multiple Input Files

```c
int main(int argc, char *argv[]) {
    int i;
    FILE *fp;
    char line[256];

    /* Process each argument */
    for (i = 1; i < argc; i++) {
        fp = fopen(argv[i], "r");
        if (!fp) {
            /* perror uses argv[i]
               as the prefix */
            perror(argv[i]);
            continue; /* skip it */
        }
        while (fgets(line,
                     sizeof(line),
                     fp) != NULL)
            fputs(line, stdout);
        fclose(fp);
    }
    return 0;
}
```

---

## stderr vs stdout

```c
/* Normal output → stdout */
printf("Result: %d\n", result);
fprintf(stdout, "Done.\n");

/* Errors, usage → stderr */
fprintf(stderr,
    "Error: file not found\n");
perror("fopen");
```

- Allows separate redirection:
  `./prog > out.txt 2> err.txt`
- Usage messages always go to `stderr`

---

## Exit Status Conventions

```c
#include <stdlib.h>

/* Success */
return 0;
exit(EXIT_SUCCESS);  /* = 0 */

/* Failure */
return 1;
exit(EXIT_FAILURE);  /* = 1 */
```

- Shell can inspect with `$?`
- Scripts can chain: `./prog && ./next`
- Use `EXIT_SUCCESS` / `EXIT_FAILURE` for clarity

---

## `exit()` vs `return` from `main`

```c
#include <stdlib.h>

void cleanup(void) {
    printf("Cleaning up...\n");
}

int main(void) {
    /* atexit runs cleanup()
       on exit() OR return */
    atexit(cleanup);

    /* Both flush stdio & run
       atexit handlers */
    exit(EXIT_SUCCESS);
    /* OR */
    return 0;
}
```

---

## Environment Variables

```c
#include <stdlib.h>

int main(void) {
    /* getenv returns NULL if
       variable not set */
    char *home = getenv("HOME");

    if (home != NULL) {
        printf("Home: %s\n", home);
    } else {
        printf("HOME not set\n");
    }
    return 0;
}
```

Useful for configuration without command-line args.

---

## Robust Argument Parsing Pattern

```c
typedef struct {
    char *infile;
    char *outfile;
    int   verbose;
} Options;

int parse_args(
    int argc, char *argv[],
    Options *opts)
{
    opts->infile  = NULL;
    opts->outfile = NULL;
    opts->verbose = 0;
    /* ... parse ... */
    return 0;
}
```

Encapsulate options in a struct for larger programs.

---

## Security Considerations

- **Never trust `argv`** — always validate
- Filenames can contain spaces, special chars
- **Buffer overflows**: use `strncpy`, not `strcpy`
- Validate numeric ranges after conversion

```c
int n = atoi(argv[1]);
if (n < 1 || n > 1000) {
    fprintf(stderr,
        "n must be 1-1000\n");
    return 1;
}
```

---

## Lecture 2 Summary

| Topic | Key Points |
|-------|-----------|
| `argc`/`argv` | Count + array of C strings |
| `argv[0]` | Program name |
| Conversion | `atoi`, `strtol`, `strtod` |
| Flags | Manual `-x` or `getopt()` |
| `stderr` | Errors and usage messages |
| Exit codes | `0` success, nonzero failure |
| Pattern | Validate → Open → Process → Close |

---

## Week Summary: File I/O + CLI Args

```c
int main(int argc, char *argv[]) {
    /* 1. Validate arguments */
    if (argc != 2) { /* usage */ }

    /* 2. Open file */
    FILE *fp = fopen(argv[1], "r");
    if (!fp) { perror(argv[1]); }

    /* 3. Process */
    /* ... read/write operations */

    /* 4. Clean up */
    fclose(fp);
    return EXIT_SUCCESS;
}
```

---

## Looking Ahead

**Next week:** Dynamic Memory Allocation
- `malloc`, `calloc`, `realloc`, `free`
- Heap vs stack
- Memory leaks and valgrind
- Dynamic arrays and linked structures

These concepts will make our file I/O programs much more powerful.
