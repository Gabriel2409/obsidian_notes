#sd

Example signed integers in Rust: i32
Example unsigned integers in Rust: u32

Range unsigned integers: $0$ to $2^n -1$
Range signed integers (usually): $-2^{n-1}$ to $2^{n-1} -1$

In Rust, two complement notations with leftmost bit = sign bit

0000 => 0
0111 => 7
1111 => -1
1000 => -8

To find complement value of nb when there is a negative sign, invert bits and add 1
1011 => invert: 0100 = 4, add 1: 5 therefore 1011 is -5
in bits: 1011 => 0100 => 0101

Note: does not work for 1000 as you can't represent the positive equivalent on 4 bits

```

```
