---
title: "Lab 5, week 6, 3.11.2025-9.11.2025" 
date: 2025-11-02
lastmod: 2025-09-24
tags: ["lab", "csa"]
author: ["Suciu Mihai"]
description: "lab 5 csa" 
summary: "Instructions for comparisons, conditional jumps and repetitive loops; string operations." 

showToc: true
disableAnchoredHeadings: false

---

<!-- <h1>Laboratory 5 - Instructions for comparisons, conditional jumps and repetitive loops. String operations.</h1> -->

## Theory
__In this lab__: instructions that compare operands, conditional jump instructions, string instructions.

### 1. The CMP instruction - Compare Two Operands

```asm
CMP D, S; effect: D-S, adjust flags according to the result
```


- The *CMP* instruction (see documentation [here](../../../instr/cmp.html)) computes a fictional substraction between the numerical values of the two operands: *D-S*.
- => it compares the value of the operands (without modifying them) by doing a fictive *D-S* affecting only the flags:OF,SF,ZF,AF,PF şi CF	


- __CMP__ decrements the value of the source operand from the destination operand, but unlike the instruction <strong>SUB</strong>, the result is not retained, it does not affect any of the operands' initial values. The effect of this instruction is only to change the value of some flags in accordance with the D-S operation. The instruction <strong>CMP</strong> is most often used in combination with conditional jump instructions.
- <strong>Although its name is CMP, it is important to underline that in reality this instruction DOES NOT COMPARE anything, without having any criteria to compare and in fact it does not take any decision, but it only PREPARES the decision with the flag set, the actual comparison and the corresponding decision being taken concretely by the conditional jump instruction that will be used after the CMP instruction! If we do not use any decision-making instructions later, the CMP does not have any concrete role in any comparison, it is just a simple fictive subtraction with the role of changing flags, and it will not represent the CMP (compare) name.</strong>
- The destination operator may be a register or variable in memory.
- The source operand may be a register, a variable in memory or a constant.
- Both operands of the CMP instruction must be of the same size.

__Example 1:__
```asm
cmp eax, ebx ; ”compares” the values stored in the two registers (fictional subtraction eax-ebx)
jle done ;Depending on the conditional jump instruction used (here JLE), the comparison criteria is established.
;In this case: If the EAX content in the signed interpretation is less than or equal to the content in EBX then JUMP to the Done label. 
;Otherwise continue with the following instruction (the flag being tested here is ZF).
done:
;instructions after the label 'done'
```

__Example 2:__
```asm
mov al,200 ; AL = C8h
mov bl,100
cmp al, bl ; fictive subtraction al-bl and set flangs 
; accordingly (in our case this means SF=0, OF=1, CF=0 şi ZF=0)
JB et2 ;the conditional jump statement establishes the comparison criterion, in this case Jump if Below - comparison for unsigned numbers (is 200 below 100?) and test CF content: if CF = 1 the jump will be performed, if CF = 0 NO jump will be done. 
;In our case CF=0, so the jump will not be performed.
;............. ;instructions set
et2:
;............. ;instructions set after this label
```

__Example 3:__
```asm 
mov al,-56 ; AL = C8h = 200 in unsigned interpretation
mov bl,100
cmp al, bl ;fictive subtraction al-bl which sets the flags (for our case, we will have SF = 0, OF = 1, CF = 0 and ZF = 0)
JNGE et2 ;verify condition JNGE - Jump if not greater or equal 
;(SIGNED comparison -56 versus 100) 
;verifies in fact if the two flags, SF and OF, have different values 
; Considering that in our case SF=0 and OF=1, so SF <> OF, the condition is fullfilled (and truly -56 is „NOT GREATER OR ;EQUAL” to 100) so the jump to label et2 will be performed
mov dx,1 
et2:
mov cx,1
```

__Example 4:__
```asm 
mov al,-56 ; AL = C8h = 200 in unsigned interpretation
mov bl,100
cmp al, bl ;fictive subtraction al-bl is performed, setting   flags accordingly (in our case SF=0, OF=1, CF=0 and ZF=0)
JNBE et2 ;verify condition JNBE - Jump if not below or equal 
;(UNSIGNED comparison   200 versus 100)
;verifies in fact if CF=0 and ZF=0
;Considering that in our case CF=0 and ZF=0, the condition is fulfilled (and truly 200 is „NOT BELOW OR EQUAL” ;to 100) so the jump to label et2 will be performed
mov dx,1
et2:
mov cx,1
```

