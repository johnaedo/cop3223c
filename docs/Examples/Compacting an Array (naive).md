---
share_cop3223c: "true"
site-folder: docs/Examples
---


Here's a simple method for compacting an array to effectively delete an item:

```c
#include <stdio.h>

int main() {
    int size = 6;
    int arr[6] = {10, 20, 30, 40, 50, 60}; 
    int position = 3; // Position 3 (index 2)
    int i;

    printf("Initial array (size %d): 10, 20, 30, 40, 50, 60\n", size);

    // Position is 1-based, convert to 0-based index
    int index = position - 1; 

    if (index < 0 || index >= size) {
        printf("Invalid position for deletion.\n");
        return 1;
    }
    
    // 1. Shift elements one position to the left (starting from the deletion index)
    // This overwrites the element at 'index'
    for (i = index; i < size - 1; i++) {
        arr[i] = arr[i + 1];
    }

    // 2. Decrement the size of the array
    size--;

    printf("Array after deleting element at position %d (index %d):\n", position, index);
    for (i = 0; i < size; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");

    return 0;
}
```
