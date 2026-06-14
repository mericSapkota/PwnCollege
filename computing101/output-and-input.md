## Core idea: `write` syscall

On Linux x86-64:

- `rax` = syscall number
- `rdi` = 1st argument
- `rsi` = 2nd argument
- `rdx` = 3rd argument

For `write`:

```c
write(fd, buf, count)
```

So:

- `rax = 1` → `write`
- `rdi = 1` → stdout
- `rsi = address of data`
- `rdx = number of bytes`

---

## What code does

```asm
mov rdi, 1
mov rsi, [rsp+16]
mov rdx, 1
mov rax, 1
syscall
```

This means:

- write to **stdout**
- use the pointer stored at **`[rsp+16]`**
- write **1 byte**
- invoke syscall **1** (`write`)

---

## Why `[rsp+16]`?

At program start, the stack looks like:

- `[rsp]` = `argc`
- `[rsp+8]` = pointer to `argv[0]` (program name)
- `[rsp+16]` = pointer to `argv[1]` (first user argument)

So if you run:

```bash
./write x
```

then `[rsp+16]` points to the string `"x"`.

Because `rdx = 1`, only the first byte is written.

---

## Why it segfaulted

Your program successfully called `write`, but after `syscall`, execution continued into whatever bytes came next.

 **did not exit**.

So the CPU kept running garbage/unintended memory as instructions, which caused the crash.

---

## Main lesson from this level

- `rdi` = where to write
- `rsi` = what memory to write from
- `rdx` = how many bytes to write

---

## Common mistakes

- Mixing up `rdi` and `rdx`
- Forgetting that `argv[1]` is a **pointer**
- Forgetting to make an `exit` syscall afterward
```asm
 mov rdi, 1
 mov rsi, [rsp+16]
 mov rdx, 1
 mov rax, 1
 syscall
 mov rax, 60
 syscall

 ```
for read 
`read(0, some_address, 5);`
args: rdi, rsi, rdx

.intel_syntax noprefix
.global \_start
\_start:
    # 1. READ 64 bytes from stdin (sys_read)
    mov rax, 0      # sys_read ID
    mov rdi, 0      # file descriptor: stdin
    mov rsi, rsp    # memory address buffer: using the stack pointer
    mov rdx, 64     # buffer size: 64 bytes
    syscall

    # 2. WRITE 64 bytes to stdout (sys_write)
    mov rax, 1      # sys_write ID
    mov rdi, 1      # file descriptor: stdout
    mov rsi, rsp    # memory address buffer: where we just saved the data
    mov rdx, 64     # count: 64 bytes
    syscall

    # 3. EXIT with code 42 (sys_exit)
    mov rax, 60     # sys_exit ID
    mov rdi, 42     # exit code status
    syscall

.intel_syntax noprefix
.global _start
_start:
    # 1. Read up to 64 bytes
    mov rdi, 0
    mov rsi, rsp
    mov rdx, 64
    mov rax, 0
    syscall         # After this, rax contains the number of bytes read (e.g., 4)

    # 2. Write ONLY the bytes that were read
    mov rdi, 1
    mov rsi, rsp
    mov rdx, rax    # Move the actual number of bytes read into rdx!
    mov rax, 1
    syscall

    # 3. Exit
    mov rdi, 42
    mov rax, 60
    syscall

### 1. Characters and Strings in Memory

- **Single Quotes (`'a'`) vs. Numbers:** In assembly, using single quotes like `'f'` is just a human-friendly way of writing its **ASCII numeric value** (e.g., `'f'` is automatically converted by the assembler to `0x66` or `102`).

- **The Null Terminator (`0`):** Linux system calls like `open` expect strings to be **null-terminated**. This means the operating system scans the memory byte-by-byte until it hits a `0`. Without that `0` at the end (`mov BYTE PTR [rsp+5], 0`), the OS would keep reading random RAM garbage after `"/flag"`, resulting in a "File Not Found" error.


### 2. Assembly Directives & Memory Sizing

