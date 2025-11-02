---
title: "Lab 4" 
date: 2025-10-19
lastmod: 2025-09-24
tags: ["lab", "csa"]
author: ["Suciu Mihai"]
description: "lab 4 csa." 
summary: "Bitwise operations." 

showToc: true
disableAnchoredHeadings: false

---

<h1>Laboratory 4 - Bitwise operation</h1>

## Theory
__In this lab__: instructions that manipulate operands at bit level.

Instructions used:
- [AND](../../../instr/and.html) - logical **AND**,
- [OR](../../../instr/or.html) - logical inclusive **OR**,
- [XOR](../../../instr/xor.html) - logical e**X**clusive **OR**,
- [TEST](../../../instr/test.html) - logical compare,
- [NOT](../../../instr/not.html) - one's complement negation,
- [SHL](../../../instr/SAL_SAR_SHL_SHR.html) - **SH**ift logical **L**eft,
- [SHR](../../../instr/SAL_SAR_SHL_SHR.html) - **SH**ift logical **R**ight,
- [SAL](../../../instr/SAL_SAR_SHL_SHR.html) - **S**hift **A**rithmetic **L**eft,
- [SAR](../../../instr/SAL_SAR_SHL_SHR.html) - **S**hift **A**rithmetic **R**ight,
- [ROL](../../../instr/RCL_RCR_ROL_ROR.html) - **RO**tate **L**eft,
- [ROR](../../../instr/RCL_RCR_ROL_ROR.html) - **RO**tate **R**ight,
- [RCL](../../../instr/RCL_RCR_ROL_ROR.html) - **R**otate through **C**arry **L**eft,
- [RCR](../../../instr/RCL_RCR_ROL_ROR.html) - Rotate through **C**arry **R**ight.

## Example
```asm
;Given the words A and B, compute the word C as follows:
;- the bits 0-2 of C are the same as the bits 10-12 of B
;- the bits 3-6 of C have the value 1
;- the bits 7-10 of C are the same as the bits 1-4 of A
;- the bits 11-12 have the value 0
;- the bits 13-15 of C are the invert of the bits 9-11 of B

; We will obtain the word C by successive "isolation" of bits sequences. The isolation
; of the bits 10-12 of B is done by obtaining the unchanged values of these bits,  
; and initialising all other bits to 0. The isolation operation can be performed
; using the operator AND between the word B and the mask
; 0001110000000000. Once isolated, the sequence of bits is put on the right  position by using a rotation operation. 
; The final word is obtained by applying the operator OR between all intermediate results obtained by using isolations and rotations.

; Observation: bits are numbered from right to left

bits 32 ;assembling for the 32 bits architecture
; the start label will be the entry point in the program
global  start 

extern  exit ; we inform the assembler that the exit symbol is foreign, i.e. it exists even if we won't be defining it
import  exit msvcrt.dll; exit is a function that ends the process, it is defined in msvcrt.dll
        ; msvcrt.dll contains exit, printf and all the other important C-runtime functions
segment  data use32 class=data ; the data segment where the variables are declared 
     a dw 0111011101010111b
     b dw 1001101110111110b
     c dw 0
segment  code use32 class=code ; code segment
start: 

     mov  bx, 0 ; we compute the result in bx

     mov  ax, [b] ; we isolate bits 10-12 of B
     and  ax, 0001110000000000b
     mov  cl, 10
     ror  ax, cl ; we rotate 10 positions to the right
     or   bx, ax ; we put the bits into the result

     or   bx, 0000000001111000b ; we force the value of bits 3-6 of the result to the value 1

     mov  ax, [a] ; we isolate bits 1-4 of A
     and  ax, 0000000000011110b
     mov  cl, 6
     rol  ax, cl ; we rotate 6 positions to the left
     or   bx, ax ; punem bitii in rezultat

     and  bx, 1110011111111111b ; facem biti 11-12 din rezultat sa aiba valoarea 0

     mov  ax, [b]
     not  ax ; we invert the value of b
     and  ax, 0000111000000000b ; we isolate the bits 9-11 of B
     mov  cl, 4
     rol  ax, cl ; we rotate 4 positions to the left
     or   bx, ax ; punem biti in rezultat

     mov  [c], bx ; we move the result from the register to the result variable

     push dword 0 ;saves on stack the parameter of the function exit
     call [exit] ;function exit is called in order to end the execution of the program	
```
## Exercices

