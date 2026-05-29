# ARM Patching Cheatsheet (ARM32, ARM64 & THUMB)

This document is a compilation of useful Hex codes for modding/patching Unity IL2CPP games on both **ARM32 (AArch32)** and **ARM64 (AArch64)** architectures.

---

## 💡 Understanding Return Registers

Before patching a function, you must identify its return data type to target the correct return register:

| Data Type (C#) | ARM32 Register | ARM64 Register | Description |
| :--- | :---: | :---: | :--- |
| `bool`, `int`, `uint`, `short`, `char` | `R0` | `W0` (32-bit) | Lower 32-bit integer register. |
| `long`, `ulong` | `R0` (lower), `R1` (upper) | `X0` (64-bit) | 64-bit general-purpose integer register. |
| `float` | `R0` (or `S0` depending on ABI) | `S0` | Single-precision floating-point register. |
| `double` | `R0`, `R1` (or `D0`) | `D0` | Double-precision floating-point register. |

> [!IMPORTANT]
> - In **ARM32**, the return instruction is `bx lr` (Hex: `1E FF 2F E1`).
> - In **ARM64**, the return instruction is `ret` (Hex: `C0 03 5F D6`).

---

## 🚀 ARM64 (AArch64) Cheatsheet

### 1. Booleans (True / False)
Commonly used to patch check functions like `get_IsPremium`, `get_IsDoubleDamage`, `CheckLicense`, etc.

| Mod Target | Assembly | Hex Code (Little-endian) | Explanation |
| :--- | :--- | :--- | :--- |
| **Return TRUE** | `mov x0, #1`<br>`ret` | `20 00 80 D2 C0 03 5F D6` | Loads `1` (True) into the return register and returns. |
| **Return FALSE** | `mov x0, #0`<br>`ret` | `00 00 80 D2 C0 03 5F D6` | Loads `0` (False) into the return register and returns. |

### 2. 32-bit Integers (`int` / `uint`)
Used to mod integer properties like `GetHp`, `get_Level`, `get_Coins`, `get_Gem`...

| Mod Target | Assembly | Hex Code (Little-endian) | Mod Effect |
| :--- | :--- | :--- | :--- |
| **Return Max Int** | `mov w0, #0x7fffffff`<br>`ret` | `00 00 B0 12 C0 03 5F D6` | Returns `2147483647` (Max Int). Perfect for God Mode or Unlimited Coins. |
| **Return High Value** | `mov w0, #0x7f000000`<br>`ret` | `00 E0 A7 52 C0 03 5F D6` | Returns `2130706432`. Prevents integer overflow crashes in some games. |
| **Return 0** | `mov w0, #0`<br>`ret` | `00 00 80 D2 C0 03 5F D6` | Returns `0`. Useful for lowering item upgrade prices to zero. |
| **Return Negative** | `mov w0, #-9999`<br>`ret` | `C0 E1 84 12 C0 03 5F D6` | Returns `-9999`. Useful for testing currency bypass/negative value bugs. |

### 3. Floating-Point (`float`)
Used for stats such as speed (`get_Speed`), damage (`get_Damage`), and HP float properties.

| Mod Target | Assembly | Hex Code (Little-endian) | Mod Effect |
| :--- | :--- | :--- | :--- |
| **Return Positive Infinity** | `movz w0, #0x7f80, lsl #16`<br>`fmov s0, w0`<br>`ret` | `00 F0 A7 52 00 00 27 1E C0 03 5F D6` | Returns **Float Positive Infinity**. Complete invincibility / God Mode. |
| **Return Big Float** | `movz w0, #0x8000`<br>`movk w0, #0x72ae, lsl #16`<br>`fmov s0, w0`<br>`ret` | `60 00 80 52 A0 93 AE 72 00 00 27 1E C0 03 5F D6` | Returns approx. $9.95 \times 10^{31}$. One-hit damage or super speed. |
| **Return 1000.0** | `movz w0, #0xa8fa`<br>`fmov s0, w0`<br>`ret` | `40 8F A8 52 00 00 27 1E C0 03 5F D6` | Returns `1000.0`. Ideal for custom attack speed. |
| **Return 100.0** | `movz w0, #0xa859`<br>`fmov s0, w0`<br>`ret` | `00 59 A8 52 00 00 27 1E C0 03 5F D6` | Returns `100.0`. Useful for forcing 100% Critical Rate. |
| **Return 1.0** | `fmov s0, #1.0`<br>`ret` | `00 10 2E 1E C0 03 5F D6` | Returns `1.0`. Useful for scale multipliers or `deltaTime`. |
| **Return 0.0** | `fmov s0, #0.0`<br>`ret` | `00 00 20 1E C0 03 5F D6` | Returns `0.0`. Used to eliminate cooldown times (`get_Cooldown`). |

### 4. Double-Precision (`double`)
Often found in Idle games or incremental strategy games that require double-precision arithmetic.

| Mod Target | Assembly | Hex Code (Little-endian) | Mod Effect |
| :--- | :--- | :--- | :--- |
| **Return Double Infinity** | `movz x0, #0xefe0, lsl #16`<br>`fmov d0, x0`<br>`ret` | `00 E0 EF D2 00 00 67 9E C0 03 5F D6` | Returns **Double Positive Infinity**. Absolute god mode or infinite resources. |
| **Return 1e+100** | `movz x0, #0xea96`<br>`fmov d0, x0`<br>`ret` | `00 96 EA D2 00 00 67 9E C0 03 5F D6` | Returns $10^{100}$. Extremely high damage / gold multiplier. |
| **Return 1e+50** | `movz x0, #0xe94a`<br>`fmov d0, x0`<br>`ret` | `00 4A E9 D2 00 00 67 9E C0 03 5F D6` | Returns $10^{50}$ (1-Hit Kill). |
| **Return 1000.0** | `movz x0, #0xe812`<br>`fmov d0, x0`<br>`ret` | `00 12 E8 D2 C0 03 5F D6` | Returns `1000.0`. |
| **Return 1.0** | `fmov d0, #1.0`<br>`ret` | `00 10 6E 1E C0 03 5F D6` | Returns `1.0`. |
| **Return 0.0** | `fmov d0, #0.0`<br>`ret` | `00 E4 00 2F C0 03 5F D6` | Returns `0.0`. |

### 5. 64-bit Integers (`long` / `int64`)
Used for large numbers exceeding 32-bit limits, such as `long Gold` or `long Exp`.

| Mod Target | Assembly | Hex Code (Little-endian) | Mod Effect |
| :--- | :--- | :--- | :--- |
| **Return 999999999** | `movz x0, #0x99e0, lsl #16`<br>`movk x0, #0x73a7`<br>`ret` | `E0 3F 99 D2 40 73 A7 F2 C0 03 5F D6` | Returns `999,999,999` (nearly 1 Billion). |
| **Return 0** | `mov x0, #0`<br>`ret` | `00 00 80 D2 C0 03 5F D6` | Returns `0`. Resets wait timers or upgrade costs. |

### 6. Function Bypassing & NOP
Commonly used on `void` methods (`public void Update()`, `public void InitAntiCheat()`) to disable unwanted security modules or bypass triggers.

* **Delete Function / Return Empty (Void Methods)**
  ```assembly
  ret
  ```
  * **Hex:** `C0 03 5F D6`
  * **Mod Effect:** Immediately exits the method upon entry. Great for disabling ads, loggers, or basic anti-cheat functions.

* **NOP (No Operation) - Single Instruction**
  ```assembly
  nop
  ```
  * **Hex:** `1F 20 03 D5`
  * **Mod Effect:** Does nothing and moves to the next instruction. Used to override branch checks (e.g. `B.EQ`, `B.NE`) so the execution flows straight through.

* **NOP & Ret (Erase Function Body)**
  * **Hex:** `1F 20 03 D5 C0 03 5F D6`
  * **Mod Effect:** Replaces the first instruction with NOP and returns immediately.

* **Bypass with Parameters (Skip Code in Parameters)**
  When bypassing a function that has parameters while preserving stack alignment:
  * **Hex Template:** `E0 03 00 AA C0 03 5F D6`
  * **Usage:** Replace the byte `00` (at the third position) with the hex representation of the number of parameters the function accepts (e.g., for a function with 7 parameters $\rightarrow$ replace with `07` $\rightarrow$ `E0 03 07 AA C0 03 5F D6`).

---

## 📱 ARM32 (AArch32) Cheatsheet

### 1. Booleans (True / False)

| Mod Target | Assembly | Hex Code (Little-endian) |
| :--- | :--- | :--- |
| **Return TRUE** | `mov r0, #1`<br>`bx lr` | `01 00 A0 E3 1E FF 2F E1` |
| **Return FALSE** | `mov r0, #0`<br>`bx lr` | `00 00 A0 E3 1E FF 2F E1` |

### 2. Integers (`int`)

| Mod Target | Assembly | Hex Code (Little-endian) | Mod Effect |
| :--- | :--- | :--- | :--- |
| **Return Max Int** | `movt r0, #0x7fff`<br>`bx lr` | `FF 0F 47 E3 1E FF 2F E1` | Returns `2147483647` (Max Int). |
| **Return 1000** | `mov r0, #1000`<br>`bx lr` | `FA 0F A0 E3 1E FF 2F E1` | Returns `1000`. |
| **Return 0** | `mov r0, #0`<br>`bx lr` | `00 00 A0 E3 1E FF 2F E1` | Returns `0`. |
| **Return Negative** | `mov r0, #-1`<br>`bx lr` | `00 00 48 E3 1E FF 2F E1` | Returns `-1`. |

### 3. Floating-Point (`float`)

| Mod Target | Assembly | Hex Code (Little-endian) | Mod Effect |
| :--- | :--- | :--- | :--- |
| **Return Float Infinity** | `movt r0, #0x7f80`<br>`bx lr` | `80 0F 47 E3 1E FF 2F E1` | Returns **Float Positive Infinity** (God Mode). |
| **Return Float Negative Infinity** | `movt r0, #0xff80`<br>`bx lr` | `7F 0F 4F E3 1E FF 2F E1` | Returns **Float Negative Infinity**. |
| **Return Big Float** | `movt r0, #0x7444`<br>`bx lr` | `9D 04 47 E3 1E FF 2F E1` | Returns approx. $9.95 \times 10^{31}$. |
| **Return 1000.0** | `movt r0, #0x447a`<br>`bx lr` | `7A 04 44 E3 1E FF 2F E1` | Returns `1000.0` (Float). |
| **Return 100.0** | `movt r0, #0x42c8`<br>`bx lr` | `C8 02 44 E3 1E FF 2F E1` | Returns `100.0` (100% Crit Rate). |
| **Return 20.0** | `movt r0, #0x41a0`<br>`bx lr` | `A0 01 44 E3 1E FF 2F E1` | Returns `20.0`. |
| **Return 10.0** | `movt r0, #0x4120`<br>`bx lr` | `20 01 44 E3 1E FF 2F E1` | Returns `10.0`. |
| **Return 1.0** | `movt r0, #0x3f80`<br>`bx lr` | `80 0F 43 E3 1E FF 2F E1` | Returns `1.0`. |

### 4. Double-Precision (`double`)

| Mod Target | Assembly | Hex Code (Little-endian) | Mod Effect |
| :--- | :--- | :--- | :--- |
| **Return 1e+100** | `movt r0, #0x4514`<br>`bx lr` | `B2 14 45 E3 1E FF 2F E1` | Returns $9.84 \times 10^{99}$. |
| **Return 1e+50** | `movt r0, #0x441a`<br>`bx lr` | `51 1A 44 E3 1E FF 2F E1` | Returns $10^{50}$ (1-Hit Kill). |
| **Return 10.0** | `movt r0, #0x4024`<br>`bx lr` | `24 10 44 E3 1E FF 2F E1` | Returns `10.0`. |
| **Return 1.0** | `movt r0, #0x3ff0`<br>`bx lr` | `F0 1F 43 E3 1E FF 2F E1` | Returns `1.0`. |
| **Return 0.0** | `movt r0, #0x0000`<br>`bx lr` | `00 10 40 E3 1E FF 2F E1` | Returns `0.0`. |

### 5. 64-bit Integers (`long`)

| Mod Target | Assembly | Hex Code (Little-endian) | Mod Effect |
| :--- | :--- | :--- | :--- |
| **Return 999999999** | `movw r0, #0xca00`<br>`movt r0, #0x3b9a`<br>`mov r1, #0`<br>`bx lr` | `FF 09 0C E3 00 10 A0 E3 9A 0B 43 E3 1E FF 2F E1` | Returns `999,999,999` (using registers `R0` and `R1`). |
| **Return 999999** | `mov r0, #999999`<br>`mov r1, #0`<br>`bx lr` | `3F 02 04 E3 00 10 A0 E3 0F 00 40 E3 1E FF 2F E1` | Returns `999,999`. |
| **Return 9999** | `mov r0, #9999`<br>`mov r1, #0`<br>`bx lr` | `0F 07 02 E3 00 10 A0 E3 1E FF 2F E1` | Returns `9999`. |
| **Return 0** | `mov r0, #0`<br>`mov r1, #0`<br>`bx lr` | `00 00 A0 E3 00 10 A0 E3 1E FF 2F E1` | Returns `0` (long). |

### 6. Function Bypassing & NOP

* **Delete Function / Return Empty (Void Methods)**
  * **Hex:** `1E FF 2F E1`
  * **Mod Effect:** Instantly exits the function.

* **Bypass with Parameters (Skip Code in Parameters)**
  * **Hex Template:** `00 00 A0 E1 1E FF 2F E1`
  * **Usage:** Replace the first two bytes `00` with the number of parameters to bypass.

* **NOP (No Operation)**
  * **Hex:** `00 F0 20 E3`

---

## ⚡ THUMB Cheatsheet (16-bit)
Thumb instructions are compressed 16-bit instructions, commonly used in optimized ARM32 binaries.

| Mod Target | Assembly | Hex Code (Little-endian) | Mod Effect |
| :--- | :--- | :--- | :--- |
| **Return 1** | `movs r0, #1`<br>`bx lr` | `01 20 70 47` | Commonly used to return `true`. |
| **Return 9** | `movs r0, #9`<br>`bx lr` | `09 20 70 47` | Returns `9`. |
| **Return 99** | `movs r0, #99`<br>`bx lr` | `63 20 70 47` | Returns `99`. |
| **NOP** | `nop`<br>`bx lr` | `00 BF 70 47` | Skips operation and returns. |

---

## 🎯 Advanced Patching Tactics

### 1. Hooking & Modifying Class Fields (Offsets)
If you don't want to change a method's return value, but instead want to overwrite a class's member variable (`Field`) using its offset from `dump.cs`.

#### ❖ ARM32 Example:
* **Force Float Field to `1000.0` at offset `0x10`:**
  ```assembly
  MOV r1, #0x447A
  STR r1, [r0, #0x10]   // R0 holds the `this` pointer, offset is 0x10
  BX lr
  ```
  **Hex Code:** `7A 14 04 E3 10 10 80 E5 1E FF 2F E1`

* **Force Int Field to `1000` at offset `0x14`:**
  ```assembly
  MOV r1, #1000
  STR r1, [r0, #0x14]
  BX lr
  ```
  **Hex Code:** `FA 1F A0 E3 14 10 80 E5 1E FF 2F E1`

---

### 2. Disabling IEnumerator Coroutines (Bypass MoveNext)
Unity uses `IEnumerator` coroutines for background tasks (e.g., connection checks, heartbeats, anti-cheat updates). Patching `MoveNext()` to return `false` cleanly halts the coroutine without crashing the game.

* **Action:** Find the `public bool MoveNext() { }` method in the target IEnumerator class, and patch:
  
#### ❖ ARM32 Example:
```assembly
MOV r1, #0xFFFFFFFF
STR r1, [r0, #0x10]   // Adjusts the state machine state to finished
MOV r0, #0            // Returns false (stops MoveNext)
BX lr
```
**Hex Code:** `00 10 E0 E3 10 10 80 E5 00 00 A0 E3 1E FF 2F E1`
