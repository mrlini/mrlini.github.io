---
title: "Lab 3" 
date: 2025-10-12
lastmod: 2025-09-24
tags: ["lab", "csa"]
author: ["Suciu Mihai"]
description: "lab 3 csa." 
summary: "Complex arithmetic expressions." 

showToc: true
disableAnchoredHeadings: false

---

<h1>Laboratory 3 - complex arithmetic expressions</h1>

__In this lab__: complex arithmetic expressions (additions, subtractions, multiplications, divisions, conversions, variable declarations).

# Theory

## Arithmetic instructions
- [ADC](../../../instr/adc.html) - Add with Carry,
- [SBB](../../../instr/sbb.html) - Integer Subtraction with Borrow,
- [IMUL](../../../instr/imul.html) - Signed Multiply,
- [IDIV](../../../instr/idiv.html) - Signed Divide.

## Signed conversion instructions
- [CBW](../../../instr/CBW_CWDE_CDQE.html) - Convert Byte to Word
- [CWD](../../../instr/CWD_CDQ_CQO.html) - Convert Word to Doubleword (DX:AX ← sign-extend of AX)
- [CWDE](../../../instr/CBW_CWDE_CDQE.html) - Convert Word to Doubleword (EAX ← sign-extend of AX)
- [CDQ](../../../instr/CWD_CDQ_CQO.html) - Convert Doubleword to Quadword (EDX:EAX ← sign-extend of EAX)

## Unsigned conversions
There are no instructions for conversions in the unsigned representation! In assembly language, the unsigned conversions are done by putting 0 in the high byte, word or doubleword:

```asm
mov AH,0 ; for converting AL → AX
mov DX,0 ; for converting AX → DX:AX
mov EDX,0 ; for converting EAX → EDX:EAX
```

## Declaring variables / constants
- Declaring variables and assigning an initial value:
```asm
a DB 0A2h ;declaring the variable a of type BYTE and initialising it with the value 0A2h
b DW 'ab' ;declaring the variable a of type WORD and initialising it with the value 'ab'
c DD 12345678h ;declaring the variable a of type DOUBLE WORD and initialising it with the value 12345678h
d DQ 1122334455667788h ;declaring the variable a of type QUAD WORD and initialising it with the value 1122334455667788h
```

- Declaring variables without assignment (reserve space in memory):
```asm
a RESB 1 ;reserving 1 byte
b RESB 64 ;reserving 64 bytes
c RESW 1 ;reserving 1 word
```

- Defining constants:
```asm
ten EQU 10 ;defining the constant ten that has the value 10
```

## Notations
```
<op8>    - operand on 8 bits
<op16>   - operand on 16 bits
<op32>   - operand on 32 bits

<reg8>   - register on 8 bits
<reg16>  - register on 16 bits
<reg32>  - register on 32 bits
<reg>    - register
<regd>   - destination register 
<regs>   - source register 

<mem8>   - memory variable on 8 bits
<mem16>  - memory variable on 16 bits
<mem32>  - memory variable on 32 bits
<mem>    - memory variable

<con8>   - constant (immediate value) on 8 bits
<con16>  - constant (immediate value) on 16 bits
<con32>  - constant (immediate value) on 32 bits
<con>    - constant (immediate value)
```

## Little endian representation
<ol>
  <li>Each  byte has an address (the byte being the smallest addressable unit of memory)</li>
  <li>An <em>address </em><u>identifies in a unique way</u> a location in memory, and x86  processors assign each byte location a separate memory address </li>
  <li>x86  processors store and retrieve data from memory using <strong><em>little endian </em>order  (the byte representing the &bdquo;end&rdquo; of the number will be stored at the  &bdquo;little&rdquo;-est address)</strong>: </li>
  <ol>
    <li>The  least significant byte is stored at the beginning of that memory area (at the address  where allocation for the data begins). </li>
    <li>The  remaining bytes are stored in reverse order in the next consecutive memory  positions.</li>
  </ol>