<ol>
<li>
Given the words A and B, compute the doubleword C as follows:
<ul>
<li> the bits 0-4 of C are the same as the bits 11-15 of A
</li>
<li> the bits 5-11 of C have the value 1
</li>
<li> the bits 12-15 of C are the same as the bits 8-11 of B
</li>
<li> the bits 16-31 of C are the same as the bits of A</li>
</ul>
</li>
<li>
Given the words A and B, compute the doubleword C as follows:
<ul><li> the bits 0-3 of C are the same as the bits 5-8 of B
</li>
<li> the bits 4-8 of C are the same as the bits 0-4 of A
</li>
<li> the bits 9-15 of C are the same as the bits 6-12 of A
</li>
<li> the bits 16-31 of C are the same as the bits of B</li>
</ul>
</li>
<li>
Given the words A and B, compute the doubleword C as follows:
<ul>
<li> the bits 0-2 of C are the same as the bits 12-14 of A
</li>
<li> the bits 3-8 of C are the same as the bits 0-5 of B
</li>
<li> the bits 9-15 of C are the same as the bits 3-9 of A</li>
<li> the bits 16-31 of C are the same as the bits of A</li>
</ul>
</li>
<li>
Given the byte A, obtain the integer number n represented on the bits 2-4 of A. 
Then obtain the byte B by rotating A n positions to the right.
Compute the doubleword C as follows:
<ul>
<li>the bits 8-15 of C have the value 0</li>
<li>the bits 16-23 of C are the same as the bits of B</li>
<li>the bits 24-31 of C are the same as the bits of A</li>
<li>the bits 0-7 of C have the value 1</li>
</ul>
</li>
<li>
Given the bytes A and B, compute the doubleword C as follows:
<ul>
<li> the bits 16-31 of C have the value 1</li>
<li> the bits 0-3 of C are the same as the bits 3-6 of B
</li>
<li> the bits 4-7 of C have the value 0
</li>
<li> the bits 8-10 of C have the value 110
</li>
<li> the bits 11-15 of C are the same as the bits 0-4 of A
</li></ul>
</li>
<li>
Given the word A, obtain the integer number n represented on the bits 0-2 of A. 
Then obtain the word B by rotating A n positions to the right.
Compute the doubleword C:
<ul>
<li>the bits 8-15 of C have the value 0</li>
<li>the bits 16-23 of C are the same as the bits of 2-9 of B</li>
<li>the bits 24-31 of C are the same as the bits of 7-14 of A</li>
<li>the bits 0-7 of C have the value 1</li>
</ul>
</li>
<li>
Given the words A and B, compute the doubleword C:
<ul><li> the bits 0-4 of C have the value 1
</li>
<li> the bits 5-11 of C are the same as the bits 0-6 of A</li>
<li> the bits 16-31 of C have the value 0000000001100101b</li>
<li> the bits 12-15 of C are the same as the bits 8-11 of B
</i></ul>
</li>
<li>
Given the words A and B, compute the byte C as follows:
<ul><li>the bits 0-5 are the same as the bits 5-10 of A</li>
<li>the bits 6-7 are the same as the bits 1-2 of B.</li></ul>
Compute the doubleword D as follows:
<ul><li> the bits 8-15 are the same as the bits of C</li>
<li>the bits 0-7 are the same as the bits 8-15 of B</li>
<li>the bits 24-31 are the same as the bits 0-7 of A</li>
<li>the bits 16-23 are the same as the bits 8-15 of A.</li>
</ul>
</li>
<li>
Given the word A and the byte B, compute the doubleword C as follows:
<ul><li> the bits 0-3 of C are the same as the bits 6-9 of A
</li><li> the bits 4-5 of C have the value 1
</li><li> the bits 6-7 of C are the same as the bits 1-2 of B</li>
<li>the bits 8-23 of C are the same as the bits of A</li>
<li>the bits 24-31 of C are the same as the bits of B</li>
</ul>
</li>
<li>
Replace the bits 0-3 of the byte B by the bits 8-11 of the word A.
</li>
<li>
Given the byte A and the word B, compute the byte C as follows:
<ul><li>the bits 0-3 are the same as the bits 2-5 of A
</li><li>the bits 4-7 are the same as the bits 6-9 of B.
</li></ul>
</li>
<li>
Given the words A and B, compute the doubleword C as follows:
<ul><li> the bits 0-6 of C have the value 0
</li><li> the bits 7-9 of C are the same as the bits 0-2 of A
</li><li> the bits 10-15 of C are the same as the bits 8-13 of B
</li><li> the bits 16-31 of C have the value 1
</li></ul>
</li>
<li>
Given 4 bytes, compute in AX the sum of the integers represented by the bits 4-6 of the 4 bytes.
</li>
<li>
Given the doubleword A, obtain the integer number n represented on the bits 14-17 of A. Then obtain the doubleword B by rotating A n positions to the left.
Finally, obtain the byte C as follows:
<ul><li> the btis 0-5 of C are the same as the bits 1-6 of B
</li><li> the bits 6-7 of C are the same as the bits 17-18 of B
</li></ul>
</li>
<li>
Given the words A and B, compute the doubleword C as follows:
<ul><li> the bits 0-2 of C have the value 0
</li>
<li> the bits 3-5 of C have the value 1
</li>
<li> the bits 6-9 of C are the same as the bits 11-14 of A
</li>
<li> the bits 10-15 of C are the same as the bits 1-6 of B
</li>
</li><li> the bits 16-31 of C have the value 1</ul>
	</li>
	