### 2. The TEST instruction - Logical Compare

```asm
TEST D,S; fictive execution  D AND S; OF = 0, CF = 0, SF,ZF,PF -  modified, AF - undefined
```

- The *TEST* instruction (see documentation [here](../../../instr/test.html)) performs the *AND* logical operation between the two operands (fictive execution of ops AND opd) without saving the result of the operation in any of the operands.
- Both operands of the TEST instructions must be of the same size.
- The only effect of a TEST statement is to modify the flag content specified in the above table, corresponding to the result of the AND operation performed.

__Example 1:__
```asm
bothOpZero: ;instruction set for this label....
test ECX, ECX ; set ZF to 1 if ECX == 0
je bothOpZero ; according to the jump instruction utilised
; (here JE) we establish the comparison criteria. 
;In our case: jump to label if ZF = 1
; Alternatively, the jz bothOpZero option could be used here, these two conditional jump instructions (JE and JZ) being similar in terms of the tested condition (true if ZF = 1)
```

__Example 2:__
```asm
mov AH,[v]
test AH,0F2h 
js et2 ; according to the jump instruction utilised (here JS) we establish the comparison criteria.  
; In our case: if the result of the operation AH AND 0F2h is a negative number in signed interpretation (the signed bit of the result is 1) then Jump ;to label et2 (the flag that is set is SF).
et2: ;instructions set for this label....
```

- Similar to the situation from the CMP instruction, we have to make the following observation:
- __Although its name is TEST, it is important to underline that in reality this instruction DOES NOT TEST anything, not establishing any test criteria and not actually taking any decision, but it just PREPARES the appropriate decision with the set flags, for the test criterion, the actual testing and the decision corresponding to the concrete conditional jump instruction that will be used after the TEST statement! If we do not use any decision-making instructions later, TEST has no specific role in doing any test, it is just a simple AND bit-by-bit operation with affecting flags and will not be representive of its name TEST).__

### 3. Conditional jumps determined by flags

- When comparing two signed numbers, the terms "*less than*" and "*greater than*" are used, and when comparing two unsigned numbers, the terms "*below*" and respectively "*above*" are used.

__JMP – Unconditional Jump__ 

- see documentation [here](../../../instr/jmp.html)

| Mnemonic | Description |
| ---- | ---- |
|*JMP rel8*|Jump short, relative, displacement relative to next instruction.|
|*JMP rel16*| 	Jump near, relative, displacement relative to next instruction.|
|*JMP rel32*	| Jump near, relative, displacement relative to next instruction.|
|*JMP r/m16*	|Jump near, absolute indirect, address given in r/m16.|
|*JMP r/m32*	|Jump near, absolute indirect, address given in r/m32.|
|*JMP ptr16:16*	|Jump far, absolute, address given in operand.|
|*JMP ptr16:32*|	Jump far, absolute, address given in operand.|
|*JMP m16:16*|	Jump far, absolute indirect, address given in m16:16.|
|*JMP m16:32*|	Jump far, absolute indirect, address given in m16:32.|

- Transfers program control to a different point in the instruction stream without recording return information.
- The destination (target) operand specifies the address of the instruction being jumped to.
- This operand can be an immediate value, a general-purpose register, or a memory location.
- This instruction can be used to execute four different types of jumps:
- __Near jump__: A jump to an instruction within the current code segment (the segment currently pointed to by the CS register), sometimes referred to as an intrasegment jump.
- __Short jump__: A near jump where the jump range is limited to -128 to +127 from the current EIP value.
- __Far jump__: A jump to an instruction located in a different segment than the current code segment but at the same privilege level, sometimes referred to as an intersegment jump.

__Jcc — Jump if Condition Is Met__

- See documentation [here](../../../instr/jcc.html)
- The conditions for each *Jcc mnemonic* are given in the "{description}" column of the table on the preceding page. The terms "*less*" and "*greater*" are used for comparisons of signed integers and the terms "above" and "below" are used for unsigned integers.

