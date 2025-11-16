---
title: "Lab 7, week 8, 17.11.2025-23.11.2025" 
date: 2025-11-16
lastmod: 2025-09-24
tags: ["lab", "csa"]
author: ["Suciu Mihai"]
description: "lab 7 csa" 
summary: "Function calls. Working with text files." 

showToc: true
disableAnchoredHeadings: false
---

# Theory
__In this lab__: call different functions, calling conventions, working with text files.

## Function calls
Even if the assembly language directly uses the hardware components of the system, there are frequently used pieces of code and writing them every single time would be impractical (for example, communicating with input/output devices, which often implies complex protocols). One of the roles of the operating system is to abstract the hardware machine for the programmer, providing multiple libraries of functions that can be called for certain frequent operations, such as printing the data in a certain format, finding a substring in a string or different mathematical functions.

Using a functions means transferring control to the procedure address, executing the code corresponding to the function, and returning to the instruction immediatley after the call of the function:
![function call](../../../lab7-img1.png)

## Using external functions. Call conventions
For functions calls we use the instruction CALL.

Syntax:
```asm
call <system_function_name>
```

Semantics:
- CALL pushes the address of the instruction immediately after itself (return address) onto the stack and then transfers control to the procedure address. This allows a RET instruction to pop the return address and thus return control to the instruction immediately after the CALL instruction.

## Calling a system function. Parameters.

Often functions have parameters and a return value. There are many conventions for passing parameters, but we will use the cdecl (which stands for C declaration) calling convention.

The calling convention is not related to the syntax of the assembly language, but it is a ”contract” between the caller and the callee, specifying how parameters are passed and how the value is returned.

___cdecl convention___


- Arguments are placed on the stack from right to left
- Implicit result returned by the functions are stored in EAX
- Registers EAX, ECX, EDX are used within the functions ( so they can be overwritten! )
- <span style="color:red">Pay attention! If you need the values soterd in EAX, ECX, EDS you need to store them somewhere else before the function call (in memory variables, on the stack or in different registers)</span>
- The function doesn’t empty the stack, it is the responsibility of the programmer to pop the arguments out of the stack after the function call.
- Example: printf, scanf

## Standard msvcrt functions.

### Printing on the screen
For printing a text on the screen we use the printf function, which requires a certain format.

Syntax of the printf function in a high level programming language is:
```c 
int printf(const char * format, variable_1, constant_2, ...);
```

- The printf function respects the cdecl convention.

- The first argument of the function is the a string containing the printing format, followed by the same number of arguments (constant values or variable names) as specified in the format.

- The character string representing the format can contain certain formatting markups, starting with the character ’%’, which will be replaced by the values given in the following arguments.

| Code | Type| Example | Representation | 
|---|---| --| --|
c|	Character	|a	|byte|
|d or i|	Signed decimal integer|	392|	dword|
|u	|Unsigned decimal integer	|7235	|dword|
|x|	Unsigned hexadecimal integer|	7fa	|dword|
|s|	String (terminated with a 0)	|example	|string of bytes terminated with 0|

<span style="color:red">When the function printf() is executed, the doubleword from the top of the stack contains the Return Address and below this doubleword we have another doubleword which contains the offset of the formatting string (which is the first argument of the printf function). On the stack, below this second doubleword we have other doublewords corresponding to the other arguments of the printf() function. The format descriptors %d, %i, %c, %x, %u will always be applied to a doubleword from the stack (even if the format descriptor is %c, it also uses a doubleword from the stack, but it will take only the least significant byte of this doubleword). Consequently, the first format descriptor will be applied to the third doubleword from the stack (counting down from the top of the stack). The second (possible) format descriptor will be applied o the fourth doubleword from the stack (counting down from the top of the stack), the third (possible) format descriptor from the formatting string will pe applied to the fifth doubleword from the stack (counting down from the top of the stack) and so on.</span>

___Examples___:

- ___Pprinting a message:___

In a high level programming language:
```C
printf("I have to go to school today.");
```
In assembly language:
```asm
segment data use32 class=data
	message  db "I have to go to school today.", 0  ; define the message
segment code use32 class=code
	; ...
	 push dword message  ; push the parameter on the stack
	 call [printf]       ; call the printf function
	 add esp, 4 * 1     ; remove the parameters from the stack
	; ...
```

- ___Printing a signed integer in base 10___

In a high level programming language:
```c
printf("%d", -17);
```
In assembly language:
```asm
segment data use32 class=data
	format  db "%d", 0  ; definining the format
segment code use32 class=code
	; ...
	 push dword -17  ; pushing the parameters on the stack from right to left
	 push dword format  
	 call [printf]       ; calling the printf function
	 add esp, 4 * 2     ; cleaning the parameters from the stack
	; ...
```

- ___Printing an integer in base 16___

In a high level programming language:
```c
printf("%x", 0xAB);
```
In assembly language:
```asm
segment data use32 class=data
	format  db "%x", 0  ; definining the format
segment code use32 class=code
	; ...
	 push dword 0xAB  ; pushing the parameters on the stack from right to left
	 push dword format  
	 call [printf]       ; calling the printf function
	 add esp, 4 * 2     ; cleaning the parameters from the stack
	; ...
```

- ___Printing a message that contains an integer in base 10, stored in a variable n___

In a high level programming language:
```c
printf("It's week %d of school", n);
```
In assembly language:
```asm
segment data use32 class=data
	n dd 8
	format  db "It's week %d of school", 0  ; definining the format
segment code use32 class=code
	; ...
	 push dword [n]  ; pushing the value of n on the stack
	 push dword format  
	 call [printf]       ; calling the printf function
	 add esp, 4 * 2     ; cleaning the parameters from the stack
	; ...
```

