### Tools
1.  **Disassembler**
	- objdump -d -M intel /tmp/your-program
2. **What syscalls does it make?**
    - `strace ...`
3. **What library calls does it make?**
    - `ltrace ...`
4. **Debug it interactively**
    - `gdb /challenge/whatever`

### Note
The alarm() function **causes the system to generate a SIGALRM signal for the process after the number of real-time seconds specified by seconds have elapsed**

- int3 for coorporate debug

# Memory inspection

Examine memory with `x`.

Format:
```gdb
x/FMT ADDRESS
```

Examples:

```gdb
x/s $rdi
x/4gx $rsp
x/10i $rip
```

Common formats:

- `s` — string
- `a` — address
- `i` — instructions
- `x` — hex
- `d` — decimal
- `g` — 8-byte chunks

Very useful:

- `x/s $rdi` → inspect string pointed to by register
- `x/10i $rip` → show next instructions

### Stack Pointers 

if your program is run as `/challenge/debug-me Hi`:

```text
     Address    │ Contents
   +────────────────────────────+
   │  rsp + 0   │ 2             │◀── argc
   +────────────────────────────+
   │  rsp + 8   │ 0x1234000     │──────┐
   +────────────────────────────+      │
   │  rsp + 16  │ 0x1234560     │────┐ │
   +────────────────────────────+    │ │
                                     │ │
                                     │ │
     Address    │ Contents           │ │
   +──────────────────────────────+  │ │
   │ 0x1234000  │ "/challenge/..."│◀─│─┘ the program name
   +──────────────────────────────+  │
   │ ...        │ ...                │
   +──────────────────────────────+  │
   │ 0x1234560  │ "Hi"            │◀─┘   the first argument
   +──────────────────────────────+
```


- 


### gdb commands

- `starti`
    - stops at the very first instruction (`_start`)
    - useful for low-level stepping
    - `starti` **start**s the program at the very first **i**nstruction. Once the program is running, you can use other gdb commands to inspect its actual runtime state. We'll start with the code that's running, which you can disassemble using the `disassemble` command! For example:
```gdb
(gdb) starti
```

- `disassemble`

```gdb
(gdb) disassemble
Dump of assembler code for function main:
=> 0x0000000000401000 <+0>:     mov    rdi,0x539
   0x0000000000401007 <+7>:     mov    rdi,0x0
   0x000000000040100e <+14>:    mov    rax,0x3c
   0x0000000000401015 <+21>:    syscall
End of assembler dump.
```

- `run`
    - actually launches the program normally
    - can include command-line arguments
    - Whatever comes after `run` becomes:
	    - `argv[1]`, `argv[2]`, etc.
	- Outside GDB, you pass args like:
    
    ```bash
    /challenge/debug-me hello
    ```
    
	- Inside GDB, you pass them to `run` (or `r`):
    
    ```gdb
    run hello
    ```


- `stepi`
	To execute a single instruction in GDB, use the `stepi` command (**step** one **i**nstruction, also abbreviated `si`):

```gdb
(gdb) stepi
```

- `print`
	- The `print` command displays the value of an expression. Register names in GDB are prefixed with `$`, so you can read `rdi` like this:

```gdb
(gdb) print $rdi
$1 = 1337
```


