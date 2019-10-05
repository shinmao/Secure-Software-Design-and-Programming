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
