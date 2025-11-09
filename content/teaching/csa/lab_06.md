---
title: "Lab 6, week 7, 10.11.2025-16.11.2025" 
date: 2025-11-09
lastmod: 2025-09-24
tags: ["lab", "csa"]
author: ["Suciu Mihai"]
description: "lab 6 csa" 
summary: "Instructions working with strings of bytes/words/double words." 

showToc: true
disableAnchoredHeadings: false

---

# Theory
__In this lab__: instructions that work on strings: string operations, data transfer, data comparison, repetitive execution of an instruction.

## String operations

Instructions working on strings have default operands (parameters).
Types of string instructions:

  1. which use a source string and a destination string (MOVSB, MOVSW, MOVSD, CMPSB, CMPSW, CMPSD)
  2. which use only a source string (LODSB, LODSW, LODSD)
  3. which use only a destination string (STOSB, STOSW, STOSD, SCASB, SCASW, SCASD)

- You will remember them more easily
  - **MOVS**_X_ = **MOV**e **S**tring at _X_ level, where _X_ is B - byte, W - word, D - double word => MOVSB - moves bytes of a string from one location to another
  - **CMPS**_X_ = **C**o**MP**are **S**tring at _X_ level
  - **LODS**_X_ = **LO**a**D** **S**tring at _X_ level
  - **STOS**_X_ = **STO**re **S**tring at _X_ level
  - **SCAS**_X_ = **SCA**n **S**tring at _X_ level

A string is characterized by:
- the __type__ of the elements (bytes or words) => is given by the last letter of the instruction that is used (B=byte, W=word, D=doubleword), both strings having the same type
- the __address__ of the first element => is a FAR memory address which is memorised in:
  - in DS:ESI - for the source string
  - in ES:EDI - for the destination string
- the __parsing direction__ => is given by the value of the DF flag (0 - from small addresses to large addresses, 1 - from large addresses to small addresses.)
- the __number of elements__ => when needed is placed in CX or ECX

## Instructions for data transfer

| instruction | explanation|
|---|---|
|LODSB	|The byte from the address <DS:ESI> is loaded in AL. If DF=0 then inc(ESI), else dec(ESI)|
|LODSW|	The word from the address <DS:ESI> is loaded in AX. If DF=0 then ESI:=ESI+2, else ESI:=ESI-2|
|LODSD|	The double word from the address <DS:ESI> is loaded in EAX. If DF=0 then ESI:=ESI+4, else ESI:=ESI-4|
|STOSB|	Store AL into the byte from the address <ES:EDI>. If DF=0 then inc(EDI), else dec(EDI)|
|STOSW|	Store AX into the word from the address <ES:EDI>.If DF=0 then EDI:= EDI+2, else EDI:= EDI-2|
|STOSD|	Store EAX into the double word from the address <ES:EDI>. If DF=0 then EDI:= EDI+4, else EDI:= EDI-4|
|MOVSB|	Store the byte from the address <DS:ESI> to the address <ES:EDI>.If DF=0 then inc(SI), inc(DI), else dec(SI), dec(DI)|
|MOVSW|	Store the word from the address <DS:ESI> to the address <ES:EDI>.If DF=0 then ESI:= ESI+2, EDI:= EDI+2, else ESI:= ESI-2, EDI:= EDI-2|
|MOVSD|	Store the double word from the address <DS:ESI> to the address <ES:EDI>.If DF=0 then ESI:= ESI+4, EDI:= EDI+4, else ESI:= ESI-4, EDI:= EDI-4|

__Obs.__ Considering the use of the flat memory model, at any start of the program execution, the OS will initialize with the same value the segment registers DS = ES. The programmer has no responsibility in what concerns loading / updating / modifying these values. In the source code that uses the instructions on strings the programmer will only need to manage the offset of these strings.

Example
```asm
;We have a source string of words. Copy this string into another string. We assume we know the length of this string.
mov ECX, dim_sir ; no of elements in string
mov ESI, sir_sursa ; load offset sir_sursa in ESI
mov EDI, sir_dest ; load offset sir_dest in EDI
CLD
Again:
	LODSW
	STOSW
LOOP Again
```