|	Mnemonic	|	Description	|
| ---- | ----|
|	JA rel8	|	Jump short if above (CF=0 and ZF=0).	|
|	JAE rel8	|	Jump short if above or equal (CF=0).	|
|	JB rel8	|	Jump short if below (CF=1).	|
|	JBE rel8	|	Jump short if below or equal (CF=1 or ZF=1).	|
|	JC rel8	|	Jump short if carry (CF=1).	|
|	JCXZ rel8	|	Jump short if CX register is 0.	|
|	JECXZ rel8	|	Jump short if ECX register is 0.	|
|	JE rel8	|	Jump short if equal (ZF=1).	|
|	JG rel8	|	Jump short if greater (ZF=0 and SF=OF).	|
|	JGE rel8	|	Jump short if greater or equal (SF=OF).	|
|	JL rel8	|	Jump short if less (SF<>OF).	|
|	JLE rel8	|	Jump short if less or equal (ZF=1 or SF<>OF).	|
|	JNA rel8	|	Jump short if not above (CF=1 or ZF=1).	|
|	JNAE rel8	|	Jump short if not above or equal (CF=1).	|
|	JNB rel8	|	Jump short if not below (CF=0).	|
|	JNBE rel8	|	Jump short if not below or equal (CF=0 and ZF=0).	|
|	JNC rel8	|	Jump short if not carry (CF=0).	|
|	JNE rel8	|	Jump short if not equal (ZF=0).	|
|	JNG rel8	|	Jump short if not greater (ZF=1 or SF<>OF).	|
|	JNGE rel8	|	Jump short if not greater or equal (SF<>OF).	|
|	JNL rel8	|	Jump short if not less (SF=OF).	|
|	JNLE rel8	|	Jump short if not less or equal (ZF=0 and SF=OF).	|
|	JNO rel8	|	Jump short if not overflow (OF=0).	|
|	JNP rel8	|	Jump short if not parity (PF=0).	|
|	JNS rel8	|	Jump short if not sign (SF=0).	|
|	JNZ rel8	|	Jump short if not zero (ZF=0).	|
|	JO rel8	|	Jump short if overflow (OF=1).	|
|	JP rel8	|	Jump short if parity (PF=1).	|
|	JPE rel8	|	Jump short if parity even (PF=1).	|
|	JPO rel8	|	Jump short if parity odd (PF=0).	|
|	JS rel8	|	Jump short if sign (SF=1).	|
|	JZ rel8	|	Jump short if zero (ZF = 1).	|
|	JA rel16/32	|	Jump near if above (CF=0 and ZF=0).	|
|	JAE rel16/32	|	Jump near if above or equal (CF=0).	|
|	JB rel16/32	|	Jump near if below (CF=1).	|
|	JBE rel16/32	|	Jump near if below or equal (CF=1 or ZF=1).	|
|	JC rel16/32	|	Jump near if carry (CF=1).	|
|	JE rel16/32	|	Jump near if equal (ZF=1).	|
|	JZ rel16/32	|	Jump near if 0 (ZF=1).	|
|	JG rel16/32	|	Jump near if greater (ZF=0 and SF=OF).	|
|	JGE rel16/32	|	Jump near if greater or equal (SF=OF).	|
|	JL rel16/32	|	Jump near if less (SF<>OF).	|
|	JLE rel16/32	|	Jump near if less or equal (ZF=1 or SF<>OF).	|
|	JNA rel16/32	|	Jump near if not above (CF=1 or ZF=1).	|
|	JNAE rel16/32	|	Jump near if not above or equal (CF=1).	|
|	JNB rel16/32	|	Jump near if not below (CF=0).	|
|	JNBE rel16/32	|	Jump near if not below or equal (CF=0 and ZF=0).	|
|	JNC rel16/32	|	Jump near if not carry (CF=0).	|
|	JNE rel16/32	|	Jump near if not equal (ZF=0).	|
|	JNG rel16/32	|	Jump near if not greater (ZF=1 or SF<>OF).	|
|	JNGE rel16/32	|	Jump near if not greater or equal (SF<>OF).	|
|	JNL rel16/32	|	Jump near if not less (SF=OF).	|
|	JNLE rel16/32	|	Jump near if not less or equal (ZF=0 and SF=OF).	|
|	JNO rel16/32	|	Jump near if not overflow (OF=0).	|
|	JNP rel16/32	|	Jump near if not parity (PF=0).	|
|	JNS rel16/32	|	Jump near if not sign (SF=0).	|
|	JNZ rel16/32	|	Jump near if not zero (ZF=0).	|
|	JO rel16/32	|	Jump near if overflow (OF=1).	|
|	JP rel16/32	|	Jump near if parity (PF=1).	|
|	JPE rel16/32	|	Jump near if parity even (PF=1).	|
|	JPO rel16/32	|	Jump near if parity odd (PF=0).	|
|	JS rel16/32	|	Jump near if sign (SF=1).	|
|	JZ rel16/32	|	Jump near if 0 (ZF=1).	|


