---
share_cop3223c: "true"
site-folder: docs/Lecture Slides
theme: ucf-knights.css
height: "1080"
width: "1920"
---

# File I/O and Command-Line Arguments
## COP 3223C — Lecture 1 of 2
### Reading & Writing Files in C

---

## Why File I/O?

- Programs that only use `stdin`/`stdout` lose data when they exit
- Files let us **persist** data across runs
- Real-world programs read config files, logs, data sets
- C gives us direct, explicit control over every step

---

## The File I/O Mental Model

```
Your Program
     |
  fopen()       <- open a channel
     |
FILE *fp        <- handle to the file
     |
 fprintf()      <- write through it
 fscanf()       <- read through it
     |
  fclose()      <- close the channel
```

> [!TIP]
> Think of `FILE *` like a TV remote —  
> it controls the device, it's not the device.

---

## `FILE *` — The File Handle

```c
#include <stdio.h>

int main(void) {
    /* Declare a file pointer */
    FILE *fp;

    /* fp is just a pointer —
       it doesn't open anything yet */
    fp = NULL;
    return 0;
}
```

- `FILE` is a struct defined in `<stdio.h>`
- We never inspect its internals directly
- Always initialize to `NULL` until opened

---

## `fopen()` — Opening a File

```c
FILE *fopen(
    const char *path,
    const char *mode
);
```

| Mode | Meaning |
|------|---------|
| `"r"` | Read (file must exist) |
| `"w"` | Write (creates/truncates) |
| `"a"` | Append (creates if needed) |
| `"r+"` | Read + write |
| `"w+"` | Read + write (truncates) |
| `"b"` | Binary (append to any mode) |

---

## `fopen()` in Practice

```c
#include <stdio.h>

int main(void) {
    FILE *fp;

    /* Open for reading */
    fp = fopen("data.txt", "r");

    /* ALWAYS check the return value */
    if (fp == NULL) {
        perror("fopen failed");
        return 1;
    }

    /* ... use the file ... */

    fclose(fp);
    return 0;
}
```

---

## Why Check for `NULL`?

`fopen` returns `NULL` when:

- The file does not exist (mode `"r"`)
- You lack permission to open it
- The path is invalid
- The OS has no more file handles available

**Never skip the NULL check** — dereferencing a NULL `FILE *` is undefined behavior.

---

## `perror()` and `errno`

```c
#include <stdio.h>
#include <errno.h>

fp = fopen("missing.txt", "r");
if (fp == NULL) {
    /* Prints: "fopen: No such file
       or directory" */
    perror("fopen");

    /* errno holds the numeric code */
    printf("errno = %d\n", errno);
    return 1;
}
```

`perror()` automatically appends the OS error string to your message.

---

## `fclose()` — Closing a File

```c
int fclose(FILE *fp);
```

- Flushes any buffered output to disk
- Releases the OS file handle
- Returns `0` on success, `EOF` on error
- **Always close every file you open**

```c
if (fclose(fp) != 0) {
    perror("fclose");
}
/* Set to NULL after closing */
fp = NULL;
```

---

## Writing: `fprintf()`

```c
/* fprintf works like printf,
   but targets a FILE * */
int fprintf(
    FILE *fp,
    const char *fmt,
    ...
);
```

```c
FILE *fp = fopen("out.txt", "w");
if (fp == NULL) { /* handle error */ }

fprintf(fp, "Name: %s\n", "Alice");
fprintf(fp, "Score: %d\n", 95);

fclose(fp);
```

---

## Writing: `fputs()` and `fputc()`

```c
/* Write a whole string */
fputs("Hello, file!\n", fp);

/* Write a single character */
fputc('A', fp);
fputc('\n', fp);
```

- `fputs` does **not** add a newline automatically
- `fputc` is useful for character-by-character output
- Both return `EOF` on error

---

## Reading: `fscanf()`

```c
int fscanf(
    FILE *fp,
    const char *fmt,
    ...
);
```

```c
char name[64];
int score;

/* Returns number of items matched */
int n = fscanf(fp, "%63s %d",
               name, &score);
if (n != 2) {
    /* Short read or error */
}
```

---

## Reading: `fgets()`

```c
char *fgets(
    char *buf,
    int size,
    FILE *fp
);
```

```c
char line[128];

/* Reads up to 127 chars + '\0' */
while (fgets(line, sizeof(line),
             fp) != NULL) {
    printf("Line: %s", line);
}
```

- Safer than `fscanf` for line-oriented data
- **Includes the newline** in the buffer
- Returns `NULL` at EOF or on error

---

## `fgets()` vs `fscanf()` — When to Use Each

| | `fgets` | `fscanf` |
|-|---------|----------|
| **Unit** | One line | One token/field |
| **Buffer overflow** | Safe (size param) | Risky without width |
| **Newline** | Included | Skipped |
| **Best for** | Log files, text lines | Structured data |

**Rule of thumb:** Use `fgets` + `sscanf` for structured line-oriented files.

---

## Combining `fgets` + `sscanf`

```c
char line[256];
char name[64];
int score;

while (fgets(line, sizeof(line),
             fp) != NULL) {
    /* Parse the line in memory */
    if (sscanf(line, "%63s %d",
               name, &score) == 2) {
        printf("%s -> %d\n",
               name, score);
    }
}
```

Two-pass approach: read safely, then parse safely.

---

## Reading: `fgetc()` and EOF

```c
int c;

/* fgetc returns int, not char */
while ((c = fgetc(fp)) != EOF) {
    putchar(c);
}
```

- Return type is `int` — must hold `EOF` (-1) **and** all 256 byte values
- Assigning to `char` before checking `EOF` is a **classic bug**

```c
/* BUG: char may wrap at -1 */
char c;
while ((c = fgetc(fp)) != EOF) { }
```