Considering that LODS + STOS = MOVS the above loop is equivalent to:
```asm
Again:
MOVSW
LOOP Again
```

or (see the prefix instruction section below)
```asm
rep MOVSW
```

## Instructions for data consultation and comparison

| instruction | explanation|
|---|---|
|SCASB|	CMP AL, <ES:EDI>. If DF=0 Then inc(EDI), Else dec(EDI)|
|SCASW|	CMP AX, <ES:EDI>. If DF=0 Then EDI:= EDI+2, Else EDI:= EDI-2|
|SCASD|	CMP EAX, <ES:EDI>. If DF=0 Then EDI:= EDI+4, Else EDI:= EDI-4|
|CMPSB|	CMP <DS:ESI>, <ES:EDI>. If DF=0 Then inc(ESI), inc(EDI), Else dec(ESI), dec(EDI)|
|CMPSW|	CMP <DS:ESI>, <ES:EDI>. If DF=0 Then ESI:= ESI+2, EDI:= EDI+2, Else ESI:= ESI-2, EDI:= EDI-2|
|CMPSD|	CMP <DS:ESI>, <ES:EDI>. If DF=0 Then ESI:= ESI+4, EDI:= EDI+4, Else ESI:= ESI-4, EDI:= EDI-4|

Example
```asm
;A sequence of bytes is given. Find the last character "0".
;... all data about the "destination" string is loaded
MOV AL, '0'
MOV ECX, lung_sir
STD
Cont_caut: ;continue search...
	SCASB
	JE Found
LOOP Cont_caut
;...
Found:
	INC EDI;I return to the character found before EDI was decremented 
```

## Prefix instructions for the repetitive execution of a string instruction

```asm
repetitive_prefix string_instruction 
```
is equivalent to, assuming ECX is set 
```asm
Again:
	string_instruction 
LOOP Again
```

- where repetitive_prefix can be REP, equivalent to REPE (Repeat While Equal), REPZ (Repeat While Zero) - which repeat the execution of instructions SCAS or CMPS until ECX becomes 0 or an unmatch occurs ( => ZF=0)
- or it can be REPNE (Repeat While Not Equal) or REPNZ (Repeat While Not Zero) - which repeat the execution of instructions SCAS or CMPS until ECX becomes 0 or when a match occurs ( => ZF=1)

__Observations__:
- string instructions don not change the flags as a result of modifications to ESI, EDI or ECX
- LODS, STOS, MOVS - do not change any flag, while SCAS and CMPS change the flags because they compare data.

## Example
```asm
;Problem. A string of quadwords is given. Compute the number of multiples of 8 from
;the string of the low bytes of the high word of the high doubleword from the elements of the quadword string 
;and find the sum of the digits (in base 10) of this number.


;Solution: We first parse the quadword string and obtain the number of multiples of 8
;from the string of the low bytes of the high word of the high doubleword from the elements of the quadword string.
;Then we obtain the digits of this number by successive divisions to 10 and then compute the sum of these digits.


bits 32 
global start
extern exit; tell nasm that exit exists even if we won't be defining it
import exit msvcrt.dll; exit is a function that ends the calling process. It is defined in msvcrt.dll
; our data is declared here (the variables needed by our program)
segment data use32 class=data
  sir  dq  123110110abcb0h,1116adcb5a051ad2h,4120ca11d730cbb0h
  len equ ($-sir)/8;the length of the string (in quadwords)
  opt db 8;variabile used for testing divisibility to 8
  zece dd 10;variabile used for determining the digits in base 10 of a number by successive divisions to 10
  suma dd  0;variabile used for holding the sum of the digits 
; our code starts here
segment code use32 class=code
    start:
  mov esi, sir;in eds:esi we will store the FAR address of the string "sir"
  cld;parse the string from left to right(DF=0).    
  mov ecx, len;we will parse the elements of the string in a loop with len iterations.
  mov ebx, 0;in ebx we will hold the number of multiples of 8.
  repeta:
    lodsd;in eax we will have the low doubleword (least significant) of the current quadword from the string
    lodsd;in eax we will have the high doubleword (least significant) of the current quadword from the string
    shr eax, 16
    mov ah, 0;we are interested in the low byte (least significant) of this word (AL)
      
    div byte[opt];check whether al is divisible to 8
    cmp ah, 0;if the remainder is 0, resume the loop "repeta". 
        ;else increment the number of multiples of 8 from EBX. 
    jnz nonmultiplu
    inc ebx

    nonmultiplu:
  loop repeta;if there are more elements (ecx>0) resume the loop.

  ;next, we obtain the 10-th base digits of the number EBX by successive divisions to 10 and then compute the sum of these digits

  mov eax, ebx
  mov edx, 0
    
  transf:
    div dword[zece];divide the number by 10 in order to obtain the last digit; this digit will be in EDX
    add dword[suma], edx;add the digit to the sum.
    cmp eax, 0
  jz sfarsit;if the quotient (from al) is 0 it means we obtained all the digits and we can leave the loop "transf"
        ;else prepare the quotient for a new iteration 
  mov edx, 0        
  jmp transf;resume the loop in order to obtain a new digit.

sfarsit:;end the program
           
        push dword 0; push the parameter for exit onto the stack
        call [exit]; call exit to terminate the program
```

