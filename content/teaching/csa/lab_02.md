---
title: "Lab 2" 
date: 2025-10-05
lastmod: 2025-09-24
tags: ["lab", "csa"]
author: ["Suciu Mihai"]
description: "lab 2 csa." 
summary: "lab 2 csa." 
# cover:
#     image: "comp_sec.jpeg"
#     alt: "Figure caption"
#     relative: true
showToc: true
disableAnchoredHeadings: false

---

<h1>Laboratory 2</h1>

# Theory

## Arithmetic instructions
- instructions used in this laboratory: ADD, SUB, MUL, DIV, INC, DEC, NEG
- [link](../../../instr) to instruction reference (just search for the desired instruction in this page and click on the appropriate link)

## Example
```asm
; Write a program in the assembly language that computes the following arithmetic expression, considering the following data types for the variables:
; a - byte, b - word
; (b / a * 2 + 10) * b - b * 15;
; ex. 1: a = 10; b = 40; Result: (40 / 10 * 2 + 10) * 40 - 40 * 15 = 18 * 40 - 1600 = 720 - 600 = 120
; ex. 2: a = 100; b = 50; Result: (50 / 100 * 2 + 10) * 50 - 50 * 15 = 12 * 50 - 750 = 600 - 750 = - 150
bits 32 ;assembling for the 32 bits architecture
; the start label will be the entry point in the program
global  start 

extern  exit ; we inform the assembler that the exit symbol is foreign, i.e. it exists even if we won't be defining it

import  exit msvcrt.dll; exit is a function that ends the process, it is defined in msvcrt.dll
        ; msvcrt.dll contains exit, printf and all the other important C-runtime functions
segment  data use32 class=data ; the data segment where the variables are declared 
	a  db 10
	b  dw 40
segment  code use32 class=code ; code segment
start: 
	mov  AX, [b] ;AX = b
	div  BYTE [a] ;AL = AX / a = b / a and AH = AX % a = b % a
	
	mov  AH, 2 ;AH = 2
	mul  AH ;AX = AL * AH = b / a * 2	
	
	add  AX, 10 ;AX = AX + b = b / a * 2 + 10
	
	mul  word [b] ;DX:AX = AX * b = (b / a * 2 + 10) * b
	
	push  DX ;the high part of the doubleword DX:AX is saved on the stack
	push  AX ;the low part of the doubleword DX:AX is saved on the stack
	pop  EBX ;EBX = DX:AX = (b / a * 2 + 10) * b
	
	mov  AX, [b] ;AX = b
	mov  DX, 15 ;DX = 15
	mul  DX ;DX:AX = b * 15
	
	push  DX ;the high part of the doubleword DX:AX is saved on the stack
	push  AX ;the low part of the doubleword DX:AX is saved on the stack
	pop  EAX ;EAX = DX:AX = b * 15
	
	sub  EBX, EAX ;EBX = EBX - EAX = (b / a * 2 + 10) * b - b * 15
	
	push   dword 0 ;saves on stack the parameter of the function exit
	call   [exit] ; function exit is called in order to end the execution of the program
```

## Problems
Write a program in the assembly language that computes the following arithmetic expression, considering the given data types for the variables.

### Simple exercises
Compute and analyze the result:

<h3>Compute and analyze the result:</h3>
		<ol>
		<li>1+9
