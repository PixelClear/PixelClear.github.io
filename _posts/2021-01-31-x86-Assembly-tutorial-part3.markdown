---
layout: post
title:  "X86 Assembly - Tutorial 3"
date:   2021-01-09 15:13:36 +0100
categories: technical post
---

<ins>**Controling the Flow**</ins> 
=====================================================================================================================================

We break programs in logical structures know as functions.Function is logical set of instructions that tansforms data in very particular way.
This logical separation is really great form of abstraction building.This construct in programing language is called function. The way we utilize 
this function is by calling it.We will see in detail what it means to call function.We may want to call function based on some condition.The 
language construct used for this is branches.Also, transforming data sometime means performing repeatative operations on data.The construct we 
use for this is called loops.All of this constructs are implemented with help of compilers.I hope at the end of chapter you will have better 
understanding about workings of branch, loop, function.

**Note** : Please read [Part I](https://pixelclear.github.io/technical/post/2021/01/02/x86-Assembly-tutorial-part1.html) and [Part II](https://pixelclear.github.io/technical/post/2021/01/09/x86-Assembly-tutorial-part2.html) of this tutorial series.

<ins>**How Program Is Executed?**</ins> 
=====================================================================================================================================

Processor has special purpose register called as **Instruction Pointer** that holds the address of the next instruction to execute. The nature of this operation
looks very sequential. This was reason why processors were slow. Later inventions of **Instruction Prefetch Cache**, **Out Of Order Execution Engine** and 
**Retirement Unit** were made to speed up the CPU operations.

Many instructions ahead of current instruction pointer location are prefeched in **Instruction Prefetch Cache**.**Out Of Order Execution Engine** takes these instructions 
and starts executing them and the results of this execution is then placed in **Retirement Unit**.The **Retirement Unit** will not execute the result until its time to 
do so in program logic.So, instruction pointer reached on instruction then, **Out Of Order Execution Engine** might have already placed the result of it in **Retirement Unit**.
The **Retirement Unit** then will execute the result of this instruction which was placed by **Out Of Order Execution Engine**. Once the **Retirement Unit** executes this result then 
and then only we say the instruction is executed and instruction pointer will be increamented.This explaination describes what actually happens in processors. 
For simplicity we will consider that the instruction pointer behaves in sequential manner and processor execute instructions sequentially.

**EIP** is instruction pointer for processor. As a programmer we cannot directly change the content of EIP register. But, there are instructions who has side
effects and by executing these instructions we can indirectly manipulate EIP. Manipulating the EIP is how we alter or control the flow of program.
So, Lets dive deep to see these instructions.
		   
<ins>**Unconditional Branches**</ins> 
=====================================================================================================================================

The place in logic where we manipulate **EIP** creates branch in flow of program.**Unconditional Branches** are used when we want to change the flow or branch out without 
any condition.Example of this could be function call or loops.Lets see the instructions used to do this

• <ins>Jump</ins>

   This instruction is similar to goto statement for C. It will help the control to go to certain location. When below instruction is executed then **EIP** is
   updated to location(address mentioned with JUMP).

>    jmp location - location is memory address where the jump will be made.

   There are multiple types of jumps. Based of on the distance between the current **EIP** location and jump to location, one of the following jumps should be used.
   
>    Short Jump - Used when jump offset is less than 128 bytes.

>    Far Jump   - Used when location of jump is in another segment. 

>    Near Jump  - In all places where Short and Far jump is not applied.

   GAS will take care of converting to proper jump if you just use mnemonic **jmp**.But, beware this will have performance implications. In next parts we will see 
   how this might affect the performance.Lets see small example
   
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~	 
.section .text
.globl _start
_start:
    nop
    movl $1, %eax
    jmp label
	movl $3, %ebx
label:
    int $0x80
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~	   
   
   We are loading value 1 in **EAX** and jump to make **sys_exit()** linux system call. We have some instructions after the call to **jmp** but those will not be executed.
   Now lets assemble this code and use one more tool called as **Objdump** to examine the object file generated.
   Observe the output generated by **Objdump**.Specifically see the address next to jump instruction. Observe for jump you mentioned the label and 
   it is replaced by the address of label. Label is just place holder name given to refer to address of instruction we want to point to.
   Commands to generate following output are as below 
   
>    as --32 -o Jump.o Jump.s

>    objdump -m i386 -D Jump.o

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~	 
Jump.o:     file format elf32-i386


Disassembly of section .text:

00000000 <_start>:
   0:   90                      nop
   1:   b8 01 00 00 00          mov    $0x1,%eax
   6:   eb 05                   jmp    d <label>
   8:   bb 03 00 00 00          mov    $0x3,%ebx

0000000d <label>:
   d:   cd 80                   int    $0x80
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~	    

**Note** : As explained in previour tutorials x86 systems uses segmented memory model.Hence, we saw different types of **jmp**.
           Please read more about **Objdump** it is very useful tool. Best way to learn more about them is use man pages.
           Addresses you are seeing in above code segment are called **Translation Time Addresses**.Compiler choses its own origin for 
		   starting addresses called as **Translated Origin**.Translation Unit is final input to Compiler/Assembler after 
		   preprocessing and other steps.Linker will chose its own address origin called as **Linked Origin** and addresses 
           generated by linker are called **Linked Time Addresses**.Similary Loader will have **Load Origin** and **Load Time Addresses**.		   
		   These are virtual addresses.When your program is loaded for execution the physical addresses are allocated and virtual address will be mapped to it. 

• <ins>Calls</ins> 
	  
   **CALL** instruction is similar to **JUMP**.The only difference is **CALL** remembers from where it jumped and if needed it can return.

>    call address - Address here is label that will be converted to address of first instruction in function.

   **CALL** instruction is used with **RET** statement.When **RET** is executed at end of function the **EIP** will be set to address just after place of CALL instruction.
   When we execute **CALL** the current value of **EIP** and all parameters are pushed on stack.After this is done then **EIP** is updated to start of function.
      
   Lets look at below program.
   
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~	 
.section .data
output:
     .asciz "This is %d"
.section .text
.globl _start
_start:
     pushl $1                     ----------------------------------> Push 1 on stack. (4 bytes)
     pushl $output                ----------------------------------> Push the address of string output. (4 bytes)
     call printf
     add $8, %esp                 ----------------------------------> Clear parameters from stack. (4 bytes + 4 bytes)
     call overhere
     pushl $2                     ----------------------------------> Push 2 on stack (4 bytes).
     pushl $output                ----------------------------------> Push the address of string output (4 bytes).
     call printf
     add $8, %esp                 ----------------------------------> Clear parameters from stack. (4 bytes + 4 bytes)
overhere:
     pushl %ebp                   ----------------------------------> Store current value in EBP.
     movl %esp, %ebp              ----------------------------------> Store current value of ESP to EBP. We need to restore stack pointer before RET is executed.
     pushl $3
     pushl $output
     call printf
     add $8, %esp
     movl %ebp, %esp              ----------------------------------> Restore the value of ESP as it was before the CALL was made.
     popl %ebp                    ----------------------------------> Restore EBP value.
     ret                          ----------------------------------> RET will make execution continue and print value 2.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~	 

   Above program will print 1 3 2.Please find the explaination next to program in above section.

• <ins>Interrupts</ins>

   **INT** instruction is used to issue interrupt and processor will take control to ISR. We saw explaination for interrupts and ISR in Part I of this tutorial series.
   Hence, we will not discuss much about this instruction here.

<ins>**Conditional Branches**</ins> 
=====================================================================================================================================

   When ever we want to transfer the flow based on some condition we use these types of instructions.The conditional statement is executed and as side effect **EFLAGS** is changed.
   Based on the status of **EFLAGS** conditional jump instruction will be executed.These types of instruction consults below 5 flags
   
>    Carry flag (CF) - bit 0 (lease significant bit)

>    Overflow flag (OF) - bit 11

>    Parity flag (PF) - bit 2

>    Sign flag (SF) - bit 7

>    Zero flag (ZF) - bit 6

   Please see example and explaination for conditional branches in section **Indexed Memory Addressing Mode** of Part II of the series.
   
<ins>**Branch Prediction**</ins> 
=====================================================================================================================================

Lets see how If statement in C program is implemented by assembler. we will write simple C program given below to implement If and Else.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~	 
include<stdio.h>

int main()
{

    int a = 100;
    int b = 50;
    if(a>b)
    {
        printf("A is greater than B");
    }
    else
    {
        printf("B is greater than A");
    }
    return 0;
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~	

After assembling we see following output 

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~	
       .file   "Branching.c"
        .text
        .section        .rodata
.LC0:
        .string "A is greater than B"
.LC1:
        .string "B is greater than A"
        .text
        .globl  main
        .type   main, @function
main:
.LFB0:
        .cfi_startproc
        endbr32
        leal    4(%esp), %ecx
        .cfi_def_cfa 1, 0
        andl    $-16, %esp
        pushl   -4(%ecx)
        pushl   %ebp
        .cfi_escape 0x10,0x5,0x2,0x75,0
        movl    %esp, %ebp                        ------------------------> Store current stack pointer to EBP so we can manipulate ESP freely and restore it later.
        pushl   %ebx
        pushl   %ecx
        .cfi_escape 0xf,0x3,0x75,0x78,0x6
        .cfi_escape 0x10,0x3,0x2,0x75,0x7c
        subl    $16, %esp
        call    __x86.get_pc_thunk.ax
        addl    $_GLOBAL_OFFSET_TABLE_, %eax
        movl    $100, -16(%ebp)                   ------------------------> -16(%ebp) represents variable a.
        movl    $50, -12(%ebp)                    ------------------------> -12(%ebp) represents variable b.
        movl    -16(%ebp), %edx                   ------------------------> Move a to EDX.
        cmpl    -12(%ebp), %edx                   ------------------------> Compare b with a.
        jle     .L2                               ------------------------> Jump to else part of b <= a. (Find more explaination in section below)
        subl    $12, %esp                         ------------------------> Continue with If part.
        leal    .LC0@GOTOFF(%eax), %edx
        pushl   %edx
        movl    %eax, %ebx
        call    printf@PLT
        addl    $16, %esp
        jmp     .L3
.L2:
        subl    $12, %esp                         -----------------------> Else part.
        leal    .LC1@GOTOFF(%eax), %edx
        pushl   %edx
        movl    %eax, %ebx
        call    printf@PLT
        addl    $16, %esp
.L3:
        movl    $0, %eax
        leal    -8(%ebp), %esp
        popl    %ecx
        .cfi_restore 1
        .cfi_def_cfa 1, 0
        popl    %ebx
        .cfi_restore 3
        popl    %ebp
        .cfi_restore 5
        leal    -4(%ecx), %esp
        .cfi_def_cfa 4, 4
        ret
        .cfi_endproc
.LFE0:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~	

We will look at only interesting details.If we observe the flow, compiler has reverse condition than what we have in program.So It might look like reverse of If-Else block we have in program.
This is done for some specific purpose.To understand this we will need to look at **Branch Prediction Unit**.**Out Of Order Unit** must predict next instructions to be executed but, when 
we have branches in program this task becomes little bit complicated.Also the original purpose of adding **Out Of Order Unit** is to optimize the processor but, what if it predicts the branch 
in wrong way?.This will actually affect the performance a lot.So, **Out Of Order Unit** take help of **Branch Prediction Unit**.This unit has some basic rules on how to evaluate the branches in program path.

>     Backward Branches are always taken - Loop is example of backward branch where we will go back to start of loop to execute instructions those might be previously executed.

>     Forward Branches are never taken -  More details in section below.

>     Braches previously taken are taken again

Now lets examine our program again. Please see instructions (cmpl -12(%ebp), %edx) and (jle .L2).Compiler changed the code because it saw that If-Then part is likely to be taken so,
it changed the logic such that, If-Else part becomes the forward branch. So **Branch Prediction Unit** will predict this branch will not be executed and hence instead of If-Else part If-Then 
code will be prefeched and we will get desired performance.So, in short placing code that is most likely to be taken as the fall-through of the JUMP forward statement increases the likelihood 
that it will be in the instruction prefetch cache when needed.

**Note** : General Optimization tips you will find online is to avoid branches all together or unroll loops.I would urge you to profile first when ever you are trying to optimize.If profiler shows
           certain branch is very slow then try to see the assembly and try to figure out what is happening.Same follows for loop unrolling.Most of the platforms provide **Conditional Move** to implement branching
		   which are not that costly.Lets say, if **Conditional Move** instruction was not present then, we might have used JUMP and implmented conditions.Thi might have affected the performance.When it comes to loop unrolling
		   we have to be careful.If we unroll loop, it might expand in lots of instruction.As result of this we might end up with no space in instruction prefectch cache and it will again affect the performance.

<ins>**Loops**</ins> 
=====================================================================================================================================

We have now idea how loops might have implemented using the **JUMP** instructions.But, lets see how assembler generates the loops.
We will use following simple C program for that.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
        .file   "Branching.c"
        .text
        .section        .rodata
.LC0:
        .string "Hello : %d"
        .text
        .globl  main
        .type   main, @function
main:
.LFB0:
        .cfi_startproc
        endbr32
        leal    4(%esp), %ecx
        .cfi_def_cfa 1, 0
        andl    $-16, %esp
        pushl   -4(%ecx)
        pushl   %ebp
        .cfi_escape 0x10,0x5,0x2,0x75,0
        movl    %esp, %ebp
        pushl   %ebx
        pushl   %ecx
        .cfi_escape 0xf,0x3,0x75,0x78,0x6
        .cfi_escape 0x10,0x3,0x2,0x75,0x7c
        subl    $16, %esp
        call    __x86.get_pc_thunk.bx
        addl    $_GLOBAL_OFFSET_TABLE_, %ebx
        movl    $0, -12(%ebp)                     --------------------------------> -12(%ebp) represent i. Init it to 0.
        jmp     .L2
.L3:
        subl    $8, %esp
        pushl   -12(%ebp)                        ---------------------------------> Push i on stack.      
        leal    .LC0@GOTOFF(%ebx), %eax
        pushl   %eax
        call    printf@PLT
        addl    $16, %esp
        addl    $1, -12(%ebp)                    --------------------------------> Increment the i by 1.
.L2:
        cmpl    $9, -12(%ebp)                    --------------------------------> Do comparison i <= 9
        jle     .L3                              --------------------------------> If i <=9 then jump back to start of loop label is L3.
        movl    $0, %eax
        leal    -8(%ebp), %esp
        popl    %ecx
        .cfi_restore 1
        .cfi_def_cfa 1, 0
        popl    %ebx
        .cfi_restore 3
        popl    %ebp
        .cfi_restore 5
        leal    -4(%ecx), %esp
        .cfi_def_cfa 4, 4
        ret
        .cfi_endproc
.LFE0:
        .size   main, .-main
        .section        .text.__x86.get_pc_thunk.bx,"axG",@progbits,__x86.get_pc_thunk.bx,comdat
        .globl  __x86.get_pc_thunk.bx
        .hidden __x86.get_pc_thunk.bx
        .type   __x86.get_pc_thunk.bx, @function
__x86.get_pc_thunk.bx:
.LFB1:
        .cfi_startproc
        movl    (%esp), %ebx
        ret
        .cfi_endproc
.LFE1:
        .ident  "GCC: (Ubuntu 9.3.0-17ubuntu1~20.04) 9.3.0"
        .section        .note.GNU-stack,"",@progbits
        .section        .note.gnu.property,"a"
        .align 4
        .long    1f - 0f
        .long    4f - 1f
        .long    5
0:
        .string  "GNU"
1:
        .align 4
        .long    0xc0000002
        .long    3f - 2f
2:
        .long    0x3
3:
        .align 4
4:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Please see the comment section in above code segment to understand part where loop is implemented.
So, with this we have now covered loops, branches and functions.