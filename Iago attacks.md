## Model
- Running trusted code on an untrusted OS
- Protection mechanisms in place to keep a malicious kernel from directly manipulating a trusted application's state. 
- Application and kernel are peers and the system call API defined an RPC interface between them. 

## Iago attacks
- Attacks that a malicious kernel can mount in the above model. 
- A carefully chosen sequence of integer return values to Linux system calls can lead a supposedly protected process to act against its interests, and even to undertake arbitrary computation at the malicious kernel's behest. 