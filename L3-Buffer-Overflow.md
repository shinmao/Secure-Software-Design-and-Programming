## Programming language and buffer overflow
Some language are not memory safe, allow buffer overflow.  
e.g. C, C++, Object-C, assembly lanaguage.  
Most language counter buffer overflow.  
e.g. detect/prevent overflow: Ada strings, Pascal.  
e.g. Autosize: Java, Python, perl.  
**But this doesn't mean using such we can change to use java or python to prevent buffer overflow**.  
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
e.g. `gets()`, `strcpy(dest, src)`, `strcat(dest, src)`,  
`sprintf()`, `vsprintf()`,  
`scanf()`, `fscanf()`, `sscanf()`, `vscanf()`...  

**Bounds-checking** and **resize automatically**  
* if oversized, stop processing input.  
* if oversized, truncate the data. But it is still difficult, the place you truncate might be in middle of multi-byte character, or strip off the critical data...etc.  
* Most language would resize automatically, or move string if necessary.

**Safer function is hard to use correctly**  
```c
char *strncpy(char *string1, const char *string2, size_t count);
char *strncat(char *string1, const char *string2, size_t count);
int sprintf(char *buffer, const char *format-string, argument-list);
```  
* There is one problem with both `strncpy()` and `strncat()`: The length in their parameters is not the buffer size. Therefore, mis-calculation would still cause to buffer overflow.  
* If overflow happens or data discarded, neither of `strncpy` or `strncat` would give the reports.  
* `strncpy()`: If count is less than or equal to the length of string2, `\0` would not be appended to the end of copied string. If count is larger than the length of string, `\0` would be padded to the end of copied string until the length of count, but this also cause to a big performance penalty!  
* `strncat()`: first `count` bytes of string2 would be concatenated to string1 and ended with `\0`. If `count` is greater than the length of string2, the `count` would be replaced with the length of string2!  
* `sprintf()`: Format string might also prevent buffer overflow. `%10s` sets the minimum length, `%.10s` sets the maximum length. With `%.*s`, we can even pass a parameter to set up the max length, e,g, `sprintf(dst, "%.*s", maxlen, src);`  
* `snprintf()`: always write NUL character to the end of dst string! (Good)  
When it comes to the return value, there is difference between return value of `sprintf` and `snprintf`. `sprintf` would return the length of successfully overwritten string. `snprintf` would return the length of intended overwritten string, which means the length you planned to write into the buffer. Therefore, we can use the return value to deal with some exception: **if return value is less than 0, then there is an error, if return value is bigger or equal to the size of buffer, then string must be truncated!**  
However, this also causes to the inefficiency of the function. Due to the reason that function want to return the value of whole string, it would keep reading to the end. **This would become a problem if input string is long or not with null character terminated.**  
* Here is one way to make `snprintf` more safer and efficient: use `snprintf` with precision spec!  
```c
len = snprintf(dst, dstsize, "%.*s", (int)srcsize, src);
if(len < 0 | len >= buflen) ...
```  
Here src even don't need to have NUL character because `snprintf()` would stop reading after `srcsize`!  

> Not only resulted string should be ended with \0, but also the parameters should be ended with \0 because program needs to read the content of parameters.

**strlcpy() and strlcat()**  
* `strlcpy(char *dst, const char *src, size_t size);` and `strlcat(char *dst, const char *src, size_t size);` are much better to use, why?  
* The size is the buffer space! Which means it would be easier for you to calculate the space!  
* Always terminated with NUL character if dst has any space.  
* `strlcpy` won't fill the buffer with NUL, so won't be so inefficient such likes `strncpy`!  
* Easy to detect with "truncation" because return value such like `snprintf`.  
* (But still inefficient because it works like snprintf, so input string too long or not terminated with NUL would become the case!)  
* (Easy to detect truncation, but still difficult to get the original word.)  
What's more, these two functions are from OpenBSD developers. They are not available on Unix-like system.  

**C++ std::string class**  
* There is a solution to prevent buffer overflow with `std::string` in C++!  
* `std::string` would automatically resize!  
* C++ also support the class natively.  
* However, you should avoid the use of **char* strings**.  
It would happen because not all systems or libraries support the class.  
Therefore, you would need to do the type-conversion.  
Then, the buffer overflow would happen again!  

