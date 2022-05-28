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

## Counting bits set, Brian Kernighan's way
https://godbolt.org/z/e46cqjW38

```
int count_bits_kernighan(unsigned int v) {
    unsigned int c; // c accumulates the total bits set in v
    for (c = 0; v; c++)
    {
    	v &= v - 1; // clear the least significant bit set
    }
}
```

```
count_bits_kernighan:
        push    {r7}
        sub     sp, sp, #20
        add     r7, sp, #0
        str     r0, [r7, #4]
        movs    r3, #0
        str     r3, [r7, #12]
        b       .L2
.L3:
        ldr     r3, [r7, #4]
        subs    r3, r3, #1
        ldr     r2, [r7, #4]
        ands    r3, r3, r2
        str     r3, [r7, #4]
        ldr     r3, [r7, #12]
        adds    r3, r3, #1
        str     r3, [r7, #12]
.L2:
        ldr     r3, [r7, #4]
        cmp     r3, #0
        bne     .L3
        nop
        mov     r0, r3
        adds    r7, r7, #20
        mov     sp, r7
        ldr     r7, [sp], #4
        bx      lr
```


##  Counting bits set by lookup table 

https://godbolt.org/z/j7MY47rjs

```
#define B2(n) n,     n+1,     n+1,     n+2
#define B4(n) B2(n), B2(n+1), B2(n+1), B2(n+2)
#define B6(n) B4(n), B4(n+1), B4(n+1), B4(n+2)

int count_bits_lookup_table(unsigned int v) {
    static const unsigned char BitsSetTable256[256] = 
    {
    	B6(0), B6(1), B6(1), B6(2)
    };

    unsigned int c; // c is the total bits set in v

    // Option 1:
    c = BitsSetTable256[v & 0xff] + 
        BitsSetTable256[(v >> 8) & 0xff] + 
        BitsSetTable256[(v >> 16) & 0xff] + 
        BitsSetTable256[v >> 24]; 
    
    return c;
}
```

```
count_bits_lookup_table:
        push    {r7}
        sub     sp, sp, #20
        add     r7, sp, #0
        str     r0, [r7, #4]
        ldr     r3, [r7, #4]
        uxtb    r2, r3
        movw    r3, #:lower16:BitsSetTable256.0
        movt    r3, #:upper16:BitsSetTable256.0
        ldrb    r3, [r3, r2]    @ zero_extendqisi2
        mov     r1, r3
        ldr     r3, [r7, #4]
        lsrs    r3, r3, #8
        uxtb    r2, r3
        movw    r3, #:lower16:BitsSetTable256.0
        movt    r3, #:upper16:BitsSetTable256.0
        ldrb    r3, [r3, r2]    @ zero_extendqisi2
        adds    r2, r1, r3
        ldr     r3, [r7, #4]
        lsrs    r3, r3, #16
        uxtb    r1, r3
        movw    r3, #:lower16:BitsSetTable256.0
        movt    r3, #:upper16:BitsSetTable256.0
        ldrb    r3, [r3, r1]    @ zero_extendqisi2
        add     r2, r2, r3
        ldr     r3, [r7, #4]
        lsrs    r1, r3, #24
        movw    r3, #:lower16:BitsSetTable256.0
        movt    r3, #:upper16:BitsSetTable256.0
        ldrb    r3, [r3, r1]    @ zero_extendqisi2
        add     r3, r3, r2
        str     r3, [r7, #12]
        ldr     r3, [r7, #12]
        mov     r0, r3
        adds    r7, r7, #20
        mov     sp, r7
        ldr     r7, [sp], #4
        bx      lr
BitsSetTable256.0:
        .ascii  "\000\001\001\002\001\002\002\003\001\002\002\003\002"
        .ascii  "\003\003\004\001\002\002\003\002\003\003\004\002\003"
        .ascii  "\003\004\003\004\004\005\001\002\002\003\002\003\003"
        .ascii  "\004\002\003\003\004\003\004\004\005\002\003\003\004"
        .ascii  "\003\004\004\005\003\004\004\005\004\005\005\006\001"
        .ascii  "\002\002\003\002\003\003\004\002\003\003\004\003\004"
        .ascii  "\004\005\002\003\003\004\003\004\004\005\003\004\004"
        .ascii  "\005\004\005\005\006\002\003\003\004\003\004\004\005"
        .ascii  "\003\004\004\005\004\005\005\006\003\004\004\005\004"
        .ascii  "\005\005\006\004\005\005\006\005\006\006\007\001\002"
        .ascii  "\002\003\002\003\003\004\002\003\003\004\003\004\004"
        .ascii  "\005\002\003\003\004\003\004\004\005\003\004\004\005"
        .ascii  "\004\005\005\006\002\003\003\004\003\004\004\005\003"
        .ascii  "\004\004\005\004\005\005\006\003\004\004\005\004\005"
        .ascii  "\005\006\004\005\005\006\005\006\006\007\002\003\003"
        .ascii  "\004\003\004\004\005\003\004\004\005\004\005\005\006"
        .ascii  "\003\004\004\005\004\005\005\006\004\005\005\006\005"
        .ascii  "\006\006\007\003\004\004\005\004\005\005\006\004\005"
        .ascii  "\005\006\005\006\006\007\004\005\005\006\005\006\006"
        .ascii  "\007\005\006\006\007\006\007\007\010"
```

## Analysis

The naive implementation is very simple to understand and maintain, however, it's the least efficient in terms of computation since the number of loops is determined by the highest order bit which is 1. If only the msb is set in a 32-bit word, it will loop 32 times.

Brian Kernighan's algorithm is more efficient since it always clears the least significant bit set.  If only the msb is set in a 32-bit word, it will loop only once.  The downside is that the algorithm is more difficult to understand and possibly maintain

Finally lookup table is on average the most efficient as it always execute in constant time no matter the number of bits set. There are no loops in the implementation. The downside is that the lookup table consumes memory.  However, if this operation is done frequently it may be worth the tradeoff.

Overall: Choose lookup table if extra memory is available and constant performance is desired.  If memory is a premium, Brian Kernighan's algorithm is the next best choice.