- ___Priting a message that contains more integers in base 10.___

In a high level programming language:
```c
printf("It's semester %d, week %d of school.", 1, 8);
```
In assembly language:
```asm
segment data use32 class=data
	format  db "It's semester %d, week %d of school.", 0  ; definining the format
segment code use32 class=code
	; ...
	 push dword 8  ; pushing parameters on the stack
	 push dword 1 
	 push dword format  
	 call [printf]       ; calling the printf function
	 add esp, 4 * 3     ; cleaning the parameters from the stack
	; ...
```

- ___Reading an input from the keyboard___

  - For reading input data from keyboard we use the scanf function.
  - Syntax of the scanf function in a high level programming language is:
```c
int scanf(const char * format, variable_address_1, ...);
```

The syntax of the scanf function is similar to the syntax of the printf function. The main difference is that its argument do not have to be constants or values of variables, but only addresses of variables, where the values read will be stored.

___Example:___
Reading an integer and storing it in the variable n
In a high level programming language:
```c
scanf("%d", &n);
		
//&n represent the address of the variable n where the function scanf stores the value read from keyboard
```
In assembly language:
```asm
segment data use32 class=data
	n dd  0       ; defining the variable n
	format  db "%d", 0  ; definining the format
segment code use32 class=code
	; ...
	push dword n       ; pushing the parameters on the stack from right to left
	push dword format
	call [scanf]       ; calling the scanf function for reading
	add esp, 4 * 2     ; cleaning the parameters from the stack
	; ...
```

## Text file operations
A file is a sequence of bytes. In order to read from a file, we need to follow several steps:
1. Open file, which falls into one of the following two cases:
    - Open existing file
    - Create new file and open it
2. Read from a file / Write into a file
3. Close file

## Text file operations - opening a file
- For opening an existing file or creating a new file, we use the fopen function.
- Syntax of the function
    - fopen in a high level programming language:
```c
FILE * fopen(const char* file_name, const char * access_mode)
```
- The fopen function respects the cdecl convention and it can be found in the msvcrt.dll library.
- ___Arguments of the fopen function:___
- The first argument of the function is the address of a character string containing the name of the function. The second argument is the address of a character string containing the access mode for opening the file.

<table border="1" cellspacing="0" cellpadding="5">
<tbody>
<tr>
<td>Access mode</td>
<td>Meaning</td>
<td>Description</td>
</tr>
<tr>
<td>r</td>
<td>read</td>
<td>-	Open file for reading. The file must exist.
</td>
</tr>
<tr>
<td>w</td>
<td>write</td>
<td>-    If the file does not exist, it creates a new file with the given name and opens it for writing. <br>
- If a file with the given name exists, it opens it for writing. It overwrites the content of the file and starts writing from the beginning of the file.
</tr>
<tr>
<td>a</td>
<td>append</td>
<td>-    If the file does not exist, it creates a new file and opens it for writing. </br>
- If a file with the given name exists, it opens it for writing. It does not overwrite the content, it continues writing at the end of the file.
</tr>
<tr>
<td>r+</td>
<td>read and write from/into existing file</td>
<td>-    Open file for reading and writing. The file must exist.
</td>
</tr>
<tr>
<td>w+</td>
<td>read and write</td>
<td>-    If the file does not exist, it creates a new file and opens it for reading and writing. </br>
- If a file with the given name exists, it opens it for reading and writing. It overwrites the content of the file and starts writing at the beginning of the file.
</tr>
<tr>
<td>a+</td>
<td>read and append</td>
<td>-    If the file does not exist, it creates a new file and opens it for reading and writing. </br>
- If a file with the given name exists, it opens it for reading and writing. It does not overwrite the content, it continues writing at the end of the file.
</tr>
</tbody>
</table>

___Observations___:
- The name of the function must include the extension (ex: name.txt, example.asm).
- File are created or opened in the current directory (in the same directory where the asm source is located). Important: in order to open a file using its name, the file must be placed in the same folder as the asm source file, otherwise the file will not be found.
- Writing operations will fail if the file was opened only for reading (ex. access mode "r"). Reading operations will fail for files opened only for writing or appending (ex. access mode "w", "a").
- Both arguments of the fopen function represent character strings that have to be terminated with a 0 (similar to the format for the printf function).
- The value returned by the fopen function:
- If the file is successfully opened, EAX will contain the file descriptor (an identifier) which can be used for working with the file (reading and writing). If an error occurs, fopen will set EAX to the value 0.

___Other observations:___
- It is important to check the value returned by the function in EAX before continuing to work with the file, in order to know whether the file was correctly opened. If a program opens more files using the fopen function, each value returned by the function must be stored separately, since each file has a unique identifier. When we are done working with a file, it is important to close the file ( it can be done at the end of the program - before exit). For closing the file we use the fclose function.

## Text file operations - writing into a file
- For writing text into a file we use the fprintf function.
- ___Syntax of the function___
- fprintf in a high level programming language:
```c
int fprintf(FILE * stream, const char * format, <variable_1>, <constant_2>, <...>)
```
- The fprintf function respects the cdecl convention and it can be found in the msvcrt.dll library. Syntax of the fprintf function is similar to the syntax of the printf function (used for printing on the screen). The difference is that, in addition to the parameters of the printf function, the fprintf function has the file descriptor as the first argument.
- ___Arguments of the fprintf function___
- First argument of the function represents the file descriptor (identifier) returned by the fopen function call. The next argument of the function is a character string containing the format for printing, followed by the same number of arguments (constant values or variable names) as specified in the format. Similar to the printf function, the character string representing the format can contain certain formatting markups, starting with the character ’%’, which will be replaced by the values given in the following arguments.