</ol>

<p>(much  more clearer as an informal statement:   
if we have on paper or in a register a 4 bytes number and we denote the order  of these bytes as 1 2 3 4 , in the memory the little-endian representation will  store that number in the reverse order of its bytes: 4 3 2 1).
<blockquote class="i">Careful ! – only  the BYTES order is reversed <strong><u>in memory</u></strong>,  NOT the BITS inside that bytes !!!!! The order of the bits which compose each  byte remains the same !</blockquote><br>
For  instance, if we have the following data segment:

```asm
a db 12h
b dw 3456h
c dd 7890abcdh
d dq 1122334455667788h 
```
Data representation in memory will be as shown in the lowest left corner from the below debugger configuration:

![debugger](../../../3le_1.jpg)

### Example 1: Addition: quadword+quadword
```asm 
bits 32 ;assembling for the 32 bits architecture
; the start label will be the entry point in the program
global  start 

extern  exit ; we inform the assembler that the exit symbol is foreign, i.e. it exists even if we won't be defining it

import  exit msvcrt.dll; exit is a function that ends the process, it is defined in msvcrt.dll
        ; msvcrt.dll contains exit, printf and all the other important C-runtime functions
segment  data use32 class=data ; the data segment where the variables are declared 
	a dq 1122334455667788h
	b dq 0abcdef1a2b3c4d5eh
	r resq 1 ; reserve 1 quadword in memory to save the result
; our code starts here
segment  code use32 class=code ; code segment
start: 
	;11223344  55667788 h -> EDX : EAX 
	;   EDX   :   EAX 
	mov eax, dword [a+0] 
	mov edx, dword [a+4] 

	;  abcdef1a 2b3c4d5e h  -> ECX : EBX 
	;  ECX     :   EBX 
	mov ebx, dword [b+0] 
	mov ecx, dword [b+4] 

	;a + b 
	; edx :  eax + 
	; ecx :  ebx 
	clc ; clear Carry Flag (punem 0 in CF) 
	add eax, ebx  ; eax=  eax+ebx 
	adc edx, ecx ; edx =  edx+ecx + CF 
	;(CF is set is add eax, ebx produce a carry) 

	;edx:eax  -> r 
	mov dword [r+0], eax 
	mov dword [r+4], edx 
	push  dword 0  ; push  the parameter for exit onto the stack 
	call  [exit] ; call exit to terminate the program
```

Step 1 – before perform the addition
![before addition](../../../3le_2.jpg)

Step 2 – after addition
![after addition](../../../3le_3.jpg)


### Example 2. Division: quadword/doubleword
```asm
bits 32 ;assembling for the 32 bits architecture
; the start label will be the entry point in the program
global  start 

extern  exit ; we inform the assembler that the exit symbol is foreign, i.e. it exists even if we won't be defining it

import  exit msvcrt.dll; exit is a function that ends the process, it is defined in msvcrt.dll
        ; msvcrt.dll contains exit, printf and all the other important C-runtime functions
segment  data use32 class=data ; the data segment where the variables are declared 
	m dq 1122334455667788h 
	n  dd 0ccddeeddh 
	rezd  resd 1 
	; our code starts here 
segment  code use32 class=code ; code segment
start: 
	mov  ebx, [n] 
	
	;11223344  55667788 h -> EDX : EAX 
	;   EDX   :   EAX 
	mov eax, dword [m+0] 
	mov edx, dword [m+4] 
	
	div ebx ; edx:eax/ebx=eax cat si edx rest 
	
	mov dword[rezd], eax 
	
	push  dword 0  ; push  the parameter for exit onto the stack 
	call  [exit] ; call exit to terminate the program
```

Debugger
![debugger](../../../3le_4.jpg)

