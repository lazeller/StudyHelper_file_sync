- creates two worlds
	- normal world (non-secure state): where user-facing stuff is
	- isolated world (secure world / secure state): as deployer, you control all software that executes there (keep small!)
- useful for phones bc assume no physical attacker
- prev. generation
- memory is not encrypted ??? (different on lecture slides!)
- Vertical isolation -> in secure world, everything in the stack is trusted (code here keeps getting bigger, tho)

![[ARM RME paper - ARM TrustZone security states.png]]
## From Keystone paper
- more flexible than SGX or SEV
- for edge-sensors or IoT applications
- Supports only a single hardware-enforced insolated domain called the Secure World. 
- Further isolation needs multiplexing btw secure applications via software-based Secure world OS sols
- only TWO security domains
- to implement multiple enclaves, must use MMU for further isolation
- HW: relies on system-wide bus-address filters to separate secure from insecure DRAM partitions

## From Exercise Session Slides 03
### Trust Assumptions Graphic
![[Ex sess 03 - Trust Assumptions ARM TrustZone.png]]

### How does ARM TrustZone enforce runtime isolation?
- Isolation of EL2 to EL1 into "two worlds", one secure, one non-secure / normal
- Based on NS bit, which is propagated on the bus
- Context switches managed by software in EL3
- Memory Isolation etc. must be enforced on subordinate side
- No protection against physical attacker per se
### Advantages
- No interrupt / memory management by untrusted code
- infrastructure (partially) supports secure state
- Availability guarantees for secure world possible
### Disadvantages
- No physical attacker
- Only one "protected" state
- No general attestation scheme
- All software running before and under software in secure world needs to be trusted -> LARGE software TCB
- Secure state is often locked for use by device vendors and not generally open for application developers.