<table border="1" cellspacing="0" cellpadding="5">
<tbody>
<tr>
<th >Code</th>
<th>Type</th>
<th>Example</th>
<th>Value representation dimension</th>
</tr>
<tr>
<td >c</td>
<td>Character</td>
<td>a</td>
<td>byte</td>
</tr>
<tr>
<td >d or i</td>
<td>Signed decimal integer</td>
<td>392</td>
<td>dword</td>
</tr>
<tr>
<td >u</td>
<td>Unsigned decimal integer</td>
<td>7235</td>
<td>dword</td>
</tr>
<tr>
<td >x</td>
<td>Unsigned hexadecimal integer</td>
<td>7fa</td>
<td>dword</td>
</tr>
<tr>
<td >s</td>
<td>String (terminated with a 0)</td>
<td>example</td>
<td>string of bytes terminated with 0</td>
</tr>
</tbody>
</table>

## Text file operations - reding from a file

- For reading from a file we use the fread function.
- ___Syntax of the function___
- fread in a high level programming language:
```c
int fread(void * str, int size, int count, FILE * stream)
```
- The fread function respects the cdecl convention and it can be found in the msvcrt.dll library.
- ___Arguments of the fread function___
- The first argument of the fread functions represents the address of a string where the data read from file is stored. The second argument represent the size of one element that will be read from the file. The third argument represents the maximum number of elements to be read. The last argument of the function represents the file descriptor (identifier) returned by the fopen function call. When reading from a text file, the first argument of the fread function is a byte string and the second argument is 1 (= dimension of one byte). The third argument is the size of the byte string (number of elements).
- The value returned by the fread function::
- The fread function stores in EAX the number of elements read. If this number is below the value of the count argument, it means that either there was an error, or that the function got to the end of the file.

___Observations___:
- If text files have large sizes, one cannot read the whole content of the file with one function call. In that case multiple fread function calls are necessary, until the whole content of the file is read. In the „Examples” section we will present such an example. In order to check whether we got to the end of the file, we can check if the value returned by fread is 0.

## Text file operations - closing an opened file
- When we finish working with an opened file, this must be closed. This step must not be forgotten in a program that opened files. For closing a file we use the fclose function.
- ___Syntax of the function___
- fclose in a high level programming language:
```c
int fclose(FILE * descriptor)
```
- The fclose function respects the cdecl convention and it can be found in the msvcrt.dll library.
- ___Argument of the fclose function___
- The argument of the fclose function is the file descriptor (identifier) returned by the fopen function call.

## Examples

## Example: Attention when using scanf function
- Pay attention to the representation of the variables according to the format used. See table from the section Standard msvcrt functions from the theory. <span style="color:red">See the following WRONG example</span> (this is a frequent mistake) and try to understand what happens:
```asm
; The following program should read a number and print the message together with the number on the screen.
bits 32
global start        

; declaring extern functions used in the program
extern exit, printf, scanf            
import exit msvcrt.dll     
import printf msvcrt.dll     ; indicating to the assembler that the printf fct can be found in the msvcrt.dll library
import scanf msvcrt.dll      ; similar for scanf
                          
segment  data use32 class=data
	n db 0       ; this is the variable where we store the number read from keyboard
	message  db "Numarul citit este n= %d", 0  
	format  db "%d", 0  ; %d <=> a decimal number (base 10)
    
segment  code use32 class=code
    start:
                                               
        ; calling scanf(format, n) => we read the number and store it in the variable n
        ; push parameters on the stack from right to left
        push  dword n       ; ! address of n, not the value
        push  dword format
        call  [scanf]       ; call scanf for reading
        add  esp, 4 * 2     ; taking parameters out of the stack; 4 = dimension of a dword; 2 = nr of parameters
        
        ;convert n to dword for pushing its value on the stack 
        mov  eax,0
        mov  al,[n]
        
        ;print the message and the value of n
        push  eax
        push  dword message 
        call  [printf]
        add  esp,4*2 
        
        ; exit(0)
        push  dword 0     ; push the parameter for exit on the stack
        call  [exit]       ; call exit
```
- In the example the beginning of the message is overwritten when we read the number n, hence nothing is printed on the screen.     

## Example: Printing a message
```asm
; The code below will print the message „Ana has 17 apples”
bits 32
global start        

; declare extern functions used by the program
extern exit, printf         ; add printf as extern function            
import exit msvcrt.dll    
import printf msvcrt.dll    ; tell the assembler that function printf can be found in library msvcrt.dll
                          
segment data use32 class=data
; char arrays are of type byte
format db "Ana has %d apples", 0  ; %d will be replaced with a number
				; char strings for C functions must terminate with 0
segment  code use32 class=code
    start:
        mov eax, 17
        
        ; will call printf(format, 17) => will print: „Ana has 17 apples”
        ; place parameters on the stack from right to left
        push dword eax
        push dword format ; ! on the stack is placed the address of the string, not its value
        call [printf]      ; call function printf for printing 
        add esp, 4 * 2     ; free parameters on the stack; 4 = size of dword; 2 = number of parameters

        ; exit(0)
        push dword 0      ; push on stack the parameter for exit
        call [exit]       ; call exit to terminate the programme
```

## Example: Printing a quadword on the screen
```asm
segment data use32 class=data
a dq  300765694775808
format db '%lld',0

segment code use32 class=code
start:

push dword [a+4]
push dword [a]
push dword format
call [printf]
add esp,4*3

push    dword 0      
call    [exit]
```        

