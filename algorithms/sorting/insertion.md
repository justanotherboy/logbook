# Insertion sort

## Description

Insertion sort takes one element of input data and inserts the component into the correct position of a growing sorting list. This method compares each element with the components of the sorted list to find the correct place for the element.

## Analysis

- Space complexity: O(1)
- Runtime complexity: O(n²) (More efficient in practice than bubble or selection sort)
- Easy to implement
- Can sort elements online

## Code

```C
void insertion_sort(int *array) {
    for (int i=1; i < ARRAY_SIZE; i++) {
        int tmp = array[i];
        int j = i-1;
        while (j >= 0 && array[j] > tmp) {
            array[j+1] = array[j];
            j--;
        }
        array[j+1] = tmp;
    }
    return;
}
```