## Exercices
The following exercises need to be solved using specific string instructions: LODSB, STOSB, MOVSB, SCASB, CMPSB, LODSW, STOSW, MOVSW, SCASW, CMPSW, LODSD, STOSD, MOVSD, SCASD, CMPSD.

<ol><li>&nbsp;An array with doublewords containing packed data (4 bytes written as a single doubleword) is given. Write an asm program in order to obtain a new array of doublewords, where each doubleword will be composed by the rule: the sum of the bytes from an odd position will be written on the word from the odd position and the sum of the bytes from an even position will be written on the word from the even position. The bytes are considered to represent signed numbers, thus the extension of the sum on a word will be performed according to the signed arithmetic.<br>
  <h3><i>Example:</i></h3>for the initial array: <pre><code>127F5678h, 0ABCDABCDh, ...</code></pre>
  The following should be obtained: <pre><code>006800F7h, 0FF56FF9Ah, ... </code></pre></li>
<li> An array of words is given. Write an asm program in order to obtain an array of doublewords, where each doubleword will contain each nibble unpacked on a byte (each nibble will be preceded by a 0 digit), arranged in an ascending order within the doubleword.  <br>
  <h3><i>Example:</i></h3>for the initial array: <pre><code>1432h, 8675h, 0ADBCh, ...</code></pre>
  The following should be obtained: <pre><code>01020304h, 05060708h, 0A0B0C0Dh, ...</code></pre></li>
<li> An array of doublewords, where each doubleword contains 2 values on a word (unpacked, so each nibble is preceded by a 0) is given. Write an asm program to create a new array of bytes which contain those values (packed on a single byte), arranged in an ascending manner in memory, these being considered signed numbers. <br>
  <h3><i>Example:</i></h3>for the initial array: <pre><code>0702090Ah, 0B0C0304h, 05060108h  </code></pre>
  the following should be obtained: <pre><code>72h, 9Ah, 0BCh, 34h, 56h, 18h </code></pre>
  which arranged in an ascending manner will give: <pre><code>9Ah, 0BCh, 18h, 34h, 56h, 72h</code></pre></li>
<li> A byte string s is given. Build the byte string d such that every byte d[i] is equal to the count of ones in the corresponding byte s[i] of s.<br>
  <h3><i>Example:</i></h3>
  <pre><code>s: 5, 25, 55, 127</code></pre>
  in binary: <pre><code>101, 11001, 110111, 1111111</code></pre>
  <pre><code>d: 2, 3, 5, 7</code></pre>
</li>
<li>Two byte strings s1 and s2 are given. Build the byte string d such that, for every byte s2[i] in s2, d[i] contains either the position of byte s2[i] in s1, either the value of 0.<br>
  <h3><i>Example:</i></h3>
  <pre><code>pos:1 2 3 4 5</code></pre>
 <pre><code>s1: 7, 33, 55, 19, 46</code></pre>
  <pre><code>s2: 33, 21, 7, 13, 27, 19, 55, 1, 46 </code></pre>
  <pre><code>d: 2,  0, 1, 0, 0, 4, 3, 0, 5</code></pre></li>