## Example: Reading a number from keyboard
```asm
; The code below will print message ”n=”, then will read from keyboard the value of perameter n.
bits 32

global start        

; declare extern functions used by the programme
extern exit, printf, scanf ; add printf and scanf as extern functions            
import exit msvcrt.dll    
import printf msvcrt.dll    ; tell the assembler that function printf is found in msvcrt.dll library
import scanf msvcrt.dll     ; similar for scanf
                          
segment data use32 class=data
	n dd  0       ; in this variable we'll store the value read from the keyboard
    ; char strings are of type byte
	message  db "n=", 0  ; char strings for C functions must terminate with 0(value, not char)
	format  db "%d", 0  ; %d <=> a decimal number (base 10)
    
segment code use32 class=code
    start:
       
        ; will call printf(message) => will print "n="
        ; place parameters on stack
        push dword message ; ! on the stack is placed the address of the string, not its value
        call [printf]      ; call function printf for printing
        add esp, 4*1       ; free parameters on the stack; 4 = size of dword; 1 = number of parameters
                                                   
        ; will call scanf(format, n) => will read a number in variable n
        ; place parameters on stack from right to left
        push dword n       ; ! addressa of n, not value
         push dword format
        call [scanf]       ; call function scanf for reading
        add esp, 4 * 2     ; free parameters on the stack
                           ; 4 = size of a dword; 2 = no of perameters
        
        ; exit(0)
        push dword 0      ; place on stack parameter for exit
        call [exit]       ; call exit to terminate the program
```

## Example: Saving register values before function calls
```asm
; The code below will calculate the result of some arithmetic operations in the EAX register, save the value of the registers, then display the result value and restore the value of the registers.
bits 32

global start        

; declare extern functions
extern exit, printf  
import exit msvcrt.dll    
import printf msvcrt.dll    ; tell assembler function is found in library msvcrt.dll
                          
segment data use32 class=data
	; string of bytes
	format  db "%d", 0  ; %d <=> decimal number
	
segment code use32 class=code
	start:
		; will calculate 20 + 123 + 7 in EAX
		mov eax, 20
		add eax, 123
		add eax, 7         ; eax = 150 (base 10) or 0x96 (base 16)
		
		; save the value of the registers because printf function call will modify their values 
		; use instruction PUSHAD which saves on stack the values of several registers: EAX, ECX, EDX and EBX 
		; in this example the value of EAX must be saved, but the instruction can be generally applied
		PUSHAD
	   
		; will call printf(format, eax) => will print value from eax
		; place parameters on stack from right to left
		push dword eax
		push dword format ; ! address of string on stack, not value
		call [printf]      ; call function for printing
		add esp, 4*2       ; free parameters on the stack; 4 = size of dword; 2 = number of parameters
		
		; after function call printf EAX register has a value set by this function (not the value 150 we have computer at the beginning of the program)
		; restore value of registers saved on stack using POPAD
		; this instruction takes values from the stack and restores them in several registers: EAX, ECX, EDX and EBX
		; it is important that before calling instruction POPAD we make sure there are enough values on stack  
		; to be placed in registers (for example make sure that PUSHAD was called before)
		POPAD
		
		; now value from EAX is restored to its previous value just before calling PUSHAD (in this case, value 150)
		
		; exit(0)
		push dword 0      ; we place on stack parameter for exit
		call [exit]       ; call exit to end the program
```

### Text files operations
## Example: Create a new file
```asm
; The following code will create an empty file called "ana.txt" in the current folder.

; The program will use:
; - the function fopen() to open/create the file
; - the function fclose() to close the created file.

; Because the program uses the file access mode "w", if a file with the same name already
; exists, its contents will be discarded and the file will be treated as a new empty file.
; For details about the file access modes see the section "Theory".

bits 32

global start

; declare external functions needed by our program
extern exit, fopen, fclose
import exit msvcrt.dll
import fopen msvcrt.dll
import fclose msvcrt.dll

; our data is declared here (the variables needed by our program)
segment data use32 class=data
    file_name db "ana.txt", 0   ; filename to be created
    access_mode db "w", 0       ; file access mode:
                                ; w - creates an empty file for writing
    file_descriptor dd -1       ; variable to hold the file descriptor

; our code starts here
segment code use32 class=code
    start:
        ; call fopen() to create the file
        ; fopen() will return a file descriptor in the EAX or 0 in case of error
        ; eax = fopen(file_name, access_mode)
        push dword access_mode     
        push dword file_name
        call [fopen]
        add esp, 4*2                ; clean-up the stack

        mov [file_descriptor], eax  ; store the file descriptor returned by fopen
        
        ; check if fopen() has successfully created the file (EAX != 0)
        cmp eax, 0
        je final
        
        ; call fclose() to close the file
        ; fclose(file_descriptor)
        push dword [file_descriptor]
        call [fclose]
        add esp, 4
        
      final:
        
        ; exit(0)
        push dword 0      
        call [exit]
```

## Example: Write a text in a new file
```asm
; The following code will create an empty file called "ana.txt" in the current folder,
; and it will write a text to this file.

; The program will use:
; - the function fopen() to open/create the file
; - the function fprintf() to write a text to file
; - the function fclose() to close the created file.

; Because the program uses the file access mode "w", if a file with the same name already
; exists, its contents will be discarded and the file will be treated as a new empty file.
; For details about the file access modes see the section "Theory".

bits 32

global start

; declare external functions needed by our program
extern exit, fopen, fprintf, fclose
import exit msvcrt.dll
import fopen msvcrt.dll
import fprintf msvcrt.dll
import fclose msvcrt.dll

; our data is declared here (the variables needed by our program)
segment data use32 class=data
    file_name db "ana.txt", 0   ; filename to be created
    access_mode db "w", 0       ; file access mode:
                                ; w - creates an empty file for writing
    file_descriptor dd -1       ; variable to hold the file descriptor
    text db "Ana are mere.", 0  ; text to be write to file

; our code starts here
segment code use32 class=code
    start:
        ; call fopen() to create the file
        ; fopen() will return a file descriptor in the EAX or 0 in case of error
        ; eax = fopen(file_name, access_mode)
        push dword access_mode     
        push dword file_name
        call [fopen]
        add esp, 4*2                ; clean-up the stack

        mov [file_descriptor], eax  ; store the file descriptor returned by fopen
        
        ; check if fopen() has successfully created the file (EAX != 0)
        cmp eax, 0
        je final

        ; write the text to file using fprintf()
        ; fprintf(file_descriptor, text)
        push dword text
        push dword [file_descriptor]
        call [fprintf]
        add esp, 4*2

        ; call fclose() to close the file
        ; fclose(file_descriptor)
        push dword [file_descriptor]
        call [fclose]
        add esp, 4

      final:

        ; exit(0)
        push dword 0      
        call [exit]
``` 