<li>Given the byte A and the word B, compute the doubleword C as follows:
<ul><li>the bits 0-7 of C have the value 1</li><li>
the bits 8-11 of C are the same as the bits 4-7 of A</li><li>
the bits 12-19 are the same as the bits 2-9 of B</li><li>
the bits 20-23 are the same as the bits 0-3 of A</li>
<li>the bits 24-31 are the same as the high byte of B</li>
</ul>
</li><li>
Given the word A and the byte B, compute the doubleword C:
<ul><li>the bits 0-3 of C have the value 1</li><li>
the bits 4-7 of C are the same as the bits 0-3 of A</li><li>
the bits 8-13 of C have the value 0</li><li>
the bits 14-23 of C are the same as the bits 4-13 of A</li><li>
the bits 24-29 of C are the same as the bits 2-7 of B</li><li>
the bits 30-31 have the value 1</li></ul></li>	
<li>
Given the word A, compute the doubleword B as follows: 
<ul><li>the bits 0-3 of B have the value 0;
</li><li>the bits 4-7 of B are the same as the bits 8-11 of A 
</li><li> the bits 8-9 and 10-11 of B are the invert of the bits 0-1 of A (so 2 times) ;
</li><li> the bits 12-15 of B have the value 1 
</li><li> the bits 16-31 of B are the same as the bits 0-15 of B.</li></ul>
</li><li>
Given the word A, compute the doubleword B as follows: 
<ul><li> the bits 28-31 of B have the value 1;
</li><li> the bits 24- 25 and 26-27 of B are the same as the bits 8-9 of A 
</li><li> the bits 20-23 of B are the invert of the bits 0-3 of A ;
</li><li> the bits 16-19 of B have the value 0 
</li><li> the bits 0-15 of B are the same as the bits 16-31 of B.</li></ul>
</li>
<li>
Given the words A and B, compute the doubleword C as follows:
<ul><li>the bits 0-5 of C are the same as the bits 3-8 of A
</li><li>the bits 6-8 of C are the same as the bits 2-4 of B
</li><li>the bits 9-15 of C are the same as the bits 6-12 of A
</li><li>the bits 16-31 of C have the value 0</li></ul>
</li><li>
Given the words A and B, compute the doubleword C as follows:
<ul><li>the bits 0-3 of C are the same as the bits 5-8 of B 
</li><li>the bits 4-10 of C are the invert of the bits 0-6 of B
</li><li>the bits 11-18 of C have the value 1
</li><li>the bits 19-31 of C are the same as the bits 3-15 of B</li></ul>
</li>
<li>
Given the doubleword A and the word B, compute the word C as follows:
<ul>
<li>the bits 0-4 of C are the invert of the bits 20-24 of A</li>
<li>the bits 5-8 of C have the value 1</li>
<li>the bits 9-12 of C are the same as the bits 12-15 of B</li>
<li>the bits 13-15 of C are the same as the bits 7-9 of A</li>
</ul>
</li>
<li>
Given the byte A and the word B, compute the doubleword C as follows:
<ul>
<li>the bits 24-31 of C are the same as the bits of A</li>
<li>the bits 16-23 of C are the invert of the bits of the lowest byte of B</li>
<li>the bits 10-15 of C have the value 1</li>
<li>the bits 2-9 of C are the same as the bits of the highest byte of B</li>
<li>the bits 0-1 both contain the value of the sign bit of A</li>
</ul>
</li>
<li>
Given the doubleword M, compute the doubleword MNew as follows:
<ul><li>the bits 0-3 a of MNew are the same as the bits 5-8 a of M.
</li><li>the bits 4-7 a of MNew have the value 1
</li><li>the bits 27-31 a of MNew have the value 0
</li><li>the bits 8-26 of MNew are the same as the bits 8-26 a of M.</li></ul>
</li><li>
Given the doublewords M and N, compute the doubleword P as follows.
<ul><li>the bits 0-6 of P are the same as the bits 10-16 of M
</li><li>the bits 7-20 of P are the same as the bits 7-20 of (M AND N).
</li><li>the bits 21-31 of P are the same as the bits 1-11 of N.</li></ul>
</li><li>
Given 2 dublucuvinte R and T. Compute the doubleword Q as follows:
<ul><li>the bits 0-6 of Q are the same as the bits 10-16 of T
</li><li>the bits 7-24 of Q are the same as the bits 7-24 of (R XOR T).
</li><li>the bits 25-31 of Q are the same as the bits 5-11 of R.</li></ul>
</li>
<li>
Given the quadword A, obtain the integer number N represented on the bits 35-37 of A. Then obtain the the doubleword B by rotating the low doubleword of A N positions to the right.
Obtain the byte C as follows:
<ul>
<li>the bits 0-3 of C are the same as the bits 8-11 of B</li>
<li>the bits 4-7 of C are the same as the bits 16-19 of B</li>
</ul>
</li>
<li>
Given the quadword A, obtain the integer number N represented on the bits 17-19 of A. Then obtain the the doubleword B by rotating the high doubleword of A N positions to the left.
Obtain the byte C as follows:
<ul>
<li>the bits 0-2 of C are the same as the bits 9-11 of B</li>
<li>the bits 3-7 of C are the same as the bits 20-24 of B</li>
</ul>
</li>

