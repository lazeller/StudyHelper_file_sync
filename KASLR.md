= Kernel Address Space Randomization
- activated by default
- a feature for KVM guest virtual machines. KASLR enables randomizing the physical and virtual address at which the kernel image is decompressed, and thus prevents guest security exploits based on the location of kernel objects.

source: [3.3. Kernel Address Space Randomization | Red Hat Product Documentation](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/virtualization_security_guide/sect-virtualization_security_guide-guest_security-kaslr#sect-virtualization_security_guide-guest_security-KASLR)

- KASLR (linux) also defends against Meltdown

## Meltdown 
- a speculative execution CPU vulnerability [[Speculative Execution + Attacks]]
- more info on Meltdown: [Meltdown (security vulnerability) - Wikipedia](https://en.wikipedia.org/wiki/Meltdown_(security_vulnerability))
	- "The vulnerability allows an unauthorized process to read data from any address that is mapped to the current process's [memory](https://en.wikipedia.org/wiki/Virtual_memory "Virtual memory") space."