## Example: Append a text in a new file
```asm
; The following code will open/create a file called "ana.txt" in the current folder,
; and it will write a text at the end of this file.

; The program will use:
; - the function fopen() to open/create the file
; - the function fprintf() to write a text to file
; - the function fclose() to close the created file.

; Because the fopen() call uses the file access mode "a", the writing operations will
; append text at the end of the file. The file will be created if it does not exist.
; For details about the file access modes see the section "Theory".

bits 32

global start

; declare external functions needed by our program
extern exit, fopen, fprintf, fclose
import exit msvcrt.dll
import fopen msvcrt.dll
import fprintf msvcrt.dll
import fclose msvcrt.dll

; our data is declared here (the variables needed by our program)
segment data use32 class=data
    file_name db "ana.txt", 0       ; filename to be read
    access_mode db "a", 0           ; file access mode:
                                    ; a - appends to a file. Append data at
                                    ; the end of the file.
    file_descriptor dd -1           ; variable to hold the file descriptor
    text db "Ana are si pere.", 0   ; text to append to file

; our code starts here
segment code use32 class=code
    start:
        ; call fopen() to create the file
        ; fopen() will return a file descriptor in the EAX or 0 in case of error
        ; eax = fopen(file_name, access_mode)
        push dword access_mode     
        push dword file_name
        call [fopen]
        add esp, 4*2                ; clean-up the stack

        mov [file_descriptor], eax  ; store the file descriptor returned by fopen

        ; check if fopen() has successfully created the file (EAX != 0)
        cmp eax, 0
        je final

        ; append the text to file using fprintf()
        ; fprintf(file_descriptor, text)
        push dword text
        push dword [file_descriptor]
        call [fprintf]
        add esp, 4*2

        ; call fclose() to close the file
        ; fclose(file_descriptor)
        push dword [file_descriptor]
        call [fclose]
        add esp, 4

      final:

        ; exit(0)
        push dword 0
        call [exit]
```

## Example: Read a short text (maximum 100 characters) from a file
```asm
; The following code will open a file called "ana.txt" from current folder,
; and it will read maximum 100 characters from this file.

; The program will use:
; - the function fopen() to open/create the file
; - the function fread() to read from file
; - the function fclose() to close the created file.

; Because the fopen() call uses the file access mode "r", the file will be open for
; reading. The file must exist, otherwise the fopen() call will fail.
; For details about the file access modes see the section "Theory".

bits 32

global start

; declare external functions needed by our program
extern exit, fopen, fread, fclose
import exit msvcrt.dll
import fopen msvcrt.dll
import fread msvcrt.dll
import fclose msvcrt.dll

; our data is declared here (the variables needed by our program)
segment data use32 class=data
    file_name db "ana.txt", 0   ; filename to be read
    access_mode db "r", 0       ; file access mode:
                                ; r - opens a file for reading. The file must exist. 
    file_descriptor dd -1       ; variable to hold the file descriptor
    len equ 100                 ; maximum number of characters to read
    text times len db 0         ; string to hold the text which is read from file

; our code starts here
segment code use32 class=code
    start:
        ; call fopen() to create the file
        ; fopen() will return a file descriptor in the EAX or 0 in case of error
        ; eax = fopen(file_name, access_mode)
        push dword access_mode     
        push dword file_name
        call [fopen]
        add esp, 4*2                ; clean-up the stack

        mov [file_descriptor], eax  ; store the file descriptor returned by fopen

        ; check if fopen() has successfully created the file (EAX != 0)
        cmp eax, 0
        je final

        ; read the text from file using fread()
        ; after the fread() call, EAX will contain the number of chars we've read 
        ; eax = fread(text, 1, len, file_descriptor)
        push dword [file_descriptor]
        push dword len
        push dword 1
        push dword text        
        call [fread]
        add esp, 4*4

        ; call fclose() to close the file
        ; fclose(file_descriptor)
        push dword [file_descriptor]
        call [fclose]
        add esp, 4

      final:

        ; exit(0)
        push dword 0
        call [exit]
```

