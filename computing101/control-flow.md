## Challenge 1: Comparing argc with 42

### Key Concepts:

- **cmp instruction**: Compares two values by subtracting second from first
    
    - Updates CPU flags (doesn't store result directly)
        
    - `cmp rdi, 42` computes rdi - 42 internally
        
- **Zero Flag (ZF)**:
    
    - Set to 1 if result is zero (values equal)
        
    - Set to 0 if result is non-zero (values not equal)
        
- **setz instruction**: "Set if Zero"
    
    - Writes 1 to byte destination if ZF = 1 (values equal)
        
    - Writes 0 if ZF = 0 (values not equal)
        
    - Only writes to byte-sized register (8 bits)
        
    - `dil` = low byte of `rdi` register
        
- **Memory access**: `[rsp]` contains argc (number of command-line args)
    

### Solution:

asm

cmp QWORD PTR [rsp], 42    ; Compare argc with 42
setz dil                    ; Set dil to 1 if equal, 0 if not
mov rax, 60                 ; syscall number for exit
syscall

---

## Challenge 2: Checking First Character of argv[1]

### Key Concepts:

- **argv[1] pointer**: Located at `[rsp+16]`
    
- **String access**: Load pointer first, then dereference
    
    - `mov rax, [rsp+16]` - get pointer
        
    - `[rax]` - first character
        
    - `[rax+1]` - second character, etc.
        
- **BYTE PTR**: Specifies you're working with single byte (not 64-bit value)
    
- **ASCII values**: Characters like 'p' are represented by ASCII codes
    

### Solution (5 instructions):

asm

mov rax, [rsp+16]          ; Load argv[1] pointer
cmp BYTE PTR [rax], 'p'    ; Compare first char with 'p'
setz dil                   ; Set exit code (1 if match, 0 if not)
mov rax, 60                ; syscall number
syscall

---

## Challenge 3: Conditional Jumps

### Key Concepts:

- **jne**: "Jump if Not Equal" - jumps when ZF = 0
    
- **je**: "Jump if Equal" - jumps when ZF = 1
    
- **Labels**: Named locations in code (e.g., `fail:`)
    
    - Don't generate machine instructions
        
    - Mark spots for jump instructions
        
    - Assembler resolves to addresses
        
- **Branching**: Two different execution paths
    
    - Branch taken → jump to different code
        
    - Branch not taken → "fall through" to next instruction
        

### Solution:

asm

mov rax, [rsp+16]          ; Load argv[1] pointer
cmp BYTE PTR [rax], 'p'    ; Compare first char
jne fail                   ; Jump if not 'p'
; Success case (fall through)
mov rdi, 0                 ; Exit code 0
mov rax, 60                ; syscall number
syscall
fail:
mov rdi, 1                 ; Exit code 1
mov rax, 60
syscall

---

## Challenge 4: Multi-character String Comparison

### Key Concepts:

- **Chained comparisons**: Multiple cmp/jne pairs
    
- **String layout**: Contiguous bytes in memory
    
    - `[rax]` = first character
        
    - `[rax+1]` = second character
        
    - `[rax+2]` = third character, etc.
        
- **Early exit**: Jump to fail on first mismatch
    
    - Only checks remaining characters if all previous match
        

### Solution for "pwn":

asm

mov rax, [rsp+16]          ; Load argv[1] pointer
cmp BYTE PTR [rax], 'p'    ; Check 'p'
jne fail
cmp BYTE PTR [rax+1], 'w'  ; Check 'w'
jne fail
cmp BYTE PTR [rax+2], 'n'  ; Check 'n'
jne fail
; Success path (all characters matched)
mov rdi, 0
mov rax, 60
syscall
fail:
mov rdi, 1
mov rax, 60
syscall

---

## Challenge 5: Reverse Engineering

### Key Concepts:

- **objdump**: Tool to disassemble binaries
    
    - `objdump -d -M intel /challenge/reverse-me`
        
    - Shows assembly instructions
        
- **Hex ASCII values**: Characters represented as hex in disassembly
    
    - `0x70` = 'p'
        
    - Use `man ascii` to convert hex to characters
        

### Strategy:

1. Use objdump to view disassembly
    
2. Find cmp instructions comparing bytes to hex values
    
3. Convert hex values to ASCII characters
    
4. Reconstruct the password string
    
5. Run program: `/challenge/reverse-me PASSWORD`
    

### Example:

If disassembly shows:

asm

cmp BYTE PTR [rax], 0x70  ; 'p'
cmp BYTE PTR [rax+1], 0x77 ; 'w'
cmp BYTE PTR [rax+2], 0x6e ; 'n'

Password = "pwn"
