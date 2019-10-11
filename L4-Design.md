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