## Example: Read a short text (maximum 100 caractere) from a file and display it
```asm
; The following code will open a file called "ana.txt" from current folder,
; it will read maximum 100 characters from this file,
; and it will display to the console the number of chars and the text we've read.

; The program will use:
; - the function fopen() to open/create the file
; - the function fread() to read from file
; - the function printf() to display a text
; - the function fclose() to close the created file.

; Because the fopen() call uses the file access mode "r", the file will be open for
; reading. The file must exist, otherwise the fopen() call will fail.
; For details about the file access modes see the section "Theory".

; Any string used by printf() must be null-terminated ('\0').

bits 32

global start

; declare external functions needed by our program
extern exit, fopen, fread, fclose, printf
import exit msvcrt.dll
import fopen msvcrt.dll
import fread msvcrt.dll
import fclose msvcrt.dll
import printf msvcrt.dll

; our data is declared here (the variables needed by our program)
segment data use32 class=data
    file_name db "ana.txt", 0   ; filename to be read
    access_mode db "r", 0       ; file access mode:
                                ; r - opens a file for reading. The file must exist.
    file_descriptor dd -1       ; variable to hold the file descriptor
    len equ 100                 ; maximum number of characters to read
    text times (len+1) db 0     ; string to hold the text which is read from file
    format db "We've read %d chars from file. The text is: %s", 0

; our code starts here
segment code use32 class=code
    start:
        ; call fopen() to create the file
        ; fopen() will return a file descriptor in the EAX or 0 in case of error
        ; eax = fopen(file_name, access_mode)
        push dword access_mode     
        push dword file_name
        call [fopen]
        add esp, 4*2                ; clean-up the stack

        mov [file_descriptor], eax  ; store the file descriptor returned by fopen

        ; check if fopen() has successfully created the file (EAX != 0)
        cmp eax, 0
        je final

        ; read the text from file using fread()
        ; after the fread() call, EAX will contain the number of chars we've read 
        ; eax = fread(text, 1, len, file_descriptor)
        push dword [file_descriptor]
        push dword len
        push dword 1
        push dword text        
        call [fread]
        add esp, 4*4

        ; display the number of chars we've read and the text
        ; printf(format, eax, text)
        push dword text
        push dword EAX
        push dword format
        call [printf]
        add esp, 4*3

        ; call fclose() to close the file
        ; fclose(file_descriptor)
        push dword [file_descriptor]
        call [fclose]
        add esp, 4

      final:

        ; exit(0)
        push dword 0
        call [exit]
```

## Example: Read the whole text from a file (in stages)
```asm
; Codul de mai jos va deschide un fisier numit "input.txt" din directorul curent si va citi intregul text din acel fisier, in etape, cate 100 de caractere intr-o etapa.
; Deoarece un fisier text poate fi foarte lung, nu este intotdeauna posibil sa citim fisierul intr-o singura etapa pentru ca nu putem defini un sir de caractere suficient de lung pentru intregul text din fisier. De aceea, prelucrarea fisierelor text in etape este necesara.
; Programul va folosi functia fopen pentru deschiderea fisierului, functia fread pentru citirea din fisier si functia fclose pentru inchiderea fisierului creat.
; Deoarece in apelul functiei fopen programul foloseste modul de acces "r", daca un fisier numele dat nu exista in directorul curent,  apelul functiei fopen nu va reusi (eroare). Detalii despre modurile de acces sunt prezentate in sectiunea "Suport teoretic".

bits 32 

global start        

; declare external functions needed by our program
extern exit, fopen, fclose, fread
import exit msvcrt.dll 
import fopen msvcrt.dll 
import fread msvcrt.dll 
import fclose msvcrt.dll 

; our data is declared here 
segment data use32 class=data
    nume_fisier db "input.txt", 0   ; numele fisierului care va fi deschis
    mod_acces db "r", 0             ; modul de deschidere a fisierului; r - pentru scriere. fisierul trebuie sa existe 
    descriptor_fis dd -1            ; variabila in care vom salva descriptorul fisierului - necesar pentru a putea face referire la fisier
    nr_car_citite dd 0              ; variabila in care vom salva numarul de caractere citit din fisier in etapa curenta
    len equ 100                     ; numarul maxim de elemente citite din fisier intr-o etapa
    buffer resb len                 ; sirul in care se va citi textul din fisier

; our code starts here
segment code use32 class=code
    start:
        ; apelam fopen pentru a deschide fisierul
        ; functia va returna in EAX descriptorul fisierului sau 0 in caz de eroare
        ; eax = fopen(nume_fisier, mod_acces)
        push dword mod_acces
        push dword nume_fisier
        call [fopen]
        add esp, 4*2
        
        ; verificam daca functia fopen a creat cu succes fisierul (daca EAX != 0)
        cmp eax, 0                  
        je final
        
        mov [descriptor_fis], eax   ; salvam valoarea returnata de fopen in variabila descriptor_fis
        
        ; echivalentul in pseudocod al urmatoarei secvente de cod este:
        ;repeta
        ;   nr_car_citite = fread(buffer, 1, len, descriptor_fis)
        ;   daca nr_car_citite > 0
        ;       ; instructiuni pentru procesarea caracterelor citite in aceasta etapa
        ;pana cand nr_car_citite == 0
        
        bucla:
            ; citim o parte (100 caractere) din textul in fisierul deschis folosind functia fread
            ; eax = fread(buffer, 1, len, descriptor_fis)
            push dword [descriptor_fis]
            push dword len
            push dword 1
            push dword buffer
            call [fread]
            add esp, 4*4
            
            ; eax = numar de caractere / bytes citite
            cmp eax, 0          ; daca numarul de caractere citite este 0, am terminat de parcurs fisierul
            je cleanup

            mov [nr_car_citite], eax        ; salvam numarul de caractere citie
            
            ; instructiunile pentru procesarea caracterelor citite in aceasta etapa incep aici
            ; ...
            
            ; reluam bucla pentru a citi alt bloc de caractere
            jmp bucla
        
      cleanup:
        ; apelam functia fclose pentru a inchide fisierul
        ; fclose(descriptor_fis)
        push dword [descriptor_fis]
        call [fclose]
        add esp, 4
        
      final:  
        ; exit(0)
        push    dword 0      
        call    [exit]       
```

## Exercices - Function calls
<ol><li> Read two numbers a and b (in base 10) from the keyboard and calculate their product. This value will be stored in a variable called "result" (defined in the data segment).