- These jumps checks the state of one or more of the status flags in the EFLAGS register (CF, OF, PF, SF, and ZF) and, if the flags are in the specified state (condition), performs a jump to the target instruction specified by the destination operand.
- A condition code (cc) is associated with each instruction to indicate the condition being tested for. If the condition is not satisfied, the jump is not performed and execution continues with the instruction following the Jcc instruction.
- The target instruction is specified with a relative offset (a signed offset relative to the current value of the instruction pointer in the EIP register).
- __A relative offset (rel8, rel16, or rel32) is generally specified as a label in assembly code, but at the machine code level, it is encoded as a signed, 8-bit or 32-bit immediate value, which is added to the instruction pointer.__
- Instruction coding is most efficient for offsets of -128 to +127. If the operand-size attribute is 16, the upper two bytes of the EIP register are cleared, resulting in a maximum instruction pointer size of 16 bits.
- The Jcc instruction does not support far jumps (jumps to other code segments). When the target for the conditional jump is in a different segment, use the opposite condition from the condition being tested for the Jcc instruction, and then access the target with an unconditional far jump (JMP instruction) to the other segment.
- The JECXZ and JCXZ instructions differ from the other Jcc instructions because they do not check the status flags. Instead they check the contents of the ECX and CX registers, respectively, for 0. Either the CX or ECX register is chosen according to the address-size attribute.
These instructions are useful at the beginning of a conditional loop that terminates with a conditional loop instruction (such as LOOPNE). They prevent entering the loop when the ECX or CX register is equal to 0, which would cause the loop to execute 232 or 64K times, respectively, instead of zero times.
- Summarizing, by analyzing the table above, one observes that some instructions test exactly the same condition, so they are similar in effect. The fact that the same instruction appears in several equivalent syntactic forms comes from the possibility of expressing the same situation under several equivalent formulations. For example, the condition op1 "less than or equal" (JLE) than op2 can also be expressed in op1 "NO is greater than" (JNG) op2. Similarly, the condition JB (Jump if below) can also be expressed as "NOT above or equal" (JNAE), etc.
- In the following table, you have all the instructions equivalent as effect grouped, their equivalence being given precisely on the basis of the tested condition. As a result, it's the same if you use a JAE, JNB or JNC comparison, their effect being the same: "jump if CF = 0".

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td><strong>&nbsp;&nbsp;MNEMONIC</strong></td>
<td><strong>MEANING (jump if..&lt;&lt;relation&gt;&gt;)</strong></td>
<td><strong>Condition verified</strong></td>
</tr>
<tr>
<td><strong>JB</strong>
<br>
<strong>JNAE</strong>
<br>
<strong>JC</strong></td>
<td>is lower
<br>
is not above or equal
<br>
there is carry</td>
<td>CF=1</td>
</tr>
<tr>
<td><strong>JAE</strong>
<br>
<strong>JNB</strong>
<br>
<strong>JNC</strong></td>
<td>is above or equal
<br>
is not below
<br>
there is no carry</td>
<td>CF=0</td>
</tr>
<tr>
<td><strong>JBE</strong>
<br>
<strong>JNA</strong></td>
<td>is below or equal
<br>
is not above</td>
<td>CF=1 or ZF=1</td>
</tr>
<tr>
<td><strong>JA</strong>
<br>
<strong>JNBE</strong></td>
<td>is above
<br>
is not below or equal</td>
<td>CF=0&nbsp; and&nbsp; ZF=0</td>
</tr>
<tr>
<td><strong>JE</strong>
<br>
<strong>JZ</strong></td>
<td>is equal
<br>
is zero</td>
<td>ZF=1</td>
</tr>
<tr>
<td><strong>JNE</strong>
<br>
<strong>JNZ</strong></td>
<td>is not equal
<br>
is not zero</td>
<td>ZF=0</td>
</tr>
<tr>
<td><strong>JL</strong>
<br>
<strong>JNGE</strong></td>
<td>is lower than
<br>
is not greater than</td>
<td>SF≠OF</td>
</tr>
<tr>
<td><strong>JGE</strong>
<br>
<strong>JNL</strong></td>
<td>is greater than
<br>
is not lower than</td>
<td>SF=OF</td>
</tr>
<tr>
<td><strong>JLE</strong>
<br>
<strong>JNG</strong></td>
<td>is lower or equal
<br>
is not greater than</td>
<td>ZF=1 or SF≠OF</td>
</tr>
<tr>
<td><strong>JG</strong>
<br>
<strong>JNLE</strong></td>
<td>is greater than
<br>
nis not lower or equal to</td>
<td>ZF=0 and SF=OF</td>
</tr>
<tr>
<td><strong>JP</strong>
<br>
<strong>JPE</strong></td>
<td>has parity
<br>
parity is even</td>
<td>PF=1</td>
</tr>
<tr>
<td><strong>JNP</strong>
<br>
<strong>JPO</strong></td>
<td>no parity
<br>
parity is odd</td>
<td>PF=0</td>
</tr>
<tr>
<td><strong>JS</strong></td>
<td>negative signed</td>
<td>SF=1</td>
</tr>
<tr>
<td><strong>JNS</strong></td>
<td>not negative sign</td>
<td>SF=0</td>
</tr>
<tr>
<td><strong>JO</strong></td>
<td>we have overflow</td>
<td>OF=1</td>
</tr>
<tr>
<td><strong>JNO</strong></td>
<td>no overflow</td>
<td>OF=0</td>
</tr>
</tbody>
</table>