<li>
Given the doublewords A si B, obtain the quadword C as follows:
<ul>
<li>the bits 0-7 of C are the same as the bits 21-28 of A</li>
<li>the bits 8-15 of C are the same as the bits 23-30 of B complemented</li>
<li>the bits 16-21 of C have the value 101010</li>
<li>the bits 22-31 of C have the value 0</li>
<li>the bits 32-42 of C are the same as the bits 21-31 of B</li>
<li>the bits 43-55 of C are the same as the bits 1-13 of A</li>
<li>the bits 56-63 of C are the same as the bits 24-31 of the result A XOR 0ABh</li>
</ul>
</li>

<li>
Given the word A,obtain the doubleword B as follows:
<ul>
<li>the bits 0-3 of B are the same as the bits 1-4 of the result A XOR 0Ah</li>
<li>the bits 4-11 of B are the same as the bits 7-14 of A</li>
<li>the bits 12-19 of B have the value 0</li>
<li>the bits 20-25 of B have the value 1</li>
<li>the bits 26-31 of C are the same as the bits 3-8 of A complemented</li>
</ul>
</li>

<li>
Given the words A, B si C, obtain the word D as the sum of the integers represented by:
<ul>
<li>the bits 1-5 of A</li>
<li>the bits 6-10 of B</li>
<li>the bits 11-15 of C</li>
</ul>
</li>

<li>
Given the words A, B si C, obtain the byte D as the sum of the integers represented by:
<ul>
<li>the bits 0-4 of A</li>
<li>the bits 5-9 of B</li>
</ul>
The byte E is the integer represented by the bits 10-14 of C. 
Obtain the byte F by computing the subtraction D-E.  
</li>

</ol>	