## Example - arithmetic expressions
__Unsigned representation__
```asm
; Write a program in assembly language which computes one of the following arithmetic expressions, considering the following domains for the variables: 
; a - doubleword; b, d - byte; c - word; e - qword
; a + b / c - d * 2 - e
bits 32 ;assembling for the 32 bits architecture
; the start label will be the entry point in the program
global  start 

extern  exit ; we inform the assembler that the exit symbol is foreign, i.e. it exists even if we won't be defining it

import  exit msvcrt.dll; exit is a function that ends the process, it is defined in msvcrt.dll
        ; msvcrt.dll contains exit, printf and all the other important C-runtime functions
segment  data use32 class=data ; the data segment where the variables are declared 
	a dd 125
	b db 2
	c dw 15
	d db 200
	e dq 80
segment  code use32 class=code ; code segment
start: 
	;for computing b/c, we convert b from byte to doubleword, so that we can divide it by the word c
	mov al, [b]
	mov ah, 0 ;unsigned conversion from al to ax
	mov dx, 0 ;unsigned conversion from ax to dx:ax
	;dx:ax = b
	div word [c] ;unsigned division dx:ax by c
	;ax=b/c	
	;catul impartirii este in ax (restul este in dx, dar mergem mai departe doar cu catul)

	mov bx, ax ;we save b/c in bx so that we can use ax for multiplying d by 2
	mov al, 2
	mul byte [d] ;ax=d*2

	sub bx, ax ;bx = b / c - d * 2
	; we convert the word bx to doubleword so that we can add it with the doubleword a
	mov cx, 0 ; unsigned conversion from bx to cx:bx
	;cx:bx=b/c-d*2

	mov ax, word [a]
	mov dx, word [a+2] ;dx:ax=a

	add ax, bx
	adc dx, cx ;dx:ax = a + b / c - d * 2
	
	push dx
	push ax
	pop eax ;eax = a + b / c - d * 2
	
	mov edx, 0 ;edx:eax = a + b / c - d * 2
	sub eax, dword [e]
	sbb edx, dword [e+4] ;edx:eax = a + b / c - d * 2 - e
	
	push   dword 0 ;saves on stack the parameter of the function exit
	call   [exit] ; function exit is called in order to end the execution of the program

```

__Signed representation__
```asm
; Write a program in the assembly language that computes the following arithmetic expression, considering the following data types for the variables:
; a - doubleword; b, d - byte; c - word; e - qword
; a + b / c - d * 2 - e
bits 32 ;assembling for the 32 bits architecture
; the start label will be the entry point in the program
global  start 

extern  exit ; we inform the assembler that the exit symbol is foreign, i.e. it exists even if we won't be defining it

import  exit msvcrt.dll; exit is a function that ends the process, it is defined in msvcrt.dll
        ; msvcrt.dll contains exit, printf and all the other important C-runtime functions
segment  data use32 class=data ; the data segment where the variables are declared 
	a dd 125
	b db 2
	c dw 15
	d db 200
	e dq 80
segment  code use32 class=code ; code segment
start: 	
	;for computing b/c, we convert b from byte to doubleword, so that we can divide it by the word c
	mov al, [b]
	cbw ;signed conversion from al to ax
	cwd ;signed conversion from ax to dx:ax
	;dx:ax = b
	idiv word [c] ;signed division dx:ax by c
	;ax=b/c	
	;the quotient of the division is in ax (the remainder is in dx, but we only use the quotient in the further computations)

	mov bx, ax ;we save b/c in bx so that we can use ax for multiplying d and 2	
	mov al, 2
	imul byte [d] ;ax=d*2	

	sub bx, ax ;bx=b/c-d*2	
	; we convert the word bx to doubleword so that we can add it with the doubleword a	
	mov ax, bx
	cwd ; signed conversion from ax to dx:ax	
	;dx:ax=b/c-d*2	

	mov bx, word [a]	
	mov cx, word [a+2] ;cx:bx=a	

	add ax, bx	
	adc dx, cx ;the result of a + b / c - d * 2 is in dx:ax	
	
	push dx
	push ax
	pop eax ;eax = a + b / c - d * 2
	cdq ;edx:eax = a + b / c - d * 2	
	
	sub eax, dword [e]
	sbb edx, dword [e+4] ;edx:eax = a + b / c - d * 2 - e
	
	push   dword 0 ;saves on stack the parameter of the function exit
	call   [exit] ; function exit is called in order to end the execution of the program
```