</li><li> Read two numbers a and b (in base 10) from the keyboard and calculate a/b. This value will be stored in a variable called "result" (defined in the data segment). The values are considered in signed representation.

</li><li> Two natural numbers a and b (a, b: dword, defined in the data segment) are given. Calculate their sum and display the result in the following format: "&lt;a&gt; + &lt;b&gt; = &lt;result&gt;".
Example: "1 + 2 = 3"<br>
The values will be displayed in decimal format (base 10) with sign.<br>

</li><li> Two natural numbers a and b (a, b: dword, defined in the data segment) are given. Calculate their product and display the result in the following format: "&lt;a&gt; * &lt;b&gt; = &lt;result&gt;".
Example: "2 * 4 = 8"<br>
The values will be displayed in decimal format (base 10) with sign.<br>

</li><li> Two natural numbers a and b (a, b: dword, defined in the data segment) are given. Calculate a/b and display the quotient and the remainder in the following format: "Quotient = &lt;quotient&gt;, remainder = &lt;remainder&gt;".
Example: for a = 23, b = 10 it will display: "Quotient = 2, remainder = 3".<br>
The values will be displayed in decimal format (base 10) with sign.<br>

</li><li> Two natural numbers a and b (a: dword, b: dword, defined in the data segment) are given. Calculate a/b and display the quotient in the following format: "&lt;a&gt;/&lt;b&gt; = &lt;quotient&gt;".
Example: for a = 200, b = 5 it will display: "200/5 = 40".<br>
The values will be displayed in decimal format (base 10) with sign.<br>

</li><li> Two natural numbers a and b (a: dword, b: dword, defined in the data segment) are given. Calculate a/b and display the remainder in the following format: "&lt;a&gt; mod &lt;b&gt; = &lt;remainder&gt;".
Example: for a = 23, b = 5 it will display: "23 mod 5 = 3".<br>
The values will be displayed in decimal format (base 10) with sign.<br>

</li><li> A natural number a (a: dword, defined in the data segment) is given. Read a natural number b from the keyboard and calculate: a + a/b. Display the result of this operation. The values will be displayed in decimal format (base 10) with sign.

</li><li> Read two numbers a and b (base 10) from the keyboard and calculate: (a+b)/(a-b). The quotient will be stored in a variable called "result" (defined in the data segment). The values are considered in signed representation.

</li><li> Read a number in base 10 from the keyboard and display the value of that number in base 16
Example: Read: 28; display: 1C

</li><li> Read a number in base 16 from the keyboard and display the value of that number in base 10
Example: Read: 1D; display: 29

</li><li> A negative number a (a: dword) is given. Display the value of that number in base 10 and in the base 16 in the following format: "a = &lt;base_10&gt; (base 10), a = &lt;base_16&gt; (base 16)"

</li><li> Read two numbers a and b (base 10) from the keyboard and calculate: (a+b)\*(a-b). The result of multiplication will be stored in a variable called "result" (defined in the data segment).

</li><li> Read two numbers a and b (in base 16) from the keyboard and calculate a+b. Display the result in base 10

</li><li> Read two numbers a and b (in base 10) from the keyboard and calculate a+b. Display the result in base 16

</li><li> Read two numbers a and b (in base 10) from the keyboard. Calculate and print their arithmetic average in base 16

</li><li> Read a number in base 10 from the keyboard and display the value of that number in base 16

</li><li> Read a decimal number and a hexadecimal number from the keyboard. 
Display the number of 1's of the sum of the two numbers in decimal format.
Example:<br>
	a = 32  = 0010 0000b<br>
	b = 1Ah = 0001 1010b<br>
	32 + 1Ah  = 0011 1010b<br>
The value printed on the screen will be 4

</li><li> Read one byte and one word from the keyboard. Print on the screen "YES" if the bits of the byte read are found consecutively among the bits of the word and "NO" otherwise.
Example:
	a = 10  = 0000 1010b<br>
	b = 256 = 0000 0001 0000 0000b<br>
The value printed on the screen will be NO.<br>

	a = 0Ah  = 0000 1010b<br>
	b = 6151h = 0110 0001 0101 0001b<br>
The value printed on the screen will be YES (you can find the bits on positions 5-12).<br>

</li><li> Read two doublewords a and b in base 16 from the keyboard. Display the sum of the low parts of the two numbers and the difference between the high parts of the two numbers in base 16
Example:<br>
a = 00101A35h<br>
b = 00023219h<br>
sum = 4C4Eh<br>
difference = Eh<br>

</li><li> Read two words a and b in base 10 from the keyboard. Determine the doubleword c such that the low word of c is given by the sum of the a and b and the high word of c is given by the difference between a and b. Display the result in base 16
Example:<br>
a = 574, b = 136<br>
c = 01B602C6h<br>

</li><li> Two numbers a and b are given. Compute the expression value: (a+b)\*k, where k is a constant value defined in data segment. Display the expression value (in base 10).

</li><li> Read a hexadecimal number with 2 digits from the keyboard. Display the number in base 10, in both interpretations: as an unsigned number and as an signed number (on 8 bits).

</li><li> Two numbers a and b are given. Compute the expression value: (a/b)\*k, where k is a constant value defined in data segment. Display the expression value (in base 2).

</li><li> Read two numbers a and b (in base 10) from the keyboard and determine the order relation between them (either a &lt; b, or a = b, or a &gt; b). Display the result in the following format: "&lt;a&gt; &lt; &lt;b&gt;, &lt;a&gt; = &lt;b&gt; or &lt;a&gt; &gt; &lt;b&gt;".

</li><li> Two numbers a and b are given. Compute the expression value: (a-b)\*k, where k is a constant value defined in data segment. Display the expression value (in base 16).

