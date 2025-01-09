![[Intel SGX graphic.png]]- current generation
- enclaves (contain small pieces of user code)
- private memory for CPU (encrypted!!)
- user-level abstraction (no syscalls)
- OS / hypervisor level
- deployed on the cloud (must encrypt memory bc attacker might have physical access)
- creates Enclave Measurement of building activities
- Enclave Identity (MRENCLAVE) represents TCB
- Uses Remote Attestation
- SGX Sealed Storage (incl. SGX Sealing Process)

## Assumptions
- Trusted hardware (manufacturer + design & implementation are trusted)
- No side-channels (in HW & SW)
- No bugs in TEE-bound code
- Secure crypto primitives

## Attacker Capabilities
- Cloud Service Provider 
	- Admin access
	- Physical access
- Untrusted privileged software (OS, Hypervisor)
- Peripherals (connected to the host)

## Enclave Measurement 
- when building enclave, Intel SGX generates a cryptographic log of all the build activities
	- Content: Code, Data, Stack, Heap
	- Location of each page within the enclave
	- Security flags being used
	- Order in which it was built
### MRENCLAVE ("Enclave Identity")
= a 256-bit digest of the log
- represents the enclave's software TCB (trusted computing base)
- Is essentially computed by repeatedly taking a data chunk and a metadata chunk and putting them through SHA-256 to add that to MRENCLAVE

## From Keystone paper
SGXv1:
- requires statically sized enclaves
- lacks secure I/O and syscall support
- vulnerable to significant side-channels
SGXv2:
- dynamic resizing

- has large software stack?
- does not support any config of its memory protection systems

## From watching lecture recording (date: 11.11.2024)
- all data to be protected has to go into enclave
- v1 was for laptops 
- goal: minimize TCB
- enclave runs in own memory that is physically partitioned from rest of untrusted software
- Protects from software adversary
- Protects against physical adversary (memory is encrypted and integrity protected -> huge performance cost, tho)
	- MEE: Memory Encryption Engine
	- Built to get performance to acceptable level
	- is on CPU
	- does encr and integrity protection
- SGX forces you to declare the stack, heap, code region etc to enclave explicitly in v1 also statically def. sizes for each of these segments. In v2 dynamic. 
- EPC: Enclave Page Cache -> where encrypted pages are going to be stored in a very particular order, dep on enclave physical page, etcetc
- EPC dedicated to purely enclave memory
- EPCM: Enclave Page Cache Map: management of EPC (no enclave can be assigned this part of the memory, where the integrity tree for EPC memory is kept, enclave metadata)
- Upon machine (with SGX support) boot, have to declare in teh BIOS how much is region that's going to be dedicated to EPC and EPCM (fixed, can't be changed later without BIOS reboot. etc)
- for each physical page which is 4 KB, the EPCM has to maintain a hash and then this hash is organized in a Merkle tree. More memory -> larger Merkle tree -> slower MEE

- Scalable SGX (for cloud deployment) -> put away Merkle Tree. Does not protect against physical adversary. 
	- still maintains integrity about the page (but no versioning and no linkage btw all the physical pages)


Untrusted Memory - what to protect?
- enclave protection
- enclave code & data
- data gen. by enclave
- timers, rand. numbers, etc (services)
- SGX uses special Intel SGX SDK
- start + end of enclave
- paiiiiiiin to program, new attacks, breaks abstractions: 
	- Enclave-enclave communication
	- Memory management of enclave (still done by OS for physical regions, but virtual addresses are a problem!)
	- IO / syscalls by enclave

- host has own virtual memory that it can allocate itself
- enclave can access host's memory (important for passing information) but host can't access enclave memory
- SGX is the only production TEE which has the minimal TCB
- hard limit on size of private memory
- microarchitectural side-channels
- Breaks abstractions and compatibility of a lot of stuff
- CPU does access control for memory
- TCB consists of only code in enclave...
- enclaves are created on VIRTUAL memory here!!!
- If the hypervisor tries to read a VM's memory, it just reads zeroes and writes will just fail
- Attestation: Intel cores itself do the attestation
- Protections are implemented in microcode