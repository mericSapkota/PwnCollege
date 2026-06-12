## Unsigned numbers

For **unsigned**, every bit contributes positively.

Range of an **n-bit unsigned** number:

- minimum: `0`
- maximum: `2^n - 1`

Examples:

- 8-bit unsigned: `0` to `255`
- 16-bit unsigned: `0` to `65535`
- 32-bit unsigned: `0` to `4294967295`

---

## Signed numbers: two’s complement

In pwn.college, signed integers use **two’s complement**.

### Rule

For an **n-bit signed** number:

- if the top bit (most significant bit) is **0**: the number is **positive**
- if the top bit is **1**: the number is **negative**

Range of an **n-bit signed** number:

- minimum: `-2^(n-1)`
- maximum: `2^(n-1) - 1`

Examples:

- 8-bit signed: `-128` to `127`
- 16-bit signed: `-32768` to `32767`
- 32-bit signed: `-2147483648` to `2147483647`

---

## 5) Fast way to read signed values

For an **n-bit** value:

- First compute the **unsigned** value.
- If top bit is `0`, signed value is the same.
- If top bit is `1`, then:

```text
signed = unsigned - 2^n
```

### 32-bit example

If unsigned value is:

```text
3046272335
```

then signed is:

```text
3046272335 - 4294967296 = -1248694961
```


---


##  Tiny cheat sheet

```text
Unsigned n-bit range: 0 to 2^n - 1
Signed n-bit range: -2^(n-1) to 2^(n-1) - 1

If MSB = 0:
    signed = unsigned

If MSB = 1:
    signed = unsigned - 2^n
```
## RULE (one line only)

|If number is...|Write binary of...|
|---|---|
|**0 or positive**|`number` (pad to 8 bits)|
|**Negative**|`number + 256` (pad to 8 bits)|

---

## QUICK EXAMPLES

|Number|Compute|Binary (8-bit)|
|---|---|---|
|42|42|`00101010`|
|0|0|`00000000`|
|-1|-1 + 256 = 255|`11111111`|
|-5|-5 + 256 = 251|`11111011`|
|-128|-128 + 256 = 128|`10000000`|


## 1) Binary ↔ Hex

Split binary into **two nibbles** (4 bits each):

- `1001 0100` → `9 4` → `0x94`
- `1100 0010` → `c 2` → `0xc2`

Nibble table:

|Binary|Hex|
|---|---|
|0000|0|
|0001|1|
|0010|2|
|0011|3|
|0100|4|
|0101|5|
|0110|6|
|0111|7|
|1000|8|
|1001|9|
|1010|a|
|1011|b|
|1100|c|
|1101|d|
|1110|e|
|1111|f|

