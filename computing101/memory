## 1. Core Idea

Memory stores data at addresses.

A register can hold:

- A normal value
- An address (a pointer)

---

## 2. Important Intel Syntax

```asm
mov rdi, 42
```

Put the literal value `42` into `rdi`.

```asm
mov rdi, [123400]
```

Read the value stored at memory address `123400`.

```asm
mov rdi, [rax]
```

Treat `rax` as an address and read memory from there.

---

## 3. Biggest Concept: Brackets Mean Dereference

- **No brackets** → use the value directly.
- **Brackets** → go to memory at that address and load what's there.

### Example

Suppose:

```text
rax = 133700
memory[133700] = 123400
```

Then:

```asm
mov rdi, rax
```

Result:

```text
rdi = 133700
```

```asm
mov rdi, [rax]
```

Result:

```text
rdi = 123400
```

---

## 4. What I Learned Through the Dojo

### a. Reading from a Known Address

If given a fixed address:

```asm
mov rdi, [address]
```

### b. Reading from an Address Stored in a Register

If a register already contains the address:

```asm
mov rdi, [rax]
```

### c. Pointers

Sometimes memory does not hold the final value.

Instead, it holds another address.

```text
addr2 -> addr1 -> secret
```

### d. Double Dereference

If `rax` points to a location that contains another address, you need two memory reads:

```asm
mov rdi, [rax]
mov rdi, [rdi]
```

---

## 5. Exiting with the Value

Linux syscall `exit`:

```asm
mov rax, 60
syscall
```

- `60` is the syscall number for `exit`
- `rdi` holds the exit status

Typical goal:

```asm
put secret into rdi

mov rax, 60
syscall
```



---

## 6. Good Mental Rule

Always ask:

> Does this register hold the value I want, or does it hold the address of the value I want?

If it holds the value:

```asm
mov reg, other_reg
```

If it holds the address:

```asm
mov reg, [other_reg]
```

---
## 7. Final Level Mental Model

Suppose:

```text
rax contains SECRET_LOCATION_2

memory[SECRET_LOCATION_2] = SECRET_LOCATION_1

memory[SECRET_LOCATION_1] = SECRET_VALUE
```

Flow:

```text
rax
 ↓
SECRET_LOCATION_2
 ↓
SECRET_LOCATION_1
 ↓
SECRET_VALUE
```

Or:

```text
rax → memory → pointer → memory → value
```
--- 

