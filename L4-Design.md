## What's design?
ISO/IEC 12207:2008: Software detailed design process is to provide a design for the software that implements andcan be varified aginst the requirements...  
In short: Determine components that you will use, and how you will use them to solve some problems.  

## Developing secure software is not new concept  
Problem is that most devlopers don't know how to do it.  
design principles:  
* Least privilege  
* Economy of mechanism/simplicity: Protection system design should be simple and small as posiible.  
* Open design: even not open, attackers can try to reverse or steal the source code by other ways.  
* Complete mediation: check everywhere.  
* Fail-safe defaults  
* Separation of privilege  
* Least common mechanism: minimum the amount and use of shared mechanism.  
* Easy to use

## Minimizing privileges  
Primary privilege determiner is EUID/EGID!!  
**Create special groups** than user.  
Limit database right for application.  
Minimize modules granted privileges.  
Limit accessible resources.  

`chroot target` can change the root directory to the parameter target.  
User would not be able to access the old directory out the current target anymore.  
Which is called **chroot jail**.  
```sh
chroot target /bin/sh
```  
target is a pseudo environment, the command would run `/bin/sh` in the pseudo environment. The `/bin/sh` isn't the one on the old environment. After the command finished, it would return back to the old environment.  
**chroot cannot protect against root privilege**!  

## Race condition
Multiple threads try to read or write into the content of the same address, and the result depends on the timing or orders of threads themselves.  
If conflict happens, it would cause to a wrong result!  
This problem would happen on the multi threads on the same computer, and could also cause by interrupt from outside attacker! 

In past, race condition not become a problem because there is only single processor to run one program at one time.  
Nowadays, with more threads share more resource, it become a serious problem.  
So the key point is: before your program access the resource, make sure there is no other program would access the same resource and cause to the interrupt.  

TOCTOU time-of-check time-of-use:  
Program check the status or specific information, and then utilize the information. However, attackers change the status between these two steps.  
```c
// popular example on wiki
if (access("file", W_OK) != 0){
  exit(1);
}
attacker --> symlink("/etc/passwd", "file"); // file points to the password database
fd = open("file", O_WRONGLY);
write(fd, buffer, sizeof(buffer));
```  
To prevent TOCTOU, here are some solutions:  
* Don't use `access` to check status, attacker would possibly changes the status. At beginning, you could set the permission of the file at least with what you want. Then, use `open()` directly.  
* When creating a new file, use `O_CREAT|O_EXCL`.  
* For the temporary file shared among the process  
`/tmp` and `/var/tmp`  
Write a loop  
Create file name repeatedly  
`O_CREAT|O_EXCL` it with least privilege  
stop repeating when open successfully  

[Lock](https://www.ibm.com/developerworks/cn/linux/l-sprace.html)  
**File as lock --**  
lock also has problems: deadlock, livelock  
system need to clean up the stucked locks  
file locking is the very old way to assert the lock.  
But, make sure attackers cannot access the locking file!  
**POSIX record locks --**  
record locks are supported on nealy all the Unix-like system.  
It can lock on just the part of the file.  
It can differentiate between reading locks and writing locks.  
When the process dies, the locks can also be release automatically.  

## Detect, Contain, Respond
Detect: Logging, log analysis...  
Contain: Limited privileges, sandboxes...  
Respond: Record into evidence...

## Cloud computing
On-demand self-service, broad network access, resource pooling, rapid elasticity...  
Virtualization is common, but **not necessary** to be a cloud!  
Cloud service models:  
* IaaS(Infracstructure as a service): cloud->infrastructure, consumers can deploy application and platform  
e.g. AWS, Windows Azure, Google Compute Engine
* PaaS(Platform as a service): cloud->infrastructure, platform. consumers can deploy application  
e.g. Google app engine, AWS also supports  
* Saas(Software as a service): cloud->infrastructure, platform, application, consumers can use application run on it  
e.g. Google docs

## Threat modeling
Think like an attacker before implementation

## Analysis approach
* Attacker centric  
* Design centric  
* Asset centric