**asprintf() and vasprintf()**  
* `int asprintf(char **str-ptr, const char *fmt, ...)` and `int vasprintf(char **str-ptr, const char *fmt, va_list ap)`.  
* after it, use `free(str-ptr)` to deallocate the buffer.  
* return the number of bytes successfully printed, return -1 if error.  
* You don't need to calculate the size by yourself. Function would do all the work for you, from the size of string, allocation of buffer, and written to the buffer. Looks great!  
* Widely used to get work done without buffer overflow.  
* This means that this function is a **hidden** `malloc()`. If you forget to `free()` the pointer, it would easy for attacker to leak the memory!  

**memccpy() v.s. memcpy()**  
* `void * memccpy(void *dst, const void *src, unsigned char ch, size_t n);` and `void * memcpy(void *dst, const void *src, size_t n)`  
* `memccpy` would copy n characters from src to dst. **If it runs into the character ch in src, it would stop copying**. c could be `\0`.  
* Return value would be the first address after `ch` in the src. If there is no `ch` in the first `n` bytes of src, return value would be NUL.  
* You still need to calculate the value of size.  
```c
size_t dsize = sizeof(d);
...
char *p = memccpy(d, s1, "\0", dsize);
dsize -= ( p - d - 1 );
if (dsize <= 0) goto overflow...
char *q = memccpy(p-1, s2, '\0', dsize);
dsize -= (q - (p-1) -1);
if (dsize <= 0) goto overflow...
```  
Easier but still painful...  

**Compiled solutions**  
* recompiled the source code  
* canary-based  
stackguard  
gcc `-fstack-protector`  
* Libsafe  
wrap check around traditional functions  
examine current stack pointer and base pointer to deny the attempts to overwrite data  
But, only protect certain functino calls, only protect certain data, thwarted by some compilter optimizations.  
* -D_FORTIFY_SOURCE  
replace some traditional function calls with bounds-checking version  
Ubuntu and Fedora use both `-D_FORTIFY_SOURCE=2` and `-fstack-protector`  
* Address sanitizer(ASan)  
Compilation-time countermeasures  
gcc tools: `gcc -fsanitize=address`  
After compiling the program and run it, it would show detailed information about some issues such like out-of-bound read, buffer overflow, use-after-free, double-free...  
ASan uses **shadow bytes** to record memory accessibility.  
Every 8 bytes of memory’s addressability represented by one byte of shadow byte  
Every read/write checks shadow bytes to see if access ok.  
use **redzone** to wrap the buffer, if buffer overflow, readzone can detect it.  

> Best approach so far is to ensure code not vulnerable to buffer overflow first, everything else is second best! The easiest approach is to avoid programming language such like c/c++ and unsafe mode which lacks memory safety.

## Related attack
Format string attack:  
`printf()` can use to leak the content of memory.  
`%n` in format string can use to overwrite momory!  
`scanf()` accept so much data and cause to buffer overflow.  
Attacker can determine what kind of data to enter system.  

Double free:  
C/C++ always need us to free the allocated memory manually.  
But if we free the allocation > once, it can also corrupt internal data structure!  
Most other languages include automatic garbic collection, such like Java, python, Perl, etc.

## But don't create your own memory allocation system!  
Some program re-implement memory allocation internally.  
But many defensive system only works for the default memory allocator such like `malloc()`.  
If you create your own system, the defensive system might not recognize or detect the problems.  
e.g. OpenSSL Heartbleed vulnerability undetected because such reason!  

## Hash Collision attack
Hash function: takes original data to generate fixed-length output.  
Good hash function has a uniform distrubtion of hash values.  
Hash table is able to do insertion/lookup/removal in O(1) time, and O(n) in worst case.  
```c
-----
| 0 |
-----
| 1 |
-----    ----    ----    ----
| 2 | -> |  | -> |  | -> |  |
-----    ----    ----    ----
| 3 |
-----
```  
The worst case is that all the elements have the same hash value.  
If attacker can predict resulting hash value that would be used:  
* can cause a large number of data having same hash value.  
* result in DoS because system trying to figure out the worst case.  

Countermeasures:  
* randomized hash functions  
**Universal hashing**: use randomized hash functions.  
* use another data structures  
mapping user data to a not vulnerable predictable situation  
* Limit the number of worst-case situation in HTTP request  
* Limit max CPU time/request  
* add random data to value to be hashed when development  
If you are usng JAVA HashMap or String.hashCode  
