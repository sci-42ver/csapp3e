# miscs
- hwo to [post](https://stackoverflow.com/help/minimal-reproducible-example) question
# TODO
- `.-multstore` meaning
- [ROP](https://tc.gts3.org/cs6265/tut/tut06-01-rop.html)
  - [endbr64](https://stackoverflow.com/questions/56905811/what-does-the-endbr64-instruction-actually-do) with `endbr64` <a id="rop"></a>
- `pwndbg` '<>' label in 'DISASM' meaning (in doc / code)
```bash
────────────────────────────────────────────────────────────────[ DISASM / x86-64 / set emulate on ]─────────────────────────────────────────────────────────────────
  0x401040<printf@plt>      jmp    qwordptr[rip+0x2fc2]          <printf@got[plt]>
 
 ►0x401046<printf@plt+6>    push   1
  0x40104b<printf@plt+11>   jmp    0x401020                      <0x401020>
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
pwndbg> x $rip+0x2fc2
0x404008 <printf@got.plt>:      0x00401046
pwndbg> got
GOT protection: Partial RELRO | GOT functions: 2
[0x404000] __stack_chk_fail@GLIBC_2.4 -> 0x401036 (__stack_chk_fail@plt+6) ◂— push 0 /* 'h' */
[0x404008] printf@GLIBC_2.2.5 -> 0x401046 (printf@plt+6) ◂— push 1
```
- [segment descriptor](https://www.felixcloutier.com/x86/push.html) / [this](https://stackoverflow.com/questions/51322139/what-does-the-d-flag-in-the-code-segment-descriptor-do-for-x86-64-instructions)
  - in [pdf p3095](../references/intel_64.pdf) 
  - [unable](https://stackoverflow.com/questions/28263624/can-i-print-the-gdtr-and-gdt-descriptor-under-gdb) to view (GDTR, LDTR, IDTR, and TR) reg, same with segment descriptor in GDT, also [CR](https://stackoverflow.com/questions/7415515/how-to-access-the-control-registers-cr0-cr2-cr3-from-a-program-getting-segmenta) reg like CR0 except in ring 0 (kerenel code) 
    - [workaround](https://forum.osdev.org/viewtopic.php?f=1&t=25968) in emulator like qemu
  - but [this](https://wiki.osdev.org/Global_Descriptor_Table#Segment_Descriptor) says to ignore Segment Descriptor ‘In 64-bit mode’.
- `GOT` more [specific](http://www.infosecwriters.com/text_resources/pdf/GOT_Hijack.pdf) than '## 8' below.
- [DIY](https://www.righto.com/2016/06/y-combinators-xerox-alto-restoring.html) old computer
## [intel reference](../references/intel_64.pdf)
- p597 `Op/En` in 'Opcode Column in the Instruction Summary Table (Instructions without VEX Prefix)'
- p529 ModR/M table
- 533 REX meaning
## [crackme](https://reverseengineering.stackexchange.com/questions/15774/crack-me-material) from [this](http://archive.hack.lu/2015/radare2-workshop-slides.pdf)
- [book](https://github.com/mikesiko/PracticalMalwareAnalysis-Labs) or [r2libc](https://cheese-hub.github.io/secure-coding/05-ret2libc/index.html)
## ret2 series
- [plt](https://ir0nstone.gitbook.io/notes/types/stack/aslr/plt_and_got) , this blog also tells what to do if got has nothing, [see](#rela)
### miscs
- [1](https://sharkmoos.medium.com/binary-exploitation-exploiting-ret2libc-328eefb0421b)
## blogs
- [1](https://reverseengineering.stackexchange.com/questions/1992/what-is-plt-got)
- [elf](https://stackoverflow.com/questions/43048932/why-does-the-plt-exist-in-addition-to-the-got-instead-of-just-using-the-got) and shared library
## weird problems
- [1 (Viewed 10 times when 23/4/17)](https://stackoverflow.com/questions/76033107/why-same-instruction-have-different-result-with-same-input-elags-regs-etc)
## miscs
- how to avoid gcc omit substraction or not generate weird 'cmp,jcc' pair
```c++
$ cat seta_setg.c
// Build with: gcc -m32 -no-pie -g -o plt plt.c
#include <stdio.h>

typedef unsigned int src_t ;
typedef int dest_t;

//3
//typedef unsigned char src_t ;
//typedef long dest_t;

//4
//typedef int src_t ;
//typedef char dest_t;

int main() {
        src_t test_src=-1;
        dest_t test_dest=-1;
        long sub;
        sub = -2-(-1);
        sub = -2-(unsigned int)(-1);
        if(sub<-1){
                putchar(1);
        }
        if(test_src>=(unsigned int)-1){
                //test_src=2;
                printf("1");
        }
        if(test_dest>-1){
                test_dest=2;
        }
}

// https://stackoverflow.com/questions/38552116/how-to-remove-noise-from-gcc-clang-assembly-output
$ gcc  seta_setg.c -S seta_setg.s -fno-asynchronous-unwind-tables
$ cat seta_setg.s                                                                           
        .file   "seta_setg.c"
        .text
        .globl  main
        .type   main, @function
main:
        pushq   %rbp
        movq    %rsp, %rbp
        subq    $16, %rsp
        movl    $-1, -16(%rbp)
        movl    $-1, -12(%rbp)
        movq    $-1, -8(%rbp)
        // here omit '-2-(unsigned int)(-1)'
        movl    $4294967295, %eax
        movq    %rax, -8(%rbp)
        cmpq    $-1, -8(%rbp)
        jge     .L2
        movl    $1, %edi
        call    putchar@PLT
.L2:
        cmpl    $-1, -16(%rbp)
        // weird jne, I use >= in .c
        jne     .L3
        movl    $49, %edi
        call    putchar@PLT
.L3:
        cmpl    $0, -12(%rbp)
        js      .L4
        movl    $2, -12(%rbp)
.L4:
        movl    $0, %eax
        leave
        ret
        .size   main, .-main
        .ident  "GCC: (GNU) 12.2.1 20230201"
        .section        .note.GNU-stack,"",@progbits
```
### [debug](http://www.ece.uah.edu/~milenka/docs/milenkovic_WDDD02.pdf) BPT(branch predictor table)
## computer basics
- stack always [multiple of eight](https://stackoverflow.com/questions/66816732/memory-allocation-in-the-power-of-two-vs-multiple-of-eight)
# resources
## categories
- [registers](https://en.wikibooks.org/wiki/X86_Assembly/X86_Architecture) [or](https://wiki.osdev.org/CPU_Registers_x86-64) 
- [online opcode](https://www.felixcloutier.com/x86/push.html) or [pdf p615](../references/intel_64.pdf) or [this](http://ref.x86asm.net/coder64.html)
- how to [interpret](https://stackoverflow.com/questions/28664856/how-to-interpret-x86-opcode-map) opcode
- gdb [rc tutorial](https://sourceware.org/gdb/onlinedocs/gdb/Process-Record-and-Replay.html) or [target](https://sourceware.org/gdb/onlinedocs/gdb/Target-Commands.html#Target-Commands) in [this](https://sourceware.org/gdb/onlinedocs/gdb/index.html#SEC_Contents)
  - [this is out-dated](https://sourceware.org/gdb/wiki/ProcessRecord)
- [wiki](https://stackoverflow.com/tags/sse/info) in stackoverflow
## miscs
- `as` used by `gcc` assembly code comment [definition](https://sourceware.org/binutils/docs/as/index.html#SEC_Contents), also [see](https://stackoverflow.com/questions/24787769/what-are-lfb-lbb-lbe-lvl-loc-in-the-compiler-generated-assembly-code) [source](https://github.com/gcc-mirror/gcc/blob/releases/gcc-4.8.2/gcc/dwarf2out.c) code
  - [`L..`](https://slowstarter80.github.io/gcc/labels_in_assembly_code/) like [`.LFB0`](https://sourceware.org/binutils/docs/as/Symbol-Names.html)
- [.cfi_def_cfa_offset](https://sourceware.org/binutils/docs/as/CFI-directives.html#g_t_002ecfi_005fdef_005fcfa_005foffset-offset)
- [pushq](https://stackoverflow.com/questions/48251829/whats-the-difference-between-push-and-pushq-in-att-assembly)
  - [opcode p96 & 327](../references/amd64.pdf)
- [r/m](https://cs.fit.edu/~mmahoney/cse3101/nasmdocb.html) from [this](https://stackoverflow.com/questions/22445347/how-do-i-interpret-this-instruction)
- [CS.D](https://stackoverflow.com/questions/11970750/rex-prefix-decoding-for-intel-x64-arch)
- callee and caller saved [regs](https://stackoverflow.com/questions/9268586/what-are-callee-and-caller-saved-registers)
- [N.E.](https://stackoverflow.com/questions/71753270/intel-reference-specification-questions#comment126817567_71753270) in [pdf p605](../references/intel_64.pdf) means x86 instruction **not encodable** in 64-bit OS.(po)
- [IA-32e](https://news.ycombinator.com/item?id=22280184)
- [RTLD](https://nxmnpg.lemoda.net/1/rtld) -> RTLD_GLOBAL
- [fs:[0x18]](https://rvsec0n.wordpress.com/2019/09/13/routines-utilizing-tebs-and-pebs/)
- [_rtld_global_ro](https://comp.os.linux.development.apps.narkive.com/sVzevUx4/rtld-global-ro-glibc-private) related with ld,'ro' maybe 'readonly' 
- [fast code](http://www.c-jump.com/CIS77/ASM/Instructions/I77_0080_fast_code.htm)
- [_ITM_deregisterTMCloneTable](https://gcc.gnu.org/onlinedocs/gcc-7.1.0/libitm/The-libitm-ABI.html) [seems](https://gcc.gnu.org/onlinedocs/libitm/Enabling-libitm.html#Enabling-libitm) used in c++ not c
  - [Transactional Memory](https://www.ibm.com/docs/en/xffbg/121.141?topic=fortran-transactional-memory)
```bash
# https://stackoverflow.com/questions/6045809/link-error-undefined-reference-to-gxx-personality-v0-and-g
$ gcc -fPIE -no-pie -fgnu-tm test_got.c -o test_got -lstdc++
# or g++ https://stackoverflow.com/questions/329059/what-is-gxx-personality-v0-for
$ g++ -fPIE -no-pie -fgnu-tm test_got.c -o test_got
```
  - [TODO](https://stackoverflow.com/questions/41274482/why-does-register-tm-clones-and-deregister-tm-clones-reference-an-address-past-t)
- [paddr](https://reverseengineering.stackexchange.com/questions/19782/what-does-paddr-baddr-laddr-haddr-and-hvaddr-refer-to)
- why segment reg [not](https://stackoverflow.com/questions/21165678/why-64-bit-mode-long-mode-doesnt-use-segment-registers) used in 64, because 'When your normal addressing modes with normal 64-bit registers can already address vastly more memory than your computer can contain...'
### cpu specific
- my cpu 4800h doesn‘t [support](https://gist.github.com/kohnakagawa/fb77904fcc44fc5652ef6d338c35a718) 'CET'， so `ENDBR64` does [nothing](https://vstinner.readthedocs.io/assembly_x86.html), see [pdf p413 INDIRECT BRANCH TRACKING](../references/intel_64.pdf)
  - from [this](https://stackoverflow.com/questions/56905811/what-does-the-endbr64-instruction-actually-do), some keywords are ‘control flow violations’， ‘WAIT_FOR_ENDBRANCH’， ‘state machine’
  > from [this](https://en.wikipedia.org/wiki/Return-oriented_programming) , [above](#rop) maybe related with return address overwriting， so related with instructions like `jmp` -> related with `ENDBR64`.
  - TODO see kernel code related with above gist.
- [cache,memory,register](https://www.geeksforgeeks.org/difference-between-cache-memory-and-register/)
  - why no [two](https://www.quora.com/Why-can%E2%80%99t-two-operands-both-be-memory-operands-in-assembly-language) memory operand (related with cache) [also](https://stackoverflow.com/questions/14510280/why-cant-mov-have-both-operands-as-memory-locations)
## registers
- r9 save parameter and [rax](https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/x64-architecture) save return [pdf p7](../references/x64_cheatsheet.pdf) [save order](http://6.s081.scripts.mit.edu/sp18/x86-64-architecture-guide.html)
- whether [callee pdf p26](../references/abi.pdf)
### [r9,etc Caller-saved registers](https://stackoverflow.com/questions/9268586/what-are-callee-and-caller-saved-registers)
- according to [this](https://web.stanford.edu/class/archive/cs/cs107/cs107.1174/guide_x86-64.html), although it said `Caller-saved`， but it doesn't mean **must to save**
  - more [intuitive p4 and in p6 caller and callee save can both exist with the same call ( see below "'r12-15' is used ( except r14-15 )to save caller")](https://courses.cs.washington.edu/courses/cse410/17wi/lectures/CSE410-L13-procedures-II_17wi.pdf)
    - p13 said exactly Caller-save may be not needed
    - p12 why caller/ee save
      - po here caller not use but callee use register, caller‐save is better(then the reg can be tweaked all as what callee want and after ret, caller would restore all)
    - p23,24 show `rbp` `rsp` relation
      - also [csapp](http://csapp.cs.cmu.edu/3e/waside/waside-ia32.pdf), showed condition with rsp or not.
```bash
# here only have `mov` to pass parameter, but no `push` because `main` don't use these regs like `esi` anymore. `push   rbp` is generated by `-pg`
pwndbg> dr
Dump of assembler code for function main:
   0x000000000040124e <+0>:     55                      push   rbp
   0x000000000040124f <+1>:     48 89 e5                mov    rbp,rsp
   0x0000000000401252 <+4>:     e8 f9 fd ff ff          call   0x401050 <mcount@plt>
   0x0000000000401257 <+9>:     b9 04 00 00 00          mov    ecx,0x4
   0x000000000040125c <+14>:    ba 03 00 00 00          mov    edx,0x3
   0x0000000000401261 <+19>:    be 02 00 00 00          mov    esi,0x2
   0x0000000000401266 <+24>:    bf 08 20 40 00          mov    edi,0x402008
   0x000000000040126b <+29>:    b8 00 00 00 00          mov    eax,0x0
   0x0000000000401270 <+34>:    e8 cb fd ff ff          call   0x401040 <printf@plt>
=> 0x0000000000401275 <+39>:    b8 00 00 00 00          mov    eax,0x0
   0x000000000040127a <+44>:    e8 17 ff ff ff          call   0x401196 <cpu_supports_cet_shadow_stack>
   0x000000000040127f <+49>:    85 c0                   test   eax,eax
   0x0000000000401281 <+51>:    74 0a                   je     0x40128d <main+63>
   0x0000000000401283 <+53>:    bf 2e 20 40 00          mov    edi,0x40202e
   0x0000000000401288 <+58>:    e8 a3 fd ff ff          call   0x401030 <puts@plt>
   0x000000000040128d <+63>:    b8 00 00 00 00          mov    eax,0x0
   0x0000000000401292 <+68>:    e8 5b ff ff ff          call   0x4011f2 <cpu_supports_cet_ibt>
   0x0000000000401297 <+73>:    85 c0                   test   eax,eax
   0x0000000000401299 <+75>:    74 0a                   je     0x4012a5 <main+87>
   0x000000000040129b <+77>:    bf 4c 20 40 00          mov    edi,0x40204c
   0x00000000004012a0 <+82>:    e8 8b fd ff ff          call   0x401030 <puts@plt>
   0x00000000004012a5 <+87>:    b8 00 00 00 00          mov    eax,0x0
   0x00000000004012aa <+92>:    5d                      pop    rbp
   0x00000000004012ab <+93>:    c3                      ret
pwndbg> l
14          return (edx & (1 << 20)) != 0;
15      }
16
17      int main() {
18              printf("test Caller-saved registers %d %d %d ",2,3,4);
pwndbg> bt
#0  main () at check_cet_supported.c:19
```
### mcount
```bash
$ gcc check_cet_supported.c -o check_cet_supported.out -g -no-pie -fno-pic -p -pg
$ cd /mnt/ubuntu/home/czg/csapp3e/asm;gdb ./check_cet_supported.out -ex 'start' -ex 'set record full stop-at-limit off' -nx -iex "source ~/.gdbinit_pwndbg_voltron.gdb" -ex 'wa $rdi' -ex 'target record-full'
...
0x40118f<cpu_supports_cet_shadow_stack+9>    call   mcount@plt                      <mcount@plt>
        rdi: 0x1
        rsi: 0x7fffffffe018 —▸ 0x7fffffffe480 ◂— '/mnt/ubuntu/home/czg/csapp3e/asm/check_cet_supported.out'
        rdx: 0x7fffffffe028 —▸ 0x7fffffffe4b9 ◂— 'BEMENU_BACKEND=wayland'
        rcx: 0x403de0 —▸ 0x401150 ◂— endbr64
```
  - here `mcount` is [from](https://ftp.gnu.org/old-gnu/Manuals/gprof-2.9.1/html_node/gprof_25.html) `-pg`
```bash
# mcount save program state as above said (maybe mainly because it call another func)
pwndbg> nearpc $pc 0x8
  0x7ffff7fdcb5c  sub    rsp,qwordptr[rip+0x2010d]     <_rtld_global_ro+432>
  0x7ffff7fdcb63  mov    qwordptr[rsp],rax
  0x7ffff7fdcb67  mov    qwordptr[rsp+8],rcx
  0x7ffff7fdcb6c  mov    qwordptr[rsp+0x10],rdx
  0x7ffff7fdcb71  mov    qwordptr[rsp+0x18],rsi
  0x7ffff7fdcb76  mov    qwordptr[rsp+0x20],rdi
  0x7ffff7fdcb7b  mov    qwordptr[rsp+0x28],r8
  0x7ffff7fdcb80  mov    qwordptr[rsp+0x30],r9
pwndbg> bt
#0  0x00007ffff7fdcb85 in ?? () from /lib64/ld-linux-x86-64.so.2
#1  0x0000000000401247 in main () at check_cet_supported.c:17
#2  0x00007ffff7d9b790 in ?? () from /usr/lib/libc.so.6
#3  0x00007ffff7d9b84a in __libc_start_main () from /usr/lib/libc.so.6
#4  0x0000000000401075 in _start ()
pwndbg> d- 0x0000000000401247
Dump of assembler code from 0x401242 to 0x40124c:
   0x0000000000401242 <main+4>: e8 f9 fd ff ff          call   0x401040 <mcount@plt>
   0x0000000000401247 <main+9>: b8 00 00 00 00          mov    eax,0x0
End of assembler dump.

...

pwndbg> d-- 0x7ffff7fdcbc9
Dump of assembler code from 0x7ffff7fdcb94 to 0x7ffff7fdcbfe:
   0x00007ffff7fdcb94:  48 89 94 24 58 02 00 00 mov    QWORD PTR [rsp+0x258],rdx
   0x00007ffff7fdcb9c:  48 89 94 24 60 02 00 00 mov    QWORD PTR [rsp+0x260],rdx
   0x00007ffff7fdcba4:  48 89 94 24 68 02 00 00 mov    QWORD PTR [rsp+0x268],rdx
   0x00007ffff7fdcbac:  48 89 94 24 70 02 00 00 mov    QWORD PTR [rsp+0x270],rdx
   0x00007ffff7fdcbb4:  48 89 94 24 78 02 00 00 mov    QWORD PTR [rsp+0x278],rdx
   0x00007ffff7fdcbbc:  0f c7 64 24 40          xsavec [rsp+0x40]
   0x00007ffff7fdcbc1:  48 8b 73 10             mov    rsi,QWORD PTR [rbx+0x10]
   0x00007ffff7fdcbc5:  48 8b 7b 08             mov    rdi,QWORD PTR [rbx+0x8]
=> 0x00007ffff7fdcbc9:  e8 12 db ff ff          call   0x7ffff7fda6e0
...

# obvious here, it is the caller 'mcount' to save and restore regs like `r9` bsed on rsp location
pwndbg> nearpc 0x7ffff7e831da -e
  0x7ffff7e831cf              mov    qwordptr[rip+0xcd086],0
 ►0x7ffff7e831da              ret                                  <0x7ffff7e83c88; mcount+56>
    ↓
  0x7ffff7e83c88<mcount+56>   mov    r9,qwordptr[rsp+0x30]
  0x7ffff7e83c8d<mcount+61>   mov    r8,qwordptr[rsp+0x28]
  0x7ffff7e83c92<mcount+66>   mov    rdi,qwordptr[rsp+0x20]
  0x7ffff7e83c97<mcount+71>   mov    rsi,qwordptr[rsp+0x18]
  0x7ffff7e83c9c<mcount+76>   mov    rdx,qwordptr[rsp+0x10]
  0x7ffff7e83ca1<mcount+81>   mov    rcx,qwordptr[rsp+8]
  0x7ffff7e83ca6<mcount+86>   mov    rax,qwordptr[rsp]
  0x7ffff7e83caa<mcount+90>   add    rsp,0x38
  0x7ffff7e83cae<mcount+94>   ret    
pwndbg> bt
#0  0x00007ffff7e831da in ?? () from /usr/lib/libc.so.6
#1  0x00007ffff7e83c88 in mcount () from /usr/lib/libc.so.6
...
```
### callee reg (continue after above)
```bash
$ cd /mnt/ubuntu/home/czg/csapp3e/asm;gdb ./check_cet_supported.out -ex 'br *40123e' -ex 'r' -ex 'set record full stop-at-limit off' -nx -iex "source ~/.gdbinit_pwndbg_voltron.gdb" -ex '' -ex 'br *401242' -ex 'c' -ex 'wa $rbp' -ex 'wa $rsp' -ex 'awatch $rsp' -ex 'target record-full' -ex "wm rsp"
pwndbg> d-- 0x7ffff7fda6ed
   0x00007ffff7fda6e4:  41 55                   push   r13
   0x00007ffff7fda6e6:  45 31 c9                xor    r9d,r9d
   0x00007ffff7fda6e9:  41 54                   push   r12
   0x00007ffff7fda6eb:  55                      push   rbp
   0x00007ffff7fda6ec:  53                      push   rbx
=> 0x00007ffff7fda6ed:  48 89 fb                mov    rbx,rdi
# after 4 `push`
──────────────────────────────────────────────────────────────────────────────────────[ STACK ]──────────────────────────────────────────────────────────────────────────────────────
00:0000│ rsp 0x7fffffffdb18 —▸ 0x7fffffffdee0 —▸ 0x7fffffffe018 —▸ 0x7fffffffe480 ◂— '/mnt/ubuntu/home/czg/csapp3e/asm/check_cet_supported.out'
01:0008│     0x7fffffffdb20 —▸ 0x7fffffffdf00 ◂— 0x1
02:0010│     0x7fffffffdb28 ◂— 0x0
03:0018│     0x7fffffffdb30 —▸ 0x7fffffffe028 —▸ 0x7fffffffe4b9 ◂— 'BEMENU_BACKEND=wayland'
...
# then spare space
 ►0x7ffff7fda6f0  sub    rsp,0x18
──────────────────────────────────────────────────────────────────────────────────────[ STACK ]──────────────────────────────────────────────────────────────────────────────────────
00:0000│ rsp 0x7fffffffdb00 —▸ 0x7fffffffdc40 —▸ 0x7ffff7e744e5 (sbrk+149) ◂— test eax,eax
01:0008│     0x7fffffffdb08 —▸ 0x7ffff7fca9d8 ◂— 0x10001100000243
02:0010│     0x7fffffffdb10 ◂— 0x6
03:0018│     0x7fffffffdb18 —▸ 0x7fffffffdee0 —▸ 0x7fffffffe018 —▸ 0x7fffffffe480 ◂— '/mnt/ubuntu/home/czg/csapp3e/asm/check_cet_supported.out'
pwndbg> wm rsp
Watchpoint 19: *(long long *)$rsp
# do anything subroutine want to do, also push/change stack 
 ►0x7ffff7fda74e  mov    qwordptr[rsp],rax
──────────────────────────────────────────────────────────────────────────────────────[ STACK ]──────────────────────────────────────────────────────────────────────────────────────
00:0000│ rsi rsp 0x7fffffffdb00 —▸ 0x400470 ◂— 0x120000002f /* '/' */
01:0008│         0x7fffffffdb08 —▸ 0x7ffff7fca9d8 ◂— 0x10001100000243
02:0010│         0x7fffffffdb10 ◂— 0x6
03:0018│         0x7fffffffdb18 —▸ 0x7fffffffdee0 —▸ 0x7fffffffe018 —▸ 0x7fffffffe480 ◂— '/mnt/ubuntu/home/czg/csapp3e/asm/check_cet_supported.out'
04:0020│         0x7fffffffdb20 —▸ 0x7fffffffdf00 ◂— 0x1
05:0028│         0x7fffffffdb28 ◂— 0x0
06:0030│         0x7fffffffdb30 —▸ 0x7fffffffe028 —▸ 0x7fffffffe4b9 ◂— 'BEMENU_BACKEND=wayland'
07:0038│         0x7fffffffdb38 —▸ 0x7ffff7fdcbce ◂— mov r11,rax
────────────────────────────────────────────────────────────────────────[ DISASM / x86-64 / set emulate on ]─────────────────────────────────────────────────────────────────────────
  0x7ffff7fda7c9  mov    eax,dwordptr[rax]
  ►0x7ffff7fda752  cmp    r8d,7

# 'rdx' caller reg pushed(saved) by caller '0x7ffff7fda7dd'
pwndbg> d- 0x7ffff7fda7ce
Dump of assembler code from 0x7ffff7fda7c9 to 0x7ffff7fda7d3:
   0x00007ffff7fda7c9:  8b 00                   mov    eax,DWORD PTR [rax]
   0x00007ffff7fda7cb:  6a 00                   push   0x0
   0x00007ffff7fda7cd:  52                      push   rdx
=> 0x00007ffff7fda7ce:  41 b9 01 00 00 00       mov    r9d,0x1
──────────────────────────────────────────────────────────────────────────────────────[ STACK ]──────────────────────────────────────────────────────────────────────────────────────
00:0000│ rsp 0x7fffffffdaf0 ◂— 0x1
01:0008│     0x7fffffffdaf8 ◂— 0x0
02:0010│ rsi 0x7fffffffdb00 —▸ 0x400470 ◂— 0x120000002f /* '/' */
03:0018│     0x7fffffffdb08 —▸ 0x7ffff7fca9d8 ◂— 0x10001100000243
04:0020│     0x7fffffffdb10 ◂— 0x6
05:0028│     0x7fffffffdb18 —▸ 0x7fffffffdee0 —▸ 0x7fffffffe018 —▸ 0x7fffffffe480 ◂— '/mnt/ubuntu/home/czg/csapp3e/asm/check_cet_supported.out'
06:0030│     0x7fffffffdb20 —▸ 0x7fffffffdf00 ◂— 0x1
07:0038│     0x7fffffffdb28 ◂— 0x0
# '0x7ffff7fda7dd' call ‘0x7ffff7fd4180’
 ►0x7ffff7fda7dd  call   0x7ffff7fd4180                <0x7ffff7fd4180>
 
  0x7ffff7fda7e2  mov    r13,rax
# 
pwndbg> si
──────────────────────────────────────────────────────────────────────────────────────[ STACK ]──────────────────────────────────────────────────────────────────────────────────────
00:0000│ rsp 0x7fffffffdae8 —▸ 0x7ffff7fda7e2 ◂— mov r13,rax
01:0008│     0x7fffffffdaf0 ◂— 0x1
02:0010│     0x7fffffffdaf8 ◂— 0x0
03:0018│ rdx 0x7fffffffdb00 —▸ 0x400470 ◂— 0x120000002f /* '/' */
04:0020│     0x7fffffffdb08 —▸ 0x7ffff7fca9d8 ◂— 0x10001100000243
05:0028│     0x7fffffffdb10 ◂— 0x6
06:0030│     0x7fffffffdb18 —▸ 0x7fffffffdee0 —▸ 0x7fffffffe018 —▸ 0x7fffffffe480 ◂— '/mnt/ubuntu/home/czg/csapp3e/asm/check_cet_supported.out'
07:0038│     0x7fffffffdb20 —▸ 0x7fffffffdf00 ◂— 0x1
────────────────────────────────────────────────────────────────────────[ DISASM / x86-64 / set emulate on ]─────────────────────────────────────────────────────────────────────────
 ►0x7ffff7fd4180  endbr64 
# then after several steps, 'r12-15' is used ( except r14-15 )to save caller reg passed by caller ‘0x7ffff7fda7dd’ after pushed by callee ‘0x7ffff7fd4180’
pwndbg> d-- 0x7ffff7fd4186
Dump of assembler code from 0x7ffff7fd4151 to 0x7ffff7fd41bb:
   0x00007ffff7fd4151:  33 00                   xor    eax,DWORD PTR [rax]
   0x00007ffff7fd4153:  00 48 8b                add    BYTE PTR [rax-0x75],cl
   0x00007ffff7fd4156:  4c 24 08                rex.WR and al,0x8
   0x00007ffff7fd4159:  e9 d2 fb ff ff          jmp    0x7ffff7fd3d30
   0x00007ffff7fd415e:  48 8b 7c 24 38          mov    rdi,QWORD PTR [rsp+0x38]
   0x00007ffff7fd4163:  ff 15 e7 88 02 00       call   QWORD PTR [rip+0x288e7]        # 0x7ffff7ffca50
   0x00007ffff7fd4169:  48 8d 3d 90 ea 01 00    lea    rdi,[rip+0x1ea90]        # 0x7ffff7ff2c00
   0x00007ffff7fd4170:  31 c0                   xor    eax,eax
   0x00007ffff7fd4172:  e8 49 36 00 00          call   0x7ffff7fd77c0 <_dl_fatal_printf>
   0x00007ffff7fd4177:  66 0f 1f 84 00 00 00 00 00      nop    WORD PTR [rax+rax*1+0x0]
   0x00007ffff7fd4180:  f3 0f 1e fa             endbr64
   0x00007ffff7fd4184:  41 57                   push   r15
=> 0x00007ffff7fd4186:  41 56                   push   r14
   0x00007ffff7fd4188:  41 55                   push   r13
   0x00007ffff7fd418a:  49 89 f5                mov    r13,rsi
   0x00007ffff7fd418d:  41 54                   push   r12
   0x00007ffff7fd418f:  4d 89 c4                mov    r12,r8
   0x00007ffff7fd4192:  55                      push   rbp
   0x00007ffff7fd4193:  48 89 d5                mov    rbp,rdx
   0x00007ffff7fd4196:  53                      push   rbx
   0x00007ffff7fd4197:  48 89 fb                mov    rbx,rdi
   0x00007ffff7fd419a:  48 81 ec 88 00 00 00    sub    rsp,0x88

# skip subsubroutine
pwndbg> dis
pwndbg> until *0x7ffff7fda7e2
# back to see subsubroutine; pop order is reverse which is fine
pwndbg> d-- 0x7ffff7fd4326
Dump of assembler code from 0x7ffff7fd42f1 to 0x7ffff7fd435b:
   0x00007ffff7fd42f1:  0f 84 99 02 00 00       je     0x7ffff7fd4590
   0x00007ffff7fd42f7:  4c 8b 54 24 50          mov    r10,QWORD PTR [rsp+0x50]
   0x00007ffff7fd42fc:  41 8b b7 fc 03 00 00    mov    esi,DWORD PTR [r15+0x3fc]
   0x00007ffff7fd4303:  85 f6                   test   esi,esi
   0x00007ffff7fd4305:  0f 84 75 02 00 00       je     0x7ffff7fd4580
   0x00007ffff7fd430b:  f6 05 ae 87 02 00 04    test   BYTE PTR [rip+0x287ae],0x4        # 0x7ffff7ffcac0 <_rtld_global_ro>
   0x00007ffff7fd4312:  0f 85 e8 01 00 00       jne    0x7ffff7fd4500
   0x00007ffff7fd4318:  4c 89 55 00             mov    QWORD PTR [rbp+0x0],r10
=> 0x00007ffff7fd431c:  48 81 c4 88 00 00 00    add    rsp,0x88
   0x00007ffff7fd4323:  4c 89 f8                mov    rax,r15
   0x00007ffff7fd4326:  5b                      pop    rbx
   0x00007ffff7fd4327:  5d                      pop    rbp
   0x00007ffff7fd4328:  41 5c                   pop    r12
   0x00007ffff7fd432a:  41 5d                   pop    r13
   0x00007ffff7fd432c:  41 5e                   pop    r14
   0x00007ffff7fd432e:  41 5f                   pop    r15
   0x00007ffff7fd4330:  c3                      ret
# 
pwndbg> en
pwndbg>c
# weird why pop to rsi not `rdx` above pushed
──────────────────────────────────────────────────────────────────────────────────────[ STACK ]──────────────────────────────────────────────────────────────────────────────────────
00:0000│ rsp 0x7fffffffdaf8 ◂— 0x0
01:0008│     0x7fffffffdb00 —▸ 0x7ffff7d8b058 ◂— 0xf0022000079d3
02:0010│     0x7fffffffdb08 —▸ 0x7ffff7fca9d8 ◂— 0x10001100000243
03:0018│     0x7fffffffdb10 ◂— 0x6
04:0020│     0x7fffffffdb18 —▸ 0x7fffffffdee0 —▸ 0x7fffffffe018 —▸ 0x7fffffffe480 ◂— '/mnt/ubuntu/home/czg/csapp3e/asm/check_cet_supported.out'
05:0028│     0x7fffffffdb20 —▸ 0x7fffffffdf00 ◂— 0x1
06:0030│     0x7fffffffdb28 ◂— 0x0
07:0038│     0x7fffffffdb30 —▸ 0x7fffffffe028 —▸ 0x7fffffffe4b9 ◂— 'BEMENU_BACKEND=wayland'
────────────────────────────────────────────────────────────────────────[ DISASM / x86-64 / set emulate on ]─────────────────────────────────────────────────────────────────────────
  0x7ffff7fda7ed  pop    rsi
 ►0x7ffff7fda7ee  pop    rdi
#  after several `c`, above reg pushed by '0x7ffff7fda6ed' is restored and then return. same as above "‘0x7ffff7fda7dd’ after pushed by callee ‘0x7ffff7fd4180’"
────────────────────────────────────────────────────────────────────────[ DISASM / x86-64 / set emulate on ]─────────────────────────────────────────────────────────────────────────
  0x7ffff7fda860  mov    qwordptr[r12],rax
 ►0x7ffff7fda864  add    rsp,0x18
  0x7ffff7fda868  pop    rbx
pwndbg> d-- 0x7ffff7fda864
Dump of assembler code from 0x7ffff7fda82f to 0x7ffff7fda899:
   0x00007ffff7fda82f:  00 48 8b                add    BYTE PTR [rax-0x75],cl
   0x00007ffff7fda832:  83 58 03 00             sbb    DWORD PTR [rax+0x3],0x0
   0x00007ffff7fda836:  00 48 85                add    BYTE PTR [rax-0x7b],cl
   0x00007ffff7fda839:  c0 0f 84                ror    BYTE PTR [rdi],0x84
   0x00007ffff7fda83c:  91                      xchg   ecx,eax
   0x00007ffff7fda83d:  00 00                   add    BYTE PTR [rax],al
   0x00007ffff7fda83f:  00 48 c1                add    BYTE PTR [rax-0x3f],cl
   0x00007ffff7fda842:  e5 05                   in     eax,0x5
   0x00007ffff7fda844:  48 01 c5                add    rbp,rax
   0x00007ffff7fda847:  8b 45 1c                mov    eax,DWORD PTR [rbp+0x1c]
   0x00007ffff7fda84a:  85 c0                   test   eax,eax
   0x00007ffff7fda84c:  0f 84 9e 00 00 00       je     0x7ffff7fda8f0
   0x00007ffff7fda852:  48 8b 45 00             mov    rax,QWORD PTR [rbp+0x0]
   0x00007ffff7fda856:  8b 15 b4 22 02 00       mov    edx,DWORD PTR [rip+0x222b4]        # 0x7ffff7ffcb10 <_rtld_global_ro+80>
   0x00007ffff7fda85c:  85 d2                   test   edx,edx
   0x00007ffff7fda85e:  75 04                   jne    0x7ffff7fda864
   0x00007ffff7fda860:  49 89 04 24             mov    QWORD PTR [r12],rax
=> 0x00007ffff7fda864:  48 83 c4 18             add    rsp,0x18
   0x00007ffff7fda868:  5b                      pop    rbx
   0x00007ffff7fda869:  5d                      pop    rbp
   0x00007ffff7fda86a:  41 5c                   pop    r12
   0x00007ffff7fda86c:  41 5d                   pop    r13
   0x00007ffff7fda86e:  c3                      ret
```
## stack with `rsp`
- based on [this](https://stackoverflow.com/questions/36529449/why-are-rbp-and-rsp-called-general-purpose-registers), `rsp` doesn't imply stack start location always.
```bash
pwndbg> |vmmap|grep stack
    0x7ffffffde000     0x7ffffffff000 rw-p    21000      0 [stack]
pwndbg> telescope 0x7ffffffde000 0x3d00
00:0000│  0x7ffffffde000 ◂— 0x0
... ↓     15450 skipped
3c5b:1e2d8│  0x7fffffffc2d8 —▸ 0x7ffff7fd122a ◂— mov r10d,dwordptr[rbp-0x108]
...
pwndbg> stack
00:0000│ r13 rsp 0x7fffffffe0a0 ◂— 0x1
01:0008│         0x7fffffffe0a8 —▸ 0x7fffffffe510 ◂— '/mnt/ubuntu/home/czg/csapp3e/asm/check_cet_supported.out'
02:0010│         0x7fffffffe0b0 ◂— 0x0
03:0018│ rcx     0x7fffffffe0b8 —▸ 0x7fffffffe549 ◂— 'BEMENU_BACKEND=wayland'
04:0020│         0x7fffffffe0c0 —▸ 0x7fffffffe560 ◂— 'BROWSER=google-chrome-stable'
05:0028│         0x7fffffffe0c8 —▸ 0x7fffffffe57d ◂— 'COLORFGBG=0;15'
06:0030│         0x7fffffffe0d0 —▸ 0x7fffffffe58c ◂— 'COLORTERM=truecolor'
07:0038│         0x7fffffffe0d8 —▸ 0x7fffffffe5a0 ◂— 'DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus'
```
- see [this 28:04 function prologue](https://www.youtube.com/watch?v=LAkYW5ixvhg)
## [why](https://stackoverflow.com/questions/6212665/why-is-eax-zeroed-before-a-call-to-printf#:~:text=Since%20the%20compiler%20doesn't,defined%20as%20having%20variable%20arguments.) clear `$rax` ('it might be the case that the function is defined as having variable arguments')
## terms
- [macro-operation MOP/µOP](https://en.wikichip.org/wiki/macro-operation#:~:text=In%20their%20context%2C%20macro%2Doperations%20are%20a%20fixed%2Dlength,%2C%20modify%2C%20and%20write%20operation.)
- [exclusive or `^`](https://en.wikipedia.org/wiki/Exclusive_or)
  - or -> 1,0 / 0,1 / 1,1; then exclusive 1,1 , get `^`
## manuals
- [1](https://www.chessprogramming.org/X86-64#Agner_Fog)
- security
  - IEEE Security and Privacy
- [ATT](https://docs.oracle.com/cd/E19253-01/817-5477/)  diffs [this](../references/att_x86.pdf) especially SSE&AVX instruction prefix `v`
  - also [1](https://docs.oracle.com/cd/E19253-01/817-5477/) [2](https://stackoverflow.com/tags/att/info) from [this](https://stackoverflow.com/questions/1776570/is-there-a-complete-x86-assembly-language-reference-that-uses-att-syntax)
    - above 2: says att is 'destination-last order'
## visual asm
- [this](https://godbolt.org/z/BzhckE)
# blogs
- [this](http://pwnable.kr/) from [this](http://archive.hack.lu/2015/radare2-workshop-slides.pdf)
## radare 2 [pdf](http://archive.hack.lu/2015/radare2-workshop-slides.pdf)
- TODO
  - hack bios & uefi p77
## small errors in some blog
- [this](https://blog.fxiao.me/got-plt/)
```bash
 ►0x401040      <printf@plt>   jmp    qwordptr[rip+0x2fc2]          <printf>
    ↓
  0x7ffff7dc7e00<printf>       endbr64 
  0x7ffff7dc7e04<printf+4>     sub    rsp,0xd8
pwndbg> dr
Dump of assembler code for function printf@plt:
=> 0x0000000000401040 <+0>:     ff 25 c2 2f 00 00       jmp    QWORD PTR [rip+0x2fc2]        # 0x404008 <printf@got.plt>
   0x0000000000401046 <+6>:     68 01 00 00 00          push   0x1
   0x000000000040104b <+11>:    e9 d0 ff ff ff          jmp    0x401020
pwndbg> telescope 0x401046+0x2fc2
00:0000│  0x404008 (printf@got[plt]) —▸ 0x7ffff7dc7e00 (printf) ◂— endbr64 
01:0008│  0x404010 (data_start) ◂— 0x0
... ↓     6 skipped
pwndbg> info symbol 0x7ffff7dc7e00
printf in section .text of /usr/lib/libc.so.6
# this is true instead of current `$rsp+0x2fc2`
pwndbg> info symbol 0x404008
printf@got[plt] in section .got.plt of /mnt/ubuntu/home/czg/csapp3e/asm/prog
```
# memory
- currently although using paged memory, but has been [abstracted](https://superuser.com/questions/318804/understanding-flat-memory-model-and-segmented-memory-model) to be flat, so just using [near](https://stackoverflow.com/questions/46187337/how-can-the-processor-discern-a-far-return-from-a-near-return) return `C3`
  - Internal Fragmentation is caused by being used by process. [whereas](https://www.tutorialspoint.com/difference-between-internal-fragmentation-and-external-fragmentation#:~:text=Internal%20Fragmentation%20occurs%20when%20a,removed%20from%20the%20main%20memory.&text=Best%20Fit%20Block%20Search%20is,the%20solution%20for%20external%20fragmentation.) External Fragmentation is cauesd by small unused spaces between used spaces.
  - [secondary](https://en.wikipedia.org/wiki/Computer_data_storage) memory
  - paging [diffs with](https://www.tutorialspoint.com/difference-between-paging-and-segmentation) segmentation 
  - [pdf p72](../references/intel_64.pdf) Real-Address Mode Model -> 'Real mode' in x86
# elf
## `.fini` [explanation](http://ftp.math.utah.edu/u/ma/hohn/linux/misc/elf/node3.html) and `.init`
- can check by debugger 
```bash
$ readelf -S check_cet_supported.out
$ objdump -d check_cet_supported.out
```
# asm code example 
> how to read manual, [1](https://stackoverflow.com/questions/43153964/how-to-read-assembly-opcode-reference) or highly [recommended](https://stackoverflow.com/questions/28664856/how-to-interpret-x86-opcode-map) 
>

> 1 & 2 is [AT&T](https://flint.cs.yale.edu/cs421/papers/x86-asm/asm.html) syntax which is [reverse](https://stackoverflow.com/questions/43764578/assembler-parameters-type#comment74573539_43764578) from intel . 
> and they are generated with `-Og` by `gcc -Og -o prog 010-main.c 010-mstore.c -g3`, if no `-Og` may output redundant codes different from listed below.
## 1
```asm
$ objdump -d 010-mstore.o
...
0000000000001139 <multstore>:
    1139:       53                      push   %rbx
    113a:       48 89 d3                mov    %rdx,%rbx
    113d:       e8 ee fe ff ff          call   1030 <_init+0x30>
    1142:       48 89 03                mov    %rax,(%rbx)
    1145:       5b                      pop    %rbx
    1146:       c3                      ret
```
- [push](https://www.felixcloutier.com/x86/push.html) `r64` -> [`rd` p598; note here `REX.W` indicates REX prefix instead of one bit](../references/intel_64.pdf)
### 1.1
- `mov r/m64 r64` [with](https://stackoverflow.com/questions/67368069/how-to-resolve-ambivalence-in-x64-assembly) `mov r64 r/m64`
```bash
0:  48 89 d3                mov    %rdx,%rbx
3:  48 8b da                mov    %rdx,%rbx
```
  > notice: `+` may mean different in opcode, detailed in [p598](../references/intel_64.pdf)
  > [this](https://stackoverflow.com/questions/71774299/how-do-you-understand-rex-w-b8-rd-io-form-for-x86-64-assembly) indicates [`MOV r64, imm64`](https://www.felixcloutier.com/x86/mov) `REX.W` not added to opcode, but prefix.
  > above `push` indicates `rd` is added to opcode.
  - [`89`, where Op/En->`MR`: 'ModRM:r/m (w)' | 'ModRM:reg (r)'](https://www.felixcloutier.com/x86/mov) -> `r/m64 r64`; d3 -> 'dx,bx' satisfies
    > notice: p528 '/digit...REG' -> p598 '/digit:... r/m'
    > so ~~second operand ~~ 'reg' in 'ModRM:r/m' default is r/m, also [see this '/0'](https://stackoverflow.com/questions/15017659/how-to-read-the-intel-opcode-notation/53976236#53976236)
    > so above in p530/529 -> 'EDX' **column** , combing with 'EBX' is `d3`
### 1.2
- TODO: notice p530 [EBX] is data in [memory](https://hackmd.io/@paolieri/x86_64) referenced by `EBX`
  but above `89` in `%rax,(%rbx)` -> `r/m64,r64`, so the latter should be register instead of data in memory ? 
### 1.3
- [call](https://stackoverflow.com/questions/65926175/why-does-e8-f2-ff-ff-ff-translate-to-call-0xfffffff7), so above `e8 ee fe ff ff          call   1030 <_init+0x30>` (here little endien can be tested in `file /mnt/ubuntu/home/czg/csapp3e/asm/010-mstore.o` output `LSB pie executable` ) just means jump to `p/x (signed)0xfffffeee+0x1142`(in gdb) -> `0x1030`
### 1.4
- memory address unit is [byte -> 'associated 8 bits of data' in link](https://sourceware.org/gdb/onlinedocs/gdb/Memory.html)
## 2
```asm
$ gdb ./prog -ex 'br 8' -ex 'r' -ex 'disassemble'
... 
Dump of assembler code for function main:
   0x0000555555555149 <+0>:     sub    $0x18,%rsp
   0x000055555555514d <+4>:     mov    %fs:0x28,%rax
   0x0000555555555156 <+13>:    mov    %rax,0x8(%rsp)
   0x000055555555515b <+18>:    xor    %eax,%eax
=> 0x000055555555515d <+20>:    mov    %rsp,%rdx
   0x0000555555555160 <+23>:    mov    $0x3,%esi
   0x0000555555555165 <+28>:    mov    $0x2,%edi
   0x000055555555516a <+33>:    call   0x5555555551ab <multstore>
   0x000055555555516f <+38>:    mov    (%rsp),%rsi
   0x0000555555555173 <+42>:    lea    0xe8a(%rip),%rdi        # 0x555555556004
   0x000055555555517a <+49>:    mov    $0x0,%eax
   0x000055555555517f <+54>:    call   0x555555555040 <p  rintf@plt>
   0x0000555555555184 <+59>:    mov    0x8(%rsp),%rax
   0x0000555555555189 <+64>:    sub    %fs:0x28,%rax
   0x0000555555555192 <+73>:    jne    0x55555555519e <main+85>
   0x0000555555555194 <+75>:    mov    $0x0,%eax
   0x0000555555555199 <+80>:    add    $0x18,%rsp
   0x000055555555519d <+84>:    ret
   0x000055555555519e <+85>:    call   0x555555555030 <__stack_chk_fail@plt>
End of assembler dump.
```
- [%fs:0x28](https://stackoverflow.com/questions/10325713/why-does-this-memory-address-fs0x28-fs0x28-have-a-random-value) 
- [0x8(%rsp)](https://stackoverflow.com/questions/19748074/meaning-of-0x8rsp)
- '<+18>': zero [extend](https://stackoverflow.com/questions/11177137/why-do-x86-64-instructions-on-32-bit-registers-zero-the-upper-part-of-the-full-6)
  - with 8bit, [use](https://www.felixcloutier.com/x86/movzx) (also refer to register renaming) `movzx`

## 3(begin intel asm flavor)
```asm
0x0000555555558006 <+6>:     00 00                   add    BYTE PTR [rax],al
```
or `att`
```asm
0x0000555555558006 <+6>:     00 00                   add    %al,(%rax)
```
- maybe 'r/m 64' [can](https://stackoverflow.com/questions/68605218/understanding-instruction-encoding) be 'r/m 32' with suffix like `w` in `movw`
  - so maybe above code `BYTE PTR` can make `r/m 64` -> `r/m 8`, and therefore in 64 bit OS -> [00 /r](https://www.felixcloutier.com/x86/add) and [pdf p530](../references/intel_64.pdf)
  > after all, disasm from binary to assembly, so binary is first to read. (assembly code can be different based on how interpreted)
## 4
```asm


0x555555558002 <__stack_chk_fail@got.plt+2>: push   rbp
```
## 5
```asm
0x00007ffff7fe5b68:  48 c1 e2 20             shl    rdx,0x20
```
- `48` -> REX.W; 
  `c1` [opcode](https://www.felixcloutier.com/x86/sal:sar:shl:shr) -> `shl r/m64, imm8`; 
  `e2` [pdf p529](../references/intel_64.pdf) -> `edx`(`rdx` when `REX.W` set ,p533), 
  `20`->`id`

## 6
```asm
0x00007ffff7fe5b61:  80 25 7e 82 01 00 df    and    BYTE PTR [rip+0x1827e],0xdf        # 0x7ffff7ffdde6 <_rtld_global+3558>
```
- `80` -> `AND r/m8, imm8	` [here](https://www.felixcloutier.com/x86/and)
  `25` -> `disp32` in column 4 [pdf p530](../references/intel_64.pdf)
  `7e 82 01 00` -> 4 byte(specified in [pdf p525](../references/intel_64.pdf)) little endian of `0x1827e`
  `df` -> `id`
## 7 AVX
```asm
0x00007ffff7ecd9a4:  c5 f9 6e c6             vmovd  xmm0,esi
```
- `vmovd  xmm0,esi` -> `VMOVD xmm1, r32/m32` -> `VEX.128.66.0F.W0 6E /`(here `/` may be mistaken, should be `/r`)
  - here `xmm1` in [pdf p603](../references/intel_64.pdf) -> `indicates the first source operand using an XMM register`
  - [4800h](https://www.techpowerup.com/cpu-specs/ryzen-7-4800h.c2280) not contain avx-512, so not use EVEX [pdf p561](../references/intel_64.pdf)
    - also check `grep avx /proc/cpuinfo --color=always | uniq` only has `avx` and `avx2`
- `c5` -> [pdf p539](../references/intel_64.pdf) -> `11000101`
  so use 2-byte VEX prefix
- `f9` -> `11111001` -> `1 1111 0 01` based on [this form '|~R |     ~vvvv     | L |   pp  |' ](https://wiki.osdev.org/X86-64_Instruction_Encoding#VEX.2FXOP_opcodes)
  - from binary to asm(recommended)
    - `R` = 1 -> 32bit [pdf p539](../references/intel_64.pdf)
    - `vvvv` should read `Instruction Operand Encoding` table [pdf p601,1997](../references/intel_64.pdf),here is encoded by `ModRM` -> `1111`
    - `L` & `pp` see below
  - from asm to binary(not recommended because the asm is generated by disassembly)
    - `128` -> `L` = 0 [pdf p599](../references/intel_64.pdf)
    - `66` -> [SIMD(single instruction, multiple data) prefix](https://codeforces.com/blog/entry/98594#:~:text=SIMD%20stands%20for%20single%20instruction,the%20code%20to%20run%20faster.)
- `6e` -> opcode
- `c6` in `R=1` 32bit mode [pdf p530](../references/intel_64.pdf) based on this 'Instruction Operand Encoding' [table](https://www.felixcloutier.com/x86/movd:movq) -> 
  xmm1: ModRM:reg -> xmm0
  r32/m32: ModRM:r/m -> ESI
## 8 `_GLOBAL_OFFSET_TABLE_` shown in `pwndbg`
- can use `r2 -AA` below
- `R_386_JUMP_SLOT` & `R_386_GLOB_DAT` [meaning](https://docs.oracle.com/cd/E19957-01/806-0641/chapter6-26/index.html) [also](http://netwinder.osuosl.org/users/p/patb/public_html/elf_relocs.html)
> [diffs (see 'References to a function address')](https://refspecs.linuxfoundation.org/ELF/zSeries/lzsabi0_zSeries/x2251.html) of `got` and `plt`
>
> also see [this `?v reloc.func_name`](https://www.megabeets.net/a-journey-into-radare-2-part-2/) and [this](https://stackoverflow.com/questions/9685699/what-is-global-offset-table). so got is updated by linker to real address. and as [this 'For example' ](https://ctf101.org/binary-exploitation/what-is-the-got/) says
>
> also [GOTOFF](https://stackoverflow.com/questions/58346418/difference-between-got-and-gotoff)(offset)
>
> [R_386_JUMP_SLOT](https://stackoverflow.com/questions/11676472/what-is-the-difference-between-got-and-got-plt-section)
- below is mainly inspired by [this ‘Global Offset Table (GOT) and Procedure Linkage Table (PLT) - Binary Exploitation PWN101’ by RazviOverflow](https://www.youtube.com/watch?v=B4-wVdQo040)
```asm
; pwndbg
0x555555555020                  push   qwordptr[rip+0x2fca]          <_GLOBAL_OFFSET_TABLE_+8>
; r2
0x5630f4198020      ff35ca2f0000   push qword [0x5630f419aff0]
```
- reverse check
  - [pwndbbg](https://reverseengineering.stackexchange.com/questions/23311/dumping-the-got-with-gdb-at-run-time)
    - guess:
    ```asm
    pwndbg> got
    GOT protection: Partial RELRO | GOT functions: 2
    [0x555555558000] __stack_chk_fail@GLIBC_2.4 -> 0x555555555036 (__stack_chk_fail@plt+6) ◂— push 0 /* 'h' */
    [0x555555558008] printf@GLIBC_2.2.5 -> 0x555555555046 (printf@plt+6) ◂— push 1
    ; above 0x555555558008 -> below 000000004008
    [czg /mnt/ubuntu/home/czg/csapp3e/asm]$ readelf --relocs ./prog 

    Relocation section '.rela.dyn' at offset 0x5a0 contains 8 entries:
      Offset          Info           Type           Sym. Value    Sym. Name + Addend
    000000003dd0  000000000008 R_X86_64_RELATIVE                    1140
    000000003dd8  000000000008 R_X86_64_RELATIVE                    10f0
    000000004018  000000000008 R_X86_64_RELATIVE                    4018
    000000003fc0  000100000006 R_X86_64_GLOB_DAT 0000000000000000 __libc_start_main@GLIBC_2.34 + 0
    000000003fc8  000200000006 R_X86_64_GLOB_DAT 0000000000000000 _ITM_deregisterTM[...] + 0
    000000003fd0  000500000006 R_X86_64_GLOB_DAT 0000000000000000 __gmon_start__ + 0
    000000003fd8  000600000006 R_X86_64_GLOB_DAT 0000000000000000 _ITM_registerTMCl[...] + 0
    000000003fe0  000700000006 R_X86_64_GLOB_DAT 0000000000000000 __cxa_finalize@GLIBC_2.2.5 + 0

    Relocation section '.rela.plt' at offset 0x660 contains 2 entries:
      Offset          Info           Type           Sym. Value    Sym. Name + Addend
    000000004000  000300000007 R_X86_64_JUMP_SLO 0000000000000000 __stack_chk_fail@GLIBC_2.4 + 0
    000000004008  000400000007 R_X86_64_JUMP_SLO 0000000000000000 printf@GLIBC_2.2.5 + 0
    ```
      - TODO weird '<>'
      ```asm
        0x55555555504b<printf@plt+11>   jmp    0x555555555020                <0x555555555020>
    ↓
        ►0x555555555020                  push   qwordptr[rip+0x2fca]          <_GLOBAL_OFFSET_TABLE_+8>

      ```
      also use `nm ./prog` to see
  - TODO r2
  ```asm
  ;https://reverseengineering.stackexchange.com/questions/17379/radare-cant-seek-to-start-symbol
  f*
  [0x5630f419a976]> f~got
  0x5630f419afc0 40 section..got
  0x5630f419afe8 40 section..got.plt
  [0x5630f419a976]> s 0x5630f419afe8
  [0x5630f419afe8]> pd 2
  │           ;-- section..got.plt:
  │           ;-- _GLOBAL_OFFSET_TABLE_:
  │       ┌─< 0x5630f419afe8      e03d           loopne 0x5630f419b027   ; reloc.__cxa_finalize+0x47 ; decrement count; jump short if ecx!=0 and zf=0; [23] -rw- section size 40 named .got.plt
  │       │   0x5630f419afea      0000           add byte [rax], al      ; adds src and dst, stores result on dst
  ```
  [.got vs .got.plt](https://maskray.me/blog/2021-08-29-all-about-global-offset-table)
### `nm` [this](https://stackoverflow.com/questions/9686764/how-can-i-get-the-global-offset-table-address-in-my-program)
```asm
$ gcc -fno-pic -Og -o test_got test_got.c -g3 -no-pie # this not
$ gcc -fPIE -no-pie test_got.c -o test_got # this works
$ nm ./test_got
0000000000404028 B __bss_start
0000000000404018 D __data_start
0000000000404018 W data_start
0000000000401090 T _dl_relocate_static_pie
0000000000404020 D __dso_handle
0000000000403df8 d _DYNAMIC
0000000000404028 D _edata
0000000000404030 B _end
                 U exit@GLIBC_2.2.5
000000000040117c T _fini
0000000000403fe8 d _GLOBAL_OFFSET_TABLE_
                 w __gmon_start__
...
$ r2 -AA -d /mnt/ubuntu/home/czg/csapp3e/asm/test_got
...
[0x7f0a71588ed0]> is~GLOBAL_
5   0x00002fe8 0x00403fe8 LOCAL  OBJ    0        _GLOBAL_OFFSET_TABLE_
[0x7f7bf15b9ed0]> iS~got.p
23  0x00002fe8   0x30 0x00403fe8   0x30 -rw- PROGBITS    .got.plt
[0x7f7bf15b9ed0]> pd 3@0x00403fe8
            ;-- section..got.plt:
            ;-- _GLOBAL_OFFSET_TABLE_:
...
```
### disassembly 'wrong' if 'cutted' from [this 17:52](https://www.youtube.com/watch?v=B4-wVdQo040) (po because it is located in `.got/.got.plt`(sections can be seen by `iS` in r2) it should be viewed as address instead of instruction) <a id="dis"></a>
```asm
[0x555f3c702040]> pd 2;dr rip
            ; CALL XREF from main @ 0x555f3c70217f(x)
┌ 6: int sym.imp.printf (const char *format);
│ rg: 0 (vars 0, args 0)
│ bp: 0 (vars 0, args 0)
│ sp: 0 (vars 0, args 0)
└           0x555f3c702040 b    ff25c22f0000   jmp qword [reloc.printf] ; [0x555f3c705008:8]=0x555f3c702046 rip ; "F p<_U" ; jump
            ;-- rip:
            0x555f3c702046      6801000000     push 1                  ; rbp ; push word, doubleword or quadword onto the stack
0x555f3c702046
[0x555f3c702040]> pd 5@reloc.printf
│           ;-- reloc.printf:
│           ; DATA XREF from sym.imp.printf @ 0x555f3c702040(x)
│           0x555f3c705008      4620703c       and byte [rax + 0x3c], r14b ; binary and operation between src and dst, stores result on dst
│           0x555f3c70500c      5f             pop rdi                 ; pops last element of stack and stores the result in argument
│           0x555f3c70500d      55             push rbp                ; push word, doubleword or quadword onto the stack
...
; notice in little endien, above three instrucion is 555f3c702046(here 4620703c get inversed besed on byte unit)
```
### dynamic link( whether `-no-pie` and `checksec` no canary or not ,all fine [use two `printf/puts` to make two `dc` work without quit])
```asm
[czg ~/czg_scripts]$ cat test_got.c
// Build with: gcc -m32 -no-pie -g -o plt plt.c

#include <stdio.h>
#include <stdlib.h>

int main(int argc, char **argv) {
  puts("Hello world!");
  puts("Hello world!");
  exit(0);
}
[czg ~/czg_scripts]$ gcc -g -o test_got test_got.c # -no-pie to make address same every execute
[czg ~/czg_scripts]$ r2 -A -d ~/czg_scripts/test_got
INFO: Analyze all flags starting with sym. and entry0 (aa)
INFO: Analyze all functions arguments/locals (afva@@@F)
INFO: Analyze function calls (aac)
INFO: Analyze len bytes of instructions for references (aar)
INFO: Finding and parsing C++ vtables (avrr)
INFO: Skipping type matching analysis in debugger mode (aaft)
INFO: Propagate noreturn information (aanr0x5653c8441000ytes>@<offset>.  For example: 3x20@0x33 will show 3 hexdumps of 20 bytes at 0x33
[0x7f840813fed0]> db sym.imp.puts;dc;pxr 20@reloc.puts
INFO: hit breakpoint at: 0x5653c843e030
0x5653c8441000 0x00005653c843e036   6.C.SV.. @ reloc.puts /home/czg/czg_scripts/test_got .plt program R X 'push 0' 'test_got'
0x5653c8441008 0x00005653c843e046   F.C.SV.. @ reloc.exit /home/czg/czg_scripts/test_got .plt program R X 'push 1' 'test_got'
[0x5653c843e030]> dc;pxr 20@reloc.puts
Hello world!
INFO: hit breakpoint at: 0x5653c843e030
0x5653c8441000 0x00007f8407f4bc10   ........ @ reloc.puts
0x5653c8441008 0x00005653c843e046   F.C.SV.. @ reloc.exit /home/czg/czg_scripts/test_got .plt program R X 'push 1' 'test_got'
[0x5653c843e030]> dmi libc ; tell r2 libc location for comment https://reverseengineering.stackexchange.com/questions/18819/radare2-debugging-how-to-get-into-lazy-imported-functions dynamic link
Do you want to print 3041 lines? (y/N)n
[0x5653c843e030]> pxr 20@reloc.puts
0x5653c8441000 0x00007f8407f4bc10   ........ @ reloc.puts /usr/lib/libc.so.6 library R X 'endbr64' 'libc.so.6'
0x5653c8441008 0x00005653c843e046   F.C.SV.. @ reloc.exit /home/czg/czg_scripts/test_got .plt program R X 'push 1' 'test_got'
```

```asm
$ gcc -g -no-pie -o test_got test_got.c # -no-pie to make address same every execute
[czg ~/czg_scripts]$ r2 -A -d ~/czg_scripts/test_got
[0x7fef740cced0]> afl~puts
0x00401030    1      6 sym.imp.puts
[0x7fef740cced0]> iS~plt
11  0x000005b0   0x30 0x004005b0   0x30 -r-- RELA        .rela.plt
13  0x00001020   0x30 0x00401020   0x30 -r-x PROGBITS    .plt
23  0x00002fe8   0x28 0x00403fe8   0x28 -rw- PROGBITS    .got.plt
db sym.imp.puts;dc
pxr 20@reloc.puts ; these two same as above


; it shows '.got.plt' -> '_GLOBAL_OFFSET_TABLE_'
[0x00404000]> s reloc.puts
[0x00404000]> iS.
Current section

nth paddr       size vaddr       vsize perm type     name
―――――――――――――――――――――――――――――――――――――――――――――――――――――――――
0   0x00002fe8  0x28 0x00403fe8   0x28 -rw- PROGBITS .got.plt

[0x00404000]> pd 2 @0x00403fe8
            ;-- section..got.plt:
            ;-- _GLOBAL_OFFSET_TABLE_:
            0x00403fe8      f8             clc                         ; clear carry flag; [23] -rw- section size 40 named .got.plt
            0x00403fe9      3d40000000     cmp eax, 0x40               ; elf_phdr ; compare two operands
[0x00404000]> dc
Hello world!
INFO: hit breakpoint at: 0x401030
^C
[0x00401030]> dm~ld
0x00007fef740b2000 - 0x00007fef740b3000 - usr     4K s r-- /usr/lib/ld-linux-x86-64.so.2 /usr/lib/ld-linux-x86-64.so.2
0x00007fef740b3000 - 0x00007fef740d9000 - usr   152K s r-x /usr/lib/ld-linux-x86-64.so.2 /usr/lib/ld-linux-x86-64.so.2 ; map._usr_lib_ld_linux_x86_64.so.2.r_x
0x00007fef740d9000 - 0x00007fef740e3000 - usr    40K s r-- /usr/lib/ld-linux-x86-64.so.2 /usr/lib/ld-linux-x86-64.so.2 ; map._usr_lib_ld_linux_x86_64.so.2.r__
0x00007fef740e3000 - 0x00007fef740e5000 - usr     8K s r-- /usr/lib/ld-linux-x86-64.so.2 /usr/lib/ld-linux-x86-64.so.2 ; map._usr_lib_ld_linux_x86_64.so.2.rw_
0x00007fef740e5000 - 0x00007fef740e7000 - usr     8K s rw- /usr/lib/ld-linux-x86-64.so.2 /usr/lib/ld-linux-x86-64.so.2 ; r15
[0x00401030]> pd 3
        ╎   ;-- rip:
        ╎   ; CALL XREFS from dbg.main @ 0x40114f(x), 0x40115e(x)
┌ 6: int sym.imp.puts (const char *s);
│ rg: 0 (vars 0, args 0)
│ bp: 0 (vars 0, args 0)
│ sp: 0 (vars 0, args 0)
└       ╎   0x00401030 b    ff25ca2f0000   jmp qword [reloc.puts]      ; [0x404000:8]=0x7fef73ed8c10 ; jump
        ╎   0x00401036      6800000000     push 0                      ; push word, doubleword or quadword onto the stack
        └─< 0x0040103b      e9e0ffffff     jmp section..plt            ; jump
[0x00401030]> 2ds
; here dynamic link works
[0x7fef73ed8c10]> pd 2
            0x7fef73ed8c10      f30f1efa       endbr64
            ;-- rip:
            0x7fef73ed8c14      4156           push r14                ; push word, doubleword or quadword onto the stack
```
### `_GLOBAL_OFFSET_TABLE_` actually means (based on comparison of `r2` and `pwndbg`) (use `-no-pie` with `gcc` to compare)

### endbr64 (also ~~see~~ has contents related with dynamic link with `pxr @ reloc.printf`)
```asm
$ gcc -Og -o prog 010-main.c 010-mstore.c -g3 -no-pie
$ r2 -A -d /mnt/ubuntu/home/czg/csapp3e/asm/prog
...
[0x00401040]> dc
2 * 3 --> 6
INFO: hit breakpoint at: 0x401040
[0x00401040]> pxr @ reloc.printf
0x00404008 0x00007f9e7dc87e00   .~.}.... @ reloc.printf ;plt works
0x00404010 ..[ null bytes ]..   00000000 loc.__data_start
[0x00401040]> pd 3
        ╎   ;-- rip:
        ╎   ; CALL XREFS from dbg.main @ 0x401170(x), 0x401181(x)
┌ 6: int sym.imp.printf (const char *format);
│ rg: 0 (vars 0, args 0)
│ bp: 0 (vars 0, args 0)
│ sp: 0 (vars 0, args 0)
└       ╎   0x00401040 b    ff25c22f0000   jmp qword [reloc.printf]    ; [0x404008:8]=0x7f9e7dc87e00 ; jump
        ╎   0x00401046      6801000000     push 1                      ; rbp ; push word, doubleword or quadword onto the stack
        └─< 0x0040104b      e9d0ffffff     jmp section..plt            ; jump
[0x00401040]> 2ds
[0x7f9e7dc87e00]> dmi libc~7f9e7dc87e00 ; section..plt
1618 0x0004fe00 0x7f9e7dc87e00 GLOBAL FUNC 205       _IO_printf
2511 0x0004fe00 0x7f9e7dc87e00 GLOBAL FUNC 205       printf
[0x7f9e7dc87e00]> pd-- 2
        └─< 0x7f9e7dc87df9      eb8f           jmp 0x7f9e7dc87d8a      ; jump
            0x7f9e7dc87dfb      0f1f440000     nop dword [rax + rax]   ; no operation
            0x7f9e7dc87e00      f30f1efa       endbr64 ; here ‘section..plt’
            ;-- rip:
            0x7f9e7dc87e04      4881ecd80000.  sub rsp, 0xd8           ; substract src and dst, stores result on dst
```
#### pwndbg
```bash
# not dynamic link
pwndbg> telescope 0x401046+0x2fc2
00:0000│  0x404008 (printf@got[plt]) —▸ 0x401046 (printf@plt+6) ◂— push 1
01:0008│  0x404010 (data_start) ◂— 0x0
... ↓     6 skipped
pwndbg> br printf
pwndbg> c
pwndbg> telescope 0x401046+0x2fc2

00:0000│  0x404008 (printf@got[plt]) —▸ 0x7ffff7dc7e00 (printf) ◂— endbr64 
01:0008│  0x404010 (data_start) ◂— 0x0
... ↓     6 skipped
```
### `rip` register save the next instruction location [pdf p536](../references/intel_64.pdf) ~~updates before `jmp`~~
```bash
pwndbg> p $rip+0x2fc2
$3 = (void (*)()) 0x404008 <printf@got[plt]>
pwndbg> dr # disass /r
Dump of assembler code for function printf@plt:
   0x0000000000401040 <+0>:     ff 25 c2 2f 00 00       jmp    QWORD PTR [rip+0x2fc2]        # 0x404008 <printf@got.plt>
=> 0x0000000000401046 <+6>:     68 01 00 00 00          push   0x1
   0x000000000040104b <+11>:    e9 d0 ff ff ff          jmp    0x401020
End of assembler dump.
pwndbg> p $rip
$4 = (void (*)()) 0x401046 <printf@plt+6>
pwndbg> x $rip+0x2fc2
0x404008 <printf@got.plt>:      0x00401046
```
#### how memory save data [pdf p90](../references/intel_64.pdf)
```bash
────────────────────────────────────────────────────────────────[ DISASM / x86-64 / set emulate on ]─────────────────────────────────────────────────────────────────
  0x401020        push   qwordptr[rip+0x2fca]          <_GLOBAL_OFFSET_TABLE_+8>
 ►0x401026        jmp    qwordptr[rip+0x2fcc]          <0x7ffff7fdcb50>
    ↓
  0x7ffff7fdcb50  endbr64
pwndbg> disass /r 0x401026,0x401040
Dump of assembler code from 0x401026 to 0x401040:
=> 0x0000000000401026:  ff 25 cc 2f 00 00       jmp    QWORD PTR [rip+0x2fcc]        # 0x403ff8
   0x000000000040102c:  0f 1f 40 00             nop    DWORD PTR [rax+0x0]
   0x0000000000401030 <__stack_chk_fail@plt+0>: ff 25 ca 2f 00 00       jmp    QWORD PTR [rip+0x2fca]        # 0x404000 <__stack_chk_fail@got.plt>
   0x0000000000401036 <__stack_chk_fail@plt+6>: 68 00 00 00 00          push   0x0
   0x000000000040103b <__stack_chk_fail@plt+11>:        e9 e0 ff ff ff          jmp    0x401020
End of assembler dump.
# little endien
pwndbg> x/g 0x000000000040102c+0x2fca
0x403ff6:       0x7ffff7fdcb500000 # -> above `  0x7ffff7fdcb50  endbr64`
pwndbg> x/b 0x000000000040102c+0x2fca
0x403ff6:       0x00
pwndbg> x/w 0x000000000040102c+0x2fca
0x403ff6:       0xcb500000
```
### '<_GLOBAL_OFFSET_TABLE_+8>' [meaning](https://reverseengineering.stackexchange.com/questions/23311/dumping-the-got-with-gdb-at-run-time)
```bash
────────────────────────────────────────────────────────────────[ DISASM / x86-64 / set emulate on ]─────────────────────────────────────────────────────────────────
  0x401040<printf@plt>      jmp    qwordptr[rip+0x2fc2]          <printf@got[plt]>
 
 ►0x401046<printf@plt+6>    push   1
  0x40104b<printf@plt+11>   jmp    0x401020                      <0x401020>
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
pwndbg> x $rip+0x2fc2
0x404008 <printf@got.plt>:      0x00401046
pwndbg> got
GOT protection: Partial RELRO | GOT functions: 2
[0x404000] __stack_chk_fail@GLIBC_2.4 -> 0x401036 (__stack_chk_fail@plt+6) ◂— push 0 /* 'h' */
[0x404008] printf@GLIBC_2.2.5 -> 0x401046 (printf@plt+6) ◂— push 1
```
so this '<printf@got[plt]>' probably indicates instruction address -> '0x401046'

```bash
────────────────────────────────────────────────────────────────[ DISASM / x86-64 / set emulate on ]─────────────────────────────────────────────────────────────────
  0x40104b<printf@plt+11>   jmp    0x401020                      <0x401020>
    ↓
 ►0x401020                  push   qwordptr[rip+0x2fca]          <_GLOBAL_OFFSET_TABLE_+8>
  0x401026                  jmp    qwordptr[rip+0x2fcc]          <0x7ffff7fdcb50>
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
pwndbg> tele 0x401026+0x2fca
00:0000│  0x403ff0 (_GLOBAL_OFFSET_TABLE_+8) —▸ 0x7ffff7ffe2c0 ◂— 0x0
01:0008│  0x403ff8 (_GLOBAL_OFFSET_TABLE_+16) —▸ 0x7ffff7fdcb50 ◂— endbr64 
02:0010│  0x404000 (__stack_chk_fail@got.plt) —▸ 0x401036 (__stack_chk_fail@plt+6) ◂— push 0 /* 'h' */
03:0018│  0x404008 (printf@got[plt]) —▸ 0x401046 (printf@plt+6) ◂— push 1
04:0020│  0x404010 (data_start) ◂— 0x0
... ↓     3 skipped
pwndbg> vmmap
LEGEND: STACK | HEAP | CODE | DATA | RWX | RODATA
             Start                End Perm     Size Offset File
          0x400000           0x401000 r--p     1000      0 /mnt/ubuntu/home/czg/csapp3e/asm/prog
          0x401000           0x402000 r-xp     1000   1000 /mnt/ubuntu/home/czg/csapp3e/asm/prog
          0x402000           0x403000 r--p     1000   2000 /mnt/ubuntu/home/czg/csapp3e/asm/prog
          0x403000           0x404000 r--p     1000   2000 /mnt/ubuntu/home/czg/csapp3e/asm/prog
          0x404000           0x405000 rw-p     1000   3000 /mnt/ubuntu/home/czg/csapp3e/asm/prog
    0x7ffff7c8d000     0x7ffff7c90000 rw-p     3000      0 [anon_7ffff7c8d]
    0x7ffff7c90000     0x7ffff7c9e000 r--p     e000      0 /usr/lib/libm.so.6
    0x7ffff7c9e000     0x7ffff7d18000 r-xp    7a000   e000 /usr/lib/libm.so.6
    0x7ffff7d18000     0x7ffff7d76000 r--p    5e000  88000 /usr/lib/libm.so.6
    0x7ffff7d76000     0x7ffff7d77000 r--p     1000  e5000 /usr/lib/libm.so.6
    0x7ffff7d77000     0x7ffff7d78000 rw-p     1000  e6000 /usr/lib/libm.so.6
    0x7ffff7d78000     0x7ffff7d9a000 r--p    22000      0 /usr/lib/libc.so.6
    0x7ffff7d9a000     0x7ffff7ef4000 r-xp   15a000  22000 /usr/lib/libc.so.6
    0x7ffff7ef4000     0x7ffff7f4c000 r--p    58000 17c000 /usr/lib/libc.so.6
    0x7ffff7f4c000     0x7ffff7f50000 r--p     4000 1d4000 /usr/lib/libc.so.6
    0x7ffff7f50000     0x7ffff7f52000 rw-p     2000 1d8000 /usr/lib/libc.so.6
    0x7ffff7f52000     0x7ffff7f5f000 rw-p     d000      0 [anon_7ffff7f52]
    0x7ffff7f7d000     0x7ffff7f85000 r--p     8000      0 /usr/lib/liblua.so.5.4.4
    0x7ffff7f85000     0x7ffff7fb3000 r-xp    2e000   8000 /usr/lib/liblua.so.5.4.4
    0x7ffff7fb3000     0x7ffff7fbf000 r--p     c000  36000 /usr/lib/liblua.so.5.4.4
    0x7ffff7fbf000     0x7ffff7fc1000 r--p     2000  42000 /usr/lib/liblua.so.5.4.4
    0x7ffff7fc1000     0x7ffff7fc2000 rw-p     1000  44000 /usr/lib/liblua.so.5.4.4
    0x7ffff7fc2000     0x7ffff7fc4000 rw-p     2000      0 [anon_7ffff7fc2]
    0x7ffff7fc4000     0x7ffff7fc8000 r--p     4000      0 [vvar]
    0x7ffff7fc8000     0x7ffff7fca000 r-xp     2000      0 [vdso]
    0x7ffff7fca000     0x7ffff7fcb000 r--p     1000      0 /usr/lib/ld-linux-x86-64.so.2
    0x7ffff7fcb000     0x7ffff7ff1000 r-xp    26000   1000 /usr/lib/ld-linux-x86-64.so.2
    0x7ffff7ff1000     0x7ffff7ffb000 r--p     a000  27000 /usr/lib/ld-linux-x86-64.so.2
    0x7ffff7ffb000     0x7ffff7ffd000 r--p     2000  31000 /usr/lib/ld-linux-x86-64.so.2
    0x7ffff7ffd000     0x7ffff7fff000 rw-p     2000  33000 /usr/lib/ld-linux-x86-64.so.2
    0x7ffffffde000     0x7ffffffff000 rw-p    21000      0 [stack]
0xffffffffff600000 0xffffffffff601000 --xp     1000      0 [vsyscall]
# notice above 0x7ffff7ffe2c0 is in '0x7ffff7ffd000 ~ 0x7ffff7fff000' -> /usr/lib/ld-linux-x86-64.so.2
pwndbg> apropos section
...
maintenance info sections -- List the BFD sections of the exec and core files.
...
pwndbg> maintenance info sections
Exec file: `/mnt/ubuntu/home/czg/csapp3e/asm/prog', file type elf64-x86-64.
 [0]      0x00400318->0x00400334 at 0x00000318: .interp ALLOC LOAD READONLY DATA HAS_CONTENTS
 [1]      0x00400338->0x00400378 at 0x00000338: .note.gnu.property ALLOC LOAD READONLY DATA HAS_CONTENTS
 [2]      0x00400378->0x0040039c at 0x00000378: .note.gnu.build-id ALLOC LOAD READONLY DATA HAS_CONTENTS
 [3]      0x0040039c->0x004003bc at 0x0000039c: .note.ABI-tag ALLOC LOAD READONLY DATA HAS_CONTENTS
 [4]      0x004003c0->0x004003dc at 0x000003c0: .gnu.hash ALLOC LOAD READONLY DATA HAS_CONTENTS
 [5]      0x004003e0->0x00400488 at 0x000003e0: .dynsym ALLOC LOAD READONLY DATA HAS_CONTENTS
 [6]      0x00400488->0x00400523 at 0x00000488: .dynstr ALLOC LOAD READONLY DATA HAS_CONTENTS
 [7]      0x00400524->0x00400532 at 0x00000524: .gnu.version ALLOC LOAD READONLY DATA HAS_CONTENTS
 [8]      0x00400538->0x00400578 at 0x00000538: .gnu.version_r ALLOC LOAD READONLY DATA HAS_CONTENTS
 [9]      0x00400578->0x004005d8 at 0x00000578: .rela.dyn ALLOC LOAD READONLY DATA HAS_CONTENTS
 [10]     0x004005d8->0x00400608 at 0x000005d8: .rela.plt ALLOC LOAD READONLY DATA HAS_CONTENTS
 [11]     0x00401000->0x0040101b at 0x00001000: .init ALLOC LOAD READONLY CODE HAS_CONTENTS
 [12]     0x00401020->0x00401050 at 0x00001020: .plt ALLOC LOAD READONLY CODE HAS_CONTENTS
 [13]     0x00401050->0x004011bc at 0x00001050: .text ALLOC LOAD READONLY CODE HAS_CONTENTS
 [14]     0x004011bc->0x004011c9 at 0x000011bc: .fini ALLOC LOAD READONLY CODE HAS_CONTENTS
 [15]     0x00402000->0x00402013 at 0x00002000: .rodata ALLOC LOAD READONLY DATA HAS_CONTENTS
 [16]     0x00402014->0x00402050 at 0x00002014: .eh_frame_hdr ALLOC LOAD READONLY DATA HAS_CONTENTS
 [17]     0x00402050->0x00402110 at 0x00002050: .eh_frame ALLOC LOAD READONLY DATA HAS_CONTENTS
 [18]     0x00403de8->0x00403df0 at 0x00002de8: .init_array ALLOC LOAD DATA HAS_CONTENTS
 [19]     0x00403df0->0x00403df8 at 0x00002df0: .fini_array ALLOC LOAD DATA HAS_CONTENTS
 [20]     0x00403df8->0x00403fc8 at 0x00002df8: .dynamic ALLOC LOAD DATA HAS_CONTENTS
 [21]     0x00403fc8->0x00403fe8 at 0x00002fc8: .got ALLOC LOAD DATA HAS_CONTENTS
 [22]     0x00403fe8->0x00404010 at 0x00002fe8: .got.plt ALLOC LOAD DATA HAS_CONTENTS
 [23]     0x00404010->0x00404020 at 0x00003010: .data ALLOC LOAD DATA HAS_CONTENTS
 [24]     0x00404020->0x00404028 at 0x00003020: .bss ALLOC
 [25]     0x00000000->0x0000001b at 0x00003020: .comment READONLY HAS_CONTENTS
 [26]     0x00000000->0x00000060 at 0x0000303b: .debug_aranges READONLY HAS_CONTENTS
 [27]     0x00000000->0x00000272 at 0x0000309b: .debug_info READONLY HAS_CONTENTS
 [28]     0x00000000->0x000001c2 at 0x0000330d: .debug_abbrev READONLY HAS_CONTENTS
 [29]     0x00000000->0x00000170 at 0x000034cf: .debug_line READONLY HAS_CONTENTS
 [30]     0x00000000->0x00005c25 at 0x0000363f: .debug_str READONLY HAS_CONTENTS
 [31]     0x00000000->0x000001d0 at 0x00009264: .debug_line_str READONLY HAS_CONTENTS
 [32]     0x00000000->0x00000065 at 0x00009434: .debug_loclists READONLY HAS_CONTENTS
 [33]     0x00000000->0x0000134f at 0x00009499: .debug_macro READONLY HAS_CONTENTS
# here 0x403ff0 is in `.got.plt`

# or more simple
pwndbg> | maintenance info target-sections | grep 403f

 [20]     0x00403df8->0x00403fc8 at 0x00002df8: .dynamic ALLOC LOAD DATA HAS_CONTENTS
          Start: 0x00403df8, End: 0x00403fc8, Owner token: 0x55ea8716bc98
 [21]     0x00403fc8->0x00403fe8 at 0x00002fc8: .got ALLOC LOAD DATA HAS_CONTENTS
          Start: 0x00403fc8, End: 0x00403fe8, Owner token: 0x55ea8716bc98
 [22]     0x00403fe8->0x00404010 at 0x00002fe8: .got.plt ALLOC LOAD DATA HAS_CONTENTS
          Start: 0x00403fe8, End: 0x00404010, Owner token: 0x55ea8716bc98
pwndbg> | maintenance info target-sections | grep 0x55ea87f508d0 -B 2 | head

From '/lib64/ld-linux-x86-64.so.2', file type elf64-x86-64:
 [0]      0x000002a8->0x000002e8 at 0x000002a8: .note.gnu.property ALLOC LOAD READONLY DATA HAS_CONTENTS
          Start: 0x7ffff7fca2a8, End: 0x7ffff7fca2e8, Owner token: 0x55ea87f508d0
 [1]      0x000002e8->0x0000030c at 0x000002e8: .note.gnu.build-id ALLOC LOAD READONLY DATA HAS_CONTENTS
          Start: 0x7ffff7fca2e8, End: 0x7ffff7fca30c, Owner token: 0x55ea87f508d0
 [2]      0x00000310->0x000004d8 at 0x00000310: .hash ALLOC LOAD READONLY DATA HAS_CONTENTS
          Start: 0x7ffff7fca310, End: 0x7ffff7fca4d8, Owner token: 0x55ea87f508d0
 [3]      0x000004d8->0x000006bc at 0x000004d8: .gnu.hash ALLOC LOAD READONLY DATA HAS_CONTENTS
          Start: 0x7ffff7fca4d8, End: 0x7ffff7fca6bc, Owner token: 0x55ea87f508d0
 [4]      0x000006c0->0x00000a68 at 0x000006c0: .dynsym ALLOC LOAD READONLY DATA HAS_CONTENTS
```
#### in `r2` (notice with `no-pie` only code is static, lib related with `ld` all diff)
```asm
$ r2 -A -d /mnt/ubuntu/home/czg/csapp3e/asm/prog
[0x00401020]> pd 3
            ;-- section..plt:
            ;-- rip:
            0x00401020      ff35ca2f0000   push qword [0x00403ff0]     ; push word, doubleword or quadword onto the stack; [13] -r-x section size 48 named .plt
            0x00401026      ff25cc2f0000   jmp qword [0x00403ff8]      ; [0x403ff8:8]=0x7f06543ddb50 ; jump
            0x0040102c      0f1f4000       nop dword [rax]             ; no operation
; here because in .got.plt , so use `pxr` instead of `pd`. [2](#dis) link above
[0x00401020]> pxr 0x8@0x00403ff0
0x00403ff0 0x00007f06543ff2c0   ..?T.... /usr/lib/ld-linux-x86-64.so.2 library R W 0x0
[0x00401020]> iS~got
22  0x00002fc8    0x20 0x00403fc8    0x20 -rw- PROGBITS    .got
23  0x00002fe8    0x28 0x00403fe8    0x28 -rw- PROGBITS    .got.plt
[0x00401020]> pxr 0x10@0x00403fe8
0x00403fe8 0x0000000000403df8   .=@..... @ obj._GLOBAL_OFFSET_TABLE_ 4210168 /mnt/ubuntu/home/czg/csapp3e/asm/prog .dynamic section..dynamic,segment.DYNAMIC,_DYNAMIC program R 0x1
0x00403ff0 0x00007f06543ff2c0   ..?T.... /usr/lib/ld-linux-x86-64.so.2 library R W 0x0 ; here is above `pwndbg` '_GLOBAL_OFFSET_TABLE_+8'
```
##### view stack (continue after above)
```asm
[0x00401020]> pd 1
            ;-- section..plt:
            0x00401020      ff35ca2f0000   push qword [0x00403ff0]     ; push word, doubleword or quadword onto the stack; [13] -r-x section size 48 named .plt
[0x00401020]> px 0x10@rsp
- offset -      3031 3233 3435 3637 3839 3A3B 3C3D 3E3F  0123456789ABCDEF
0x7fff5846fd30  0100 0000 0000 0000 7511 4000 0000 0000  ........u.@.....
[0x00401020]> pxr 0x10@rsp
0x7fff5846fd30 0x0000000000000001   ........ @ rsp 1 .comment rbp
0x7fff5846fd38 0x0000000000401175   u.@..... 4198773 /mnt/ubuntu/home/czg/csapp3e/asm/prog .text dbg.main program R X 'mov rsi, qword [rsp]' 'prog'
[0x00401020]> ds
[0x00401020]> pxr 0x10@rsp
0x7fff5846fd28 0x00007f06543ff2c0   ..?T.... @ rsp /usr/lib/ld-linux-x86-64.so.2 library R W 0x0
0x7fff5846fd30 0x0000000000000001   ........ 1 .comment rbp
[0x00401020]> px 0x10@rsp
- offset -      2829 2A2B 2C2D 2E2F 3031 3233 3435 3637  89ABCDEF01234567
0x7fff5846fd28  c0f2 3f54 067f 0000 0100 0000 0000 0000  ..?T............
[0x00401020]> ? 0x7fff5846fd30-0x7fff5846fd28~:1
uint32  8
; here quadword -> rsp should decrease 8  
```
#### `print` can print func location (most of `dmi libc` / `dmi prog` in `r2`)
### fs/gs register (other segment register no use in 64-bit mode [pdf p73,74 Canonical Addressing,79](../references/intel_64.pdf) [1 TODO:'CS/DS/ES/SS are only useful for padding to make instructions longer?'](https://stackoverflow.com/questions/50400274/why-is-the-use-of-the-ds-segment-override-illegal-in-64-bit-mode)) mainly based on [this](https://reverseengineering.stackexchange.com/questions/19979/what-does-fs-and-gs-registers-provide-in-linux)
[pdf p75](../references/intel_64.pdf) register size
> TODO more [datailed](https://unix.stackexchange.com/questions/453749/what-sets-fs0x28-stack-canary) with TLS
> > how fs:0x28 check overflow (try making program overflow and see)
```bash
[czg ~]$ cat czg_scripts/test_fs_reg.c 
// Build with: gcc -m32 -no-pie -g -o plt plt.c

#include <stdio.h>
#include <stdlib.h>

int main(int argc, char **argv) {
  exit(0);
}
$ echo "test_fs_reg" | xargs -I@ gcc -fstack-protector-strong -g -no-pie -fno-pic -o czg_scripts/@ czg_scripts/@.c # https://stackoverflow.com/questions/10325713/why-does-this-memory-address-fs0x28-fs0x28-have-a-random-value/10325915#10325915
# if nothing done in main then no linker run, so below is same with above code.
$ echo "test_fs_reg" | xargs -I@ gcc -g -no-pie -fno-pic -o czg_scripts/@ czg_scripts/@.c
[czg ~]$ r2 -AA -qc "pdf @ main" czg_scripts/test_fs_reg 
INFO: Analyze all flags starting with sym. and entry0 (aa)
INFO: Analyze all functions arguments/locals (afva@@@F)
INFO: Analyze function calls (aac)
INFO: Analyze len bytes of instructions for references (aar)
INFO: Finding and parsing C++ vtables (avrr)
INFO: Type matching analysis for all functions (aaft)
INFO: Propagate noreturn information (aanr)
INFO: Integrate dwarf function information
INFO: Scanning for strings constructed in code (/azs)
INFO: Finding function preludes (aap)
INFO: Enable anal.types.constraint for experimental type propagation
            ;-- main:
            ; DATA XREF from entry0 @ 0x401058(r)
┌ 25: int dbg.main (int argc, char **argv);
│           ; arg int argc @ rdi
│           ; arg char **argv @ rsi
│           ; var int64_t var_4h @ rbp-0x4
│           ; var char **var_10h @ rbp-0x10
│           0x00401126      55             push rbp                    ; stdlib.h:6    modify it under the terms of the GNU Lesser General Public ; push word, doubleword or quadword onto the stack; int main(int argc,char ** argv);
│           0x00401127      4889e5         mov rbp, rsp                ; moves data from src to dst
│           0x0040112a      4883ec10       sub rsp, 0x10               ; substract src and dst, stores result on dst
│           0x0040112e      897dfc         mov dword [var_4h], edi     ; moves data from src to dst; argc
│           0x00401131      488975f0       mov qword [var_10h], rsi    ; moves data from src to dst; argv
│           0x00401135      bf00000000     mov edi, 0                  ; stdlib.h:7    License as published by the Free Software Foundation; either ; moves data from src to dst; int status
└           0x0040113a      e8f1feffff     call sym.imp.exit           ; calls a subroutine, push eip into the stack (esp) ; void exit(int status)
# here no canary

$ cd /mnt/ubuntu/home/czg/csapp3e/asm;gcc -fno-pic -Og -o prog 010-main.c 010-mstore.c -g3 -no-pie -fno-stack-protector 
$ r2 -AA -qc "pdf @ main>./no_canary_prog.asm" /mnt/ubuntu/home/czg/csapp3e/asm/prog
# in /mnt/ubuntu/home/czg/csapp3e/asm/prog/no_canary_prog.asm
# in asm here not use `rax`
...
$ r2 -AA -qc "pdf @ main>./canary_prog.asm" /mnt/ubuntu/home/czg/csapp3e/asm/prog
```

#### fs related opcode
```asm
[0x0040113a]> pd-- 2
│       └─< 0x00401134      eb8a           jmp 0x4010c0                ; jump
            ;-- main:
            ; DATA XREF from entry0 @ 0x401068(r)
┌ 107: int dbg.main (int argc, char **argv, char **envp);
│ rg: 0 (vars 0, args 0)
│ bp: 1 (vars 1, args 0)
│ sp: 1 (vars 1, args 0)
│           0x00401136      4883ec18       sub rsp, 0x18               ; stdio.h:6    modify it under the terms of the GNU Lesser General Public ; substract src and dst, stores result on dst; int main();
│           0x0040113a b    64488b042528.  mov rax, qword fs:[0x28]    ; stdio.h:6    modify it under the terms of the GNU Lesser General Public ; moves data from src to dst
[0x0040113a]> px @fs+0x28
- offset -      6869 6A6B 6C6D 6E6F 7071 7273 7475 7677  89ABCDEF01234567
0x7f4718698768  0026 f2ec 9cde 433e 4b34 6aa3 7e6d 23c8  .&....C>K4j.~m#.
...
[0x0040113a]> pxr @fs+0x28
0x7f4718698768 0x3e43de9cecf22600   .&....C> rax
```
- `64488b042528`
  - `64` in [pdf p532](../references/intel_64.pdf) -> [legacy](https://wiki.osdev.org/X86-64_Instruction_Encoding#Legacy_Prefixes) prefix, just to [overload ‘They are still registers, so they can of course be used for various other assembly operations.’ ](https://reverseengineering.stackexchange.com/questions/2006/how-are-the-segment-registers-fs-gs-cs-ss-ds-es-used-in-linux) default segmentation usage
  - `48` see above `REX.W`
  - rest are MODR/M, same with above
#### shown in [this](https://www.felixcloutier.com/x86/push.html)
- [this](http://www.c-jump.com/CIS77/CPU/x86/X77_0240_prefix.htm) and [pdf p532](../references/intel_64.pdf) said `66h` prefix usage in SSE (related with AVX [pdf p537](../references/intel_64.pdf))
  - [multiple](https://stackoverflow.com/questions/2404364/combining-prefixes-in-sse) 66H prefix
### [`.rela.plt`](https://reverseengineering.stackexchange.com/questions/21521/pyelftools-relocation-section-symbols) <a id="rela"></a>
> [TODO .plt.got (it seems no use anymore)](https://systemoverlord.com/2017/03/19/got-and-plt-for-pwning.html)
```asm
[0x00401000]> iS~plt
11  0x000005d8    0x30 0x004005d8    0x30 -r-- RELA        .rela.plt
13  0x00001020    0x30 0x00401020    0x30 -r-x PROGBITS    .plt
23  0x00002fe8    0x28 0x00403fe8    0x28 -rw- PROGBITS    .got.plt
;obviously this is used when first search `reloc.printf` when linker not work
[0x00401000]> pd 0xf@0x00000000004005d8
            ;-- section..rela.plt:
            0x004005d8      .qword 0x0000000000404000 ; reloc.__stack_chk_fail ; [11] -r-- section size 48 named .rela.plt
            0x004005e0      .qword 0x0000000300000007
            0x004005e8      .qword 0x0000000000000000
            0x004005f0      .qword 0x0000000000404008 ; reloc.printf
            0x004005f8      .qword 0x0000000400000007
            0x00400600      .qword 0x0000000000000000
            0x00400608      0000           add byte [rax], al          ; adds src and dst, stores result on dst ; rbp
            0x0040060a      0000           add byte [rax], al          ; adds src and dst, stores result on dst ; rbp ; rbp ; rbp
            0x0040060c      0000           add byte [rax], al          ; adds src and dst, stores result on dst ; rbp
            0x0040060e      0000           add byte [rax], al          ; adds src and dst, stores result on dst ; rbp
            0x00400610      0000           add byte [rax], al          ; adds src and dst, stores result on dst ; rbp
            0x00400612      0000           add byte [rax], al          ; adds src and dst, stores result on dst ; rbp ; rbp ; rbp
            0x00400614      0000           add byte [rax], al          ; adds src and dst, stores result on dst ; rbp
            0x00400616      0000           add byte [rax], al          ; adds src and dst, stores result on dst ; rbp
            0x00400618      0000           add byte [rax], al          ; adds src and dst, stores result on dst ; rb
[0x00401000]> pxr @0x404008
0x00404008 0x0000000000401046   F.@..... @ reloc.printf 4198470 /mnt/ubuntu/home/czg/csapp3e/asm/prog .plt program R X 'push 1' 'prog'
0x00404010 ..[ null bytes ]..   00000000 loc.__data_start
[0x00401000]> axt 0x404008
sym.imp.printf 0x401040 [DATA:--x] jmp qword [reloc.printf]
[0x00401000]> axt 0x401040
dbg.main 0x40116a [CALL:--x] call sym.imp.printf
dbg.main 0x40117d [CALL:--x] call sym.imp.printf
[0x00401000]> pd 3@0x401040
        ╎   ; CALL XREFS from dbg.main @ 0x40116a(x), 0x40117d(x)
╭ 6: int sym.imp.printf (const char *format);
│ rg: 0 (vars 0, args 0)
│ bp: 0 (vars 0, args 0)
│ sp: 0 (vars 0, args 0)
╰       ╎   0x00401040      ff25c22f0000   jmp qword [reloc.printf]    ; [0x404008:8]=0x401046 ; "F\x10@" ; jump
        ╎   0x00401046      6801000000     push 1                      ; rbp ; push word, doubleword or quadword onto the stack
        ╰─< 0x0040104b      e9d0ffffff     jmp section..plt            ; jump
; and also got has the same table with `.rela.plt` at the beginning
[0x7f2361666ed0]> pxr @0x0000000000403fe8
0x00403fe8 0x0000000000403df8   .=@..... @ obj._GLOBAL_OFFSET_TABLE_ 4210168 /mnt/ubuntu/home/czg/csapp3e/asm/prog .dynamic section..dynamic,segment.DYNAMIC,_DYNAMIC program R W 0x1
0x00403ff0 ..[ null bytes ]..   00000000 
0x00404000 0x0000000000401036   6.@..... @ reloc.__stack_chk_fail 4198454 /mnt/ubuntu/home/czg/csapp3e/asm/prog .plt program R X 'push 0' 'prog'
0x00404008 0x0000000000401046   F.@..... @ reloc.printf 4198470 /mnt/ubuntu/home/czg/csapp3e/asm/prog .plt program R X 'push 1' 'prog'
0x00404010 ..[ null bytes ]..   00000000 loc.__data_start
[0x7f2361666ed0]> iS~403
19  0x00002de8     0x8 0x00403de8     0x8 -rw- INIT_ARRAY  .init_array
20  0x00002df0     0x8 0x00403df0     0x8 -rw- FINI_ARRAY  .fini_array
21  0x00002df8   0x1d0 0x00403df8   0x1d0 -rw- DYNAMIC     .dynamic
; after update, in pwndbg
────────────────────────────────────────────────────────────────[ DISASM / x86-64 / set emulate on ]─────────────────────────────────────────────────────────────────
 ►0x401040      <printf@plt>   jmp    qwordptr[rip+0x2fc2]          <printf>
    ↓
  0x7ffff7dc7e00<printf>       endbr64 
  0x7ffff7dc7e04<printf+4>     sub    rsp,0xd8
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
pwndbg> telescope 0x0000000000403fe8
00:0000│  0x403fe8 (_GLOBAL_OFFSET_TABLE_) —▸ 0x403df8 (_DYNAMIC) ◂— 0x1
01:0008│  0x403ff0 (_GLOBAL_OFFSET_TABLE_+8) —▸ 0x7ffff7ffe2c0 ◂— 0x0
02:0010│  0x403ff8 (_GLOBAL_OFFSET_TABLE_+16) —▸ 0x7ffff7fdcb50 ◂— endbr64 
03:0018│  0x404000 (__stack_chk_fail@got.plt) —▸ 0x401036 (__stack_chk_fail@plt+6) ◂— push 0 /* 'h' */
04:0020│  0x404008 (printf@got[plt]) —▸ 0x7ffff7dc7e00 (printf) ◂— endbr64 
05:0028│  0x404010 (data_start) ◂— 0x0
pwndbg> telescope 0x00000000004005d8
00:0000│  0x4005d8 —▸ 0x404000 (__stack_chk_fail@got.plt) —▸ 0x401036 (__stack_chk_fail@plt+6) ◂— push 0 /* 'h' */
01:0008│  0x4005e0 ◂— 0x300000007
02:0010│  0x4005e8 ◂— 0x0
03:0018│  0x4005f0 —▸ 0x404008 (printf@got[plt]) —▸ 0x7ffff7dc7e00 (printf) ◂— endbr64 
04:0020│  0x4005f8 ◂— 0x400000007
05:0028│  0x400600 ◂— 0x0
... ↓     2 skipped
```
#### how ld work is a bit complex (here give a scratch)
```asm
[0x00401000]> pd-- 4@section..plt
╰           0x0040101a      c3             ret                         ; return from subroutine. pop 4 bytes from esp and jump there.
            0x0040101b      0000           add byte [rax], al          ; adds src and dst, stores result on dst ; rbp ; rbp ; rbp
            0x0040101d      0000           add byte [rax], al          ; adds src and dst, stores result on dst ; rbp
            0x0040101f  ~   00ff           add bh, bh                  ; adds src and dst, stores result on dst ; rbp ; rbp ; rbp
            ;-- section..plt:
            0x00401020 b    ff35ca2f0000   push qword [0x00403ff0]     ; push word, doubleword or quadword onto the stack; [13] -r-x section size 48 named .plt
            0x00401026      ff25cc2f0000   jmp qword [fcn.7f903acb2b50] ; [0x403ff8:8]=0x7f903acb2b50 fcn.7f903acb2b50 ; jump
            0x0040102c      0f1f4000       nop dword [rax]             ; no operation
[0x00401000]> agf @fcn.7f903acb2b50
...
│ call fcn.7f903acb06e0;[oa]    
│ mov r11, rax    
│ jmp r11
; fcn.7f903acb06e0 is too long
; in pwndbg
────────────────────────────────────────────────────────────────[ DISASM / x86-64 / set emulate on ]─────────────────────────────────────────────────────────────────
 ►0x7ffff7fdcc0a             jmp    r11                           <printf>
    ↓
  0x7ffff7dc7e00<printf>     endbr64 
pwndbg> hexdump 0x404008

+0000 0x404008  00 7e dc f7 ff 7f 00 00  00 00 00 00 00 00 00 00  │.~......│........│
; has link to 0x7ffff7dc7e00
```
#### `.plt.got` seems to be set to `.got` in elf
```asm
; https://stackoverflow.com/questions/58076539/plt-plt-got-what-is-different
[0x7f2361666ed0]> iS~dyn
6   0x000003e0    0xa8 0x004003e0    0xa8 -r-- DYNSYM      .dynsym
7   0x00000488    0x9b 0x00400488    0x9b -r-- STRTAB      .dynstr
10  0x00000578    0x60 0x00400578    0x60 -r-- RELA        .rela.dyn
21  0x00002df8   0x1d0 0x00403df8   0x1d0 -rw- DYNAMIC     .dynamic
; four reloc in .rela.dyn
[0x7f2361666ed0]> pd 0xf4@0x00400578
            ;-- section..rela.dyn:
            0x00400578      .qword 0x0000000000403fc8 ; section..got ; reloc.__libc_start_main ; [10] -r-- section size 96 named .rela.dyn
            0x00400580      .qword 0x0000000100000006
            0x00400588      .qword 0x0000000000000000
            0x00400590      .qword 0x0000000000403fd0 ; reloc._ITM_deregisterTMCloneTable
            0x00400598      .qword 0x0000000200000006
            0x004005a0      .qword 0x0000000000000000
            0x004005a8      .qword 0x0000000000403fd8 ; reloc.__gmon_start__
            0x004005b0      .qword 0x0000000500000006
            0x004005b8      .qword 0x0000000000000000
            0x004005c0      .qword 0x0000000000403fe0 ; reloc._ITM_registerTMCloneTable
            0x004005c8      .qword 0x0000000600000006
            0x004005d0      .qword 0x0000000000000000
; here no got for __libc_start_main
[0x00403fc8]> pxr 
0x00403fc8 ..[ null bytes ]..   00000000 reloc.__libc_start_main
0x00403fe8 0x0000000000403df8   .=@..... @ obj._GLOBAL_OFFSET_TABLE_ 4210168 /mnt/ubuntu/home/czg/csapp3e/asm/prog .dynamic section..dynamic,segment.DYNAMIC,_DYNAMIC program R W 0x1
0x00403ff0 ..[ null bytes ]..   00000000 
0x00404000 0x0000000000401036   6.@..... @ reloc.__stack_chk_fail 4198454 /mnt/ubuntu/home/czg/csapp3e/asm/prog .plt program R X 'push 0' 'prog'
0x00404008 0x0000000000401046   F.@..... @ reloc.printf 4198470 /mnt/ubuntu/home/czg/csapp3e/asm/prog .plt program R X 'push 1' 'prog'
0x00404010 ..[ null bytes ]..   00000000 loc.__data_start
; after link
[0x00403fc8]> pxr
0x00403fc8 0x00007fde0eac77c0   .w...... @ reloc.__libc_start_main /usr/lib/libc.so.6 library R X 'endbr64' 'libc.so.6'
; others still zero
0x00403fd0 ..[ null bytes ]..   00000000 reloc._ITM_deregisterTMCloneTable
0x00403fe8 0x0000000000403df8   .=@..... @ obj._GLOBAL_OFFSET_TABLE_ 4210168 /mnt/ubuntu/home/czg/csapp3e/asm/prog .dynamic section..dynamic,segment.DYNAMIC,_DYNAMIC program R 0x1
[0x00403fc8]> pd 4
            ;-- section..got:
            ;-- reloc.__libc_start_main:
            ; CALL XREF from entry0 @ 0x40106f(x)
            0x00403fc8      .qword 0x00007fde0eac77c0                  ; RELOC 64 __libc_start_main ; [22] -rw- section size 32 named .got
            ;-- reloc._ITM_deregisterTMCloneTable:
            ; DATA XREF from fcn.00401090 @ 0x40109d(r)
            0x00403fd0      .qword 0x0000000000000000                  ; RELOC 64 _ITM_deregisterTMCloneTable
            ;-- reloc.__gmon_start__:
            ; DATA XREF from sym._init @ 0x401008(r)
            0x00403fd8      .qword 0x0000000000000000                  ; RELOC 64 __gmon_start__
            ;-- reloc._ITM_registerTMCloneTable:
            ; DATA XREF from entry.init0 @ 0x4010df(r)
            0x00403fe0      .qword 0x0000000000000000                  ; RELOC 64 _ITM_registerTMCloneTable
[0x00403fc8]> iS.
Current section

nth paddr       size vaddr       vsize perm type     name
―――――――――――――――――――――――――――――――――――――――――――――――――――――――――
0   0x00002fc8  0x20 0x00403fc8   0x20 -rw- PROGBITS .got

;readelf
[czg /mnt/ubuntu/home/czg/csapp3e/asm]$ readelf -r prog

Relocation section '.rela.dyn' at offset 0x578 contains 4 entries:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
000000403fc8  000100000006 R_X86_64_GLOB_DAT 0000000000000000 __libc_start_main@GLIBC_2.34 + 0
000000403fd0  000200000006 R_X86_64_GLOB_DAT 0000000000000000 _ITM_deregisterTM[...] + 0
000000403fd8  000500000006 R_X86_64_GLOB_DAT 0000000000000000 __gmon_start__ + 0
000000403fe0  000600000006 R_X86_64_GLOB_DAT 0000000000000000 _ITM_registerTMCl[...] + 0

Relocation section '.rela.plt' at offset 0x5d8 contains 2 entries:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
000000404000  000300000007 R_X86_64_JUMP_SLO 0000000000000000 __stack_chk_fail@GLIBC_2.4 + 0
000000404008  000400000007 R_X86_64_JUMP_SLO 0000000000000000 printf@GLIBC_2.2.5 + 0
;pwndbg
pwndbg> info symbol 0x00403fc8
No symbol matches 0x00403fc8.
pwndbg> info symbol 0x0000000000404000
__stack_chk_fail@got.plt in section .got.plt of /mnt/ubuntu/home/czg/csapp3e/asm/prog
```
## 9
```asm
  0x7ffff7fe5d7f  test   rax,rax
 ►0x7ffff7fe5d82  je     0x7ffff7fe5dff                <0x7ffff7fe5dff>
```
[jump](https://stackoverflow.com/questions/13064809/the-point-of-test-eax-eax) if return 0
## 10 TODO how `leave` implemented in cpu [pdf p178](../references/intel_64.pdf), [related](https://stackoverflow.com/questions/67449884/mov-rbp-rsp-in-function-prologue) with `push rbp` above
- ~~TODO~~ why gcc [not](https://stackoverflow.com/questions/29790175/assembly-x86-leave-instruction) use enter
  - [this](https://www.agner.org/optimize/instruction_tables.pdf) p151 (~~TODO execution pipe~~?)
    - po Latency based on multiple instructions runned (dependency chain) and Reciprocal throughput is mostly based on itself (detailed see p2)
    - p3,36(execution pipe)
      - Execution pipe is [pipelined](https://en.wikipedia.org/wiki/Execution_unit) Execution units
    - here `MOV r64,r64 1 0 0.25 renaming` -> `0 0.25` and `ENTER a,b 11+3b N/A ~20+b` (multiple nested condition) with `LEAVE N/A 2 N/A 3`
    - shuffle [shufps](http://www.songho.ca/misc/sse/sse.html)
- in pdf above, rbp is mainly used to access parent variable in stack.
```bash
*RBP  0x7fffffffdfc0 ◂— 0x1                                                         *RSP  0x7fffffffdfc0 ◂— 0x1
*RIP  0x40119a (main+4) ◂— sub rsp,0x10                                              EFLAGS 0x246 [ cf PF af ZF sf IF df of ]
──────────────────────────────────────────────────────────────────────────────[ STACK ]──────────────────────────────────────────────────────────────────────────────
00:0000│ rbp rsp 0x7fffffffdfc0 ◂— 0x1
01:0008│         0x7fffffffdfc8 —▸ 0x7ffff7d9b790 ◂— mov edi,eax
02:0010│         0x7fffffffdfd0 —▸ 0x7ffff7ffd000 (_rtld_global) —▸ 0x7ffff7ffe2c0 ◂— 0x0
03:0018│         0x7fffffffdfd8 —▸ 0x401196 (main) ◂— push rbp
04:0020│         0x7fffffffdfe0 ◂— 0x100400040 /* '@' */
05:0028│         0x7fffffffdfe8 —▸ 0x7fffffffe0d8 —▸ 0x7fffffffe548 ◂— '/mnt/ubuntu/home/czg/csapp3e/asm/prog'
06:0030│         0x7fffffffdff0 —▸ 0x7fffffffe0d8 —▸ 0x7fffffffe548 ◂— '/mnt/ubuntu/home/czg/csapp3e/asm/prog'
07:0038│         0x7fffffffdff8 ◂— 0xa8242b04de4995fd
────────────────────────────────────────────────────────────────[ DISASM / x86-64 / set emulate on ]─────────────────────────────────────────────────────────────────
  0x401197<main+1>   mov    rbp,rsp
 ►0x40119a<main+4>   sub    rsp,0x10
# after some run
──────────────────────────────────────────────────────────────────────────────[ STACK ]──────────────────────────────────────────────────────────────────────────────
00:0000│ rsp 0x7fffffffdfb0 ◂— 0x6
01:0008│     0x7fffffffdfb8 ◂— 0x58a4da165677c400
02:0010│ rbp 0x7fffffffdfc0 ◂— 0x1
...
pwndbg> dr
Dump of assembler code for function main:
   0x0000000000401196 <+0>:     55                      push   rbp
   0x0000000000401197 <+1>:     48 89 e5                mov    rbp,rsp
...
 ►0x4011ff<main+105>   leave
>si
*RBP  0x1                                                                           *RSP  0x7fffffffdfc8 —▸ 0x7ffff7d9b790 ◂— mov edi,eax
*RIP  0x401200 (main+106) ◂— ret                                                     EFLAGS 0x246 [ cf PF af ZF sf IF df of ]
──────────────────────────────────────────────────────────────────────────────[ STACK ]──────────────────────────────────────────────────────────────────────────────
00:0000│ rsp 0x7fffffffdfc8 —▸ 0x7ffff7d9b790 ◂— mov edi,eax
01:0008│     0x7fffffffdfd0 —▸ 0x7ffff7ffd000 (_rtld_global) —▸ 0x7ffff7ffe2c0 ◂— 0x0
02:0010│     0x7fffffffdfd8 —▸ 0x401196 (main) ◂— push rbp
03:0018│     0x7fffffffdfe0 ◂— 0x100400040 /* '@' */
04:0020│     0x7fffffffdfe8 —▸ 0x7fffffffe0d8 —▸ 0x7fffffffe548 ◂— '/mnt/ubuntu/home/czg/csapp3e/asm/prog'
05:0028│     0x7fffffffdff0 —▸ 0x7fffffffe0d8 —▸ 0x7fffffffe548 ◂— '/mnt/ubuntu/home/czg/csapp3e/asm/prog'
06:0030│     0x7fffffffdff8 ◂— 0x6b2a7c5df1500785
07:0038│     0x7fffffffe000 ◂— 0x0
────────────────────────────────────────────────────────────────[ DISASM / x86-64 / set emulate on ]─────────────────────────────────────────────────────────────────
  0x4011ff      <main+105>   leave  
 ►0x401200      <main+106>   ret    <0x7ffff7d9b790>
# stack same with <main>,below is when main start 
*RIP  0x401196 (main) ◂— push rbp                                                    EFLAGS 0x246 [ cf PF af ZF sf IF df of ]
──────────────────────────────────────────────────────────────────────────────[ STACK ]──────────────────────────────────────────────────────────────────────────────
00:0000│ rsp 0x7fffffffdfc8 —▸ 0x7ffff7d9b790 ◂— mov edi,eax
01:0008│     0x7fffffffdfd0 —▸ 0x7ffff7ffd000 (_rtld_global) —▸ 0x7ffff7ffe2c0 ◂— 0x0
02:0010│     0x7fffffffdfd8 —▸ 0x401196 (main) ◂— push rbp
03:0018│     0x7fffffffdfe0 ◂— 0x100400040 /* '@' */
04:0020│     0x7fffffffdfe8 —▸ 0x7fffffffe0d8 —▸ 0x7fffffffe548 ◂— '/mnt/ubuntu/home/czg/csapp3e/asm/prog'
05:0028│     0x7fffffffdff0 —▸ 0x7fffffffe0d8 —▸ 0x7fffffffe548 ◂— '/mnt/ubuntu/home/czg/csapp3e/asm/prog'
06:0030│     0x7fffffffdff8 ◂— 0xe9b166c2d1e3c3da
07:0038│     0x7fffffffe000 ◂— 0x0
────────────────────────────────────────────────────────────────[ DISASM / x86-64 / set emulate on ]─────────────────────────────────────────────────────────────────
 ►0x401196<main>     push   rbp
```
### `r8-r15` [this](https://stackoverflow.com/questions/59660118/assembly-using-rn-registers-r8-r9-have-more-code-size-than-other-registers) [rax,etc relation with r8-15 see pdf p2846](../references/intel_64.pdf)
- above  `0x4a` -> `0b01001010` -> Figure 2-4 [pdf p533](../references/intel_64.pdf)
- rasm2 [bug](https://reverseengineering.stackexchange.com/questions/18024/rasm2-giving-different-output-each-time-for-x64) still exists ,`R` -> `reg`
```bash
[0x00000100]> rasm2 -d 418b4104
mov eax, dword [r9 + 4]
# the above is right
[0x00000100]> rasm2 -a x86 -b 64 "mov eax, dword [r9 + 4]" # ignore this
448b4a04
$ rasm2 -L
...
# no x86 but x86.nasm
a__ 16 32 64    x86.nasm    LGPL3   X86 nasm assembler
a__ 16 32 64    x86.nz      LGPL3   x86 handmade assembler (by pancake)
...
$ rasm2 -a x86.nasm -b 64 "mov eax, dword [r9 + 4]" 
418b4104
```
## 11 `printf` (int & double)
```bash
  0x40115b<main+37>   mov    eax,1
  0x401160<main+42>   call   printf@plt                      <printf@plt>
  # here eax is to check xmm reg, if not 0 then not skip
pwndbg> pdisass 
 ►0x7ffff7dc7e00<printf>      endbr64 
  0x7ffff7dc7e04<printf+4>    sub    rsp,0xd8
  0x7ffff7dc7e24<printf+36>   test   al,al
  0x7ffff7dc7e26<printf+38>   je     printf+95                <printf+95>
pwndbg> pdisass 0x7ffff7dc7e26
 ►0x7ffff7dc7e26<printf+38>   je     printf+95                <printf+95>
 
  0x7ffff7dc7e28<printf+40>   movaps xmmwordptr[rsp+0x50],xmm0
  0x7ffff7dc7e2d<printf+45>   movaps xmmwordptr[rsp+0x60],xmm1
  0x7ffff7dc7e32<printf+50>   movaps xmmwordptr[rsp+0x70],xmm2
  0x7ffff7dc7e37<printf+55>   movaps xmmwordptr[rsp+0x80],xmm3
  0x7ffff7dc7e3f<printf+63>   movaps xmmwordptr[rsp+0x90],xmm4
```
- [xmm/ymm](https://wiki.osdev.org/AVX2)/[zmm](https://wiki.osdev.org/CPU_Registers_x86) , ymm [pdf p538](../references/intel_64.pdf) zmm [pdf p603](../references/intel_64.pdf), see 
## fs/gs and SS segment(rsp/rbp)
[pdf p86](../references/intel_64.pdf), see [this (fs != fs_base)](https://stackoverflow.com/questions/59797987/how-is-effective-address-calculated-with-fs-and-gs-registers)
```bash
pwndbg> x/g $fs_base+0x28
0x7ffff7c8d768: 0x6bdb9d77fe8c7e00
────────────────────────────────────────────────────────────────[ DISASM / x86-64 / set emulate on ]─────────────────────────────────────────────────────────────────
  0x40113a<main+4>    mov    rax,qwordptrfs:[0x28]
 ►0x401143<main+13>   mov    qwordptr[rsp+8],rax
pwndbg> p *(long long*)($fs_base+0x28)==$rax
$7 = 0x1
pwndbg> x $fs+0x28
0x28:   Cannot access memory at address 0x28
```
### how fs:[0x28] canary check overflow [similar check method maybe in old gcc / other assembler](https://sourceware.org/pipermail/gdb/2020-March/048428.html)
```bash
pwndbg> dga main
Dump of assembler code for function main:
   0x0000000000401136 <+0>:     48 83 ec 18             sub    rsp,0x18
   0x000000000040113a <+4>:     64 48 8b 04 25 28 00 00 00      mov    rax,QWORD PTR fs:0x28
   0x0000000000401143 <+13>:    48 89 44 24 08          mov    QWORD PTR [rsp+0x8],rax
   0x0000000000401148 <+18>:    31 c0                   xor    eax,eax
  #  begin move parameter
   0x000000000040114a <+20>:    f2 0f 10 05 de 0e 00 00 movsd  xmm0,QWORD PTR [rip+0xede]        # 0x402030
   ...
   0x0000000000401160 <+42>:    e8 db fe ff ff          call   0x401040 <printf@plt>
  #  after every call the stack should look same as when `0x000000000040114a`
=> 0x0000000000401165 <+47>:    48 89 e2                mov    rdx,rsp
──────────────────────────────────────────────────────────────────────────────[ STACK ]──────────────────────────────────────────────────────────────────────────────
00:0000│ rsp 0x7fffffffdfb0 ◂— 0x0
# `0xace30b97c80c7c00` is canary `fs:[0x28]`
01:0008│     0x7fffffffdfb8 ◂— 0xace30b97c80c7c00
02:0010│     0x7fffffffdfc0 —▸ 0x7fffffffe0d8 —▸ 0x7fffffffe54a ◂— '/mnt/ubuntu/home/czg/csapp3e/asm/prog'
# return address (the next instruction)
03:0018│     0x7fffffffdfc8 —▸ 0x7ffff7d9b790 ◂— mov edi,eax
pwndbg> dga main
...
   0x000000000040119d <+103>:   48 8b 44 24 08          mov    rax,QWORD PTR [rsp+0x8]
   0x00000000004011a2 <+108>:   64 48 2b 04 25 28 00 00 00      sub    rax,QWORD PTR fs:0x28
  #  [rsp+0x8] should same as fs:0x28 if no overflow
   0x00000000004011ab <+117>:   75 0a                   jne    0x4011b7 <main+129>
   ...
   0x00000000004011b7 <+129>:   e8 74 fe ff ff          call   0x401030 <__stack_chk_fail@plt>

pwndbg> info br
Num     Type           Disp Enb Address            What
3       acc watchpoint keep y                      *0x7fffffffdfb0
        breakpoint already hit 4 times
4       acc watchpoint keep y                      *0x7fffffffdfb8
        breakpoint already hit 2 times
#  after debug,only read between call,except '<+47>', see below 'use rsp to access function call result'
   0x0000000000401165 <+47>:    48 89 e2                mov    rdx,rsp
   0x0000000000401168 <+50>:    be 03 00 00 00          mov    esi,0x3
   0x000000000040116d <+55>:    bf 02 00 00 00          mov    edi,0x2
   0x0000000000401172 <+60>:    e8 4d 00 00 00          call   0x4011c4 <multstore>
   0x0000000000401177 <+65>:    48 8b 34 24             mov    rsi,QWORD PTR [rsp]
   0x000000000040117b <+69>:    bf 1c 20 40 00          mov    edi,0x40201c
   0x0000000000401180 <+74>:    b8 00 00 00 00          mov    eax,0x0
   0x0000000000401185 <+79>:    e8 b6 fe ff ff          call   0x401040 <printf@plt>
# so '►0x4011a2' just to check overflow.
────────────────────────────────────────────────────────────────────────[ DISASM / x86-64 / set emulate on ]─────────────────────────────────────────────────────────────────────────
 ►0x4011a2<main+108>   sub    rax,qwordptrfs:[0x28]
  0x4011ab<main+117>   jne    main+129                      <main+129>

```
## use rsp to access function call result
```bash
──────────────────────────────────────────────────────────────────────────────────────[ STACK ]──────────────────────────────────────────────────────────────────────────────────────
00:0000│ rdx rsp 0x7fffffffdfb0 ◂— 0x0
01:0008│         0x7fffffffdfb8 ◂— 0xb30961705bab4800
02:0010│         0x7fffffffdfc0 —▸ 0x7fffffffe0d8 —▸ 0x7fffffffe54a ◂— '/mnt/ubuntu/home/czg/csapp3e/asm/prog'
03:0018│         0x7fffffffdfc8 —▸ 0x7ffff7d9b790 ◂— mov edi,eax
04:0020│         0x7fffffffdfd0 —▸ 0x7fffffffe0c0 —▸ 0x7fffffffe0c8 ◂— 0x38 /* '8' */
05:0028│         0x7fffffffdfd8 —▸ 0x401136 (main) ◂— sub rsp,0x18
06:0030│         0x7fffffffdfe0 ◂— 0x100400040 /* '@' */
07:0038│         0x7fffffffdfe8 —▸ 0x7fffffffe0d8 —▸ 0x7fffffffe54a ◂— '/mnt/ubuntu/home/czg/csapp3e/asm/prog'
────────────────────────────────────────────────────────────────────────[ DISASM / x86-64 / set emulate on ]─────────────────────────────────────────────────────────────────────────
  0x401165<main+47>   mov    rdx,rsp
 ►0x401168<main+50>   mov    esi,3
# RDX the third parameter -> result
   0x0000000000401165 <+47>:    48 89 e2                mov    rdx,rsp
   0x0000000000401168 <+50>:    be 03 00 00 00          mov    esi,0x3
   0x000000000040116d <+55>:    bf 02 00 00 00          mov    edi,0x2
   0x0000000000401172 <+60>:    e8 4d 00 00 00          call   0x4011c4 <multstore>
  #  pass to printf second param
   0x0000000000401177 <+65>:    48 8b 34 24             mov    rsi,QWORD PTR [rsp]
   0x000000000040117b <+69>:    bf 1c 20 40 00          mov    edi,0x40201c
   0x0000000000401180 <+74>:    b8 00 00 00 00          mov    eax,0x0
   0x0000000000401185 <+79>:    e8 b6 fe ff ff          call   0x401040 <printf@plt>

pwndbg> dr main
Dump of assembler code for function multstore:
   0x00000000004011c4 <+0>:     53                      push   rbx
=> 0x00000000004011c5 <+1>:     48 89 d3                mov    rbx,rdx
──────────────────────────────────────────────────────────────────────────────────────[ STACK ]──────────────────────────────────────────────────────────────────────────────────────
00:0000│ rsp 0x7fffffffdfa0 —▸ 0x7fffffffe0d8 —▸ 0x7fffffffe54a ◂— '/mnt/ubuntu/home/czg/csapp3e/asm/prog'
01:0008│     0x7fffffffdfa8 —▸ 0x401177 (main+65) ◂— mov rsi,qwordptr[rsp]
02:0010│ rdx 0x7fffffffdfb0 ◂— 0x0
pwndbg> dr
Dump of assembler code for function multstore:
  #  push callee reg
   0x00000000004011c4 <+0>:     53                      push   rbx
  # use caller reg
   0x00000000004011c5 <+1>:     48 89 d3                mov    rbx,rdx
=> 0x00000000004011c8 <+4>:     e8 ef ff ff ff          call   0x4011bc <mult2>
  # here is just to rdx -> rsp before/after call multstore
   0x00000000004011cd <+9>:     48 89 03                mov    QWORD PTR [rbx],rax
  #  rbx is just at top, restore it
   0x00000000004011d0 <+12>:    5b                      pop    rbx
   0x00000000004011d1 <+13>:    c3                      ret
End of assembler dump.
# mult2 no change to stack
pwndbg> dga mult2
Dump of assembler code for function mult2:
   0x00000000004011bc <+0>:     48 89 f8                mov    rax,rdi
   0x00000000004011bf <+3>:     48 0f af c6             imul   rax,rsi
   0x00000000004011c3 <+7>:     c3                      ret
End of assembler dump.

```
### here `rbx` is redundant to use, use `r2` to change binary exe because [not](https://stackoverflow.com/questions/44534733/why-copy-the-same-value-to-rax-that-he-already-has) optimized to be better debugged
```asm
$ r2 -AAq -c 'db sym.multstore;dc;wx 90909090@0x004011c4;wx 488902@0x004011cd;wx 90@0x004011d0;db 0x000000000040118a;dc;pd 4@0x004011cd;pd 8@0x004011c4;pd-- 1@0x004011d0;dc;dc' -d /mnt/ubuntu/home/czg/csapp3e/asm/prog 
2 * 3 --> 0
test: 2.468000
INFO: hit breakpoint at: 0x4011c4
2 * 3 --> 6
INFO: hit breakpoint at: 0x40118a
│           0x004011cd      488902         mov qword [rdx], rax        ; stdc-predef.h:6    License as published by the Free Software Foundation; either ; moves data from src to dst
│           0x004011d0      90             nop                         ; stdc-predef.h:7    version 2.1 of the License, or (at your option) any later version. ; no operation
╰           0x004011d1      c3             ret                         ; return from subroutine. pop 4 bytes from esp and jump there.
            0x004011d2      0000           add byte [rax], al          ; adds src and dst, stores result on dst
        ╎   ;-- multstore:
        ╎   ; CALL XREF from dbg.main @ 0x401172(x)
╭ 14: dbg.multstore (int64_t arg1, int64_t arg3);
│       ╎   ; arg int64_t arg1 @ rdi
│       ╎   ; arg int64_t arg3 @ rdx
│       ╎   0x004011c4 b    53             push rbx                    ; stdio.h:20 #define _BITS_STDIO_H 1 ; push word, doubleword or quadword onto the stack; void multstore(long int x,long int y,long int * dest);
│       ╎   0x004011c5      90             nop                         ; no operation; arg3
│       ╎   0x004011c6      90             nop                         ; no operation
│       ╎   0x004011c7      90             nop                         ; no operation
│       ╰─< 0x004011c8      e8efffffff     call dbg.mult2              ; stdc-predef.h:5    modify it under the terms of the GNU Lesser General Public ; calls a subroutine, push eip into the stack (esp)
│           0x004011cd      488902         mov qword [rdx], rax        ; stdc-predef.h:6    License as published by the Free Software Foundation; either ; moves data from src to dst
│           0x004011d0      90             nop                         ; stdc-predef.h:7    version 2.1 of the License, or (at your option) any later version. ; no operation
╰           0x004011d1      c3             ret                         ; return from subroutine. pop 4 bytes from esp and jump there.
│           0x004011cd      488902         mov qword [rdx], rax        ; stdc-predef.h:6    License as published by the Free Software Foundation; either ; moves data from src to dst
│           0x004011d0      90             nop                         ; stdc-predef.h:7    version 2.1 of the License, or (at your option) any later version. ; no operation
ERROR: Run `aeim` to initialize a stack for the ESIL vm
2 * 3 --> 6
(6664) Process terminated with status 0
INFO: ==> Process finished
ERROR: Cannot continue, run ood?
```
## [eh_frame_hdr](https://stackoverflow.com/questions/14091231/what-do-the-eh-frame-and-eh-frame-hdr-sections-store-exactly)(hdr->header) -> linux [foundation](https://refspecs.linuxfoundation.org/LSB_3.0.0/LSB-Core-generic/LSB-Core-generic/ehframechpt.html) (`eh` maybe mean 'exception header')
[also](https://github.com/nbdd0121/unwinding) 
```bash
   0x000000000040114a <+20>:    f2 0f 10 05 de 0e 00 00 movsd  xmm0,QWORD PTR [rip+0xede]        # 0x402030
   0x0000000000401152 <+28>:    48 8b 34 24             mov    rsi,QWORD PTR [rsp]
pwndbg> telescope 0x401152+0xede
00:0000│  0x402030 ◂— 0x4003be76c0000000
01:0008│  0x402038 (__GNU_EH_FRAME_HDR) ◂— 0x3c3b031b01
```
## how xmm save [pdf p52](../references/intel_64.pdf)
[pdf p67](../references/intel_64.pdf) -> 8 regs when non-64 mode
16 regs [pdf p70](../references/intel_64.pdf)
- related with stack
```bash
# 64/8=8 byte, can test with hexdump
   0x00007ffff7dc7e0b <+11>:    48 89 74 24 28          mov    QWORD PTR [rsp+0x28],rsi
   0x00007ffff7dc7e10 <+16>:    48 89 54 24 30          mov    QWORD PTR [rsp+0x30],rdx
  #  similar
   0x00007ffff7dc7e28 <+40>:    0f 29 44 24 50          movaps XMMWORD PTR [rsp+0x50],xmm0
```
- xmm
```bash
   0x000000000040114a <+20>:    f2 0f 10 05 de 0e 00 00 movsd  xmm0,QWORD PTR [rip+0xede]        # 0x402030
   0x0000000000401152 <+28>:    48 8b 34 24             mov    rsi,QWORD PTR [rsp]

pwndbg> hexdump 0x0000000000401152+0xede
+0000 0x402030  00 00 00 c0 76 be 03 40  01 1b 03 3b 3c 00 00 00  │....v..@│...;<...│
# so 0x0->minimal memory address and then grow to 0x4,etc
pwndbg> p $ymm0
$1 = {
  v16_bfloat16 = {[0x0] = 0, [0x1] = -2, [0x2] = -0.2402, [0x3] = 2.047, [0x4] = 0 <repeats 12 times>},
  v16_half = {[0x0] = 0, [0x1] = -2, [0x2] = -1.6152, [0x3] = 2.0059, [0x4] = 0 <repeats 12 times>},
  v8_float = {[0x0] = -2, [0x1] = 2.05849981, [0x2] = 0, [0x3] = 0, [0x4] = 0, [0x5] = 0, [0x6] = 0, [0x7] = 0},
  v4_double = {[0x0] = 2.4679999351501465, [0x1] = 0, [0x2] = 0, [0x3] = 0},
  v32_int8 = {[0x0] = 0x0, [0x1] = 0x0, [0x2] = 0x0, [0x3] = 0xc0, [0x4] = 0x76, [0x5] = 0xbe, [0x6] = 0x3, [0x7] = 0x40, [0x8] = 0x0 <repeats 24 times>},
  v16_int16 = {[0x0] = 0x0, [0x1] = 0xc000, [0x2] = 0xbe76, [0x3] = 0x4003, [0x4] = 0x0 <repeats 12 times>},
  v8_int32 = {[0x0] = 0xc0000000, [0x1] = 0x4003be76, [0x2] = 0x0, [0x3] = 0x0, [0x4] = 0x0, [0x5] = 0x0, [0x6] = 0x0, [0x7] = 0x0},
  v4_int64 = {[0x0] = 0x4003be76c0000000, [0x1] = 0x0, [0x2] = 0x0, [0x3] = 0x0},
  v2_int128 = {[0x0] = 0x4003be76c0000000, [0x1] = 0x0}
}
```
## printf with `xmm`
- test: only read xmm0 (TODO libc too many calls，read C source code) 
```bash
──────────────────────────────────────────────────────────────────────────────────────[ STACK ]──────────────────────────────────────────────────────────────────────────────────────
00:0000│ rsp 0x7fffffffded0 ◂— 0x0
01:0008│     0x7fffffffded8 ◂— 0x8000
02:0010│     0x7fffffffdee0 —▸ 0x7fffffffdf78 —▸ 0x7fffffffe8c3 ◂— '/usr/lib/liblua5.4.so'
03:0018│     0x7fffffffdee8 ◂— 0x80000
04:0020│     0x7fffffffdef0 ◂— 0x300000
05:0028│     0x7fffffffdef8 ◂— 0x0
06:0030│     0x7fffffffdf00 —▸ 0x7fffffffe0e8 —▸ 0x7fffffffe570 ◂— 'BEMENU_BACKEND=wayland'
07:0038│     0x7fffffffdf08 —▸ 0x403df0 —▸ 0x401100 ◂— endbr64 
────────────────────────────────────────────────────────────────────────[ DISASM / x86-64 / set emulate on ]─────────────────────────────────────────────────────────────────────────
 ►0x7ffff7dc7e32<printf+50>   movaps xmmwordptr[rsp+0x70],xmm2
  0x7ffff7dc7e37<printf+55>   movaps xmmwordptr[rsp+0x80],xmm3
  0x7ffff7dc7e3f<printf+63>   movaps xmmwordptr[rsp+0x90],xmm4
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
pwndbg> info br
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x00007ffff7dc7e00 <printf>
        breakpoint already hit 1 time
2       breakpoint     keep y   0x00007fffffffdc48 
3       breakpoint     keep y   0x00007ffff7e6e9c0 <write>
4       acc watchpoint keep y                      *(0x7fffffffded0+0x50)
        breakpoint already hit 1 time
5       acc watchpoint keep y                      *(0x7fffffffded0+0x60)
        breakpoint already hit 1 time
```
### parameter scratch
```bash
pwndbg> dga 0x00007ffff7dc7e68,0x00007ffff7dc7ea7+0x1
Dump of assembler code from 0x7ffff7dc7e68 to 0x7ffff7dc7ea8:
   0x00007ffff7dc7e68 <printf+104>:     48 89 44 24 18          mov    QWORD PTR [rsp+0x18],rax
   0x00007ffff7dc7e6d <printf+109>:     31 c0                   xor    eax,eax
   0x00007ffff7dc7e6f <printf+111>:     48 8d 84 24 e0 00 00 00 lea    rax,[rsp+0xe0]
   0x00007ffff7dc7e77 <printf+119>:     48 89 e2                mov    rdx,rsp
   0x00007ffff7dc7e7a <printf+122>:     31 c9                   xor    ecx,ecx
   0x00007ffff7dc7e7c <printf+124>:     48 89 44 24 08          mov    QWORD PTR [rsp+0x8],rax
   0x00007ffff7dc7e81 <printf+129>:     48 8d 44 24 20          lea    rax,[rsp+0x20]
   0x00007ffff7dc7e86 <printf+134>:     48 89 fe                mov    rsi,rdi
   0x00007ffff7dc7e89 <printf+137>:     48 89 44 24 10          mov    QWORD PTR [rsp+0x10],rax
   0x00007ffff7dc7e8e <printf+142>:     48 8b 05 f3 7e 18 00    mov    rax,QWORD PTR [rip+0x187ef3]        # 0x7ffff7f4fd88
   0x00007ffff7dc7e95 <printf+149>:     c7 04 24 08 00 00 00    mov    DWORD PTR [rsp],0x8
   0x00007ffff7dc7e9c <printf+156>:     48 8b 00                mov    rax,QWORD PTR [rax]
   0x00007ffff7dc7e9f <printf+159>:     c7 44 24 04 30 00 00 00 mov    DWORD PTR [rsp+0x4],0x30
   0x00007ffff7dc7ea7 <printf+167>:     48 89 c7                mov    rdi,rax
pwndbg> telescope $rsp 0x1f
00:0000│ rsp 0x7fffffffdfa8 —▸ 0x40107f (main+47) ◂— mov rdx,rsp
01:0008│     0x7fffffffdfb0 ◂— 0x0
02:0010│     0x7fffffffdfb8 ◂— 0xd0c30a3adaf82800
03:0018│     0x7fffffffdfc0 —▸ 0x7fffffffe0d8 —▸ 0x7fffffffe54a ◂— '/mnt/ubuntu/home/czg/csapp3e/asm/prog'
04:0020│     0x7fffffffdfc8 —▸ 0x7ffff7d9b790 ◂— mov edi,eax
05:0028│     0x7fffffffdfd0 —▸ 0x7fffffffe0c0 —▸ 0x7fffffffe0c8 ◂— 0x38 /* '8' */
06:0030│     0x7fffffffdfd8 —▸ 0x401050 (main) ◂— sub rsp,0x18
07:0038│     0x7fffffffdfe0 ◂— 0x100400040 /* '@' */
08:0040│     0x7fffffffdfe8 —▸ 0x7fffffffe0d8 —▸ 0x7fffffffe54a ◂— '/mnt/ubuntu/home/czg/csapp3e/asm/prog'
09:0048│     0x7fffffffdff0 —▸ 0x7fffffffe0d8 —▸ 0x7fffffffe54a ◂— '/mnt/ubuntu/home/czg/csapp3e/asm/prog'
0a:0050│     0x7fffffffdff8 ◂— 0xee0fb9aaeca58ca7
0b:0058│     0x7fffffffe000 ◂— 0x0
0c:0060│     0x7fffffffe008 —▸ 0x7fffffffe0e8 —▸ 0x7fffffffe570 ◂— 'BEMENU_BACKEND=wayland'
0d:0068│     0x7fffffffe010 —▸ 0x403df0 —▸ 0x401180 ◂— endbr64 
0e:0070│     0x7fffffffe018 —▸ 0x7ffff7ffd000 (_rtld_global) —▸ 0x7ffff7ffe2c0 ◂— 0x0
0f:0078│     0x7fffffffe020 ◂— 0x11f0465553078ca7
10:0080│     0x7fffffffe028 ◂— 0x11f05619822f8ca7
11:0088│     0x7fffffffe030 ◂— 0x0
... ↓        2 skipped
14:00a0│     0x7fffffffe048 —▸ 0x7fffffffe0d8 —▸ 0x7fffffffe54a ◂— '/mnt/ubuntu/home/czg/csapp3e/asm/prog'
15:00a8│     0x7fffffffe050 ◂— 0x1
16:00b0│     0x7fffffffe058 ◂— 0xd0c30a3adaf82800
17:00b8│     0x7fffffffe060 ◂— 0x0
18:00c0│     0x7fffffffe068 —▸ 0x7ffff7d9b84a (__libc_start_main+138) ◂— mov r15,qwordptr[rip+0x1b4737]
19:00c8│     0x7fffffffe070 —▸ 0x401050 (main) ◂— sub rsp,0x18
1a:00d0│     0x7fffffffe078 —▸ 0x403df0 —▸ 0x401180 ◂— endbr64 
1b:00d8│     0x7fffffffe080 ◂— 0x0
... ↓        2 skipped
1e:00f0│     0x7fffffffe098 —▸ 0x4010d0 (_start) ◂— endbr64 
```
- `rsp+0x18` -> ... -> '/mnt/ubuntu/home/czg/csapp3e/asm/prog' which is overwrited by `rax = 0x1`
- clear rax after save it
- `rsp+0xe0` addr -> rax -> `rsp+0x8`
- `rsp` -> `rdx` (caller reg save)
- clear `ecx` param
- `rsp+0x20` addr -> rax -> `rsp+0x10`
- `rip+0x187ef3` -> rax; rax -> rax -> rdi, so `rdi` -> `_IO_2_1_stdout_` , which probably is to call
```bash
pwndbg> telescope 0x00007ffff7dc7e95+0x187ef3
00:0000│  0x7ffff7f4fd88 —▸ 0x7ffff7f51688 (stdout) —▸ 0x7ffff7f515a0 (_IO_2_1_stdout_) ◂— 0xfbad2084
```
- 0x8 -> rsp, 0x30 -> rsp+0x4
> above rsp+0x0/4/8/10/18 is overrided after `0x00007ffff7dc7e68`, 20~c0 -> params with printf TODO 0xd0 maybe used when exit, 0xe0 is saved
```bash
pwndbg> telescope rsp+0xd0
00:0000│  0x7fffffffe078 —▸ 0x403df0 —▸ 0x401180 ◂— endbr64 
01:0008│  0x7fffffffe080 ◂— 0x0
pwndbg> | maintenance info sections |grep 403df0

 [18]     0x00403de8->0x00403df0 at 0x00002de8: .init_array ALLOC LOAD DATA HAS_CONTENTS
 [19]     0x00403df0->0x00403df8 at 0x00002df0: .fini_array ALLOC LOAD DATA HAS_CONTENTS
pwndbg> telescope 0x00403de8

00:0000│         0x403de8 —▸ 0x4011b0 ◂— endbr64 
01:0008│ rcx r14 0x403df0 —▸ 0x401180 ◂— endbr64 
02:0010│         0x403df8 (_DYNAMIC) ◂— 0x1
```
above param save is similar to how env saved in stack
```bash
pwndbg> telescope 0x7fffffffe0d8 0xff
02:0010│ rcx  0x7fffffffe0e8 —▸ 0x7fffffffe56f ◂— 'BEMENU_BACKEND=wayland'
03:0018│      0x7fffffffe0f0 —▸ 0x7fffffffe586 ◂— 'BROWSER=google-chrome-stable'
04:0020│      0x7fffffffe0f8 —▸ 0x7fffffffe5a3 ◂— 'COLORFGBG=0;15'
05:0028│      0x7fffffffe100 —▸ 0x7fffffffe5b2 ◂— 'COLORTERM=truecolor'
06:0030│      0x7fffffffe108 —▸ 0x7fffffffe5c6 ◂— 'DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus'
07:0038│      0x7fffffffe110 —▸ 0x7fffffffe5fc ◂— 'DEBUGINFOD_URLS=https://debuginfod.archlinux.org'
08:0040│      0x7fffffffe118 —▸ 0x7fffffffe62d ◂— 'DESKTOP_SESSION=plasmawayland'
09:0048│      0x7fffffffe120 —▸ 0x7fffffffe64b ◂— 'DISPLAY=:1'
0a:0050│      0x7fffffffe128 —▸ 0x7fffffffe656 ◂— 'GBM_BACKEND=nvidia-drm'
...
93:0498│      0x7fffffffe570 ◂— 'EMENU_BACKEND=wayland'
94:04a0│      0x7fffffffe578 ◂— 'CKEND=wayland'
95:04a8│      0x7fffffffe580 ◂— 0x524200646e616c79 /* 'yland' */
96:04b0│      0x7fffffffe588 ◂— 'OWSER=google-chrome-stable'
97:04b8│      0x7fffffffe590 ◂— 'ogle-chrome-stable'
98:04c0│      0x7fffffffe598 ◂— 'ome-stable'
99:04c8│      0x7fffffffe5a0 ◂— 0x524f4c4f4300656c /* 'le' */
9a:04d0│      0x7fffffffe5a8 ◂— 'FGBG=0;15'
```
[pdf p249](../references/intel_64.pdf) -> `67h` -> addr32 prefix here `e8` -> rel , no mem op(erand)
```bash
# here addr32 no use https://stackoverflow.com/questions/72892152/whats-addr32-in-assembly-means
   0x00007ffff7dc7ec7 <+199>:   67 e8 53 7d 0c 00       addr32 call 0x7ffff7e8fc20 <__stack_chk_fail>
```
## nopie when exit
```bash
pwndbg> d-- 0x401110
   0x0000000000401100 <_dl_relocate_static_pie+0>:      f3 0f 1e fa             endbr64
   0x0000000000401104 <_dl_relocate_static_pie+4>:      c3                      ret

   0x0000000000401180:  f3 0f 1e fa             endbr64
   0x0000000000401184:  80 3d 95 2e 00 00 00    cmp    BYTE PTR [rip+0x2e95],0x0        # 0x404020
=> 0x000000000040118b:  75 13                   jne    0x4011a0
   0x000000000040118d:  55                      push   rbp
   0x000000000040118e:  48 89 e5                mov    rbp,rsp
   0x0000000000401191:  e8 7a ff ff ff          call   0x401110
   0x0000000000401196:  c6 05 83 2e 00 00 01    mov    BYTE PTR [rip+0x2e83],0x1        # 0x404020
   0x000000000040119d:  5d                      pop    rbp
   0x000000000040119e:  c3                      ret
pwndbg> bt
#0  0x000000000040118b in ?? ()
#1  0x00007ffff7fcb0e2 in ?? () from /lib64/ld-linux-x86-64.so.2
#2  0x00007ffff7fcef4e in ?? () from /lib64/ld-linux-x86-64.so.2
#3  0x00007ffff7db3445 in ?? () from /usr/lib/libc.so.6
#4  0x00007ffff7db35b0 in exit () from /usr/lib/libc.so.6
#5  0x00007ffff7d9b797 in ?? () from /usr/lib/libc.so.6
#6  0x00007ffff7d9b84a in __libc_start_main () from /usr/lib/libc.so.6
```
## hlt last to stop current running logical processor and `endbr64` always first(although the latter not supported in old cpu)
```bash
pwndbg> dr
Dump of assembler code for function _start:
...
   0x00000000004010f8 <+24>:    48 c7 c7 60 10 40 00    mov    rdi,0x401060
   0x00000000004010ff <+31>:    ff 15 c3 2e 00 00       call   QWORD PTR [rip+0x2ec3]        # 0x403fc8
   0x0000000000401105 <+37>:    f4                      hlt
pwndbg> telescope 0x401060
00:0000│  0x401060 (main) ◂— sub rsp,0x18
pwndbg> telescope 0x403fc8
00:0000│  0x403fc8 —▸ 0x7ffff7d9b7c0 (__libc_start_main) ◂— endbr64
```
## TODO why vdso&vdrr location diff between gdb and r2
- [maybe this](https://manpages.ubuntu.com/manpages/xenial/man7/vdso.7.html), but why location state related with [stack] always same.
```bash
[czg /mnt/ubuntu/home/czg/csapp3e/asm]$ pstree -apnh | grep prog     
  |   |       |   `-r2,18110 -AA -c db sym.multstore;dc -d /mnt/ubuntu/home/czg/csapp3e/asm/prog
  |   |       |       `-prog,18111
  |   |       |   `-gdb,19446 -q ./prog -ex starti -ex set record full stop-at-limit off -nx -iex ...
  |   |       |       `-prog,19480
  |   |           `-grep,19619 --color=auto prog
[czg /mnt/ubuntu/home/czg/csapp3e/asm]$ sudo cat /proc/19480/maps -vv
00400000-00401000 r--p 00000000 103:08 2105174                           /mnt/ubuntu/home/czg/csapp3e/asm/prog
00401000-00402000 r-xp 00001000 103:08 2105174                           /mnt/ubuntu/home/czg/csapp3e/asm/prog
00402000-00403000 r--p 00002000 103:08 2105174                           /mnt/ubuntu/home/czg/csapp3e/asm/prog
00403000-00404000 r--p 00002000 103:08 2105174                           /mnt/ubuntu/home/czg/csapp3e/asm/prog
00404000-00405000 rw-p 00003000 103:08 2105174                           /mnt/ubuntu/home/czg/csapp3e/asm/prog
7ffff7c8d000-7ffff7c90000 rw-p 00000000 00:00 0 
...
7ffff7f7d000-7ffff7f85000 r--p 00000000 00:18 333519                     /usr/lib/liblua.so.5.4.4
7ffff7f85000-7ffff7fb3000 r-xp 00008000 00:18 333519                     /usr/lib/liblua.so.5.4.4
7ffff7fb3000-7ffff7fbf000 r--p 00036000 00:18 333519                     /usr/lib/liblua.so.5.4.4
7ffff7fbf000-7ffff7fc1000 r--p 00042000 00:18 333519                     /usr/lib/liblua.so.5.4.4
7ffff7fc1000-7ffff7fc2000 rw-p 00044000 00:18 333519                     /usr/lib/liblua.so.5.4.4
7ffff7fc2000-7ffff7fc4000 rw-p 00000000 00:00 0 
7ffff7fc4000-7ffff7fc8000 r--p 00000000 00:00 0                          [vvar]
7ffff7fc8000-7ffff7fca000 r-xp 00000000 00:00 0                          [vdso]
7ffff7fca000-7ffff7fcb000 r--p 00000000 00:18 3960                       /usr/lib/ld-linux-x86-64.so.2
7ffff7fcb000-7ffff7ff1000 r-xp 00001000 00:18 3960                       /usr/lib/ld-linux-x86-64.so.2
7ffff7ff1000-7ffff7ffb000 r--p 00027000 00:18 3960                       /usr/lib/ld-linux-x86-64.so.2
7ffff7ffb000-7ffff7ffd000 r--p 00031000 00:18 3960                       /usr/lib/ld-linux-x86-64.so.2
7ffff7ffd000-7ffff7fff000 rw-p 00033000 00:18 3960                       /usr/lib/ld-linux-x86-64.so.2
7ffffffde000-7ffffffff000 rw-p 00000000 00:00 0                          [stack]
ffffffffff600000-ffffffffff601000 --xp 00000000 00:00 0                  [vsyscall]

[czg /mnt/ubuntu/home/czg/csapp3e/asm]$ sudo cat /proc/18111/maps -vv
00400000-00401000 r--p 00000000 103:08 2105174                           /mnt/ubuntu/home/czg/csapp3e/asm/prog
00401000-00402000 r-xp 00001000 103:08 2105174                           /mnt/ubuntu/home/czg/csapp3e/asm/prog
00402000-00403000 r--p 00002000 103:08 2105174                           /mnt/ubuntu/home/czg/csapp3e/asm/prog
00403000-00404000 r--p 00002000 103:08 2105174                           /mnt/ubuntu/home/czg/csapp3e/asm/prog
00404000-00405000 rw-p 00003000 103:08 2105174                           /mnt/ubuntu/home/czg/csapp3e/asm/prog
012bb000-012dc000 rw-p 00000000 00:00 0                                  [heap]
...
7f9e67f5b000-7f9e67f5c000 rw-p 00044000 00:18 333519                     /usr/lib/liblua.so.5.4.4
7f9e67f5c000-7f9e67f5e000 rw-p 00000000 00:00 0 
7f9e67f5e000-7f9e67f5f000 r--p 00000000 00:18 3960                       /usr/lib/ld-linux-x86-64.so.2
7f9e67f5f000-7f9e67f85000 r-xp 00001000 00:18 3960                       /usr/lib/ld-linux-x86-64.so.2
7f9e67f85000-7f9e67f8f000 r--p 00027000 00:18 3960                       /usr/lib/ld-linux-x86-64.so.2
7f9e67f8f000-7f9e67f91000 r--p 00031000 00:18 3960                       /usr/lib/ld-linux-x86-64.so.2
7f9e67f91000-7f9e67f93000 rw-p 00033000 00:18 3960                       /usr/lib/ld-linux-x86-64.so.2
7ffd32f83000-7ffd32fa4000 rw-p 00000000 00:00 0                          [stack]
7ffd32fe7000-7ffd32feb000 r--p 00000000 00:00 0                          [vvar]
7ffd32feb000-7ffd32fed000 r-xp 00000000 00:00 0                          [vdso]
ffffffffff600000-ffffffffff601000 --xp 00000000 00:00 0                  [vsyscall]
# same with in r2 and pwndbg
[0x004011e0]> dm
...
0x00007ffd32f83000 - 0x00007ffd32fa4000 - usr   132K s rw- [stack] [stack] ; map._stack_.rw_
0x00007ffd32fe7000 - 0x00007ffd32feb000 - usr    16K s r-- [vvar] [vvar] ; map._vvar_.r__
0x00007ffd32feb000 - 0x00007ffd32fed000 - usr     8K s r-x [vdso] [vdso] ; map._vdso_.r_x
0xffffffffff600000 - 0xffffffffff601000 - usr     4K s --x [vsyscall] [vsyscall] ; map._vsyscall_.__x
pwndbg> vmmap
    0x7ffff7fc4000     0x7ffff7fc8000 r--p     4000      0 [vvar]
    0x7ffff7fc8000     0x7ffff7fca000 r-xp     2000      0 [vdso]
    0x7ffff7fca000     0x7ffff7fcb000 r--p     1000      0 /usr/lib/ld-linux-x86-64.so.2
    0x7ffff7fcb000     0x7ffff7ff1000 r-xp    26000   1000 /usr/lib/ld-linux-x86-64.so.2
    0x7ffff7ff1000     0x7ffff7ffb000 r--p     a000  27000 /usr/lib/ld-linux-x86-64.so.2
    0x7ffff7ffb000     0x7ffff7ffd000 r--p     2000  31000 /usr/lib/ld-linux-x86-64.so.2
    0x7ffff7ffd000     0x7ffff7fff000 rw-p     2000  33000 /usr/lib/ld-linux-x86-64.so.2
    0x7ffffffde000     0x7ffffffff000 rw-p    21000      0 [stack]
0xffffffffff600000 0xffffffffff601000 --xp     1000      0 [vsyscall]

```
# gdb usage
- use `x/g` to avoid accidental truncate of variable in stack (can tested with `voltron`)  
## voltron
- [`add byte [eax], al`](https://reverseengineering.stackexchange.com/questions/17555/radare-produces-a-bunch-of-add-byte-rax-al-but-why) all zeroes.
## pwndbg ([pwnlib/pwntools](https://docs.pwntools.com/en/stable/about.html))
- setting with [EFLAGS](http://www.c-jump.com/CIS77/ASM/Instructions/I77_0070_eflags_bits.htm) [pdf p80](../references/intel_64.pdf) in [gdb `p $eflags`](https://gist.github.com/mepcotterell/199b86a558b3f944589674f53ff681b0)
- [readthedoc wiki](https://browserpwndbg.readthedocs.io/en/docs/commands/misc/pwndbg/)
```python
# context.py
"context-flag-set-color", "green,bold", "color for flags register (flag set)"
```
### ghidra
- must add `set context-ghidra always` in cfg even have `set context-sections ... ghidra` addeed in cfg
### `vmmap`
- `p` [permission](https://stackoverflow.com/questions/19780783/whats-the-p-permission-found-on-var-run-screen)
```bash
pwndbg> |vmmap|grep sta
    0x7ffffffde000     0x7ffffffff000 rw-p    21000      0 [stack]

$ ps aux | grep gdb
czg         4285  0.0  0.1  93736 24768 pts/4    Sl+  09:01   0:00 vim /home/czg/.gdbinit.d/func_pwndbg.gdb -c vs -c e ~/.gdbinit.d/func.gdb
czg         7381  0.4  1.1 1537832 174348 pts/5  Sl+  10:06   0:26 gdb ./check_cet_supported.out -ex br 6 -ex br 12 -ex br *0x0000555555555040 -ex set record full stop-at-limit off -ex target record-full -ex r -nx -iex source ~/.gdbinit_pwndbg_voltron.gdb
czg        10060  0.0  0.6 1473544 101436 pts/1  Sl+  11:03   0:00 gdb ./check_cet_supported.out -ex starti -ex set record full stop-at-limit off -nx -iex source ~/.gdbinit_pwndbg_voltron.gdb -ex wa $rdi -ex
czg        11233  0.0  0.0  10856  2944 pts/13   S+   11:42   0:00 grep --color=auto gdb
$ ls /proc/7381/fd -al
total 0
dr-x------ 2 czg czg 20 Apr 14 10:06 .
dr-xr-xr-x 9 czg czg  0 Apr 14 10:06 ..
lrwx------ 1 czg czg 64 Apr 14 10:06 0 -> /dev/pts/5
lrwx------ 1 czg czg 64 Apr 14 10:06 1 -> /dev/pts/5
l-wx------ 1 czg czg 64 Apr 14 11:43 10 -> 'pipe:[140131]'
lr-x------ 1 czg czg 64 Apr 14 11:43 11 -> 'pipe:[138023]'
l-wx------ 1 czg czg 64 Apr 14 11:43 12 -> 'pipe:[138023]'
...
```
### tip
- GDB's set directories <path> parameter can be used to debug e.g. glibc sources like the malloc/free functions!
## gdb
- [LWP](https://stackoverflow.com/questions/4691031/getting-info-about-threads-in-gdb-ddd)
- breakpoint source [code](https://gitlab.com/git-mirrors/binutils/-/blob/master/gdb/python/lib/gdb/dap/events.py) or [API](https://sourceware.org/gdb/onlinedocs/gdb/Breakpoints-In-Python.html#Breakpoints-In-Python)
  (recommend read API firts, **if not understand** then go to source code [not to adhere to understandign all source codes])
- memory [watchpoint](https://stackoverflow.com/questions/11004374/watch-a-memory-range-in-gdb) [more](https://sourceware.org/gdb/download/onlinedocs/gdb/Set-Watchpoints.html#Set-Watchpoints)
- [probe](https://docs.rs/probe/latest/probe/)
## rizin(version:`9ab709bc34843f04ffde3b63322c809596123e77`)
- history in `/home/czg/.cache/rizin/history` or `/home/czg/.cache/radare2/history`
## r2(radare2)
- [wiki](https://r2wiki.readthedocs.io/en/latest/home/misc/cheatsheet/) or [blog](https://rehex.ninja/posts/radare2-rizin-cheatsheet/)
  - also can see official book
- meson [build](https://book.rada.re/first_steps/compilation_portability.html)
### web: [1](https://radareorg.github.io/blog/) [2](https://rada.re/n/) [3](https://www.radare.org/r/me.html)
### installation
- capstone need build from source to make `/usr/include/capstone` version right (python all version is fine) (here `$?` [return](https://stackoverflow.com/questions/7248031/meaning-of-dollar-question-mark-in-shell-scripts) 'exit status of the most recently executed foreground pipeline.' from description in 'Special Parameters' in `man bash`)
```bash
$ pkg-config --cflags capstone
-I/usr/include/capstone
$ cat ~/radare2/sys/install.sh
...
NEED_CAPSTONE=1
pkg-config --cflags capstone > /dev/null 2>&1
echo "\$\?" $?
if [ $? = 0 ]; then
    pkg-config --atleast-version=5.0.0 capstone >/dev/null 2>&1
    if [ $? = 0 ]; then
        pkg-config --variable=archs capstone 2> /dev/null | grep -q riscv
        if [ $? = 0 ]; then
            export CFGARG="--with-syscapstone"
            NEED_CAPSTONE=0
            echo "Note: Using system-wide-capstone"
        else
            echo "Warning: Your system-wide capstone dont have enough archs"
        fi  
    else
        echo "Warning: Your system-wide capstone is too old for me"                                                                                                  
    fi  
else
    echo "Warning: Cannot find system wide capstone"
fi
```
### common cmds
> not to try `ESIL` if not knowing what it will do by running something like `aei`.
> > tested it may cause battery slump to 10% 
>
> see [this](https://book.rada.re/debugger/migration.html)
>
> also [this](https://gist.github.com/williballenthin/6857590dab3e2a6559d7)/[this](https://monosource.gitbooks.io/radare2-explorations/content/intro/debugging.html)
```bash
el # list available cfgs in rizin
e?? # in r2

# `pCd 20 @ 0x00007ffff7fe5b2c ` may drop more than `pd 20 @ 0x00007ffff7fe5b2c` (exist in rizin not in r2)
pd 20 @ 0x00007ffff7fe5b2c
            # ;-- rip:
            # 0x7ffff7fe5b2c     .string "\xf3\x0f\x1e\xfaUH\x8d5\xd4D\xfe\xffH\x8d\x05\x85\x87\x01c" ; len=19
            # 0x7ffff7fe5b33      movq  xmm1, rsi
            # 0x7ffff7fe5b38      movq  xmm2, rax
            # 0x7ffff7fe5b3d      punpcklqdq xmm1, xmm2
pCd 20 @ 0x00007ffff7fe5b2c
# 0x7ffff7fe5b2c   rip:                    0x7ffff7fe5b84 mov rax, qword [rip + 0x170x7ffff7fe5bde jbe 0x7ffff7fe5bc9        0x7ffff7fe5c20 mov rax, rdi
# 0x7ffff7fe5b2c lea rax, [rip + 0x18785]  0x7ffff7fe5b8b mov qword [rip + 0x17f2e],0x7ffff7fe5be0 mov rdi, r8               0x7ffff7fe5c23 jmp 0x7ffff7fe5bc9
# 0x7ffff7fe5b33 movq xmm1, rsi            0x7ffff7fe5b92 test rax, rax             0x7ffff7fe5be3 sub rdi, rax              0x7ffff7fe5c25 nop dword [rax]
# 0x7ffff7fe5b38 movq xmm2, rax            0x7ffff7fe5b95 je 0x7ffff7fe5c28         0x7ffff7fe5be6 cmp rdi, 0xf              0x7ffff7fe5c28 movzx edx, byte [rip + 0x18

# https://github.com/radareorg/radare2
iS;is # check whether have main or other subfunc
af;pdr; agf # get scratch of one function

# https://itnext.io/radare2-for-reverse-engineering-part1-eedf0a47b5cc
aaa;afl;
dcu sym.multstore # shown above

# https://reverseengineering.stackexchange.com/questions/16844/how-to-get-a-nice-stack-view-in-radare2
pxr @ esp # view stack

# https://r2wiki.readthedocs.io/en/latest/home/tips/
ag
pxr @ esp!32

# tweak based on `e??`
e asm.hint.call=true # show syscall func

# https://book.rada.re/visual_mode/visual_panels.html?highlight=layout#saving-layouts
v=test_r2.json # not to format, otherwise, r2 can't recognize

# ?*~...
pv1 @ 0x56003724319e # can test view instruction memory
# 0xe8
# 0x56003724319e      e88dfeffff     call sym.imp.__stack_chk_fail 

# https://book.rada.re/first_steps/basic_debugger_session.html
pd 1 @ +$l # show next instruction
F2 -> br
# https://book.rada.re/first_steps/expressions.html
???~vi # check calculation

# https://gist.github.com/williballenthin/6857590dab3e2a6559d7
/aa jbe|head # search instruction, but below it‘s better

# https://github.com/radareorg/radare2/blob/master/doc/intro.md#rop from http://archive.hack.lu/2015/radare2-workshop-slides.pdf from https://r2wiki.readthedocs.io/en/latest/home/resources/
# which would output all possible gadget, may be not original asm
/R push rax

# check func refs http://archive.hack.lu/2015/radare2-workshop-slides.pdf p63
axt sym.main
# https://www.megabeets.net/a-journey-into-radare-2-part-1/
fs symbols
f

# https://reverseengineering.stackexchange.com/questions/18024/rasm2-giving-different-output-each-time-for-x64
rasm2 -a x86 -b 64 -d "48898D80FEFFFF"
rasm2 -d 418b4104
```
#### TODO
- https://docs.google.com/document/d/1our_fcFcufIJ13QsZoDuGOEBqftF6o0zEkDsqzAy43U/edit#heading=h.zdmriiaus3dc
### [blogs](https://github.com/dukebarman/awesome-radare2)
- [this](https://www.megabeets.net/a-journey-into-radare-2-part-1/)
### notice
- with [large](https://reverseengineering.stackexchange.com/questions/16112/how-to-make-radare2-work-for-a-large-binary/16115#16115) binary
### `dsb`
- [this](https://reverseengineering.stackexchange.com/questions/31804/can-radare2-skip-syscall-when-db-maindtsdc/31805#31805)
```bash
r2 -c "dcu main;dsu -1;ood;dcu main;dts+;3ds;dr rip;dr edi;dsb;dsb;dr rip;dr edi" -d /mnt/ubuntu/home/czg/csapp3e/asm/prog
...
0x00401071
0x00402004 # edi modified
0x00401064
0x00000001
```
# gcc tweak
## parameter use
```bash
$ gcc -fomit-frame-pointer -fno-pic -no-pie -g3 -Og -o prog 010-main.c 010-mstore.c 
$ gcc -pg -fno-pic -no-pie -g3 -Og -o prog 010-main.c 010-mstore.c
$ gcc -fPIE -fPIC -no-pie -fgnu-tm test_got.c -o test_got -lstdc++
...
```
## [view](https://stackoverflow.com/questions/12697081/what-is-the-gmon-start-symbol) `__gmon_start__`
```bash
$ gcc check_cet_supported.c -o check_cet_supported.out -g -no-pie -fno-pic -p -pg
```
# csapp -> intel manual
- zero extended [pdf p77](../references/intel_64.pdf)
  - tested: above not apply to condition when dest is memory
- att manual 
  - map with intel[pdf p38](../references/att_x86.pdf)
```bash
   0x0000555555555150 <+23>:    c6 45 e3 01             mov    BYTE PTR [rbp-0x1d],0x1
   0x0000555555555154 <+27>:    c7 45 e4 04 00 00 00    mov    DWORD PTR [rbp-0x1c],0x4
=> 0x000055555555515b <+34>:    48 8d 45 e3             lea    rax,[rbp-0x1d]
...
End of assembler dump.
pwndbg> x $rbp-0x1c
0x7fffffffdfa4: 0x00000004
pwndbg> x/g $rbp-0x1c
0x7fffffffdfa4: 0xf7fe638000000004
```
## Problem 3.3 (show how to disassemble & assemble att syntax without `rasm2`(it doesn't support att))
- although access 32bit mem %ebx is a bit weird in 64bit, but if possible having the rights, no problem. 
```bash
[czg /mnt/ubuntu/home/czg/csapp3e/asm]$ gcc -g -no-pie -nostdlib test_ebx.s && ./a.out
zsh: segmentation fault (core dumped)  ./a.out # here is because no rights to write 0x1 mem
[czg /mnt/ubuntu/home/czg/csapp3e/asm]$ objdump -d ./a.out                            

./a.out:     file format elf64-x86-64


Disassembly of section .text:

0000000000401000 <_start>:
  401000:       48 c7 c0 01 00 00 00    mov    $0x1,%rax
  401007:       48 c7 c7 01 00 00 00    mov    $0x1,%rdi
  40100e:       67 c6 03 01             movb   $0x1,(%ebx)
  401012:       c6 07 01                movb   $0x1,(%rdi)
```
### use `sed` to [test](https://stackoverflow.com/questions/32223572/sed-copy-lines-from-a-file-to-specific-line-in-another-file)
- see how sed [work](https://www.gnu.org/software/sed/manual/html_node/sed-script-overview.html) (similar to vim) 
```bash
[czg /mnt/ubuntu/home/czg/csapp3e/asm]$ cat 3_4_add.s 

        movq    -24(%rbp),%rdi
        movq    -16(%rbp),%rsi
        movsbl (%rdi), %eax
        movl %eax, (%rsi)

# note above has 2 blank lines;test whether work
$ sed -n -e '/\(movq\t%rax, -16(%rbp)\)/r ./3_4_add.s' -e '/\(movq\t%rax, -16(%rbp)\)/,+20p' 3_4.s 
        movq    %rax, -16(%rbp)

        movq    -24(%rbp),%rdi
        movq    -16(%rbp),%rsi
        movsbl (%rdi), %eax
        movl %eax, (%rsi)


        movq    -24(%rbp),%rdi
        movq    -16(%rbp),%rsi
        movsbl (%rdi), %eax
        movl %eax, (%rsi)


        movq    -24(%rbp),%rdi
        movq    -16(%rbp),%rsi
        movsbl (%rdi), %eax
        movl %eax, (%rsi)

        movl    $0, %eax
        movq    -8(%rbp), %rdx
        subq    %fs:40, %rdx
        je      .L3
        call    __stack_chk_fail@PLT
# not use egrep,https://stackoverflow.com/questions/1825552/grep-a-tab-in-unix
[czg /mnt/ubuntu/home/czg/csapp3e/asm]$ cat 3_4.s|grep -P 'movq\t%rax, -16' -A 20
        movq    %rax, -16(%rbp)

        movq    -24(%rbp),%rdi
        movq    -16(%rbp),%rsi
        movsbl (%rdi), %eax
        movl %eax, (%rsi)


        movq    -24(%rbp),%rdi
        movq    -16(%rbp),%rsi
        movsbl (%rdi), %eax
        movl %eax, (%rsi)

        movl    $0, %eax
        movq    -8(%rbp), %rdx
        subq    %fs:40, %rdx
        je      .L3
        call    __stack_chk_fail@PLT
# begin use sed
$ cd /mnt/ubuntu/home/czg/csapp3e/asm;gcc -S -o 3_4.s problem_3_4.c -no-pie
$ sed -n -e '/\(movq\t%rax, -16(%rbp)\)/r ./3_4_add.s' -e 'p' 3_4.s > 3_4_char_int.s
$ file="3_4_char_int";gcc -g $file.s -o $file;pwngdb ./$file -ex 'start' -ex 'target record-full'
```
- here `rdi,rsi` is used, a bit of difficult to [call](https://cs.lmu.edu/~ray/notes/gasexamples/) `printf`
### `unsigned char long`
```bash
$ cat problem_3_4.c
// Build with: gcc -m32 -no-pie -g -o plt plt.c

//typedef char src_t ;
//typedef int dest_t;
typedef unsigned char src_t ;
typedef long dest_t;

int main() {
        src_t *sp;
        dest_t *dp;
        //src_t test_src=-1;
        src_t test_src=1;
        dest_t test_dest=1<<63;
        sp=&test_src;
        dp=&test_dest;
}
# important， see above 'weird problems' -> '1...'
$ cat ./sed_3_4.sh
s/\(leaq\t-32(%rbp), \)%rax/\1%rdx/g
#s/\(leaq\t-\)32\((%rbp), \)%rax/\140\2%rdx\n\t\132\2%rdx/g
$ cat ./3_4_add.s
# test
        movq    -24(%rbp), %rdi
        #sometimes weird fail,but above works...
        #movq   -16(%rbp),%rdx

        #weird the before lea work,the latter fail
        #leaq    -32(%rbp), %rdx
        #leaq    -40(%rbp), %rdx

        #book answer
        #movsbl (%rdi), %eax
        #movl %eax, (%rdx)
        #self
        #movb (%rdi), %al
        ## movsbl dest should be reg https://www.felixcloutier.com/x86/movsx:movsxd
        #movsbl %al,(%rdx)

        movzbl (%rdi), %eax
        movq %rax, (%rdx)
# end
$ cd /mnt/ubuntu/home/czg/csapp3e/asm;gcc -S -o 3_4.s problem_3_4.c -no-pie;sed -n -f ./sed_3_4.sh -e '/\(movq\t%rax, -16(%rbp)\)/r ./3_4_add.s' -e 'p' 3_4.s > 3_4_char_int.s;file="3_4_char_int";gcc -g $file.s -o $file;pwngdb ./$file -ex 'start' -ex 'target record-full' -ex 'br *0x0000555555555173' -ex '' -ex 'si 2' -ex 'x/b $rdi' -ex 'x/w $rsi' -ex 'wa $rdx' -ex 'c'
...
```
### subquesion 4
```bash
$ cat problem_3_4.c
...
	src_t test_src=-1;
	dest_t test_dest=3;
# 
   0x0000555555555150 <+23>:    c7 45 e4 ff ff ff ff    mov    DWORD PTR [rbp-0x1c],0xffffffff
   0x0000555555555157 <+30>:    c6 45 e3 03             mov    BYTE PTR [rbp-0x1d],0x3
   0x000055555555515b <+34>:    48 8d 45 e4             lea    rax,[rbp-0x1c]
   0x000055555555515f <+38>:    48 89 45 e8             mov    QWORD PTR [rbp-0x18],rax
   0x0000555555555163 <+42>:    48 8d 45 e3             lea    rax,[rbp-0x1d]
   0x0000555555555167 <+46>:    48 89 45 f0             mov    QWORD PTR [rbp-0x10],rax
   0x000055555555516b <+50>:    48 8b 7d e8             mov    rdi,QWORD PTR [rbp-0x18]
  #  here test with src '-1'
   0x000055555555516f <+54>:    8a 07                   mov    al,BYTE PTR [rdi]
   0x0000555555555171 <+56>:    88 06                   mov    BYTE PTR [rsi],al
=> 0x0000555555555173 <+58>:    b8 00 00 00 00          mov    eax,0x0
pwndbg> x/b $rsi
0x7fffffffe0b8: 0xff
pwndbg> x/b $rdi
0x7fffffffdf84: 0xff
```
## use as to check how att syntax assembly generate obj ([also](https://stackoverflow.com/questions/55674190/is-it-possible-to-create-a-shared-object-using-only-as-and-ld) can use `ld` to generate exe, but more troublesome than `gcc` directly)
```bash
$ gcc -S -o 3_4.s problem_3_4.c -no-pie
$ cat 3_4.s
        .file   "problem_3_4.c"
        .text
        .globl  main
        .type   main, @function
main:
.LFB0:
        .cfi_startproc
        pushq   %rbp
        .cfi_def_cfa_offset 16
        .cfi_offset 6, -16
        movq    %rsp, %rbp
        .cfi_def_cfa_register 6
        subq    $32, %rsp
        movq    %fs:40, %rax
        movq    %rax, -8(%rbp)
        xorl    %eax, %eax
        movb    $-1, -29(%rbp)
        movl    $3, -28(%rbp)
        leaq    -29(%rbp), %rax
        movq    %rax, -24(%rbp)
        leaq    -28(%rbp), %rax
        movq    %rax, -16(%rbp)
        movl    $0, %eax
        movq    -8(%rbp), %rdx
        subq    %fs:40, %rdx
        je      .L3
        call    __stack_chk_fail@PLT
.L3:
        leave
        .cfi_def_cfa 7, 8
        ret
        .cfi_endproc
.LFE0:
        .size   main, .-main
        .ident  "GCC: (GNU) 12.2.1 20230201"
        .section        .note.GNU-stack,"",@progbits
# use as
$ as 3_4.s -o 3_4.o;objdump -d 3_4.o
```
- directly `gcc` (`gcc 3_4.s -o 3_4.o;./3_4.o`)
## instruction test
### imul
```bash
────────────────────────────────────────────────────────────────────────[ DISASM / x86-64 / set emulate on ]─────────────────────────────────────────────────────────────────────────
  0x55555555516e<main+53>   imul   rcx
 ►0x555555555171<main+56>   mov    eax,0
Watchpoint 2: $rdx

Old value = 0x7fffffffe0c8
New value = 0x2aaaaaaa

Watchpoint 3: $rax

Old value = 0x7fffffffdf84
New value = 0xb417fffffadc0b60

 RCX  0x555555557dd8 —▸ 0x5555555550e0 ◂— endbr64
```
- calculation see [this](./bfloat16_half.py)
- usage `rdx` with param -> csapp p235
### `idiv`
- [why](https://stackoverflow.com/questions/72152783/why-do-x86-mul-and-div-instructions-only-take-a-source-operand) `idiv` instruction no two operand -> not to 'hard-coded'
- `rax` as Quotient -> (po) maybe to just as return value conveniently, just as `imul` save low 64-bit because it is probably to return.
## EFLAGS
- why CF flag is only [the last bit](https://stackoverflow.com/questions/71771188/why-is-the-overflow-flag-only-set-when-single-shifts-are-used) shifted, based on design (more economical)
- [why](https://stackoverflow.com/questions/13435142/why-do-the-inc-and-dec-instructions-not-affect-the-carry-flag-cf) `dec` not influence carry flag, because [redundant (ZF & OF can do the same thing)](https://stackoverflow.com/questions/13435142/why-do-the-inc-and-dec-instructions-not-affect-the-carry-flag-cf#comment86749885_13435722) and economical
### OF flag & CF
- [carry in](https://www.quora.com/What-does-carry-in-and-carry-out-mean-in-electronics#:~:text=The%20%E2%80%9Ccarry%20out%E2%80%9D%20is%20the,add%20into%20the%20next%20column.)
  - what is [overflow](http://c-jump.com/CIS77/CPU/Overflow/lecture.html#O01_0010_overflow), just unable to represent result correctly.
  - ~~TODO , so how to use in this condition~~ useful [example](https://stackoverflow.com/questions/15245762/carries-in-and-out-of-most-significant-bits-in-binary-addition)
    - so
      - [1](http://c-jump.com/CIS77/CPU/Overflow/lecture.html#O01_0080_overfow_output): 0100b+1100b has carry in `1` (because 100+100=1000, carry out from third column to fourth [carry in to fourth]) and result is 10000b -> carry out `1` to fifth column
        - the fouth column is `0+1=1`, so with carry in to fourth, must have carry out to fifth.
        - the rest is similar
      - [this](http://c-jump.com/CIS77/CPU/Overflow/lecture.html#O01_0120_signed_overflow_cont) -> must have carry in and carry out **counterbalanced**
  - because `OF` is to test whether can be represented, it doesn't care about whether negative.
```bash
$ cat seta_setg_custom.s
        movw $0x1,%ax
        pushw %ax
        subw $2,(%rsp) ## set CF no OF

        movw $0x7fff,%ax
        pushw %ax
        subw $0xffff,(%rsp) ## would always seen as -1, not unsigned... 0xfffff obviouly no use to generate overflow; 0x7fff-0xffff=0x7fff+1=0x8000 would set CF and OF
```
## simple explanation of some terms
- dereference: because data is saved in memory or reg, so memory is just reference to data(similar to alias), so `*ptr` is dereferencing.
- oct word -> ['double quadword operand' pdf p602](../references/intel_64.pdf)
- [compiler](https://english.stackexchange.com/questions/212566/etymology-of-compiler-computer-term) just 'compile' things from lib
  - [assembler](https://en.wikipedia.org/wiki/Assembly_language) 'consisting of several sections'
  - compiler may [use](https://www.quora.com/Are-assemblers-compilers) assembler
    - [cc1](https://unix.stackexchange.com/questions/77779/relationship-between-cc1-and-gcc)
    - [collect2](https://en.wikibooks.org/wiki/GNU_C_Compiler_Internals/GNU_C_Compiler_Architecture)
  - [relation](https://erg.abdn.ac.uk/users/gorry/eg2068/course/comp.html)
  - also see `man gcc` -> `/^ +-S` (where ~~compilation is a little different from above ‘compiler’ because it not directly to machine code~~ is [specfic](https://en.wikipedia.org/wiki/Compiler) to ‘assembly language’)
  - linker is [necessary](https://stackoverflow.com/questions/845355/do-programming-language-compilers-first-translate-to-assembly-or-directly-to-mac) to get object code.
```bash
$ gcc -v -o 3_4.o problem_3_4.c -no-pie
 /usr/lib/gcc/x86_64-pc-linux-gnu/12.2.1/cc1 -quiet -v problem_3_4.c -quiet -dumpdir 3_4.o- -dumpbase problem_3_4.c -dumpbase-ext .c -mtune=generic -march=x86-64 -version -o /tmp/ccLtBlXL.s
...
 as -v --64 -o /tmp/ccGChgzd.o /tmp/ccLtBlXL.s
 /usr/lib/gcc/x86_64-pc-linux-gnu/12.2.1/collect2 -plugin /usr/lib/gcc/x86_64-pc-linux-gnu/12.2.1/liblto_plugin.so -plugin-opt=/usr/lib/gcc/x86_64-pc-linux-gnu/12.2.1/lto-wrapper -plugin-opt=-fresolution=/tmp/cctTz5hJ.res -plugin-opt=-pass-through=-lgcc -plugin-opt=-pass-through=-lgcc_s -plugin-opt=-pass-through=-lc -plugin-opt=-pass-through=-lgcc -plugin-opt=-pass-through=-lgcc_s --build-id --eh-frame-hdr --hash-style=gnu -m elf_x86_64 -dynamic-linker /lib64/ld-linux-x86-64.so.2 -o 3_4.o /usr/lib/gcc/x86_64-pc-linux-gnu/12.2.1/../../../../lib/crt1.o /usr/lib/gcc/x86_64-pc-linux-gnu/12.2.1/../../../../lib/crti.o /usr/lib/gcc/x86_64-pc-linux-gnu/12.2.1/crtbegin.o -L/usr/lib/gcc/x86_64-pc-linux-gnu/12.2.1 -L/usr/lib/gcc/x86_64-pc-linux-gnu/12.2.1/../../../../lib -L/lib/../lib -L/usr/lib/../lib -L/usr/lib/gcc/x86_64-pc-linux-gnu/12.2.1/../../.. /tmp/ccGChgzd.o -lgcc --push-state --as-needed -lgcc_s --pop-state -lc -lgcc --push-state --as-needed -lgcc_s --pop-state /usr/lib/gcc/x86_64-pc-linux-gnu/12.2.1/crtend.o /usr/lib/gcc/x86_64-pc-linux-gnu/12.2.1/../../../../lib/crtn.o
```
## `seta`and`setg`
```bash
$ cat seta_setg_custom.s
...
        #add
        movl $2,%eax
        movq $-1,%rdx 
        sub $-1,%eax
        # 2-(-1) need borrow -> 10..(7 bytes)2-0xf..(8 bytes) = 0x3
        setg %cl
        sub $2,%rdx
        # (-1)-2 no borrow, keep sign bit set
        setb %cl
        #end
...
$ gcc seta_setg_custom.s -o seta_setg_custom.o -g;record_pwngdb seta_setg_custom.o -ex 'start'
```
## `rep ret`
- [pdf p144](../references/amd_opt.pdf) -> branch [prediction](https://en.wikipedia.org/wiki/Branch_predictor#Next_line_prediction) related with cpu design
  - in new cpu, this is [no needed](https://stackoverflow.com/questions/20526361/what-does-rep-ret-mean), (only old `gcc` [generate](https://stackoverflow.com/questions/20526361/what-does-rep-ret-mean#comment52568460_20526918) this instruction, clang [not](https://repzret.org/p/repzret/))
## `&&` [label](https://stackoverflow.com/questions/1777990/is-it-possible-to-store-the-address-of-a-label-in-a-variable-and-use-goto-to-jum) address
## book problem notes
### 3.5
```bash
$ gcc problem_3_5.c -S -o problem_3_5.s -O3

$ cat problem_3_5.s
        .file   "problem_3_5.c"
        .text
        .p2align 4
        .globl  decode1
        .type   decode1, @function
decode1:
.LFB0:
        .cfi_startproc
        movq    (%rdi), %r8
        movq    (%rsi), %rcx
        movq    (%rdx), %rax
        movq    %r8, (%rsi)
        movq    %rcx, (%rdx)
        movq    %rax, (%rdi)
        ret
        .cfi_endproc

```
### `3.34`
- actual behavior
```bash
pwndbg> dr
Dump of assembler code for function main:
   0x00005555555551ea <+92>:    41 51                   push   r9
   0x00005555555551ec <+94>:    41 50                   push   r8
   0x00005555555551ee <+96>:    57                      push   rdi
...
──────────────────────────────────────────────────────────────────────────────────────[ STACK ]──────────────────────────────────────────────────────────────────────────────────────
00:0000│ rbp rsp 0x7fffffffdf38 —▸ 0x7fffffffdfb0 ◂— 0x1
01:0008│         0x7fffffffdf40 —▸ 0x5555555551fc (main+110) ◂— add rsp,0x18
02:0010│         0x7fffffffdf48 ◂— 0x6
03:0018│         0x7fffffffdf50 ◂— 0x7
04:0020│         0x7fffffffdf58 ◂— 0x8
pwndbg> dr
Dump of assembler code for function test_mul_args:
   0x0000555555555149 <+0>:     55                      push   rbp
   0x000055555555514a <+1>:     48 89 e5                mov    rbp,rsp # caused above rbp relocation
...
   0x000055555555517d <+52>:    8b 45 10                mov    eax,DWORD PTR [rbp+0x10]
   0x0000555555555180 <+55>:    01 c2                   add    edx,eax
   0x0000555555555182 <+57>:    8b 45 18                mov    eax,DWORD PTR [rbp+0x18]
   0x0000555555555185 <+60>:    01 c2                   add    edx,eax
   0x0000555555555187 <+62>:    8b 45 20                mov    eax,DWORD PTR [rbp+0x20]
   0x000055555555518a <+65>:    01 d0                   add    eax,edx
# clearly in 
[czg /mnt/ubuntu/home/czg/csapp3e/asm]$ file=3_34;gcc $file.c -o $file.s -S -fverbose-asm                        
[czg /mnt/ubuntu/home/czg/csapp3e/asm]$ cat 3_34.s 
        movl    -16(%rbp), %r9d # test[8], _1
        movl    -20(%rbp), %r8d # test[7], _2
        movl    -24(%rbp), %edi # test[6], _3
        movl    -28(%rbp), %r11d        # test[5], _4
        movl    -32(%rbp), %r10d        # test[4], _5
        movl    -36(%rbp), %ecx # test[3], _6
        movl    -40(%rbp), %edx # test[2], _7
        movl    -44(%rbp), %esi # test[1], _8
        movl    -48(%rbp), %eax # test[0], _9
        pushq   %r9     # _1
        pushq   %r8     # _2
        pushq   %rdi    # _3
        movl    %r11d, %r9d     # _4,
        movl    %r10d, %r8d     # _5,
        movl    %eax, %edi      # _9,
...
```
- first push last argument
### `3.45`
```bash
        movq    %rax, -64(%rbp) # tmp84, test.a
        movss   .LC0(%rip), %xmm0       #, tmp85
        movss   %xmm0, -56(%rbp)        # tmp85, test.b
        movb    $1, -52(%rbp)   #, test.c
        movw    $1, -50(%rbp)   #, test.d
        movq    $1, -48(%rbp)   #, test.e
        movsd   .LC1(%rip), %xmm0       #, tmp86
        movsd   %xmm0, -40(%rbp)        # tmp86, test.f
        movl    $1, -32(%rbp)   #, test.g
        leaq    -69(%rbp), %rax #, tmp87
        movq    %rax, -24(%rbp) # tmp87, test.h
# require 64-24+8=48 bytes
```
### 3.46
- [strlen](https://codebrowser.dev/glibc/glibc/string/strlen.c.html)
```cpp
    if (*char_ptr == '\0')
      return char_ptr - str; // not count '\0' in 
```
### 3.49
- see fig  2.29&30
### 3.51 
- TODO: why gcc not use [CVTSS2SD](https://stackoverflow.com/questions/16597587/why-dont-gcc-and-clang-use-cvtss2sd-memory)
### 3.69
- movslq not `MOVSD` in [pdf p44](../references/att_x86.pdf) but `movsxd`
```bash
$ rasm2 -d '4863c9'
movsxd rcx, ecx
```
### 4.49
- mainly based on the following:
if `a!=b` -> `(a^b)^a=b;(a^b)^b=a`
### 5.9
- maybe because of loop overhead, the A3 two multiplication in the path which isn't critical is not considered block the execution.
### Figure 5.37
- ~~TODO why example A not left chain critical?~~
  example A: two address reading can be parallel and register reading can use aliasing, so left can be seen as one operation. -> critical is left / right, but right is more intuitive.
  compared with former multiplication critical path, the multiplication is more complex than read and addition here, so it is seen as critical path.
## miscs
- better not to use [ddd (archaic)](https://news.ycombinator.com/item?id=32125868)
- see [operation](https://www.felixcloutier.com/x86/unpcklps#operation) of instruction better than description -> `UNPCKLPS`
  - [VCVTPS2PD](https://www.felixcloutier.com/x86/cvtps2pd#vcvtps2pd--vex-128-encoded-version-) -> `cvtps2pd`
  - `cvtsi2ss` second operand -> [SRC1](https://www.felixcloutier.com/x86/cvtsi2ss#vcvtsi2ss--vex-128-encoded-version-)
```asm
$ cat jmp_test.s| grep vcvts -A 3 -B 1                           
        jmp *%rax
        vcvtsi2ss %edi, %xmm1, %xmm2
        vmulsd %xmm0, %xmm1, %xmm0 #Multiply a by x
        vcvtsi2sd %edi, %xmm1, %xmm1 #Convert i to double
        vdivsd %xmm1, %xmm2, %xmm2
;att
vcvtsi2ss %edi, %xmm1, %xmm2
;intel
    115d:       c5 f2 2a d7             vcvtsi2ss xmm2,xmm1,edi
```
- qnan/snan [pdf p94](../references/intel_64.pdf)
  - why define two types of [nan](https://softwareengineering.stackexchange.com/questions/392179/should-nan-default-to-snan-or-qnan)
- alignment is based on [base](https://stackoverflow.com/questions/381244/purpose-of-memory-alignment), -> offset should be multiply of alignment size.
- xmm1 return [pdf p26](../references/abi.pdf) / homework 3.75
- y86-64 online [simulator](https://boginw.github.io/js-y86-64/)
- Obviously, there will be more hazards if stages is above 5.
- Figure 4.66
> bubble [vs](https://stackoverflow.com/questions/42743411/understanding-bubble-vs-stall-vs-repeated-decode-fetch) stall
  - `ret` is detected when fetched, so the following **one** fetch is stalled (having a small possibility to return to next fetched instruction, so not cancel, i.e. bubble) -> decode after the following fetch is got bubble.
  - 'Load/use hazard' is detected when execute (Figure 4.54), so the following **two** instruction still work, but the related registers should be modified (So not cancelled -> not bubble). Similar to above.
  - 'Mispredicted branch', two following instruction must be wrong -> cancelled (see Figure 4.56)
- [Retirement Unit](https://stackoverflow.com/questions/22368835/what-does-intel-mean-by-retired#:~:text=In%20the%20context%20%22retired%22%20means,if%20they%20execute%20in%2Dorder.)
- AVX no parallel multiplication of 64-bit integers, but AVX512 has [similar](https://stackoverflow.com/questions/41403718/can-i-use-the-avx-fma-units-to-do-bit-exact-52-bit-integer-multiplications#comment70023106_41403718); can use conversion but [not recommended](https://stackoverflow.com/questions/41403718/can-i-use-the-avx-fma-units-to-do-bit-exact-52-bit-integer-multiplications#comment70023106_41403718).
- [EDO DRAM](https://en.wikipedia.org/wiki/Dynamic_random-access_memory#Extended_data_out_DRAM) mainly kept data cache to accelerate.
> or see Bibliographic Notes [1](https://user.eng.umd.edu/~blj/papers/isca99.pdf)
  - [SDRAM](https://en.wikipedia.org/wiki/Synchronous_dynamic_random-access_memory#Commands) use one better encoding method
  - [DDR SDRAM](https://en.wikipedia.org/wiki/Synchronous_dynamic_random-access_memory#DDR_SDRAM_prefetch_architecture) use a new cache enhancement method.
  - flash [diff](https://www.tutorialspoint.com/difference-between-eeprom-and-flash#:~:text=EEPROM%20is%20a%20type%20of,to%20and%20erased%20in%20blocks.) with EEPROMs
- [Recording density](https://www.anandtech.com/show/11925/western-digital-stuns-storage-industry-with-mamr-breakthrough-for-nextgen-hdds/2)
- [Transfer Time](https://www.javatpoint.com/seek-time-vs-transfer-time-in-disk-scheduling)
  - p630 : 1 circle -> 400 sectors/track, so 1/400 circle time -> 1 sector
- when use [col major](https://craftofcoding.wordpress.com/2017/02/03/column-major-vs-row-major-arrays-does-it-matter/#:~:text=In%20a%20large%202D%20array,%2Dfetches%20data%20required%20next).
- [when](https://computerscience.chemeketa.edu/cs160Reader/Binary/Bytes.html) use 2 or 10 with KB
## TODO
- redo 3.68 homework after understanding stack better.
- 3.73 use asm not __asm__ direct.
- relearn digital circuits and relearn chapter 4 and try designing circuits.
- [prime](https://stackoverflow.com/questions/67623801/is-it-always-necessary-to-make-hash-table-number-of-buckets-a-prime-number-for-p) hash bucket number
- [AMDuProf](https://community.amd.com/t5/server-gurus-discussions/intel-vtune-on-amd-processors/m-p/529122/highlight/true) & [callgrind](https://stackoverflow.com/questions/6663614/use-valgrind-to-know-timein-seconds-spent-in-each-function)
- SRAM DRAM [design](https://www.egr.msu.edu/classes/ece410/mason/files/Ch13.pdf), more [concise  ](https://web.cs.umass.edu/~weems/homepage/335-Notes/ewExternalFiles/Lecture%209.pdf)
# directly [use](https://cs.lmu.edu/~ray/notes/gasexamples/) syscall with asm to run (this blog get by googling 'use as to assemble')
# att syntax
- [label(%rip)](https://stackoverflow.com/questions/69464871/assembly-and-rip-usage)
  - [also](https://godbolt.org/) [from](https://www.felixcloutier.com/documents/gcc-asm.html#examples) 