- In order to facilitate the correct choice by the programmer of the variants of the conditional jump in relation to the result of a comparison (ie, if the programmer wants to interpret the result of the comparison for numbers with a sign or without sign) we give the following table:

|	Relation to test between operands	|	Signed comparison	|	Unsigned comparison	|
| ---- | ---- | ---- |
|	d = s	|	JE	|	JE	|
|	d ≠ s	|	JNE	|	JNE	|
|	d > s	|	JG	|	JA	|
|	d < s	|	JL	|	JB	|
|	d ≥ s	|	JGE	|	JAE	|
|	d ≤ s	|	JLE	|	JBE	|

- The study of these tables reiterates our previous statement: it is not the CMP's instruction that distinguishes between a sign of comparison and a sign without sign! The role of interpreting differently (with or without sign) the final result of the comparison is done ONLY by the specified jump instructions AFTER the comparison was made.
- The table above is very useful for interpreting the results of arithmetic instructions in general. Without executing a CMP statement, considering d as the result of the last executed arithmetic instruction and putting s = 0, the table remains valid.


### 4. Repetitive loop instructions

- The x86 processors have special loop instructions. They are: LOOP, LOOPE, LOOPNE and JECXZ.
- The sintax is:
```asm
instruction label
```

- The LOOP instruction orders the restart of the instruction block execution beginning on the label as long as the value in the ECX register is different to 0. First, the ECX register is decremented and then the test and possibly the jump are performed. The jump is necessarily "short" this time (up to 127 bytes - so be careful about the "distance" between the LOOP and the tag!).

__Example:__
```asm 
mov ecx, 5
start_loop:
; the code here would be executed 5 times
loop start_loop
```

- If the ending conditions of the cycle are more complex, the LOOPE and LOOPNE instructions can be used.
- The LOOPE statement (LOOP while Equal) differs from LOOP by the termination condition, the cycle terminating either if ECX = 0 or ZF = 0. For LOOPNE (LOOP while Not Equal), the cycle will end either if ECX = 0 or if ZF = 1. Even if the cycle output is based on the value in the ZF, the ECX is still decremented.
- LOOPE is also known as LOOPZ and LOOPNE is also known as LOOPNZ. These instructions are usually preceded by a CMP or SUB statement.
- The JECXZ and JCXZ instructions are conditional jump instructions but differ from the standard set of these instructions by not checking the status of the flags. JECXZ and JCXZ only check if the content in ECX or CX is 0. These instructions can be used at the beginning of a loop ending with a conditional loop to prevent loop entry when the ECX or CX is 0. If it enters the loop with CX = 0, given that "first decrementing of the ECX register is performed and then the test is performed and possibly the jump" this will cause the execution of a loop of 2 ^ 32 times or 2 ^ 16 times, instead of 0 times such as normal based on CX = 0.