<li> A word string s is given. Build the byte string d such that each element d[i] contains:<br>
  - the count of zeros in the word s[i], if s[i] is a negative number<br>
  - the count of ones in the word s[i], if s[i] is a positive number<br>
  <h3><i>Example:</i></h3>
  <pre><code>s: -22, 145, -48, 127</code></pre>
  in binary:<pre><code>1111111111101010, 10010001, 1111111111010000, 1111111</code></pre>
  <pre><code>d: 3, 3, 5, 7</code></pre></li>
<li> A string of doublewords is given. Obtain the list made out of the high bytes of the 
high words of each doubleword from the given list with the property that these bytes are 
multiple of 3. 
</br>
  <h3><i>Example:</i></h3>
  Given the string of doublewords:
  <pre><code>s DD 12345678h, 1A2B3C4Dh, FE98DC76h</code></pre>
  obtain the string of bytes:
  <pre><code>d: 12h.</code></pre></li>


<li>  A list of doublewords is given. Obtain the list made out of the low bytes of 
the high words of each doubleword from the given list with the property that these bytes are 
palindromes in base 10.
<br>
  <h3><i>Example:</i></h3>
  Given the string of doublewords:<br>
  <pre><code>s DD 12345678h, 1A2C3C4Dh, 98FCDC76h</code></pre>
  obtain the string of bytes:
  <pre><code>d DB 2Ch, FCh.</code></pre></li>
<li>A list of doublewords is given. Starting from the low part of the doubleword, obtain the doubleword made of 
the high even bytes of the low words of each doubleword from the given list. If there are not 
enough bytes, the remaining bytes of the doubleword will be filled with the byte FFh.
<br>
  <h3><i>Example:</i></h3>
  Given the string of doublewords:
 <pre><code> s DD 12345678h, 1A2C3C4Dh, 98FCDD76h, 12783A2Bh</code></pre>
  obtain the doubleword:
  <pre><code>d DD FF3A3C56h</code></pre>.</li>
<li> Given an array A of words, build two arrays of bytes:&nbsp;&nbsp;<br>
  &nbsp;- array B1 contains as elements the higher part of the words from A<br>
  &nbsp;- array B2 contains as elements the lower part of the words from A</li>

<li> Given an array A of doublewords, build two arrays of bytes:&nbsp;&nbsp;<br>
  &nbsp;- array B1 contains as elements the higher part of the higher words from A<br>
  &nbsp;- array B2 contains as elements the lower part of the lower words from A </li>
<li> Given an array A of doublewords, build two arrays of bytes:&nbsp;&nbsp;<br>
  &nbsp;- array B1 contains as elements the lower part of the lower words from A<br>
  &nbsp;- array B2 contains as elements the higher part of the higher words from A </li>

<li> Given an array S of doublewords, build the array of bytes D formed from lower bytes of lower words, bytes multiple of 7.<br>
  <h3><i>Example:</i></h3>
 <pre><code> s DD 12345607h, 1A2B3C15h, 13A33412h</code></pre>
 <pre><code> d DB 07h, 15h</code></pre></li>

<li> Given an array S of doublewords, build the array of bytes D formed from bytes of doublewords sorted as unsigned numbers in ascending order.<br>
  <h3><i>Example:</i></h3>
 <pre><code> s DD 12345607h, 1A2B3C15h</code></pre>
 <pre><code> d DB 07h, 12h, 15h, 1Ah, 2Bh, 34h, 3Ch, 56h</code></pre></li>
<li>Given an array S of doublewords, build the array of bytes D formed from bytes of doublewords sorted as unsigned numbers in descending order.<br>
 <h3><i>Example:</i></h3>
 <pre><code> s DD 12345607h, 1A2B3C15h</code></pre>
 <pre><code> d DB 56h, 3Ch, 34h, 2Bh, 1Ah, 15h, 12h, 07h</code></pre></li>
