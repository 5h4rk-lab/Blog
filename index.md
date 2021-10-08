# Understanding X86 Assembly For Reverse Engineering (32-bit)

## Introduction
	
when I first decided to learn Binary exploitation and reverse Engineering the first suggetion I got from my mentors was to learn assembly language and honestly that was the best move I Did to learn RE.Understanding Assembly language make you feel every application out there is Open source :wink: .This blog is written in such a way even if you are absolute new to assembly you would be able to understand the topics discussed. we are going to learn from absolute 0 level but also I Recomand you to have basic understanding of Computer Architecture and Programing.

By the end of the blog we would be able to understand and able to read the assembly code. 

note in this blog we are gonna discuss based on intel-32bit architecture.

### What is Assembly language?
Assembly language is a low level programing language language that is intended to communicate directly with a computer’s hardware. Unlike machine language, which consists of binary and hexadecimal characters, assembly languages are designed to be readable by humans. Apart from this a few decades ago most of the softwares were written in assembly language. Sounds intresting right? if it dosent we arent friends xD.So, enough of intro lets now dig into some actual shit.

## Data types.
	
Data types in a 32-Bit assembly are Bits,Bytes,Word,Dword
What is a Bit?
=> this is a smallest data type among all of the above.
what is a Byte?
=> A byte is 8 bits put together anything in between 0 to 255.
what is a word?
=> A word is two bytes put together, which has an maximum value of 65535.
what is a Dword?
=> as the name says its a 2word where D stand for double and has an maximum value of 4294967295.

### Registors 
	
Registors plays most important role in world of computers. Registers are a type of computer memory used to quickly accept, store, and transfer data and instructions that are being used immediately by the CPU.
These are prety fast but also very limited memory on the microprocessor.The problem is, there are only a few of them and some registers are not general purpose, meaning they can’t really be used for everything.

According to Intel Architecture- 32bit there are 8 types of Genral pourpose registors. Obviously there are number-of registers apart from this but truley only 8 of them are genral purpose and these are most important as they are seen the most in the assembly out put of a program. these are:- 

EAX (Extended Accumulator Register)
EBX (Extended Base Register) 
ECX (Extended Counter Register)  
EDX (Extended Data Register)  
ESI (Extended Source Index )
EDI (Extended Destination Index)
EBP (Extended Base Pointer )
ESP (Extended Stack Pointer )

In this EAX, EBX, ECX, and EDX are general purpose registers used for various operations and EBP and ESP are the Stack Base Pointer and the Stack Pointer. Together, these two create the Stack Frame.EDI and ESI are the Destination Index and Source Index registers and is used to deal with memory copying.
All the general purpose registers are 32-bit size in Intel’s IA-32 architecture but depending on their origin and intended purpose, a subset of some of them can be referenced in assembly. here is the list

```
+--------+--------+-------+
| 32-bit | 16-bit | 8-bit |
+--------+--------+-------+
| EAX    | AX     | AH/AL |
+--------+--------+-------+
| EBX    | BX     | BH/BL |
+--------+--------+-------+
| ECX    | CX     | CH/CL |
+--------+--------+-------+
| EDX    | DX     | DH/DL |
+--------+--------+-------+
| ESI    | SI     |       |
+--------+--------+-------+
| EDI    | DI     |       |
+--------+--------+-------+
| EBP    | BP     |       |
+--------+--------+-------+
| ESP    | SP     |       |
+--------+--------+-------+

```
## Status Flags registors 

The Flag register is a Special Purpose Register. Depending upon the value of result after any arithmetic and logical operation the flag bits become set (1) or reset (0). The flags which we see the most are

Z – zero flag, set when the result of the last operation is zero 
S – signed  flag,  set  to  determine  if  values  should  be  intercepted  as  signed  or unsigned 
O – overflow  flag,  set  when  the  result  of  the  last  operation  switches  the  most significant bit from either F to 0 or 0 to F. 
C – carry flag, set when the result of the last operation changes the most significant bit.

## EIP- registor 
	
This register plays an very important role EIP is an Extended Instruction Pointer which always contains the address of the next instruction to be executed. You cannot directly access or change the instruction pointer.However, instructions that control program flow, such as calls, jumps, loops, and interrupts, automatically change the instruction pointer.

## Instructions

The instructions are usually part of an executable program, often stored as a computer file and executed on the processor. This is another vast topic to discuss I woud add the links at last for further references but a compiler does the work for you so you really dont need to care a lot about this during reversing(just my point of view).

## Arthematic Operations

The IA-32 instruction set contains six instructions for arithmetic purposes. which Are
ADD,SUB,DIV,MUL.IDIV,IMUL

### ADD

This instruction basially adds the src value to Des and stores it in des.
Syntax `add Destnation ,Src`

example :- 

`add eax,9;`

in this 9 is addedd to eax and the value is stored in eax.

### Sub

this works simmilar to ADD instruction as in example it subtracts the 9 from eax and saves the value to eax.

### DIV
DIV Operand - Divides the 64-bit value from EDX:EAX by the unsigned operand specified. The dividend is always eax and that is also were the result of the operation is stored. the rest of this stored in edx