__Example:__
```asm
mov ecx, numar ; number of iterations
JECXZ endFor	;skip loop if numar=0
forIndex
; instructiuni
Loop forIndex	; repeat
endFor
```

- equivalent to:
```asm
mov ECX, numar
cmp ECX,0
JZ endFor
forIndex
; instructions
Loop forIndex	; repeat
endFor
```

- It is important to note here that none of the presented cycling instructions affect the flags.

## Code Example
```asm
;Given a lower case string of bytes obtain the corresponding upper case string
bits 32 
global start        
extern exit,printf ; tell nasm that exit exists even if we won't be defining it
import exit msvcrt.dll ; exit is a function that ends the calling process. It is defined in msvcrt.dll
; msvcrt.dll contains exit, printf and all the other important C-runtime specific functions
; our data is declared here (the variables needed by our program)
segment data use32 class=data
	s db 'a', 'b', 'c', 'm','n' ; declare the string of bytes
	l equ $-s ; compute the length of the string in l
	d times l db 0 ; reserve l bytes for the destination string and initialize it
segment code use32 class=code
start:
	mov ecx, l ; we put the length l in ECX in order to make the loop
	mov esi, 0     
	jecxz Sfarsit      
	Repeta:
		mov al, [s+esi]
		mov bl, 'a'-'A' ; in order to obtain the corresponding upper case letter, we will decrease the ASCII CODE
		; of 'a'-'A' from the lower case letter AL
		sub al, bl             
		mov [d+esi], al    
		inc esi
	loop Repeta
	Sfarsit:;end of the program

	; exit(0)
	push dword 0 ; push the parameter for exit onto the stack
	call [exit] ; call exit to terminate the program
```

