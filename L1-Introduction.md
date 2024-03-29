## Why software always not secure?
Most devlopers cannot think like attackers.  
Developers not learn from others' mistakes.  
Customers can't evaluate software security.  
Security always not considered seriously.  

## We must
We should always consider security through life cycle. E.g. Security test, risk analysis, abuse cases,.. and so on.  
NIST cybersecurity framework:  
1. IDENTITY: assess management  
2. PROTECT: data security, access control, awareness and training  
3. DETECT: monitoring  
4. RESPOND: ensure adequate response  
5. RECOVER: recovery planning and improvement  

for organizations  

Risk management should be part of entire system lifecycle  
Risk management process:  
Communication and consultation  
Establishing context  
Risk identification, analysis, and evaluation  
Risk treatment  
We cannot eliminate all risks, but we can manage them!  

Assurance case includes:  
claims: Top level claims for a system or product e.g. System is adequately secure against threats  
Argument: Systematic argumentation for claim e.g. no common weakness with them  
Evidence: Evidence underlying argument e.g. SQL preparestatement used  
A good assurance case makes it easier to agree "enough has been done!".

## Attacker, craker different from hacker
We always mistake hacker for attacker.  
In fact, hacker is someone who delights in having an intimate understanding of the internal workings of the system, computers and computer networks in particular.  

## User authentication
Attacker v.s. password: capture password, brute force attack, and dictionary attack  
How to defend password:  
**Long enough** is the most important  
check against dictionary of bad password e.g. HIBP  
Don't store plaintext in database of server, please use salted hash function on it.  
occasional password changes  
make it hard to exploit "forget my password"  
Alert user when password is changed!  

Alternatives to password:  
One-time password: Can't reuse!; pros: Counter network eavesdropping; cons: hard to distribute list  

Shared secret:  
Server generates nonce and sends to client  
Client encrypts nonce with secret and sends back  
Server encrpts nonce with secret and compare to the value sent from client  
If same, authentication success!  
pros: Counter network eavesdropping  
cons: secret cannot be compromised  

Public key cryptography  
hardware: challenge-response, time-based challenge-response, smartcard e.g. Yubikey  

## Authorization
ACL: Access Control List  
**DAC**: Discretionary(自主的) Access Control  
Gernerally, whether user can access subject based on the ACL of subject  
Every subject has owner, the owner can decide who can access the subject  
the disadvantage is hard to manage the permission of user  

**MAC**: Mandatory Access Control  
Born up because DAC is too hard to manage  
Each users and subjects both has flags of permission  
Whether the user can access the subject based on the relationship between flags  
For example, subject has flag of top security permission but the user doesn't have  
Then the user would not be able to access the subject  

**RBAC**: Role-Based Access Control  
User is related to roles  
roles are related to permission

## Basics of Unix/Linux/POSIX
Users & Groups:  
Each users belongs to at least one group  
Modern system allows user to belong to several groups  
In android, even applications have different UID/GID  

Process:  
In window, whether a file can be executed depends on the file extension. In linux, it depends on the execution bit.  
see running process with command:  
```
ps -ef
```
Process attributes:  
RUID, EUID, SUID  
Access permission of resource on system is EUID for users, which almost same to RUID.  
SUID is different from RUID and EUID, it is binded with subject instead of user.  
Take `/usr/bin/passwd` for example:  
```sh
-r-s–x–x 1 root root 21944 Feb 12  2006 /usr/bin/passwd；
```
To take a little introduction to the permission bit:  
* For directory, r bit means the permission of taking a view of file list, such like `ls`.  
* For directory, w bit means the permission of moving the file in it, such like `mv`.  
* For directory, x bit means the permission of switching into it, such like `cd`.  

For example, if you want to let everyone view your web directory, you should set up r bit and x bit for everyone at least!  
> Pay attention! w bit would not include the permission of removal!  

Why other user can change password by executing `passwd` when they forget it?  
It is also because of SUID of `/usr/bin/passwd`.  
The requirement is everyone can execute it.  
When other user try to execute `/usr/bin/passwd`, the EUID of process would be changed into the root,  
then this process temporarily gains the same permission as the root.  
After execution finished, the EUID of process would be changed back to the user's.  
> So, SUID is convenient, right? s bit would be always set up on x field, and only useful for binary program. The user who wants to run it should have permission of x, and he would get the permission as program's owner! If it comes to SGID, it would be set up on the x field of group, and the user who wants to run it would get the permission as program's group! SGID would be more useful if applied to the project directory. If two groups group up to work together, the group2 would like to have the same permission as group1 when they try to make some work on the directory. We can set up s bit at the group field for the directory.

Umask: can be used to set default permission of created files.

## Logging
Unix/Linux/POSIX systems often record system logs by appending to text file, such as `/var/log/messages`.  

## Pluggable Authentication Modules (PAM)
Implemented in many Unix-like systems  
Separates modules for system authentication from application, e.g. `/etc/pam.d` has config for each applications which need authentication  
application would need to call `pam_authenticate`.