<li>
Being given two alphabetical ordered strings of characters, s1 and s2, build using merge sort the ordered string of bytes that contain all characters from s1 and s2.
</li><li>
A string of doublewords is given. Order in decreasing order the string of the low words (least significant) from these doublewords. The high words (most significant) remain unchanged. 
<h3><i>Example:</i></h3>
being given <pre><code>sir DD 12345678h 1256ABCDh, 12AB4344h</code></pre>
the result will be <pre><code>1234ABCDh, 12565678h, 12AB4344h.</code></pre>
</li><li>
A string of doublewords is given. Order in increasing order the string of the high words (most significant) from these doublewords. The low words (least significant) remain unchanged.
<h3><i>Example:</i></h3>
being given <pre><code>sir DD 12AB5678h, 1256ABCDh, 12344344h </code></pre>
the result will be <pre><code>12345678h, 1256ABCDh, 12AB4344h.</code></pre>
</li><li>
Being given two strings of bytes, compute all positions where the second string appears as a substring in the first string.
</li><li>
Being given a string of bytes representing a text (succession of words separated by spaces), determine which words are palindromes (meaning may be interpreted the same way in either forward or reverse direction); ex.: "cojoc", "capac" etc.
</li><li>
Being given a string of words, obtain the string (of bytes) of the digits in base 10 of each word from this string.  
<h3><i>Example:</i></h3>
being given the string <pre><code>sir DW 12345, 20778, 4596 </code></pre>
the result will be <pre><code>1, 2, 3, 4, 5, 2, 0, 7, 7, 8, 4, 5, 9, 6.</code></pre>
</li><li>
A string of bytes 'input' is given together with two additional strings of N bytes each, 'src' and 'dst'. Obtain a new string of bytes called 'output' from the 'input' string, by replacing all the bytes with the value src[i] with the new value dst[i], for i=1..N.
</li><li>
Being given a string of bytes, build a string of words which contains in the low bytes of the words the set of distinct characters from the given string and in the high byte of a word it contains the number of occurrences of the low byte of the word in the given byte string.
<h3><i>Example:</i></h3>
given the string <pre><code>sir DB 2, 4, 2, 5, 2, 2, 4, 4 </code></pre>
the result will be <pre><code>rez DW 0402h, 0304h, 0105h.</code></pre>
</li><li>
Being given a string of doublewords, build another string of doublewords which will include only the doublewords from the given string which have an even number of bits with the value 1.
</li><li>A string of bytes is given. Obtain the mirror image of the binary representation of this string of bytes. 
<h3><i>Example:</i></h3>
given the byte string <pre><code>s DB 01011100b, 10001001b, 11100101b </code></pre>
obtain the string <pre><code>d DB 10100111b, 10010001b, 00111010b.</code></pre>
</li><li> A string of doublewords is given. Compute the string formed by the high bytes of the low words from the elements of the doubleword string and these bytes should be multiple of 10. 
<h3><i>Example:</i></h3>
given the doublewords string: <pre><code>s DD 12345678h, 1A2B3C4Dh, FE98DC76h </code></pre>
obtain the string <pre><code>d DB 3Ch, DCh.</code></pre>
</li><li> Being given a string of words, compute the longest substring of ordered words (in increasing order) from this string.
</li><li> Being given a string of bytes and a substring of this string, eliminate all occurrences of this substring from the initial string.
</li><li>Two strings of bytes A and B are given. Parse the shortest string of those two and build a third string C as follows: 
<ul><li>- up to the lenght of the shortest string C contains the largest element of the same rank from the two strings</li>
<li>- then, up to the length of the longest string C will be filled with 1 and 0, alternatively.</li></ul>
</li><li>A string of words is given. Build two strings of bytes, s1 and s2, in the following way: for each word,  
<ul><li>- if the number of bits 1 from the high byte of the word is larger than the number of bits 1 from its low byte, then s1 will contain the high byte and s2 will contain the low byte of the word 
</li><li>- if the number of bits 1 from the high byte of the word is equal to the number of bits 1 from its low byte, then s1 will contain the number of bits 1 from the low byte and s2 will contain 0 
</li><li>- otherwise, s1 will contain the low byte and s2 will contain the high byte of the word.</li></ul>
</li>
</ol>   