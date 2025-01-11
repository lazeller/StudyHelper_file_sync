(See also [[Keystone (from paper)]])
Date: 11.11.2024
![[Keystone - Architecture and Trust Model Graphic.png]]
Trusted hardware: 
- RISC-V cores
- optional HW features (such as a memory encryption engine)
- Root of trust (important for sealing and attestation -> need a piece of HW in your SoC which takes care of attestation)
Security monitor
- executes in M-mode
- Where firmware executes
- where keystone implements most of its isolation logic
- Next to the firmware in the machine more

Untrusted software stack (that runs in S-mode and U-mode): consists of OS and apps
The enclaves are next to the untrusted software stack, even down to being next to the OS
- Eapp: runs in application layer (U-mode)
- Runtime: next to OS, runs in S-mode
- Each enclave has own runtime. Choice to pick what you want


- microarchitectural optimizations = side channels. We can't avoid it, hence, Keystone does not assume absence of side channels
- if you corrupt the firmware, all bets are off
- In keystone, have a specific untrusted region of memory reserved for an enclave to access (U1)
## PMP
- fixed nr of PMP registers given by HW (range: 0-64, standard: 16)
- One PMP has: 
	- pmpaddr: address range (which range of contiguous physical addresses is this PMP entry going to protect?)
	- Permissions
	- pmpgfg: Configuration: decides who are these permissions going to be enforced on?
- highest prio PMP is checked first 
- LIMITED BY NR OF THESE REGISTERS!

## How to transition btw views
 - SM is in charge of identity management. 
 - Only entity in system that can program PMP registers
 - SM reprograms registers to change views
### start
- boot SM (1st part of memory, highest prio PMP register (pmp0) is for SM)
- put memory for OS -> all memory rwx -> lowest prio PMP (pmpN) --> if all checks fail, OS can access it
- OS gives region of memory to SM + OS gives SM page table (bc need a page table to bootstrap the mapping for the enclave). SM has to check that page table is safe bc OS could do a lot of attacks. Check in attestation. 
- keystone enclaves are directly created on physical memory
- each enclave + RT do the memory management (physical to virtual mapping)
### execution - flipping of pmp entries
- flip from OS to E1:
	- disable OS (flip pmpN from rwx for OS to ---)
	- enable E1 view (flip pmp1 from --- to rwx)
	- + the entry for the untrusted region
- Instead of virtual memory, the PMP checks are happening directly on physical memory.

1. CPU: virtual to physical memory address translation, when physical memory accessed -> PMP check!! --> fairly lightweight, performance-wise

- no hardware changes :))) as long as you have a RISC-V core, you're good
## Various memory protection mechanisms
- baseline: software attacker. CC: controlled channel attacker
- control side channel attack: adversary observes the page, page faults and learns some info about them. 
- Keystone: protection against sw attacker (memory protection) + against CC attacker (bc enclave has its own page tables -> malicious OS cannot maipulate the enclave's page tables)
- Last level cache -> no isolation there -> that's the baseline that you get
- Cache partitioning: use PMP checks not only for physical memory accesses but also for accesses to the last level cache (LLC)
- On-Chip Enclave
	- The problem with a physical adversary is that it can go and look at the DRAM content. 
	- All data sits in DRAM in plaintext -> Keystone does NOT protect against a physical adversary
	- solution: never leave that data in DRAM --> use the cache not as a cache, but as programmable on-chip memory (really small, 2mb -> but if it fits? voila XD)
- Software Encryption
	- if you have memory that's bigger than 2mb and want to protect it
	- SOFTWARE encryption! When data is movign from LLC to DRAM, encrypt it! Horribly slow, but possible!
- Hardware Encryption
	- = having a MEE

- you can pick what you protect against using your own runtime :))
CAN DO: 
- efficient memory sharing btw enclaves
- NOn-CPU TEEs
- sovereign cloud: Could protect against Nation state adversaries
- Sovereign smartphone (use it for phones as well!)

## Info from exercise session slides 03
![[Ex sess 03 - Trust Assumptions Keystone.png]]

### TCB of untrusted OS
![[Ex sess 03 - Trust Assumptions Keystone TCB untrusted OS.png]]

### TCB of eapp
![[Ex sess 03 - Trust Assumptions Keystone TCB eapp.png]]