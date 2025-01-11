## Threat Model
- trusted hypervisor or VMM (Virtual Machine Monitor)
- trusted application
- untrusted OS

### from paper
- prevents guest OS from reading or modifying application code, data + registers
- no attempt to provide availability when OS is hostile
- all non-app memory access to cloaked data (incl DMA from virtual I/O devices) -> only get encrypted data
- data secrecy, integrity ordering + freshness dependent on cryptography used (for the encryption)
- modify encrypted data -> application terminated

- NOTE: memory access patterns visible from outside! Timing side-channels!
- Assurance limited by the VMM
- NOT GIVEN: protected network I/O, no trusted path for user input or secure displays
## Goal
- execute *unmodified* applications on *unmodified* hardware while ensuring execution integrity, and data confidentiality and integrity for the applications / keep privacy of application data

## General info
- Overshadow
	- protects legacy applications from commodity OSes running them
	- requires no changes to existing OSes or applications
	- requires no additional HW support
	- extends isolation capabilities of virtualization layer
	- changes to normal execution environment
	- introduces *shim* (at load time) into address space of each cloaked application to mediate all communication with the OS
		- interposes on syscalls and signal delivery (with some help from VMM)
- *Cloaking*: present application with cleartext view of its pages, and the OS with an encrypted view
	- low-level primitive

## Multi-Shadowed Cloaking

### Classical Memory Virtualization
- Terminology:
	- VA / PA: virtual / physical address
	- VPN: virtual page number
	- PPN: physical page number
	- VMM: virtual machine monitor
	- VM: virtual machine
	- MPN: machine page number 
	- machine address + MPN refer to actual HW memory
	- "physical" memory: software abstraction, illusion of HW memory to a VM
	- GVPN: guest virtual page number
	- GPPN: guest physical page number
- page tables: VA-to-PA, resp. VPN-to-PPN
- hardware TLB: caches VPN-to-PPN translations (here: GVPN-to-MPN)
- VMM
	- provides illusion of physical memory to VM
	- maintains pmap data structure for each VM to store GPPN-to-MPN translations
	- typically manages separate shadow page tables, containing GVPN-to-MPN + keeps those consistent with GVPN-to-GPPN mappings by guest OS
- mapping GVPN-to-GPPN: address translation performed by a guest OS in a VM

Single view of guest "physical" memory:
- one-to-one GPPN-to-MPN mappings (back physical page with distinct machine page)
- to support shared memory -> use many-to-one mapping (e.g., transparent page sharing maps multiple GPPNs copy-on-write to a single MPN)
- No flexible support for one GPN-to-many MPNs
### Multi-shadowing
- presents different views of "physical" memory, depending on the context performing the access
- VMM: virtual machine monitor
- supports context-dependent, one-to-many GPPN-to-MPN mappings
- multiple shadow page tables -> different views to different shadow contexts
### Memory Cloaking
- = multi-shadowing + encryption
#### Single page, encrypted / unencrypted views
- represent each GPPN using only a single MPN
- dynamically encrypt / decrypt
- cloaked page accessed from outside the shadow context
	1. VMM encrypts page (fresh, randomly-generated IV)
	2. VMM takes a secure hash H of the ciphertext of the page
	3. Store (IV, H) securely for future use
- decryption:
	- check hash
		- if wrong -> terminate application
		- By checking hash before decryption, you detect any attempts to corrupt cloaked pages
- Overshadow uses single key $K_{VMM}$ (managed by VMM) to encrypt all pages
- AES-128, CBC mode, hash: SHA-256

#### Basic Cloaking Protocol
- compatible with copy-on-write (COW)
GPPN
- At any point in time, the page is mapped into only one shadow page table: either a application shadow (protected and used by a cloaked user-space process) or the system shadow (used for all other accesses)
	- page in application shadow: plaintext, normal 

![[Overshadow paper - graphic basic cloaking protocol.png]]
- transition 1: cloaked page accessed via system shadow
	- VMM unmaps page from application shadow, encrypts page, generates integrity hash, maps the page into the system shadow
	- Kernel can now read (encrypted) contents (e.g. swap page out to disk) or overwrite its contents (swap prev. encrypted page in from disk)
- transition 2 and 3: encrypted page accessed via application shadow
	- VMM unmaps page from system shadow, verifies integrity hash, decrypts page, maps page into application shadow
	- for transition 3 (read): page is mapped read-only and (IV, H) is retained
- Transition 4: page in application shadow in read-only map is written to by application
	- (IV, H) is discarded, page protection changes to read/write
- Transition 5: page in application shadow, in read-only mode is written to by kernel
	- same as transition 1, EXCEPT that the hash for the (unmodified) page is not recomputed

#### Virtual DMA
- For a disk read, the cloaked page contents are already encrypted on disk -> VMM simply permits kernel to issue a DMA request to read the page
- Disk write
	- page already encrypted: VMM allows DMA to be performed directly
	- page in plaintext read-only state: VMM encrypts the page with its existing (IV, H) into a separate page used for the DMA operation
	- page in plaintext read-write state: VMM encrypts its contents into a separate page used for the DMA operation. Cloaked page goes into read-only plaintext state and is associated with the newly-generated (IV, H)
	- For both plaintext states, page is still accessible, since the DMA operation uses an encrypted copy of the page
#### VMM
- usual model:
	- one-to-one mapping from guest "physical" addresses to actual machine addresses
- Multi-shadowing
	- one-to-many, context-dependent mapping -> multiple views of guest memory

## Design Goals
- Ease of Adoption
- Support for Diverse Applications
- Incremental Path to Higher Assurance
	- Can use cloaking for whole application protection as well as fine-grained compartmentalization
- 