## From Keystone paper
- isolates a full VM with a large TCB
## From Lecture Recording (11.11.2024)
![[AMD SEV graphic.png]]
- trust the CPU, no side channels
- Hypervisor is perfectly implemented but untrusted
- Untrusted VMs running next to confidential VM 
- VM has own guest OS (can choose that one)
- Hypervisor gives illusion to VM 
### Design choices
- isolate VMs from hypervisor and from each other 
- VMS execute on rings 3, 2, 1 (SGX only supports ring 3)
- 0: host OS runs
- 3: applications

- does NOT protect against physical attacker. Can still do cold boot attacks, but not bus tampering! We don't have a merkel tree to do the versioning, etc., hence, attacker can do some bus tampering
- Everything outside the CPU die is untrusted
### Secure Nested paging -> memory isolation
- HV = hypervisor 
- HV usually in charge of virtual pages, etc.
- HV read and write in VM's memory: 
	- HV can **read** a guest VM's memory, but the memory is encrypted, hence, not a problem. 
	- If HV tries to write -> exceptions!! 
	- (Read and write in In SGX: read: gets 0, write: fail)
- VMs reading / write other VM's stuff -> exceptions
- HV can change mappings -> we have reverse mapping table (RMP). It's a reverse look up table, typically a page table. RMP keep track of which physical address is individually mapped to which virtual address. One way look up -> unique mapping

### Attestation
![[AMD SEV Attestation graphic.png]]
- launch Guest VM 
- HW (processor running next to AMD cores (also sits on the processor)) does measurement of guest state
- Measurement sent to guest owner (could be local VM, local untrusted VM on same machine or a remote user)
- in SGX: intel cores itself do the attestation, here, it's offloaded to a separate security processor

#### Authenticity of Attestation Report
AMD Root CA --certifies--> AMD SEV CA (AMD Certificate Authority) --certifies--> VCEK --signs --> Attestation Report
- KDS: AMD Key Distribution Service
	- Consists of AMD Root CA, AMD SEV CA and VCEK
- VCEK: Versioned Chip-Endorsement Key (used as the attestation key)
- KDF contains 
	- bootloader SVN
	- OS SVN
	- SNP FW (firmware) SVN 
	- Microcode Patch Level (e.g., to check whether CPU has an older patch set, if so, reject measurement)
### Key Management
- Has ARM co-processor (= PSP: AMD Platform Security Processor, = AMD SP) which sits next to the AMD CPUs that manages the encryption keys
- SGX implements protections in microcode, **AMD uses PSP**
- Per-VM level memory encryption
- The CPU creates a new encryption key when a new VM is spawned, & that key is then used in the MEE when it's shipping data to and from the DRAM
- per VM different encryption key / each VM gets its own key