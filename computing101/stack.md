# Computing101 Notes: Stack

## What is the stack?
The **stack** is a region of memory used in **last-in, first-out (LIFO)** order.

- **push** adds a value to the top of the stack
- **pop** removes the top value from the stack
- The stack pointer register on x86-64 is **`rsp`**

---

## Key idea: `pop`
`pop rdi` does **two things**:

1. Reads the 8-byte value at address `[rsp]`
2. Stores it in `rdi`
3. Increases `rsp` by 8

Equivalent idea:

```asm
mov rdi, [rsp]
add rsp, 8
````

But `pop rdi` does both in one instruction.

---

## Stack at program start

When a Linux program starts, the stack begins with:

- `argc` = argument count
- followed by pointers to `argv`

So at startup:

- `[rsp]` contains `argc`

That means:

```asm
pop rdi
```

loads `argc` into `rdi`.

---

## Exiting with a syscall

On Linux x86-64:

- `rax = 60` means `exit`
- `rdi` holds the exit status

So this program exits with `argc`:

```asm
.intel_syntax noprefix
.global _start

_start:
    pop rdi
    mov rax, 60
    syscall
```

---

## Example

If the program is run as:

```bash
./prog a b c
```

then:

- `argc = 4`  
    (`./prog`, `a`, `b`, `c`)

So the program exits with status `4`.

Check with:

```bash
echo $?
```

---

## Important registers here

- `rsp` → top of the stack
- `rdi` → first syscall argument / exit code
- `rax` → syscall number

---

## Main takeaway

The stack is called a **stack** because values are accessed from the **top**, and instructions like `push` and `pop` naturally add/remove items in LIFO order.

For this exercise, the important pattern is:

- `argc` starts at the top of the stack
- `pop rdi` reads it
- `exit` uses `rdi` as the return code

---

## Quick summary

- Stack = LIFO memory structure
- `rsp` points to the top
- `pop` reads from `[rsp]` and moves `rsp`
- At program start, `[rsp] = argc`
- `mov rax, 60` + `syscall` performs `exit`
- Therefore, popping into `rdi` makes the program exit with the argument count



# Computing101 Study Guide: The Stack (`pop`, `rsp`, and `argc`)

## 1) Big idea

A **stack** is a **Last-In, First-Out (LIFO)** data structure.

Think of it like a stack of plates:

- You **add** a plate to the top
- You **remove** a plate from the top

In assembly:

- `push` = put a value on top of the stack
- `pop` = take a value off the top of the stack

---

## 2) The stack pointer: `rsp`

On x86-64 Linux, the register **`rsp`** points to the **top of the stack**.

### Visual idea

```text
High memory
+------------------+
|       ...        |
+------------------+
| next stack item  |
+------------------+
| top stack item   |  <-- rsp
+------------------+
Low memory

When you `pop`, the CPU:

1. Reads the value at `[rsp]`
2. Stores it in a register
3. Moves `rsp` forward by 8 bytes
```


## 3) What `pop rdi` does

```asm
pop rdi


This means:


mov rdi, [rsp]
add rsp, 8
```
```
```

### Before `pop rdi`

```text
Memory:
+-------------------------+
| [rsp] = top value       |  <-- rsp
+-------------------------+
| next value              |
+-------------------------+

Registers:
rdi = ?
rsp = address of top value
```

### After `pop rdi`

```text
Memory:
+-------------------------+
| old top value           |
+-------------------------+
| next value              |  <-- rsp
+-------------------------+

Registers:
rdi = old top value
rsp = rsp + 8
```

**Important:** the old value is still in memory, but the stack now considers it “removed” because `rsp` moved past it.

---

## 4) What is on the stack when a program starts?

At the beginning of a Linux program, the stack starts with:

- `argc` (argument count)
- then pointers to the argument strings (`argv`)

### Startup stack layout

```text
[rsp]      = argc
[rsp+8]    = argv[0]
[rsp+16]   = argv[1]
[rsp+24]   = argv[2]
...
```

So if your program begins with:

```asm
pop rdi
```

then `rdi` gets **`argc`**.

---

## 5) Example with command-line arguments

If you run:

```bash
./prog hello world
```

Then the arguments are:

- `argv[0] = "./prog"`
- `argv[1] = "hello"`
- `argv[2] = "world"`

So:

```text
argc = 3
```

### Stack view at startup

```text
+-------------------------+
| 3                       |  <-- rsp   (argc)
+-------------------------+
| pointer to "./prog"     |
+-------------------------+
| pointer to "hello"      |
+-------------------------+
| pointer to "world"      |
+-------------------------+
```

After:

```asm
pop rdi
```

you get:

```text
rdi = 3
rsp -> now points to argv[0]
```

---

## 6) Exiting a program with a syscall

In Linux x86-64:

- `rax = 60` means the `exit` syscall
- `rdi` holds the exit status

### Minimal program

```asm
.intel_syntax noprefix
.global _start

_start:
    pop rdi
    mov rax, 60
    syscall
```

### What it does

- Pops `argc` from the stack into `rdi`
- Calls `exit(argc)`

So the program exits with the number of command-line arguments.

---

## 7) Diagram of the full flow

### Program run

```bash
./prog a b c
```

### Step 1: initial stack

```text
+-------------------------+
| 4                       |  <-- rsp
+-------------------------+
| pointer to "./prog"     |
+-------------------------+
| pointer to "a"          |
+-------------------------+
| pointer to "b"          |
+-------------------------+
| pointer to "c"          |
+-------------------------+
```

### Step 2: `pop rdi`

```text
rdi = 4
rsp = rsp + 8
```

Now:

```text
+-------------------------+
| 4                       |
+-------------------------+
| pointer to "./prog"     |  <-- rsp
+-------------------------+
| pointer to "a"          |
+-------------------------+
| pointer to "b"          |
+-------------------------+
| pointer to "c"          |
+-------------------------+
```

### Step 3: exit syscall

```asm
mov rax, 60
syscall
```

This exits with status:

```text
4
```

---

## 8) Why is it called a stack?

Because values are used in **stack order**:

- the most recently added value is the first one removed
- access happens at the **top**
- `push` and `pop` naturally model this behavior

---

## 9) Key registers to remember

- `rsp` = stack pointer
- `rdi` = first argument to syscall/function
- `rax` = syscall number

For this example:

- `pop rdi` → gets `argc`
- `mov rax, 60` → selects `exit`
- `syscall` → exits using `rdi`

---

## 10) Quick memory aid

### Formula

```text
pop X = X <- [rsp], then rsp <- rsp + 8
```

### Startup fact

```text
[rsp] = argc
```

### Exit syscall fact

```text
rax = 60
rdi = exit code
```

---

## 11) Mini quiz

**Q1.** If you run `./prog`, what is `argc`?  
**A:** `1`

**Q2.** If you run `./prog x y`, what is `argc`?  
**A:** `3`

**Q3.** After `pop rdi`, what happens to `rsp`?  
**A:** It increases by 8

**Q4.** Why does the program exit with the argument count?  
**A:** Because `argc` is popped into `rdi`, and `exit` uses `rdi` as its status code

---

## 12) One-line summary

The stack is a LIFO memory structure, `rsp` points to its top, `pop rdi` reads `argc` from the top of the startup stack, and the program exits using that value as its return code.

```

If you want, I can also make this into a **1-page cheat sheet** or **lecture-slide style notes**.
```

- 
-