mov eax, 29  ; move the dividend into eax 
mov ecx, 2  ; move the divisor into ecx 
div ecx ; divide eax by ecx, this will result in eax containing 14 and edx containing the rest, which is 1 

for the code above we first moved the value 29 to eax and then moved 2 to ecx and then called the instraction which devides eax/ecx which is 29/2.

### MUL/IMUL 
mul/imul (unsigned/signed) multiply either eax with a value, or they multiply two values and put them into a destination register or they multiply a register with a value. 

syntax 
mul value 

### Bitwise operations

In computer programming, a bitwise operation operates on a bit string, a bit array or a binary numeral (considered as a bit string) at the level of its individual bits. It is a fast and simple action, basic to the higher level arithmetic operations and directly supported by the processor. Most bitwise operations are presented as two-operand instructions where the result replaces one of the input operands.

AND, syntax: and dest, src  
OR, syntax:  or dest, src 
XOR, syntax: xor dest, src 
NOT, syntax: not eax 

for this lets assume two values value1 and value2 and perform the operations for better understanding.

Value1: 10101010
value2: 00101011
after operation: 


Case1 And operation 

The logical AND operator (&&) returns true if both operands are true and returns false otherwise. The operands are implicitly converted to type bool before evaluation, and the result is of type bool. Logical AND has left-to-right associativity. so the out put for AND would be 101010

Case2 OR operation

The logical OR operator returns the boolean value true if either or both operands is true and returns false otherwise. The operands are implicitly converted to type bool before evaluation, and the result is of type bool. Logical OR has left-to-right associativity. output for this would be 10101011


Case3 XOR 

The bitwise XOR operator (^) returns a 1 in each bit position for which the corresponding bits of either but not both operands are 1s. Output for this is 10000001.

Case4 NOT operation

In Boolean algebra, the NOT operator is a Boolean operator that returns TRUE or 1 when the operand is FALSE or 0, and returns FALSE or 0 when the operand is TRUE or 1. output be 1010101.


## Branching and Goto instructions 
A jump instruction, like "jmp", just switches the CPU to executing a different piece of code.  It's the assembly equivalent of "goto", but unlike goto, jumps are notconsidered shameful in assembly.
found this amazing table from internet and this is how the jump instructions exactly work.

```
+-------------+---------------------------------------------------------+
| Instruction | Useful to...                                            |
+-------------+---------------------------------------------------------+
| jmp         | Always jump                                             |
+-------------+---------------------------------------------------------+
| je          | Jump if cmp is equal                                    |
|             |                                                         |
+-------------+---------------------------------------------------------+
| jne         | Jump if cmp is not equal                                |
|             |                                                         |
+-------------+---------------------------------------------------------+
| jg          | Signed >  (greater)                                     |
|             |                                                         |
+-------------+---------------------------------------------------------+
| jge         | Signed >=                                               |
+-------------+---------------------------------------------------------+
| jl          | Signed <  (less than)                                   |
|             |                                                         |
+-------------+---------------------------------------------------------+
| jle         | Signed <=                                               |
+-------------+---------------------------------------------------------+
| ja          | Unsigned >                                              |
|             |                                                         |
+-------------+---------------------------------------------------------+
| jae         | Unsigned >=                                             |
|             |                                                         |
+-------------+---------------------------------------------------------+
| jb          | Unsigned <      (below)                                 |
|             |                                                         |
+-------------+---------------------------------------------------------+
| jbe         | Unsigned <=                                             |
|             |                                                         |
+-------------+---------------------------------------------------------+
| jecxz       | Jump if ecx is 0                                        |
|             |                   (Seriously!?)                         |
+-------------+---------------------------------------------------------+
| jc          | Jump if carry: used for unsigned             overflow,  |
|             |              or multiprecision add                      |
+-------------+---------------------------------------------------------+
| jo          | Jump if there was signed             overflow           |
+-------------+---------------------------------------------------------+

```


## Call & ret

Ok so we learned a lot on the way and this part is going to be most important among all and most of gets wrong at this during reversing.
In assembly language, the call instruction handles passing the return address for you, and ret handles using that address to return back to where you called the function from. return value. The return value is the main method of transferring data back to the main program.Once the CALL instruction is hit, it will push the current Instruction Pointer to the stack (so that it the program can continue from where it left when the called function RETurns), and then it jumps to address of the called function.a ret instruction is called which uses the pushed values of rbp and rip (saved base and instruction pointers) it can continue execution right where it left off

syntax: Call,syntax:callfunction

## Calling conventions

Calling conventions are a standardized method for functions to be implemented and called by the machine. A calling convention specifies the method that a compiler sets up to access a subroutine. In theory, code from any compiler can be interfaced together, so long as the functions all have the same calling conventions. In practice however, this is not always the case. 

there are a large number of calling conventions you can refer to https://en.wikibooks.org/wiki/X86_Disassembly/Calling_Conventions of more detail 


## Conclusion

I hope You learnt something out of this. thankyou for reading till the end.


