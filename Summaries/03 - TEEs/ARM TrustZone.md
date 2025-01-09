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