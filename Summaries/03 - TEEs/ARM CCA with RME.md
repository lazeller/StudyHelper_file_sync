= Arm Confidential Computing Architecture (CCA) with Realm Management Extension (RME)
- second generation of TEEs
- geared towards cloud envs
- uses a VM abstraction, which is more native to cloud environments
- RME: name of the extension of ARM 

- **Trustzone**
	- had a normal world to run all your legacy code, your hypervisor, your VMs
	- secure world: for more sensitive computations. 
	- Problems with secure world: 
		- bare metal access all the memory + all resources
		- sec world has all the privileges and can access all the memory in the normal world
- Big changes in ARM CCA: 
		1. Introduce root world (new) - very similar to M-mode in keystone. Essentially, that's where the trusted firmware on ARM executes (called the TFA?)
			1. Monitor: very similar to the keystone monitor. In charge of enforcing isolation rules, and acts as an interface to the ISA extensions that are available on the SoC 
			2. The root world is trusted
			3. Has access to all data that is mapped into any of the 3 worlds
![[Ex sess 03 - Trust Assumptions ARM CCA with RME.png]]
### Worlds
![[ARM CCA world memory access matrix.png]]
#### Root world
- new in CCA, very similar to M-mode in keystone. 
- Trusted firmware (is the security monitor?) on ARM executes (called the TF-A) runs here
- Monitor runs here
	- Monitor: very similar to the keystone monitor. In charge of enforcing isolation rules, and acts as an interface to the ISA extensions that are available on the SoC 
- trusted and has access to all data that is mapped into any of the 3 worlds

#### Non-secure world
- = normal world
- only has access to its own memory
- accessible to everyone
- e.g, the cloud provider's hypervisor executes here
#### Secure World
- access own memory, normal world memory

#### Realm World
- new addition to create virtual machines
- get 3 exception levels in this world: El0, El1, El2
- allows you to run a VM, e.g. with the user mode and kernel mode component
- Contains RMM
	- not a hypervisor (no memory or resource management)
	- There to ensure isolation btw Realm VMs themselves
- Can access own memory and that of normal world
- CanNOT access secure world or root world memory
- Secure world and realm world are mutually exclusive to each other

## Salient features
- as opposed to TrustZone, there is a full software stack that you can execute
- gives prototype monitor, prototype RMM
- kernel support for Linux to launch realm VMs
- isolation from host OS and HV -> tailor-made for cloud computing
- HW-based security mechanisms that are in the CPU -> attestation
- realm can contain a full OS stakc
- ARM specifies how attestation should be done, but then left to manufacturer how they enable it

## Isolation
- have untrusted HV
- Wants to access VM's physical memory (PM)
- memory marked as belonging to normal, secure, root or realm world
- New hardware mechanism: GPC (granule protection checks, essentially memory filters for access control)
	- The checks which verify whether the accessor core (runs e.g. the HV) has permission to access PA it's trying to
- GPT: Granule protection table
	- enforces isolation between states
	- Table that keeps track of which PA belongs to which world
	- Monitor can reprogram this table
	- Mapping from PA to world (e.g. "realm")
	-  NOT limited to certain nr of registers like Keystone
	- GPTs: arbitrary ranges of PM marked to belong to one of the four worlds
	- whenever the table is updated, you need to refresh the checks!
- GPC looks up GPT -> ok, PA1 belongs to the realm world, core 1 is running in the realm world -> access ok
- PCI devices and peripherals can bypass the CPU & try to access memory --> ARM also places these GPC checks on devices!!
- PE: processing elements for ARM
	- cores and things such as SMMU 
	- general rule: In CCA, all PEs need to be upgraded to have GPC checks
![[ARM CCA overview.png]]

- software attacker can't access realm or root memory bc GPC checks are *everywhere*

### MPE: Memory Protection Engine
- like in SGX, there's a encryption engine sitting btw the cores, but and DRAM
- this encrypts and integrity protects data from the caches going to the DRAM & back

- have several VMs running in realm world -> isolate those!
- both are running in relam world! Same access rights. 
- trusted component: RMM!
	- has its own memory translation table!
	- typically stage-2 translation table (cluster in cloud, mappings from IPA (Guest physical address) to PA (host physical address))
	- untrusted HV also uses similar table to isolate VMs in normal world
	- one particular PA has to be exclusively accessible to only 1 VM at any point of time --> "Only 1 realm VM has mappings for 1 host physical address"
	- VM is executing in exception level 1 or 0
	- RMM is executing in exception level with higher privilege (EL2).
	- VM can't overwrite RMM table
	- can request per-CVM memory protection keys (specific to each VM)
- Encr and decr is happening at line rate -> CPU should not see any significant delys when it's tryping to load and store info

1. GPC check to check whether right world
2. Mem encr check to see whether right encr keys used

### MEC: Memory Encryption Context
- specification that encodes memory protection feature
- possible keys:
	- different keys for each VM
	- no keys at all (-> no memory encryption)
	- per-memory keys (-> e.g. one VM has two regions in memory)
	- (from ....: For a Realm, having access to more than one MECID register means that it can share encrypted Realm PAS memory with other Realms. This means that memory spaces can be allocated a Memory Encryption Context that can be shared between multiple Realms to allow these Realms to have shared encrypted memory)
- Assigned by trusted SW
- uses system registers and page table bits
- Can setup shared keys btw VMs

## Notes from sections 1 and 2 of ARM Realm Management Extension paper
- Security states:
	- Secure state (safe from non-secure and realm states)
	- Non-secure state
	- Realm state (safe from non-secure and secure states)
	- Root state
- Dynamic memory assignment possible
- The SW in secure and realm states are mutually distrusting
- for EL0, EL1, and EL2, the NS and NSE fields in SCR_EL3 control the security state
- While in EL3, the current security state is always root, regardless of the SCR_EL3.{NSE,NS} value, however, the SCR_EL3.{NSE,NS} value is used to control some operations

![[ARM RME paper - security states with RME.png]]

### Changing security state
![[ARM RME paper - changing security state.png]]

1. Execution stars in realm state (SCR_EL3.{NSE,NS} = 0b11). SW executes a SMC (Secure Monitor Call) instruction -> exception -> taken to EL3!
2. Processor enters EL3 (root state). SCR_EL3.{NSE,NS} is still the same. SW in EL3 changes SCR_EL3.{NSE,NS} to the corresponding value for the required other state and executes a ERET (Exception Return). 
3. ERET causes exit from EL3. SCR_EL3.{NSE,NS} controls which state is now entered

- Note:
	- When moving btw states, it is the responsibility of the SW, NOT the HW, to save and restore register contexts. 
	- The Monitor saves and restores this register context. 

### From Exercise Session 03
- isolation between states enforced by GPT (managed by EL3)
- Isolation between realms enforced by stage 2 translation tables (controlled by R-EL2)

#### TCB of secure world
![[Ex sess 03 - Trust Assumptions ARM CCA with RME TCB secure world.png]]

#### TCB of non-secure world
![[Ex sess 03 - Trust Assumptions ARM CCA with RME TCB non-secure world.png]]

#### TCB of one realm in realm world
![[Ex sess 03 - Trust Assumptions ARM CCA with RME TCB realm world.png]]