---

## Detecting End-of-File

```c
/* After a read returns EOF/NULL */
if (feof(fp)) {
    printf("Reached end of file\n");
}

/* Check for I/O error */
if (ferror(fp)) {
    perror("Read error");
    clearerr(fp);
}
```

- `feof()` — true only **after** a failed read
- Do **not** use `feof()` as a loop condition (common mistake!)

---

## The `feof()` Loop Bug

```c
/* WRONG — reads last item twice */
while (!feof(fp)) {
    fscanf(fp, "%d", &n);
    printf("%d\n", n);
}

/* CORRECT — check the read itself */
while (fscanf(fp, "%d", &n) == 1) {
    printf("%d\n", n);
}
```

`feof` only becomes true **after** a read has already failed. The loop body runs one extra time.

---

## File Position: `fseek()` and `ftell()`

```c
/* Move to byte offset from origin */
int fseek(
    FILE *fp,
    long offset,
    int origin
);

/* Return current position */
long ftell(FILE *fp);
```

| Origin | Meaning |
|--------|---------|
| `SEEK_SET` | From start of file |
| `SEEK_CUR` | From current position |
| `SEEK_END` | From end of file |

---

## `fseek()` Example

```c
/* Jump to byte 10 from start */
fseek(fp, 10L, SEEK_SET);

/* Save position */
long pos = ftell(fp);

/* Go back to start */
fseek(fp, 0L, SEEK_SET);

/* Jump to end */
fseek(fp, 0L, SEEK_END);

/* File size in bytes */
long size = ftell(fp);
```

---

## `rewind()`

```c
/* Equivalent to:
   fseek(fp, 0L, SEEK_SET)
   + clears error flag */
rewind(fp);
```

Useful when you want to read a file a second time from the beginning.

---

## Binary Files: `fread()` and `fwrite()`

```c
size_t fread(
    void *buf,
    size_t size,   /* bytes per item */
    size_t count,  /* number of items */
    FILE *fp
);

size_t fwrite(
    const void *buf,
    size_t size,
    size_t count,
    FILE *fp
);
```

---

## Binary File Example

```c
typedef struct {
    char name[32];
    int  score;
} Record;

Record r = {"Alice", 95};

/* Open in binary write mode */
FILE *fp = fopen("data.bin", "wb");
if (!fp) { /* handle error */ }

fwrite(&r, sizeof(Record), 1, fp);
fclose(fp);
```

---

## Reading Binary Data Back

```c
Record r2;
FILE *fp = fopen("data.bin", "rb");
if (!fp) { /* handle error */ }

size_t n = fread(
    &r2, sizeof(Record), 1, fp
);

if (n == 1) {
    printf("%s: %d\n",
           r2.name, r2.score);
}
fclose(fp);
```

---

## Text vs Binary Mode

| | Text `"r"/"w"` | Binary `"rb"/"wb"` |
|-|----------------|---------------------|
| Newlines | Translated (`\r\n` ↔ `\n`) | Exact bytes |
| EOF | `Ctrl+Z` on Windows | Byte value |
| Use for | Human-readable files | Structs, images, data |

On Linux, text and binary modes are identical — but always use `"b"` for portability.

---

## Temporary Files

```c
#include <stdio.h>

/* Creates a unique temp file,
   auto-deleted on fclose/exit */
FILE *tmp = tmpfile();
if (tmp == NULL) {
    perror("tmpfile");
}

fprintf(tmp, "scratch data\n");

/* No fclose needed — auto-deleted */
fclose(tmp);
```

---

## Complete Read-and-Echo Example

```c
#include <stdio.h>

int main(void) {
    FILE *fp;
    char line[256];

    fp = fopen("input.txt", "r");
    if (fp == NULL) {
        perror("fopen");
        return 1;
    }

    while (fgets(line,
                 sizeof(line),
                 fp) != NULL) {
        fputs(line, stdout);
    }

    fclose(fp);
    return 0;
}
```

---

## Complete Write Example

```c
#include <stdio.h>

int main(void) {
    FILE *fp;
    int i;

    fp = fopen("numbers.txt", "w");
    if (fp == NULL) {
        perror("fopen");
        return 1;
    }

    for (i = 1; i <= 10; i++) {
        fprintf(fp, "%d\n", i);
    }

    fclose(fp);
    printf("Wrote numbers.txt\n");
    return 0;
}
```

---

## Copy-File Example

```c
#include <stdio.h>

int main(void) {
    FILE *src, *dst;
    int c;

    src = fopen("in.txt",  "r");
    dst = fopen("out.txt", "w");

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

## Common File I/O Mistakes

1. **Not checking `fopen` return value**
2. **Using `char` instead of `int` for `fgetc`**
3. **Using `feof()` as a loop condition**
4. **Forgetting `fclose()`** — data may not be flushed
5. **Opening in wrong mode** — `"w"` destroys existing content
6. **Buffer overflow with `fscanf("%s")`** — always use width specifier

---

## File I/O Checklist

```
□ #include <stdio.h>
□ Declare FILE *
□ Call fopen() with correct mode
□ Check for NULL return
□ Read/write using f* functions
□ Check read return values
□ Call fclose() when done
□ Set pointer to NULL after close
```

---

## Lecture 1 Summary

| Function | Purpose |
|----------|---------|
| `fopen` | Open a file, get `FILE *` |
| `fclose` | Close and flush |
| `fprintf` / `fputs` / `fputc` | Write text |
| `fscanf` / `fgets` / `fgetc` | Read text |
| `fread` / `fwrite` | Binary I/O |
| `fseek` / `ftell` | Seek/position |
| `feof` / `ferror` | Status checks |

**Next lecture:** Command-line arguments + putting it all together