## Exercices
<ol><li>Given a byte string S of length l, obtain the string D of length l-1 as D(i) = S(i) * S(i+1)  (each element of D is the product of two consecutive elements of S).
<br><strong>Example:</strong><br>
<pre><code>S: 1, 2, 3, 4
D: 2, 6, 12
</code></pre></li><li>
Given a character string S, obtain the string D containing all special characters (!@#$%^&*) of the string S. 
<br><strong>Example:</strong><br>
<pre><code>S: '+', '4', '2', 'a', '@', '3', '$', '*'
D: '@','$','*'
</code></pre></li><li>
Two byte strings S1 and S2 are given. Obtain the string D by concatenating the elements of S1 from the left hand side to the right hand side and the elements of S2 from the right hand side to the left hand side.
    <br><strong>Example:</strong><br>
<pre><code>S1: 1, 2, 3, 4
S2: 5, 6, 7
D: 1, 2, 3, 4, 7, 6, 5
</code></pre></li><li>Two byte strings S1 and S2 are given, having the same length. Obtain the string D in the following way: each element found on the even positions of D is the sum of the corresponding elements from S1 and S2, and each element found on the odd positions of D is the difference of the corresponding elements from S1 and S2.
    <br><strong>Example:</strong><br>
<pre><code>S1: 1, 2, 3, 4
S2: 5, 6, 7, 8
D: 6, -4, 10, -4
</code></pre></li><li>A character string S is given. Obtain the string D containing all small letters from the string S.
<br><strong>Example:</strong><br>
<pre><code>S: 'a', 'A', 'b', 'B', '2', '%', 'x'
D: 'a', 'b', 'x'
</code></pre></li><li>A byte string S is given. Obtain the string D by concatenating the elements found on the even positions of S and then the elements found on the odd positions of S.
    <br><strong>Example:</strong><br>
<pre><code>S: 1, 2, 3, 4, 5, 6, 7, 8
D: 1, 3, 5, 7, 2, 4, 6, 8
</code></pre></li><li>Two byte string S1 and S2 are given, having the same length. Obtain the string D by intercalating the elements of the two strings.
    <br><strong>Example:</strong><br>
<pre><code>S1: 1, 3, 5, 7
S2: 2, 6, 9, 4
D: 1, 2, 3, 6, 5, 9, 7, 4
</code></pre></li><li>A character string S is given. Obtain the string D that contains all capital letters of the string S.
<br><strong>Example:</strong><br>
<pre><code>S: 'a', 'A', 'b', 'B', '2', '%', 'x', 'M'
D: 'A', 'B', 'M'
</code></pre></li><li>A byte string S of length l is given. Obtain the string D of length l-1 so that the elements of D represent the difference between every two consecutive elements of S.
    <br><strong>Example:</strong><br>
<pre><code>S: 1, 2, 4, 6, 10, 20, 25
D: 1, 2, 2, 4, 10, 5
</code></pre></li><li>Two character strings S1 and S2 are given. 
Obtain the string D by concatenating the elements of S2 in reverse order and the elements found on even positions in S1.
<br><strong>Example:</strong><br>
<pre><code>S1: '+', '2', '2', 'b', '8', '6', 'X', '8'
S2: 'a', '4', '5'
D: '5', '4', 'a', '2','b', '6', '8'
</code></pre></li><li>A byte string S is given. Obtain the string D1 which contains all the even numbers of S and the string D2 which contains all the odd numbers of S.
    <br><strong>Example:</strong><br>
<pre><code>S: 1, 5, 3, 8, 2, 9
D1: 8, 2
D2: 1, 5, 3, 9
</code></pre></li><li>Two character strings S1 and S2 are given. 
Obtain the string D by concatenating the elements found on even positions in S2 and the elements found on odd positions in S1.
<br><strong>Example:</strong><br>
<pre><code>S1: 'a', 'b', 'c', 'd', 'e', 'f'
S2: '1', '2', '3', '4', '5'
D: '2', '4','a','c','e'
</code></pre></li><li>A byte string S is given. Obtain the string D1 which contains the elements found on the even positions of S and the string D2 which contains the elements found on the odd positions of S.
    <br><strong>Example:</strong><br>
<pre><code>S: 1, 5, 3, 8, 2, 9
D1: 1, 3, 2
D2: 5, 8, 9
</code></pre></li><li>A byte string S is given. Obtain the string D1 which contains all the positive numbers of S and the string D2 which contains all the negative numbers of S.
    <br><strong>Example:</strong><br>
<pre><code>S: 1, 3, -2, -5, 3, -8, 5, 0
D1: 1, 3, 3, 5, 0
D2: -2, -5, -8
</code></pre></li><li>Two byte strings A and B are given.
Obtain the string R by concatenating the elements of B in reverse order and the odd elements of A.
<br><strong>Example:</strong><br>
<pre><code>A: 2, 1, 3, 3, 4, 2, 6
B: 4, 5, 7, 6, 2, 1
R: 1, 2, 6, 7, 5, 4, 1, 3, 3
</code></pre></li><li>Two character strings S1 and S2 are given. 
Obtain the string D by concatenating the elements found on odd positions in S2 and the elements found on even positions in S1.
<br><strong>Example:</strong><br>
<pre><code>S1: 'a', 'b', 'c', 'b', 'e', 'f'
S2: '1', '2', '3', '4', '5'
D: '1', '3', '5', 'b', 'b', 'f'
</code></pre></li><li>
Two byte strings S1 and S2 are given, having the same length. Obtain the string D so that each element of D represents the maximum of the corresponding elements from S1 and S2.
    <br><strong>Example:</strong><br>
<pre><code>S1: 1, 3, 6, 2, 3, 7
S2: 6, 3, 8, 1, 2, 5
D: 6, 3, 8, 2, 3, 7
</code></pre></li><li>
Two byte strings A and B are given.
Obtain the string R that contains only the odd positive elements of the two strings.
<br><strong>Example:</strong><br>
<pre><code>A: 2, 1, 3, -3
B: 4, 5, -5, 7
R: 1, 3, 5, 7
</code></pre></li><li>
Two byte strings A and B are given.
Obtain the string R that contains only the even negative elements of the two strings.
<br><strong>Example:</strong><br>
<pre><code>A: 2, 1, 3, -3, -4, 2, -6
B: 4, 5, -5, 7, -6, -2, 1
R: -4, -6, -6, -2
</code></pre></li><li>
Two byte strings A and B are given.
Obtain the string R by concatenating the elements of B in reverse order and the even elements of A.
<br><strong>Example:</strong><br>
<pre><code>A: 2, 1, 3, 3, 4, 2, 6
B: 4, 5, 7, 6, 2, 1
R: 1, 2, 6, 7, 5, 4, 2, 4, 2, 6
</code></pre></li><li>
Two byte strings A and B are given.
Obtain the string R by concatenating the elements of B in reverse order and the negative elements of A.
<br><strong>Example:</strong><br>
<pre><code>A: 2, 1, -3, 3, -4, 2, 6
B: 4, 5, 7, 6, 2, 1
R: 1, 2, 6, 7, 5, 4, -3, -4
</code></pre></li><li>
Two byte strings S1 and S2 of the same length are given.
Obtain the string D where each element contains the minimum of the corresponding elements from S1 and S2. 
<br><strong>Example:</strong><br>
<pre><code>S1: 1, 3, 6, 2, 3, 7
S2: 6, 3, 8, 1, 2, 5
D: 1, 3, 6, 1, 2, 5
</code></pre></li><li>
A byte string S is given. Obtain in the string D the set of the elements of S.
    <br><strong>Example:</strong><br>
<pre><code>S: 1, 4, 2, 4, 8, 2, 1, 1
D: 1, 4, 2, 8
</code></pre></li><li>
Two byte strings A and B are given.
Obtain the string R by concatenating the elements of B in reverse order and the elements of A in reverse order.
<br><strong>Example:</strong><br>
<pre><code>A: 2, 1, -3, 0
B: 4, 5, 7, 6, 2, 1
R: 1, 2, 6, 7, 5, 4, 0, -3, 1, 2
</code></pre></li><li>
Two character strings S1 and S2 are given. Obtain the string D which contains all the elements of S1 that do not appear in S2.
    <br><strong>Example:</strong><br>
<pre><code>S1: '+', '4', '2', 'a', '8', '4', 'X', '5'
S2: 'a', '4', '5'
D: '+', '2', '8', 'X'
</code></pre></li><li>
A byte string S is given. Obtain the maximum of the elements found on the even positions and the minimum of the elements found on the odd positions of S.
    <br><strong>Example:</strong><br>
<pre><code>S: 1, 4, 2, 3, 8, 4, 9, 5
max_poz_pare: 9
min_poz_impare: 3
</code></pre></li><li>
Two byte strings S1 and S2 of the same length are given.
Obtain the string D where each element is the difference of the corresponding elements from S1 and S2
<br><strong>Example:</strong><br>
<pre><code>S1: 1, 3, 6, 2, 3, 2
S2: 6, 3, 8, 1, 2, 5
D: -5, 0, -2, 1, 1, -3
</code></pre></li><li>
Two character strings S1 and S2 are given. Obtain the string D by concatenating the elements found on the positions multiple of 3 from S1 and the elements of S2 in reverse order.
    <br><strong>Example:</strong><br>
<pre><code>S1: '+', '4', '2', 'a', '8', '4', 'X', '5'
S2: 'a', '4', '5'
D: '+', 'a', 'X', '5', '4', 'a'
</code></pre></li><li>
A byte string S is given. Build the string D whose elements represent the sum of each two consecutive bytes of S.
<br><strong>Example:</strong><br>
<pre><code>S: 1, 2, 3, 4, 5, 6
D: 3, 5, 7, 9, 11
</code></pre></li><li>
Two byte strings S1 and S2 of the same length are given.
Obtain the string D where each element is the sum of the corresponding elements from S1 and S2
<br><strong>Example:</strong><br>
<pre><code>S1: 1, 3, 6, 2, 3, 2
S2: 6, 3, 8, 1, 2, 5
D: 7, 6, 14, 3, 5, 7
</code></pre></li><li>
A character string S is given. Obtain the string D which contains all the digit characters of S.
    <br><strong>Example:</strong><br>
<pre><code>S: '+', '4', '2', 'a', '8', '4', 'X', '5'
D: '4', '2', '8', '4', '5'
</code></pre></li><li>
A byte string S of length l is given.
Obtain the string D of length l-1, such that each element of D is the quotient of two consecutive elements of S, i.e. D(i) = S(i) / S(i+1).
<br><strong>Example:</strong><br>
<pre><code>S: 1, 6, 3, 1
D: 0, 2, 3
</code></pre></li></ol>	
		
		
		
			
			
</div>