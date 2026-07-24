---
share_cop3223c: "true"
site-folder: docs/Lecture Slides
theme: ucf-knights.css
height: "1080"
width: "1920"
---

# Algorithm Efficiency
## A Primer on Big O Notation

---

## The Problem with "Time"
- How do we know if code is fast?
- We cannot just use a stopwatch.
- Execution time changes based on:
  - CPU speed
  - Background processes
  - Compiler optimizations
  - many other factors we can't control!

---

## A Universal Metric
- Instead of seconds, we measure scaling.
- We ask: As the input size ($N$) grows, how does the number of operations grow?
- This is called Asymptotic Analysis.
- Commonly referred to as Big O Notation.

---

## What is Big O Notation?
- Describes the worst-case scenario.
- Focuses on the "order of magnitude".
- Ignore exact operation counts.
- Look at the dominant trend as $N$ approaches infinity.
---

## $O(1)$ — Constant Time
- The fastest classification.
- Time taken does not depend on $N$.
- Whether $N$ is 10 or 10,000,000, it takes the same amount of effort.
---

## $O(1)$ Examples
- Accessing an array element: `arr[5]`
- Checking if a number is even.
- Inserting a node at the head of a linked list.

```c
// N does not matter here
int get_first(int arr[]) {
    return arr[0]; 
}
````
---

## $O(N)$ — Linear Time

- Time grows directly proportional to $N$.
- If you double the input size, the algorithm takes twice as long.
- Usually involves looking at every item once.

---
## $O(N)$ Examples

- Searching an unsorted array for a value.
- Finding the maximum value in an array.
- Counting the nodes in a linked list

---

## $O(N)$ Code Example

```c
// Loop runs exactly N times
void print_all(int n) {
    int i;
    for (i = 0; i < n; i++) {
        printf("%d\n", i);
    }
}
```

---

## $O(N^2)$ — Quadratic Time

- Time grows exponentially with $N$.
- If you double the input, the time quadruples!
- Often the result of nested loops
- Dangerous for large data sets.

---

## $O(N^2)$ Code Example

```c
// Nested loops = N * N operations
void print_pairs(int n) {
    int i, j;
    for (i = 0; i < n; i++) {
        for(j = 0; j < n; j++) {
            printf("%d, %d\n", i, j);
        }
    }
}
```
---

## Golden Rules of Big O

- **Drop constants:** $O(2N)$ is just $O(N)$. We only care about the shape of the curve.
    
- **Drop smaller terms:** $O(N^2 + N)$ becomes $O(N^2)$. As $N$ gets massive, the $N^2$ completely dominates the runtime.

---
## Why Big O Matters Now

- Arrays vs. Linked Lists!
- Inserting at the front of an Array:
    - Must shift every element down.
    - Time Complexity: $O(N)$
- Inserting at the front of a Linked List:
    - Just update two pointers.
    - Time Complexity: $O(1)$
- Knowing Big O helps us choose the right tool for the job.