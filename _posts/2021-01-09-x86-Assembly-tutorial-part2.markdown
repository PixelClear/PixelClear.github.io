---
layout: post
title:  "X86 Assembly - Tutorial 2"
date:   2021-01-09 15:13:36 +0100
categories: technical post
---

<ins>**Moving Data**</ins> 
=====================================================================================================================================

Program in its fundamental structure is nothing but set of instruction that is transforming data.This transformations include moving,
reading, writing data from and to different types of memories. Out of this transformations the most frequent one is moving data in 
memory and so, its important we understand instructions in assembly that helps us do that. This tutorial is mostly dedicated to this topic.

**Note** : Please read [Part I](https://pixelclear.github.io/technical/post/2021/01/02/x86-Assembly-tutorial-part1.html) of this tutorial series.

<ins>**Defining Data in Assembly Program**</ins> 
=====================================================================================================================================

Before we jump and start looking at how to manipulate data, we must know how to define/declare it. In C generally you find data defined/declared
at start of the function or at place where it is used. In assembly we have dedicated sections to define/declare the data.

**Note** : There is subtle difference between defining data and declaring data. 

           defining data - specify type of data, placehholder name for data , initial value.
		   
           declaring data - specify type of data, placehholder name for data.

In assembly we have **.data, .rodata , .bss** sections where we need to define/declare our data so that the memory is reserved for it.

• <ins>Defining Data in .data or .rodata</ins>
      
The syntax for specifying data in these sections is 

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~	  
label :
.directive initial value**
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~	 

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~	   
.data 
pi:
  .float 3.14
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~	   
	  
The label is just like variable name in C. .directive specify what is the type of data along with the size. 
Different directives supported for .data section are 

>     .ascii Text string
>     .asciz Null-terminated text string
>     .byte Byte value
>     .double Double-precision floating-point number
>     .float Single-precision floating-point number
>     .int 32-bit integer number
>     .long 32-bit integer number (same as .int)
>     .octa 16-byte integer number
>     .quad 8-byte integer number
>     .short 16-bit integer number
>     .single Single-precision floating-point number (same as .float)

**Note** : Read only data goes in **.rodata** section.
	  
• <ins>Defining Data in .bss</ins>
	  
This section generally holds that without initial values. You can use this section to declare buffers that will be utilized later.
The syntax for declaring data in .bss section is

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~	   
.directive label, length
   
.lcomm buffer, 100
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~	   

We can think of it as temporary variable.We can have global buffer or local buffers using following directives.
	  
>     .comm   Declares common area for data 
>     .lcomm  Declared local common area for data
    
**Note** : .bss section is not part of exe. The memory you declare in this section is not reserved during compile time.
           As the data is not initialized hence, it is lazily allocated. Operating system implements virtual memory using 
		   technique called as on demand paging.Please refer to **Miscellaneous** to get some details on **On Demand Paging**. 
		   
<ins>**Moving Data**</ins> 
=====================================================================================================================================

GAS provides **MOV** instruction to move data between different types of memories.
The syntax for **MOV** instruction is

>    movx source, destination

Based on the size of data element we want to move we need to append following suffix to **MOV** instruction 

>    l - 32 bit long word value

>    w - 16 bit word value

>    b - 8 bit byte value
 
**Note** : We have to be careful here. If in declaration the size of data type is 16 bits and we move it using **MOVL** then still 4 bytes will be read
           and the resultant read value will be wrong. I think now we should start feeling the love for compiler. 

• <ins>Moving data between registers</ins> 
	  
   This is the easiest and fastest way to move data with processor. The only care we have to take is while moving data to and from control, debug, segment
   registers. Value can be moved into these special purpose registers only to and from general purpose registers.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~	 
movl %eax, %ecx  - Moving 32 bit of data from EAX to ECX 
movw %ax, %cx	 - Moving 16 bit of data from AX to CX
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~	 

   Special care needs to be taken when moving data between registers of different size.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~	 	  
movb %al, %bx    - This will generate error. As we are trying to copy 8 bits from AL to 16 bit BX. We should instead move AX to BX.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~	 

• <ins>Moving data between register and memory</ins>

   Moving data between memory and register is tricky. We have to know how to represent the address of data we want to move in instruction code.
   There are multiple addressing modes to access data 

  * **Direct Memory Addressing Mode** 

   We can directly specify data using label. For example 

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.section .data 
number:
  .int 9
.section .text
.globl _start 
_start:
      nop
      movl number, %ecx               --------------> Here we are moving value from memory to register by directly referencing to label.
      movl $1, %eax                   --------------> This is example of moving immediate data to registers.
      movl $0, %ebx
      int $0x80
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~		 

  * **Indexed Memory Addressing Mode**

   We can think of it as accessing arrays in C. Please read below section carefuly as we will be covering arrays and loops here.
   Consider following example code 

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.section .data
output:
     .asciz “The value is %d\n”
values:                                   
     .int 10, 15, 20, 25, 30, 35, 40, 45, 50, 55, 60 
.section .text
.globl _start
_start:
     nop
     movl $0, %edi                      
loop:
     movl values(, %edi, 4), %eax        
     pushl %eax
     pushl $output
     call printf
     addl $8, %esp
     inc %edi
     cmpl $11, %edi
     jne loop
     movl $0, %ebx
     movl $1, %eax
     int $0x80      
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

   Lets break this down and explaining each line.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.section .data 		 
output:
     .asciz “The value is %d\n”
values:                                   
     .int 10, 15, 20, 25, 30, 35, 40, 45, 50, 55, 60 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

   This is out data section.We are declaring null terminated string to print the values and array of integers with label values.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.section .text
.globl _start
_start:
     nop
     movl $0, %edi   
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~		 

   This is start of our text section and we will be using values in register EDX as counter for loop. So we initialize it to 0.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
loop:
    movl values(, %edi, 4), %eax        
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

   This label loop marks the start of loop. The way values are index has following syntax

>	movx base_address( offset_address, index, size), destination

   If any of the values are zero we can omit it. As we want to iterate the array from start hence, the offset_address for us will be 0.
   So, we have omitted it. The way this address deduced is using following expression 

>   base_address + offset_address + index * size

   So, with EDX holding value 0 we will access first element in values array and put it in %eax. Then we increament that value and continue.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
pushl %eax                 ----------------------> Put value in EAX on stack.
pushl $output              ----------------------> Put format string on stack.
call printf                ----------------------> Stack is prepared so call printf.
addl $8, %esp              ----------------------> Clear stack by adding to stack pointer. As result of two pushl it was decremented by 8 bytes.
inc %edi                   ----------------------> Increment the value of counter in EDX.
cmpl $11, %edi             ----------------------> Compare value of counter to length of array.
jne loop                   ----------------------> Jump to label loop if values are not equal.  
movl $0, %ebx           
movl $1, %eax
int $0x80      
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

   Please check side notes in above code snippet for detailed explaination of loop logic. I will add more detailed explaination for JUMP and CMP instructions. 
   
  * **Indirect Memory Addressing Mode** 

   Registers can also hold the address of data. We can think of this as Pointer manipulation in C.Lets take same example of printing array values but this 
   time access the data using indirect memory addressing mode.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.section .data
output:
  .asciz “The value at index 1 is %d\n”
values:
     .int 10, 15, 20, 25, 30, 35, 40, 45, 50, 55, 60
.section .text
.globl _start
_start:
     nop
     movl $values, %edi
     movl $100, 4(%edi)              
     movl values(, %edi, 4), %ebx
     pushl %ebx
     pushl $output
     call printf
     movl $0, %ebx           
     movl $1, %eax
     int $0x80    
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

   Let me explain important sections in code. 		 

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
movl $values, %edi
movl $100, 4(%edi)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

   We first move the starting address of values array to EDX. When **$** appended to label assembler will give the address of variable.
   The second instruction is actually dereferencing the address in EDI then adding 4 bytes to it and writing 100 as new value.
   In C the equivalent statement would be *(values + 1) = 100. As result of above code we have updated the value in array at index 1.

<ins>**Conditional Move**</ins> 
=====================================================================================================================================

As optimization to move operations x86 assembly also provides conditional move.Conditional move will move the data based on the status of EFLAGS.
There are multiple conditional move instructions like COMA, COMBE and so on. Consider following code for example

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
movl number2, %ebx
movl number1, %eax
cmp %ebx, %eax
cmova %eax, %ebx
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Above code snippet will load number1 in EAX and number2 in EBX.
CMP instruction will subtract number2 from number1 and set appropriate EFLAGS status.

If **number1 < number2** CMP will set **ZF = 0** and **CF = 1**.

If **number1 = number2** CMP will set **ZF = 0** and **CF = 0**.

If **number1 > number2** CMP will set **ZF = 1** and **CF = 0**.

Next COMA instruction will replace the value in EBX if  **ZF = 1** and **CF = 0**. In other words EBX will be updated if number 1 is greater or above the 
value that is already present in EBX. I will leave finding the max of array using conditional move to readers.

<ins>**Exchanging Data Between Memories**</ins> 
=====================================================================================================================================

Exchanging data is also one of the most common operations we perform. If we want to exchange values between two registers you will generally need temporary.
To optimize these exchange operation we are provided with multiple instructions.

>**XCHG**       Exchanges the values of two registers, or a register and a memory location

>**BSWAP**      Reverses the byte order in a 32-bit register

>**XADD**       Exchanges two values and stores the sum in the destination operand

>**CMPXCHG**    Compares a value with an external value and exchanges it with another

>**CMPXCHG8B**  Compares two 64-bit values and exchanges it with another

<ins>**Miscellaneous**</ins> 
=====================================================================================================================================
	  
• <ins>**On Demand Paging**</ins>
	    
   Memory management unit in operating systems maintains **Page Table** data structure. Each entry 
   in this table maps virtual page to physical page(page in RAM).
   
   When you compile/link/load program the addresses allocated are virtual addresses.The .text section and so allocated
   other sections will be divided in virtual pages generally of size 64kb. Each virtual page contains bunch of virtual addresses.
   When ever your program access some location and that is not present in RAM then processor raises page fault trap.
   As we saw in tutorial 1 it is type of exception.After raising the exception processor goes on to execute the exception handler.
   The pages in RAM are referred as physical page or frame.Similary hard driver where your program resides is also divided in block.
   These blocks are same size of physical pages and virtual pages.During execution of exception handler the required address is located 
   using Page Table. As this address is not present in RAM so it will be further dereferenced and corresponding block on hard driver 
   will be found. Then new page is created in RAM and this block is copied to this physical page.If the address you accessed doesnt belong 
   to your address space then segmentation fault is generated and program might terminate. If address you are referencing doesnt belong to
   your processes address space then it will be mapping to some page that is mapped in some other process and processor will raise **General Protection Fault (GPF)**.
   
   .bss section is implemented using **Zero Fill On Demand Paging** method. The memory is not mapped to any physical page. The first time buffer 
   is accessed then a new page is allocated and it is zero initialized.This zero initialized memory is then returned to program as buffer.
   You can try to print the size by declaring buffer inside .data section and in .bss section to see the difference in exe sizes.		