</li><li> A character string is given (defined in the data segment). Read one character from the keyboard, then count the number of occurences of that character in the given string and display the character along with its number of occurences.

</li><li> Read numbers (in base 10) in a loop until the digit '0' is read from the keyboard. Determine and display the biggest number from those that have been read.

</li><li> Read three numbers a, m and n (a: word, 0 &lt;= m, n &lt;= 15, m &gt; n) from the keyboard. Isolate the bits m-n of a and display the integer represented by those bits in base 16

</li><li> Read numbers (in base 10) in a loop until the digit '0' is read from the keyboard. Determine and display the smallest number from those that have been read.

</li>
</ol>

## Exercices - Text file operations
___Note___: the text files can have large sizes!
<ol><li> A text file is given. Read the content of the file, count the number of vowels and display the result on the screen. The name of text file is defined in the data segment.

</li><li> A text file is given. Read the content of the file, count the number of consonants and display the result on the screen. The name of text file is defined in the data segment.

</li><li> A text file is given. Read the content of the file, count the number of even digits and display the result on the screen. The name of text file is defined in the data segment.

</li><li> A text file is given. Read the content of the file, count the number of odd digits and display the result on the screen. The name of text file is defined in the data segment.

</li><li> A text file is given. Read the content of the file, count the number of special characters and display the result on the screen. The name of text file is defined in the data segment.

</li><li> A text file is given. Read the content of the file, determine the digit with the highest frequency and display the digit along with its frequency on the screen. The name of text file is defined in the data segment.

</li><li> A text file is given. Read the content of the file, determine the lowercase letter with the highest frequency and display the letter along with its frequency on the screen. The name of text file is defined in the data segment.

</li><li> A text file is given. Read the content of the file, determine the uppercase letter with the highest frequency and display the letter along with its frequency on the screen. The name of text file is defined in the data segment.

</li><li> A text file is given. Read the content of the file, determine the special character with the highest frequency and display the character along with its frequency on the screen. The name of text file is defined in the data segment.

</li><li> Read a file name and a text from the keyboard. Create a file with that name in the current folder and write the text that has been read to file.
Observations: The file name has maximum 30 characters. The text has maximum 120 characters.

</li><li> A file name is given (defined in the data segment). Create a file with the given name, then read words from the keyboard and write those words in the file, until character '$' is read from the keyboard.

</li><li> A file name is given (defined in the data segment). Create a file with the given name, then read numbers from the keyboard and write those numbers in the file, until the value '0' is read from the keyboard.

</li><li> A file name and a text (defined in the data segment) are given. The text contains lowercase letters, uppercase letters, digits and special characters. Transform all the lowercase letters from the given text in uppercase. Create a file with the given name and write the generated text to file.

</li><li> A file name and a text (defined in the data segment) are given. The text contains lowercase letters, uppercase letters, digits and special characters. Transform all the uppercase letters from the given text in lowercase. Create a file with the given name and write the generated text to file.

</li><li> A file name and a text (defined in the data segment) are given. The text contains lowercase letters, uppercase letters, digits and special characters. Replace all the special characters from the given text with the character 'X'. Create a file with the given name and write the generated text to file.

</li><li> A text file is given. Read the content of the file, count the number of letters 'y' and 'z' and display the values on the screen. The file name is defined in the data segment.

</li><li> A file name is given (defined in the data segment). Create a file with the given name, then read numbers from the keyboard and write only the numbers divisible by 7 to file, until the value '0' is read from the keyboard.

</li><li> A text file is given. The text contains letters, spaces and points. Read the content of the file, determine the number of words and display the result on the screen. (A word is a sequence of characters separated by space or point)

</li><li> A file name and a text (which can contain any type of character) are given in data segment. Calculate the sum of digits in the text. Create a file with the given name and write the result to file.

</li><li> A file name and a text (defined in the data segment) are given. The text contains lowercase letters and spaces. Replace all the letters on even positions with their position. Create a file with the given name and write the generated text to file.

</li><li> A file name and a text (defined in the data segment) are given. The text contains lowercase letters, digits and spaces. Replace all the digits on odd positions with the character ‘X’.  Create the file with the given name and write the generated text to file.

</li><li> A file name and a decimal number (on 16 bits) are given (the decimal number is in the unsigned interpretation). Create a file with the given name and write each digit composing the number on a different line to file.

</li><li> A file name and a hexadecimal number (on 16 bits) are given. Create a file with the given name and write each nibble composing the hexadecimal number on a different line to file.

</li><li> A file name and a text (defined in data segment) are given. The text contains lowercase letters, uppercase letters, digits and special characters. Replace all digits from the text with character 'C'. Create a file with the given name and write the generated text to file.

</li><li> A file name and a text (defined in data segment) are given. The text contains lowercase letters, uppercase letters, digits and special characters. Replace all spaces from the text with character 'S'. Create a file with the given name and write the generated text to file.

</li><li> A file name (defined in data segment) is given. Create a file with the given name, then read words from the keyboard until character '$' is read. Write only the words that contain at least one uppercase letter to file.

</li><li> A text file is given. The text file contains numbers (in base 10) separated by spaces. Read the content of the file, determine the minimum number (from the numbers that have been read) and write the result at the end of file.

</li><li> A file name (defined in data segment) is given. Create a file with the given name, then read words from the keyboard until character '$' is read. Write only the words that contain at least one lowercase letter to file.

</li><li> A text file is given. The text file contains numbers (in base 10) separated by spaces. Read the content of the file, determine the maximum number (from the numbers that have been read) and write the result at the end of file.

</li><li> A file name (defined in data segment) is given. Create a file with the given name, then read words from the keyboard until character '$' is read from the keyboard. Write only the words that contain at least one digit to file.
</li>
</ol>