</li><li>1+15
</li><li>128+128
</li><li>5-6
</li><li>10/4
</li><li>256*1
</li><li>256/1
</li><li>128+127
</li><li>3*4
</li><li>9+7
</li><li>128*2
</li><li>4-5
</li><li>2+8
</li><li>-2*5
</li><li>6*3
</li><li>4*4
</li><li>14+2
</li><li>127+129
</li><li>12/4
</li><li>13/3
</li><li>15/3
</li><li>16/4
</li><li>256*1
</li><li>256/1
</li><li>64*4
</li><li>3-4
</li><li>4+12
</li><li>13/5
</li><li>14/6
</li><li>11+5</li>
		</ol>
		</div>
		<h2>Additions, subtractions</h2>
		<div class="article">
		<h3>a,b,c,d - byte</h3>
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
</li><li>(a+c-d) +d - (b+b-c)
</li><li>2-(c+d)+(a+b-c)
</li><li>a+b-c+d-(a-d)
</li><li>(a+d-c)-(b+b)
</li><li>a-b-d+2+c+(10-b)
</li><li>a+13-c+d-7+b
</li><li>(a+a-c)-(b+b+b+d)
</li><li>d-(a+b)+c
</li><li>d-(a+b)-c
</li><li>(a+a)-(c+b+d)
</li><li>(a-b)+(d-c)
</li><li>(a+b+b)-(c+d)
</li><li>(a-c)+(b+b+d)
</li><li>(a-b-b-c)+(a-c-c-d)
</li><li>(c+d+d)-(a+a+b)
</li><li>(a+a)-(b+b)-c
</li><li>(a+b-c)-(a+d)
</li><li>a+b-c+d
</li><li>(b+c)+(a+b-d)
</li><li>d-(a+b)-(c+c)</li>
		</ol>
		<h3>a,b,c,d - word</h3>
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
</li><li>(b-c)+(d-a)
</li><li>d-(a+b)-(c+c)
</li><li>(a+a-c)-(b+b+d)
</li><li>(c+d)+(a-b)+a
</li><li>(a-b+c)-(d+d)
</li><li>(a+b+b)+(c-d)
</li><li>a+a-b-c-(d+d)
</li><li>(a-b-c)+(a-c-d-d)
</li><li>b+a-(4-d+2)+c+(a-b)
</li><li>b-(b+c)+a
</li><li>a-c+d-7+b-(2+d)
</li><li>(b-a)-(c+c+d)
</li><li>(a+b+c)-(d+d)
</li><li>(a-c)+(b-d)
</li><li>(a+b-c)-d
</li><li>(a+c)-(b+b+d)
</li><li>a+b-(c+d)+100h
</li><li>(d-c)+(b+b-c-a)+d
</li><li>(d-a)+(b+b+c)
</li><li>a-b+(c-d+a)</li>
		</ol>
		</div>
		<h2>Multiplications, divisions</h2>
		<div class="article">
		<h3>a,b,c - byte, d - word</h3>
		<ol>
			<li>((a+b-c)*2 + d-5)*d
</li><li>d*(d+2*a)/(b*c)
</li><li>[-1+d-2*(b+1)]/a
</li><li>–a*a + 2*(b-1) – d
</li><li>[d-2*(a-b)+b*c]/2
</li><li>[2*(a+b)-5*c]*(d-3)
</li><li>[100*(d+3)-10]/d
</li><li>(100*a+d+5-75*b)/(c-5)
</li><li>3*[20*(b-a+2)-10*c]+2*(d-3)
</li><li>3*[20*(b-a+2)-10*c]/5
</li><li>[(d/2)*(c+b)-a*a]/b
</li><li>a*[b+c+d/b]+d
</li><li>[(a*b)-d]/(b+c)<!--14-->
</li><li>(d-b*c+b*2)/a
</li><li>(a*2)+2*(b-3)-d-2*c
</li><li>(a+b)/2 + (10-a/c)+b/4
</li><li>300-[5*(d-2*a)-1]
</li><li>200-[3*(c+b-d/a)-300]<!--19-->
</li><li>[(a-b)*3+c*2]-d
</li><li>(50-b-c)*2+a*a+d
</li><li>d-[3*(a+b+2)-5*(c+2)]
</li><li>[(10+d)-(a*a-2*b)]/c
</li><li>[(a+b)*3-c*2]+d
</li><li>(10*a-5*b)+(d-5*c)
</li><li>[100-10*a+4*(b+c)]-d
</li><li>d+[(a+b)*5-(c+c)*5]
</li><li>d/[(a+b)-(c+c)]
</li><li>d+10*a-b*c
</li><li>[d-(a+b)*2]/c
</li><li>[(a-b)*5+d]-2*c</li>
		</ol>
		<h3>a,b,c,d-byte, e,f,g,h-word</h3>
		<ol>
			<li>((a-b)*4)/c
</li><li>e-a*a
</li><li>(e+f)*g
</li><li>(a-c)*3+b*b
</li><li>a*(b+c)+34 <!---->
</li><li>(a*b)/c
</li><li>(a+b)*(c+d)
</li><li>2*(a+b)-e
</li><li>(2*d+e)/a
</li><li>a*d+b*c
</li><li>(e+f)*(2*a+3*b)<!---->
</li><li>(a*d+e)/[c+h/(c-b)]
</li><li>(g+5)-a*d
</li><li>a*d*e/(f-5)
</li><li>f*(e-2)/[3*(d-5)]<!---->
</li><li>a*a-(e+f)
</li><li>h/a + (2 + b) + f/d – g/c
</li><li>f+(c-2)*(3+a)/(d-4)
</li><li>(e + g) * 2 / (a * c) + (h – f) + b * 3
</li><li>[(a+b+c)*2]*3/g<!---->
</li><li>(f*g-a*b*e)/(h+c*d)
</li><li>(a+(b-c))*3
</li><li>[(a+b)*2]/(a+d)
</li><li>[(a-d)+b]*2/c
</li><li>(e+f+g)/(a+b)
</li><li>(e+g-2*b)/c
</li><li>[(e+f-g}+(b+c)*3]/5
</li><li>(e+g-h)/3+b*c
</li><li>[[b*c-(e+f)]/(a+d)
</li><li>100/(e+h-3*a)</li>
		</ol>
		</div>
	</div>		