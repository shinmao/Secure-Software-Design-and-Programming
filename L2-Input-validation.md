## Dangerous?
```php
escape_shell_cmd(input);
fp = popen(input, "r");
```
Dangerous because `escape_shell_cmd` doesn't strip out new line character, the malicious command would executed in the new line. We also need to avoid `%0a` and `\n` characters.  

## Identify the all potential attacking channels
* command line  
GUI/web-based application cab be built on command line program, it can take arguments  
Don't even trust the reported command line arguments.  
* environmental variables  
```sh
ENV ---> |PTR| ---> |SHELL=/bin/shNIL|
           |
           v
         |PTR| ---> |SHELL=/attack/shNIL|
           |
           v
         |NIL|           // program might check one value for validity, but use a different value!
```
The structure of env is a pointer to the arry of characters.  
We can use `getenv()` and `putenv()` to maintain structure of environmental variables.  
e.g. `IFS`. Unix/Linux shell use it to separate command line arguments
With IFS, attackers can even use forbidden character they want. If space character is forbidden, attacker can set `IFS` to F and send `rmF-RF*`!  
CGI sets certain environment variables which can be influenced by untrusted user. e.g. `$_SERVER['QUERY_STRING']`.  
Therefore, determine the required set of environmental variables and don't let all of them exposure to untrusted user.  
* Path manipulation  
```sh
/home/attacker/exploit:/sbin:/usr/sbin:/bin:/usr/bin
```
Don't trust path from the untrusted source.  
Always start path from the trusted directory.  
Or maybe don't let any user input become part of your path.  
* File descriptors  
Object reference to files.  
stdin, stdout, stderr  
* File names  
* File content  
File content can be modified by untrusted users.  
e.g. including indirectly...  
should verify content before used by trusted program.  
* web-based application input: URL, GET, POST data...  
receive HTTP headers from untrusted users e.g. Many web devlopers rely on HTTP header value of `host` to write links.  
* ...

