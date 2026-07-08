
## Little Endian

Every value wider than a byte has to be split into individual bytes to live in memory cause memory is addressed one byte at a time.

| size of value | decimal value | hex value            | big endian bytes (NOT x86) | little endian bytes (x86) |
| ------------- | ------------- | -------------------- | -------------------------- | ------------------------- |
| 8 (1 byte)    | `65`          | `0x41`               | `41`                       | `41`                      |
| 16 (2 bytes)  | `4660`        | `0x1234`             | `12 34`                    | `34 12`                   |
| 32 (4 bytes)  | `1145258561`  | `0x44434241`         | `44 43 42 41`              | `41 42 43 44`             |
| 64 (8 bytes)  | `1145258561`  | `0x0000000044434241` | `00 00 00 00 44 43 42 41`  | `41 42 43 44 00 00 00 00` |

say `mov rax, 0x1234; push rax` written to memory and read straight back (e.g., `pop rax`) comes out unchanged: it's written in little-endian order on the stack by `push` and endian-corrected when it's read back into the register by `pop`. 

Endianness only matters _in memory_, but the moment you look at memory _as bytes_ (in a hex dump, in a debugger, in an exploit) the byte order is right there, and you have to read it the way the CPU wrote it.

The easiest place to get turned around is the boundary between memory order and the value printed from a register. Suppose `rdi` points at these eight bytes:

```text
Address    Byte
[rdi+0]    41
[rdi+1]    42
[rdi+2]    43
[rdi+3]    44
[rdi+4]    45
[rdi+5]    46
[rdi+6]    47
[rdi+7]    48
```
```asm
mov rax, [rdi]

memory address order: 41 42 43 44 45 46 47 48
register hex order: 48 47 46 45 44 43 42 41
rax value: 0x4847464544434241
```

The bytes did not move in memory. The CPU interpreted the byte at the lowest address as the least-significant part of the number.

|Name|Bits|Bytes|Partial `rax` Access|Memory Access|
|---|---|---|---|---|
|byte|8|1|`mov al, [rdi]`|`mov BYTE PTR [rdi], 0x11`|
|word|16|2|`mov ax, [rdi]`|`mov WORD PTR [rdi], 0x1122`|
|doubleword (`dword`)|32|4|`mov eax, [rdi]`|`mov DWORD PTR [rdi], 0x11223344`|
|quadword (`qword`)|64|8|`mov rax, [rdi]`|`mov QWORD PTR [rdi], 0x1122334455667788`|

 `al` is the low byte of `rax`, `ax` the low 2 bytes, `eax` the low 4, and `rax` all 8. The size names are just another way of saying the same thing --- `al` holds a byte, `eax` holds a dword, `rax` holds a qword.

### Sign extension

It copies the sign bit, not zeroes, into the new high bit
```asm
movsx rax, BYTE PTR [rdi]
```

This reads one byte from the address in `rdi`, treats that byte as signed, and returns the 64-bit signed value in `rax`.

### Little Endian Bytes

movabs - absolute
a normal `mov`'s immediate maxes out at 32 bits, so the assembler uses this wider form "move absolute" --- when the constant fills all 64.
```asm
movabs rbx, 0x4847464544434241
mov rax, [rdi]
cmp rax, rbx 
jne fail
```

### Struct
```asm
movabs rbx, 0x................ ; a: 8-byte field at +0 
mov rax, [rdi+0] 
cmp rax, rbx 
mov eax, [rdi+8] ; b: 4-byte field at +8 
cmp eax, 0x........ 
mov ax, [rdi+12] ; c: 2-byte field at +12 
cmp ax, 0x.... 
mov al, [rdi+14] ; d: 1-byte field at +14 
cmp al, 0x.. 
mov al, [rdi+15] ; e: 1-byte field at +15 
cmp al, 0x..
```

A program can check a struct's fields in _any_ order it likes , the order the compares appear in the code has nothing to do with where those bytes live in memory.

```asm
mov ax, [rdi+12] ; the +12 word might be checked first... 
cmp ax, 0x.... 
mov al, [rdi+15] ; ...then a byte from the very end... 
cmp al, 0x.. 
movabs rbx, 0x................ ; ...then the +0 qword, and so on. 
mov rax, [rdi+0] 
cmp rax, rbx
```