- `.intel_syntax noprefix`: Tells the assembler to use Intel formatting (Destination on the left, Source on the right: `mov dest, src`) and eliminates the need to put `%` signs before registers (e.g., `rax` instead of `%rax`).

- `BYTE PTR`: Explicitly tells the CPU the **size of the memory destination**. Because `rsp` is just a memory address, the CPU doesn't know if you want to write a byte (8 bits), a word (16 bits), or a quadword (64 bits). `BYTE PTR` forces it to modify exactly **1 byte** of memory.


### 3. Registers & System Call Pipeline

- **`rax` handles two jobs:** Before a `syscall`, `rax` holds the **System Call ID** (what you want the OS to do). Right _after_ a `syscall`, the OS overwrites `rax` with the **return value** (the result of the operation).

- **File Descriptors (FD):** When `sys_open` succeeds, it returns a small integer handle (usually `3`) in `rax`. You must immediately copy this out of `rax` (into `rdi` for `sys_read`) before `rax` gets overwritten by the next syscall ID.

- **Dynamic Write Size:** Copying `rax` (the number of bytes successfully read) into `rdx` before running `sys_write` ensures your program only prints the exact size of the file, cleanly avoiding terminal garbage values.


##  Code with Line-by-Line Comments

Code snippet

```as
.intel_syntax noprefix     # Use Intel syntax; drop the '%' prefix for registers
.global _start

_start:
    # =========================================================================
    # 1. CONSTRUCT THE FILENAME STRING ON THE STACK
    # =========================================================================
    # We write "/flag" character-by-character into the memory pointed to by RSP.
    # 'BYTE PTR' ensures we only modify exactly 1 byte of memory at each offset.
    mov BYTE PTR [rsp], '/'     # Store ASCII '/' at the base stack address
    mov BYTE PTR [rsp+1], 'f'   # Store ASCII 'f' 1 byte forward
    mov BYTE PTR [rsp+2], 'l'   # Store ASCII 'l' 2 bytes forward
    mov BYTE PTR [rsp+3], 'a'   # Store ASCII 'a' 3 bytes forward
    mov BYTE PTR [rsp+4], 'g'   # Store ASCII 'g' 4 bytes forward
    mov BYTE PTR [rsp+5], 0     # Null terminator: Marks the absolute end of the string

    # =========================================================================
    # 2. OPEN THE FILE ("/flag")
    # =========================================================================
    mov rax, 2                  # Syscall ID 2: sys_open
    mov rdi, rsp                # Argument 1: Pointer to filename string (which is on the stack)
    mov rsi, 0                  # Argument 2: Flags (0 = O_RDONLY, Read-Only mode)
    mov rdx, 0                  # Argument 3: Mode/Permissions (Not creating a file, so 0)
    syscall                     # Trigger Linux kernel. RAX now contains the File Descriptor (e.g., 3)

    # =========================================================================
    # 3. READ FROM THE OPENED FILE
    # =========================================================================
    mov rdi, rax                # Argument 1: Move the File Descriptor from RAX into RDI
    mov rax, 0                  # Syscall ID 0: sys_read
    mov rdx, 64                 # Argument 3: Count (Read a maximum buffer size of 64 bytes)
    mov rsi, rsp                # Argument 2: Buffer address (Overwrite the stack space with file contents)
    syscall                     # Trigger Linux kernel. RAX now holds the actual number of bytes read

    # =========================================================================
    # 4. WRITE THE CONTENTS TO STDOUT
    # =========================================================================
    mov rdx, rax                # Argument 3: Pass the actual bytes read (from RAX) into RDX
    mov rax, 1                  # Syscall ID 1: sys_write
    mov rdi, 1                  # Argument 1: File Descriptor 1 (stdout / terminal screen)
    mov rsi, rsp                # Argument 2: Pointer to the data buffer (our stack)
    syscall                     # Trigger Linux kernel. File content prints to screen.

    # =========================================================================
    # 5. EXIT THE PROGRAM cleanly
    # =========================================================================
    mov rdi, 42                 # Argument 1: Status code 42
    mov rax, 60                 # Syscall ID 60: sys_exit
    syscall                     # Terminate program execution
```