## Server side validation v.s. Client side validation
```html
<input name="lastname" type="text" id="lastname" maxlength="100" />
```
Do this prevent user from sending data over size of 100 to the server?  
It would stop you at the first site, but it is easy to bypass. This is such kind of client side validation that you can change maxlength to be bigger in your own browser. Therefore, you should use Server-side validation if you don't want the over-size data to be sent to server.  
All the security-relevant checks should be redone on a trusted server and data is protected there.  
[Client side validation and server side validation](http://net-informations.com/faq/asp/validation.htm)  

## Whitelist but not blacklist
If you want to block something, whitelist is always better than blacklist.  
You want never be able to enumerate the payload in blacklist.  
However, blacklist can still be useful for security testing to test whitelist rules!

## Input type
* Number  
Be careful of overflow. e.g. biggest integer 2^64 - 1, then overflow.  
Check the range with not only largest but also the least.  
Fraction allowed?  
float point? NaN? Infinity?  
* String  
Name of some characters:  
`!`: exclamation-mark  
`&`: ampersand  
`^`: hat, circumflex  
`|`: pipe, bar  
`()`: parenthesis  
`<>`: bracket  
`[]`: square bracket  
`{}`: brace  
* character encoding  
ASCII  
Unicode: UTF-32, UTF-16, UTF-8  
unchecked invalid sequence might be interpreted as NIL, newline, slash by decoder.  
Attacker could use this technique to bypass checking.  
Locale would also affect how characters are interpreted.  
Visual spoofing: 2 different strings mistaken as same by user.  
e.g. "-" U+002D v.s. "-" U+2010  

## Pattern language
Globbing: always used to match filenames  
* `*` to match any 0 or more characters.  
* `?` to match any 1 character.  
* `[xxx]` to match characters inside  

RE (regular expression):  
**Always use ^ at the beginning and $ at the end**, which are the anchoring patterns.  
There are many variations of REs such like POSIX basic REs, POSIX entended REs, Perl-style.  
Make sure the RE libraries can handle NUL char in data or not!  
* `.` match any one character.  
* `\n` to escape newline.  
* `\r` to escape carriage return.  
* `\.` matches dot character.  
* `\[` matches left bracket character.  
* `\\` matches backslash.  
* `[x-y]` matches any chatacters in the range from x to y if using POSIX/C locale.  
* `\-` if you want to escape dash in the sample above, otherwise put at beginning or end.  
* `[.]` matches dot character.  
* `[^A-Z]` matches any characters other than A to Z.  
* `{N}`: for exactly N times; `{N,}`: for N or more times; `{N1, N2}` between N1 and N2 times.  

If you want to case-insensitive, you need to specify it.  
Use parenthesis (`()`) to group it.  
Use pipe (`|`) to list alternative expressions. Pipe has lower precedence than `^` and `$`.  
e.g. `^cat|bird$` matches anything beginning with cat **or** ending in bird. `^(cat|bird)$` matches **cat or bird only**.  
The implementation of RE:  
`grep` in command line.  
`regexec()` in C library.  
```c
// regcomp(*ptr, RE string, options): compiles regex into a form that can be later used by regexec
// regexec(compiled regex, string to match against RE, substring match info, options): matches string against the precompiled regex created by regcomp, return 0 if matched.
// regerror(): returns error string, given an error code generated by regcomp or regex when regcomp return nonzero.
// regfree(): free memory allocated by regcomp
#include <regex.h>
regex_t compiled_pattern;
...
error = regcomp(&compiled_pattern, pattern, ...);
if (error) {}
...
m = regexec(compiled_pattern, input, (size_t)0, NULL, 0);
if m == 0  // matched
...
regfree(&compiled_pattern);
```  
`java.util.regex` in java package.  
```java
// pattern object: a compiled representation of regex
// matcher object: engine that interpret the pattern and perform operation against the input string  
// PatternSyntaxException
import java.util.regex.Pattern;
import java.util.regex.Matcher;
Pattern nump = Pattern.compile("^[0-9]+$");
Matcher match = nump.matcher(input);
if(match.find()) {...}
```  
`findstr` works as `grep` in window.  
Two kinds of implementations:  
* DFA: Take the whole string to match with sub-part of RE -> next sub-part of RE. DFA ony needs to scan string for one time, so it is much faster, but with few features.  
* NFA: Take the RE to match with string, if there is any substring matched, recorded and go to next part of substring. NFA is too complicated and slower work with string, but with more features. Most RE engine todays such like Perl, Ruby, Python, Java, .NET are based on NFA.  

## ReDoS
Exploding times of backtrack causes to denial of service.  
It always happens on NFA because NFA is greedy.  
e.g. input string: `hello.google@aaaa`  
```php
^[a-zA-Z0-9._]+@([a-zA-Z0-9]+.)+com$
```  
First, `hello.google@` would be matched by `^[a-zA-Z0-9._]+@`.  
Second, `aaaa` would be matched by `[a-zA-Z0-9]+`.
Third, backtrack because there is nothing left to match. `aaa` would be matched by `[a-zA-Z0-9]+`, and `a` would be matched by `.`.  
Fourth, backtrack because there is nothing left to match `com`. `aa` would be matched by `[a-zA-Z0-9]+`, `a` would be matched by `.`.  
Fifth, backtrack because com still no match. `a` would be matched by `[a-zA-Z0-9]+`, `a` would be matched by `.`, match still fail.  
Sixth, backtrack because com still no match. But `[a-zA-Z0-9]+` cannot match empty string.  
The above is how the backtrack works. Now we just need to imagine the length of the a at the end of string is very big, then the backtrack times should be also very large. This would cause a big loading to computer CPU.  

What is the feature of ReDoS?  
There would be many branches of matched string at one point. Take the example above, after the point of `@`, there are many branches of matched string such as `aaaa`, `aaa`, `aa`, `a`!  
Solution?  
Use NFA-to-DFA implementation  
Use regex fuzzers and static analysis tools to verify  
**What's the most common and efficient way, limit input size before using regex!!**
