# Exercise 4: Pointer Arithmetic

## Output Analysis

Running the `pointers.c` program produced the following output:

```
1: a = 0x16f7369a8, b = 0x100ecdc80, c = 0x206be8d01
2: a[0] = 200, a[1] = 101, a[2] = 102, a[3] = 103
3: a[0] = 200, a[1] = 300, a[2] = 301, a[3] = 302
4: a[0] = 200, a[1] = 400, a[2] = 301, a[3] = 302
5: a[0] = 200, a[1] = 128144, a[2] = 256, a[3] = 302
6: a = 0x16f7369a8, b = 0x16f7369ac, c = 0x16f7369a9
```

### Explanation of Results

1.  **Line 1 (Pointers)**:
    -   `a`: Address of the array on the stack.
    -   `b`: Address of memory allocated on the heap via `malloc`.
    -   `c`: Uninitialized local variable, contains a random garbage value.

2.  **Line 2 (Initialization)**:
    -   `c = a;` sets `c` to point to the start of `a`.
    -   Loop initializes `a[i] = 100 + i`.
    -   `c[0] = 200` overwrites `a[0]`.

3.  **Line 3 (Pointer Indexing)**:
    -   `c[1] = 300;` sets `a[1]`.
    -   `*(c + 2) = 301;` is equivalent to `c[2] = 301`, sets `a[2]`.
    -   `3[c] = 302;` is equivalent to `c[3] = 302` (commutative property of addition `*(3 + c)`), sets `a[3]`.

4.  **Line 4 (Pointer Arithmetic)**:
    -   `c = c + 1;` increments the pointer `c` by `sizeof(int)` (4 bytes). `c` now points to `a[1]`.
    -   `*c = 400;` overwrites `a[1]` with 400.

5.  **Line 5 (Unaligned Access / "Corruption")**:
    -   `c = (int *) ((char *) c + 1);`:
        -   Casts `c` (pointing to `a[1]`) to `char*`.
        -   Increments by 1 byte.
        -   Casts back to `int*`. `c` now points to the address `timestamp(a[1]) + 1`.
    -   `*c = 500;` writes the 4-byte integer `500` (hex `0x000001F4`) to this unaligned address.
    -   **Memory Layout (Little Endian)** before write:
        -   `a[1]` (400 = `0x190`): `90 01 00 00`
        -   `a[2]` (301 = `0x12D`): `2D 01 00 00`
    -   **The Write**:
        -   Writes `F4 01 00 00` starting at `offset(a[1]) + 1`.
        -   `a[1]` byte 1 becomes `F4`.
        -   `a[1]` byte 2 becomes `01`.
        -   `a[1]` byte 3 becomes `00`.
        -   `a[2]` byte 0 becomes `00`.
    -   **Result**:
        -   `a[1]` becomes `90 F4 01 00` -> `0x0001F490` = **128144**.
        -   `a[2]` becomes `00 01 00 00` -> `0x00000100` = **256**.
    -   This explains the "corrupted" values; they are the result of overwriting overlapping bytes of two adjacent integers.

6.  **Line 6 (Pointer vs Byte Arithmetic)**:
    -   `b = (int *) a + 1;`: Adds `1 * sizeof(int)` (4 bytes) to address of `a`. `0x...a8` + 4 = `0x...ac`.
    -   `c = (int *) ((char *) a + 1);`: Adds `1 * sizeof(char)` (1 byte) to address of `a`. `0x...a8` + 1 = `0x...a9`.
