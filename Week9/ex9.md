# Counting bits
Algorithms are from https://graphics.stanford.edu/~seander/bithacks.html

Compiler exploration using https://godbolt.org/ with ARM gcc 11.2 (linux).

## Counting bits set (naive way)
https://godbolt.org/z/41Gz8Kn1b
```
int count_bits_naive(unsigned int v) {
    unsigned int c; // c accumulates the total bits set in v

    for (c = 0; v; v >>= 1)
    {
      c += v & 1;
    }

    return c;
}
```

```
count_bits_naive:
        push    {r7}
        sub     sp, sp, #20
        add     r7, sp, #0
        str     r0, [r7, #4]
        movs    r3, #0
        str     r3, [r7, #12]
        b       .L2
.L3:
        ldr     r3, [r7, #4]
        and     r3, r3, #1
        ldr     r2, [r7, #12]
        add     r3, r3, r2
        str     r3, [r7, #12]
        ldr     r3, [r7, #4]
        lsrs    r3, r3, #1
        str     r3, [r7, #4]
.L2:
        ldr     r3, [r7, #4]
        cmp     r3, #0
        bne     .L3
        ldr     r3, [r7, #12]
        mov     r0, r3
        adds    r7, r7, #20
        mov     sp, r7
        ldr     r7, [sp], #4
        bx      lr
```