## Problems
Write a program in assembly language that computes one of the following arithmetic expressions, considering the following domains for the variables (in the unsigned and signed representation).

### Additions, subtractions
__a - byte, b - word, c - double word, d - qword - Unsigned representation__
<ol>
<li>c-(a+d)+(b+d)
</li><li>(b+b)+(c-a)+d
</li><li>(c+d)-(a+d)+b
</li><li>(a-b)+(c-b-d)+d
</li><li>(c-a-d)+(c-b)-a
</li><li>(a+b)-(a+d)+(c-a)
</li><li>c-(d+d+d)+(a-b)
</li><li>(a+b-d)+(a-b-d)
</li><li>(d+d-b)+(c-a)+d
</li><li>(a+d+d)-c+(b+b)
</li><li>(d-c)+(b-a)-(b+b+b)
</li><li>(a+b+d)-(a-c+d)+(b-c)
</li><li>d-b+a-(b+c)
</li><li>(a+d)-(c-b)+c
</li><li>a+b-c+(d-a)
</li><li>c-a-(b+a)+c
</li><li>(c+c-a)-(d+d)-b
</li><li>(d+d)-a-b-c
</li><li>(d+d)-(a+a)-(b+b)-(c+c)
</li><li>(a+c)-b+a + (d-c)
</li><li>(c-a) + (b - d) +d
</li><li>(d+c) - (c+b) - (b+a)
</li><li>((a + a) + (b + b) + (c + c)) - d
</li><li>((a + b) + (a + c) + (b + c)) - d
</li><li>(a + b + c) - (d + d) + (b + c)
</li><li>(c-b+a)-(d+a)
</li><li>(a+c)-(d+b)
</li><li>d-(a+b)+(c+c)
</li><li>d+c-b+(a-c)
</li><li>(b+c+a)-(d+c+a)</li>
</ol>

__a - byte, b - word, c - double word, d - qword - Signed representation__
<ol>
<li>(c+b+a)-(d+d)
</li><li>(c+b)-a-(d+d)
</li><li>(b+b+d)-(c+a)
</li><li>(b+b)-c-(a+d)
</li><li>(c+b+b)-(c+a+d)
</li><li>c-(d+a)+(b+c)
</li><li>(c+c+c)-b+(d-a)
</li><li>(b+c+d)-(a+a)
</li><li>a-d+b+b+c
</li><li>b+c+d+a-(d+c)
</li><li>d-(a+b+c)-(a+a)
</li><li>(a-b-c)+(d-b-c)-(a-d)
</li><li>(b-a+c-d)-(d+c-a-b)
</li><li>c-b-(a+a)-b
</li><li>c+a+b+b+a
</li><li>(d-a)-(a-c)-d
</li><li>(c+d-a)-(d-c)-b
</li><li>(d-b)-a-(b-c)
</li><li>(d+a)-(c-b)-(b-a)+(c+d)
</li><li>a-b-(c-d)+d
</li><li>d-a+(b+a-c)
</li><li>c+b-(a-d+b)
</li><li>a + b + c + d - (a + b)
</li><li>(a + b + c) - d + (b - c)
</li><li>(a + b - c) + (a + b + d) - (a + b)
</li><li>(c-d-a)+(b+b)-(c+a)
</li><li>(d+d-c)-(c+c-a)+(c+a)
</li><li>c+d-a-b+(c-a)
</li><li>(a+a)-(b+b)-(c+d)+(d+d)
</li><li>d-a+c+c-b+a</li>
</ol>

