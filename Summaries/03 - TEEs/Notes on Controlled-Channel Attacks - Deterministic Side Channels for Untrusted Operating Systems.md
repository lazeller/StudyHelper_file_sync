## Attack Model
- Attacker controls OS
- monitor components and application's code untouched
- Memory resource management by the OS
	- virtual memory
	- OS can use demand paging
	- OS controls virtual to physical memory mappings
	- OS canNOT map page where app does not expect it
	- OS can remove virtual-to-physical page mappings
	- OS can restore page mappings to handle page faults
	- OS can get virtual base address of page faulted page but not offset within page
- Applications
	- No special measures taken to obscure their memory access patterns (exception: crypto code has been hardened)
	- public
	- version of targeted application binaries known

## Design
- input-dependent control transfers
- input-dependent data accesses
- exploits memory access patterns that depend on application secrets

## Inferring Input-Dependent Memory Accesses
- *page fault address*: the address whose access triggered the page fault
- *instruction address* (of the page fault): address of the instruction that was being executed when the page fault occurred

## Handling Page Faults
- *tracking pages*: small set of relevant pages (which we track)
- Basic approach:
  1. include all pages in the page-fault sequences identified in the offline analysis as the set of tracking pages + restrict access to them when an app starts
  2. When a page fault happens, log the page-fault event, enable access to the page,  and remove access to the previous page
  - watch out for false positives (when removing those pages from tracking pages, we see the sequence, despite this being a false positive -> keep those pages in the tracking pages)
## ASLR
- the execution of an executable always starts from a predefined entry point
	- Exception: loader. However, the execution of a loader always starts from a deterministic location, plus, it's the first code executed in user mode for a process. 
1. offline analysis (figure out first few page faults for each executable to identify / distinguish them)
2. Process starts -> restrict access to all mapped memory ranges
	1. First code page fault is loader. -> identify the base address of the loader + enable access to all of its pages
	2. next code page fault will be for a different executable
	3. -> identify executable + enable access to all its pages
	4. Repeat until you've identified all loaded executables
- special case: delayed loading
1. restrict access to newly mapped memory regions + track initial code page faults on them
2. Then use first few consecutive code page faults on the new memory regions to identify the dynamically loaded executable

## Other pot. information channels controlled-channel attacks could exploit
- thread scheduling
- patters in the app's system calls to the OS 
- low-noise cache side channels constructed by the OS

## Mitigations
### At application level
- rewrite application s.t. its memory access pattern does not depend on sensitive data
	- Note: This can be done manually or with the help of a special compiler. 
	- impacts application performance
	- may require significant development effort 
	- identifying sensitive data may require exhaustive security analysis of the application
### At system level
- shielding system could prohibit paging by the OS
	- disables important feature of shielding systems
- prohibit paging only for smaller subset of the application's pages (e.g. keeping the OS from paging executable pages in application binaries)
- Given analysis of its data dependent memory access patterns, the application might choose the set of pages that has to be protected TODO: figure out InkTag style mechanism to communicate those to shielding system
- Self-paging (moves paging from OS into app)
	- OS still manages how much memory each app controls
	- requires new HW or new paging interfaces in HV-based shielding systems
	- requires significant changes to legacy OSes
	- requires additional self-paging code in the protected processes
- Hiding application's memory access pattern through noise injection or ORAM techniques
- Attempt to obscure layout of the binaries in memory through variants of fine-grained ASLR

### Mitigation in either application or shielding system
- Attempt to detect artifacts of the attack such as page fault counts or execution time (the latter is not reliable)

## Other notes
- If physical memory is shared between the victim and the attacker, even an unprivileged attacker has direct control over cache lines used by the victim
-> enables Flush-Reload-based cache side channels (very efficient!!)