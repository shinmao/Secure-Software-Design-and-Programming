## Programming language and buffer overflow
Some language are not memory safe, allow buffer overflow.  
e.g. C, C++, Object-C, assembly lanaguage.  
Most language counter buffer overflow.  
e.g. detect/prevent overflow: Ada strings, Pascal.  
e.g. Autosize: Java, Python, perl.  
But this doesn't mean using such we can change to use java or python to prevent buffer overflow.  
Most languages implemented in C/C++.  
Most libraries, Operation Systems include C/C++.  
Some compiler even allows to disable protection!

## Some details of C
C strings terminated with `\0` character, formal name is NUL character.  
If you overwrite the terminated character, the string would seem not ended!  

C arrays allocated **fixed** size of memory.  
Arrays should be long enought to include not only the string but also the NUL character!  
e.g. `char x[10];` would include 9 characters and 1 NUL character.  

## How buffer overflow works?
First, memory map:  
```c
-> higher address  ------------       <---- frame pointer, base pointer
                  |    stack   |  | 
                   ------------   v   <---- stack pointer, top of the stack
                  |            |
                  |            |
                   ------------   ^
                  |    heap    |  |        // dynamically allocated with malloc()
                   ------------
                  |   .bss     |          // uninitialized variables
                   ------------          // global variables
                  |   data     |         // initialized variables
                   ------------
                  |   .text    |         // compiled code, often read-only
-> lower address   ------------
```  
Stack grows up from higher address to lower address, while heap grows up from lower address to higher address.  
```c
void f(int a, int b, int c){
  char buffer1[5];
  char buffer2[10];
}

int main(){
  f(1,2,3);
  return 1;
}
// pushl $3;
// pushl $2;
// pushl $1;
// call f;
-> higher address ------------
                 |     $3     |
                  ------------
                 |     $2     |
                  ------------
                 |     $1     |
                  ------------
                 | ret adr of |                                ^
                 |    main()  |                                |  o
                  ------------                                 |  v
                 | saved base |                                |  e
                 |   pointer  |                                |  r
                  ------------ <---------- base pointer        |  w
                 |   buffer1  |                                |  r
                  ------------                                 |  i
                 |   buffer2  |                                |  t
-> lower address  ------------ <---------- stack pointer       |  e
```  
If you don't limit the size of input, the input would be able to overwrite from buffer2 to buffer1, saved base pointer, return address, parameters to function.  

* Attacker can insert some shellcode in the buffer, and overwrite the return address with the address of buffer. Then, the function would return to execute the malicious shellcode.  
* ROP gadgets  

You can also try to smash the heap (Off-by-one!!), global variables.  
If you are interested in the attacking technique, you can take a look at my repositories:  
* [WhyNot-StackOverflow](https://github.com/shinmao/WhyNot-StackOverflow)  
* [WhyNot-HEAP-Exploitation](https://github.com/shinmao/WhyNot-HEAP-Exploitation)

## Mitigation
Checking bound, but many C functions don't check bound.  
e.g. `gets()`, `strcpy(dest, src)`, `strcat(dest, src)`, `scanf()`...  

**Bounds-checking** and **resize automatically**  
* if oversized, stop processing input.  
* if oversized, truncate the data. But it is still difficult, the place you truncate might be in middle of multi-byte character, or strip off the critical data...etc.  
* Most language would resize automatically, or move string if necessary.