### Multiplications, divisions  - Unsigned representation and signed representation
<ol>
<li>c+(a*a-b+7)/(2+a), a-byte; b-doubleword; c-qword<br>
</li><li>2/(a+b*c-9)+e-d; a,b,c-byte; d-doubleword; e-qword<br>
</li><li>(8-a*b*100+c)/d+x; a,b,d-byte; c-doubleword; x-qword<br>
</li><li>(a*2+b/2+e)/(c-d)+x/a; a-word; b,c,d-byte; e-doubleword; x-qword<br>
</li><li>(a+b/c-1)/(b+2)-x; a-doubleword; b-byte; c-word; x-qword<br>
</li><li>x+a/b+c*d-b/c+e; a,b,d-byte; c-word; e-doubleword; x-qword<br>
</li><li>(a-2)/(b+c)+a*c+e-x; a,b-byte; c-word; e-doubleword; x-qword<br>
</li><li>1/a+200*b-c/(d+1)+x/a-e; a,b-word; c,d-byte; e-doubleword; x-qword<br>
</li><li>(a-b+c*128)/(a+b)+e-x; a,b-byte; c-word; e-doubleword; x-qword<br>
</li><li>d-(7-a*b+c)/a-6+x/2; a,c-byte; b-word; d-doubleword; x-qword<br>
</li><li>(a+b)/(2-b*b+b/c)-x; a-doubleword; b,c-byte; x-qword<br>
</li><li>(a*b+2)/(a+7-c)+d+x; a,c-byte; b-word; d-doubleword; x-qword<br>
</li><li>x-(a+b+c*d)/(9-a); a,c,d-byte; b-doubleword; x-qword<br>
</li><li>x+(2-a*b)/(a*3)-a+c; a-byte; b-word; c-doubleword; x-qword<br>
</li><li>x-(a*b*25+c*3)/(a+b)+e; a,b,c-byte; e-doubleword<br>
</li><li>x/2+100*(a+b)-3/(c+d)+e*e; a,c-word; b,d-byte; e-doubleword; x-qword<br>
</li><li>x-(a*a+b)/(a+c/a); a,c-byte; b-doubleword; x-qword<br>
</li><li>(a+b*c+2/c)/(2+a)+e+x; a,b-byte; c-word; e-doubleword; x-qword<br>
</li><li>(a+a+b*c*100+x)/(a+10)+e*a; a,b,c-byte; e-doubleword; x-qword<br>
</li><li>x-b+8+(2*a-b)/(b*b)+e; a-word; b-byte; e-doubleword; x-qword<br>
</li><li>(a*a/b+b*b)/(2+b)+e-x; a-byte; b-word; e-doubleword; x-qword<br>
</li><li>a/2+b*b-a*b*c+e+x; a,b,c-byte; e-doubleword; x-qword<br>
</li><li>(a*b-2*c*d)/(c-e)+x/a; a,b,c,d-byte; e-word; x-qword<br>
</li><li>a-(7+x)/(b*b-c/d+2); a-doubleword; b,c,d-byte; x-qword<br>
</li><li>(a*a+b+x)/(b+b)+c*c; a-word; b-byte; c-doubleword; x-qword<br>
</li><li>(a*a+b/c-1)/(b+c)+d-x; a-word; b-byte; c-word; d-doubleword; x-qword<br>
</li><li>(100+a+b*c)/(a-100)+e+x/a; a,b-byte; c-word; e-doubleword; x-qword<br>
</li><li>x-(a*100+b)/(b+c-1); a-word; b-byte; c-word; x-qword<br>
</li><li>(a+b)/(c-2)-d+2-x; a,b,c-byte; d-doubleword; x-qword<br>
</li><li>a*b-(100-c)/(b*b)+e+x; a-word; b,c-byte; e-doubleword; x-qword</li>
</ol>