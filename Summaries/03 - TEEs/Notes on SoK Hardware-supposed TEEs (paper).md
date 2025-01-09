## High-level security properties
1. **verifiable launch**: verifiable launch of the execution environment for the sensitive code and data s.t. a remote entity can ensure that it was set up correctly
2. **run-time CPU and memory isolation**: run-time isolation to protect the confidentiality and integrity of sensitive code and data
3. **trusted IO**: trusted I/O to enable secure access to peripherals and accelerators
4. **secure storage**: secure storage for TEE data that must be stored persistently and made available tonly to authorized entities at a later point in time. 

### Verifiable launch
= provide proof regarding the correctness of the initial state of the TEE
1. establish RTM (Root of Trust for Measurement)
2. *measure* the state of the code / data within the TEE (using RTM)
3. make measurement available for verification through *attestation*
- TPM: Trusted Platform Module
	- used by Trusted Computing Group to do standard measurement and attestation processes
- measurement = fingerprint of enclave's initial state
	- cryptographic hashes
	- must be trustworthy
	- start with TPM; then chain-of-trust over TCB
#### RTM 
- SRTM: static RTM
- DRTM: dynamic RTM
- HW (RTM): hardware based RTM

(Note that **some** remote attestation schemes can be trivially reused for local attestation)
- Intel SGX: DRTM (local + remote attestation)
- Intel TDW: DRTM (local + remote attestation)
- AMD SEV-SNP: SRTM (remote attestation)
- ARM TrustZone: SRTM (- attestation)
- ARM Realms: SRTM (remote attestation)
- Keystone: SRTM (remote attestation)

![[SoK HW-supported TEEs RTMs.png]]
#### Attestation
- local 
	- verifier co-located with enclave on same platform
	- typically with symmetric cryptography
	- tends to be efficient
- remote
	- remote verifier (not on same platform)
	- usually rely on asymmetric cryptography
	- cost of checking one or more certificate chains
	- can be expensive

#### Provisioning Secrets into Enclave
- AMD SEV-SNP allows enclaves to be provisioned with secret data prior to the attestation. 
- AMD SEV supports both initial secret provisioning (put secret data into enclave before measurement and attestation and it's then also included in those two steps) and establishing a secure channel bound to an attestation (enclaves in TEEs may append some custom data (e.g., a public key certificate) to the attestation report -> this data is also authenticated during attestation -> integrity is protected -> construct secure channel)

### Run-time isolation
Dimensions of isolation strategies:
- resource partitioning
- isolation enforcement

- Intel SGX uses spatio-temporal partitioning and logical enforcement to protect enclave memory, but it uses purely spatial partitioning to protect its TCB, and cryptographic isolation to cope with $A_{bus}$
- Options for access control check for logical enforcement of spatial or spatio-temporal memory isolation
	- MPU: memory protection units
		- Operates on physical addresses
		- typically support a limited nr of rules
		- simple
		- used more in academia
		- access control rules
		- range registers are simplified MPUs
	- MMU: memory management units
		- operates on virtual addresses
		- flexible in terms of nr of rules
		- used more in commercial TEEs
		- trusted page tables
	![[SoK HW-supported TEEs partition options.png]]
	![[SoK HW-supported TEEs isolation enforcement strategies.png]]
### Trusted IO
- Common main components of trusted IO:
	- trusted path to the device (confidentiality and integrity for enclave's access to the device)
		- logical
		- cryptographic
	- trusted device architecture (to protect sensitive data just like the CPU, protect enclave data on device)

#### Trusted path to device 
- logical and cryptographic both protect against $A_{tee}, A_{app}, A_{ssw}, A_{boot}, A_{per}$
- Only cryptographic trusted paths can protect against $A_{bus}$
### Secure Storage
= *sealing* = Any sensitive data that is persistent is only available to authorized entities. 
+ reverse sealing = *unsealing*
+ TPM's PCRs: TPM's Platform Configuration Registers
+ Keystone: sealing support via software TCB -> TCB exposes interface to create sealing keys for each enclave -> no add. HW or architectural support needed
+ Intel SGX: special CPU instructions in HW to enable sealing -> include instrs to generate and access sealing keys based on different types of binding (e.g., developer identity, enclave measurement)
### Some terminology
- TEE instances
	- in SGX: enclaves
	- in TDX (Trust Domain Extensions): Trust Domains
	- in SEV-SNP (AMD Secure Encrypted VMs-Secure Nested Paging): Secure Encrypted VMs
	- ARM CCA (ARM Confidential Compute Architecture) uses Realm for their VM-isolation sol and trustlets for their application isolation feature
- TEE: refers to the entire architecture that enables the creation of enclaves
- SoC (System-on-Chip)
#### SoC (System-on-Chip)
- has off-chip memory and, optionally, off-chip peripherals
- contains one or more cores that potentially share a cache (or parts thereof) and fabric that connects them to the memory controller
- include an IO complex
	- used to connect both on-chip and off-chip peripherals to the SoC fabric and caches

### Adversaries
![[SoK HW-supported TEEs adversaries.png]]
- Co-located enclave adversary $A_{tee}$
- Unprivileged software adversary $A_{app}$
- System software adversary $A_{ssw}$
- Startup adversary $A_{boot}$
- Peripheral adversary $A_{bus}$
- Invasive adversary $A_{inv}$

![[SoK HW-supported TEEs privilege levels in modern processors.png]]


## Other stuff (TCB discussion)
- TCB recovery: post-manufacture updates to mutable components of TCBs (typically reflected in attestation report of the platform)
	- mutable -> usually SW
	- immutable -> usually HW
	- Exceptions:
		- mutable HW: $\mu code$ in certain CPUa
		- immutable SW: boot ROM
		- 