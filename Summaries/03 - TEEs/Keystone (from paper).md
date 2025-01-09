## PROs of Customizable TEEs
- Independent exploration of gaps / trade-offs in existing designs
- Quick prototyping of new feature requirements
- A shorter turn-around time for fixes
- Adaptation to threat-models
- Usage-specific deployment

### Motivation
- Threat model may differ depending on the use case, the application or the hardware platform 
- --> We allow each enclave to specify its configuration of security features

Hypervisor sol results in a trusted layer with a mix of security and virtualization responsibilities -> complicated!

- each enclave operates in its own isolated physical memory

## PMP: Physical Memory Protection
- primitive
- allows the programmable machine mode underneath the OS in RISC-V to specify arbitrary protections on physical memory regions

## SM: Security Monitor
- (trusted)
- provides security boundary (without needing to perform resource management)
- manages hardware-enforced guarantees
- Uses four properties of M-mode
	- it's programmable by platform providers
	- meets needs for minimal highest privilege mode
	- controls HW delegation of interrupts and exceptions in the system
	- M-mode's control of RISC-V PMP standard enables isolation of memory-mapped control features at runtime
- handles all machine interrupts 
- Sets machine timer before entering enclave (avoids DoS)

## RT: runtime (supervisor-mode?)
- manages the virtual memory of the enclave (and more)
- implement enclave-specific functionality here
- communicates with SM
- mediates communication with the host via shared memory
- Services the eapp (enclave user-mode application)
- Runs in S-mode (supervisor)
- resides in enclave address space -> isolated from untrusted OS or other apps
- Manages lifecycle of user code executing in the enclave, manages memory, services syscalls, etc
- Each enclave can choose own RT and the RT is never shared btw enclaves
- Handles exceptions for standard kernel abstractions and might forwards other traps to the untrusted OS via the SM

## Eapp: enclave user-mode application
- in U-mode
- resides in enclave address space -> isolated from untrusted OS or other apps
## RISC-V
- RISC-V provides per-hardware-thread views of physical memory via machine-mode and PMP registers
- --> Allows multiple concurrent and pot. multi-threaded enclaves to access disjoint memory partitions while also opening up supervisor-mode and the MMU for enclave use
- -->Allows an enclave to contain either a lightweight or even a full supervisor-mode OS
- Has four priviledge modes
	- U-mode (user) for user-space processes
	- S-mode (supervisor) for the kernel
	- H-mode (hypervisor) for the hypervisor
	- M-mode (machine) which directly accesses physical resources (e.g., interrupts, memory, devices)

## KEYSTONE
- requires no changes to CPU-cores, memory controllers, etc
- HW requirements
	- device-specific secret key visible only to the trusted boot process
	- HW source of randomness
	- trusted boot process
- Keystone enclave workflow
	- 1. Platform provider configures the SM
	- 2. Keystone compiles and generates the SM boot image
	- 3. Platform provider deploys the SM
	- 4 Developer writes an eapp, configures the enclave
	- 5. Keystone builds the binaries, computes measurements
	- 6. Untrusted host binary is deployed to the machine
	- 7. Host deploys the RT, the eapp, and initiates the enclave creation
	- 8. Remote verifier can attest based on known platform specifications, keys, and SM / enclave measurements
- Protects confidentiality and integrity of all enclave code and data at all times after creation
- Does not protect against [[Speculative Execution + Attacks]]
- No protection against timing side-channel attacks
- No protection against side-channel attacks using off-chip components
- Natively, Keystone supports N-2 simultaneously created enclaves, where N is the number of PMP entries available.
- During use (post-creation), Keystone uses the OS-generated page table for initialization and then delegates virtual-to-physical memory mapping entirely to the enclave during execution
- Page tables are always inside the isolated enclave memory space
- Supports primitives:
	- Secure Boot
	- Secure Source of Randomness
	- Remote Attestation
	- can optionally implement: trusted timers, rollback defense, sealed storage

### Attacker Models
#### Physical Attacker
- can intercept, modify, or replay signals that leave the chip package
- does not affect the components inside the chip package
#### Software Attacker 
AND:
- can control:
	- host applications
	- the untrusted OS
	- network communications
- can launch adversarial enclaves
- can arbitrarily modify any memory not protected by the TEE
- can add / drop / replay enclave messages
#### Side-channel attacker
- can glean information by observing interactions between the trusted and the untrusted components via (OR):
	- the cache side channel
	- the timing side channel
	- controlled channel 
####  Denial-of-service attacker
- can take down the enclave or the host OS
- Keystone allows the OS to DoS enclaves as the OS can refuse services to user applications at any time
### Design Principles
- Leverage programmable layer and isolation primitives below the untrusted code
- Decouple the resource management and security checks
- Design modular layers
- Allow fine-grained TCB configuration
## Some Acronyms and Abbreviations that I didn't know where to put
- SBI: RISC-V supervisor binary interface
- CMA: Contiguous Memory Allocator
- IPIs: inter-processor interrupts (Note: PMP synchronization IPIs are only sent during enclave creation and destruction)