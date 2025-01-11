## Acronyms etc
- Hardware features of modern Intel processors
	- MPX: memory protection extensions
		- for checking pointer bounds
	- AES-NI: encryption instructions
	- MPK: memory protection keys
	- SGX: software guard extensions
- virtualization features
	- VMFUNC instruction
- MemSentry:
	- self-contained deterministic memory-isolation framework

### Intel MPX Example
(From exercise session 05's slides)
``` 
a_b = bndmk a, a+79 (create pointer bounds)
bndcl a_b, ai (check memory / register operands against lower bound)
bndcu a_b, ai+7 (check memory / register operands against upper bound)
```

### MemSentry Example
(From exercise session 05's slides)
``` 
; Original
mov rdi, [rbx + 8] 

; Instrumented with MPX
lea rcx, [rbx + 8]
bndcu bnd0, rcx 
mov rdi, [rbx + 8]

; Instrumented with SFI
lea rcx, [rbx + 8]
movabs rax, #0x3ffffffffff
and rcx, rax
mov rdi, [rcx]
```