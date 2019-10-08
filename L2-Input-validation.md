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
