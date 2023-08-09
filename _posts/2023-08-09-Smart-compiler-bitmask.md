Today, while using Compiler Explorer to dive deep into a C++ snippet, I noticed an intriguing optimization. The line in question was a simple conditional statement checking if a char was one of the 4 DNA base letters, A, C, G, or T:

```cpp
if (c != 'A' && c != 'C' && c != 'G' && c != 'T') continue;
```

This ended up compiling down to the following assembly:

```nasm
mov r12d, 524357
; ... 
movzx eax, byte ptr [rsp + 7] ; moves the current char, c into eax
lea ecx, [rax - 65]           ; Maps c s.t. A = 0, B = 1, C = 2, …
cmp cl, 19                    ; Checks if c < 19
ja .LBB0_1                    ; if so, continues by jumping to LBB0_1
movzx ecx, cl                 ; zeroes upper bits of ecx (cl is low byte of ecx)
bt r12, rcx                   ; Bit Test - see below
jae .LBB0_1                   ; jumps based on result of bt
```

The really interesting part happens with the r12 register and the bt (Bit Test) instruction. Here’s what the constant 524357 looks like in binary:

```
10000001000100101
```

So here we see that the 1st, 3rd, 7th, and 20th bits are set. Recall that we’re looking for A, C, G, and T and have subtracted 65 from their ASCII representation; now A=0, C=2, G=6, and T=19. The Bit Test instruction checks the r12 bit specified by the value in the rcx register. The r12 register holds this bit mask and the rcx register holds our current char value, and by this point we’re guaranteed it’s 19 or less.

In essence, the compiler has compiled our manual check of 4 characters into an uber-efficient bit mask! We can check 4 values for the price of a single subtraction and a bitmask check. Pretty slick.
