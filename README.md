# 🧠 IDA Pro Cheat Sheet for Static Analysis

A concise reference for reverse engineering with IDA Pro.

---

## 📁 Navigation Shortcuts

| Action                      | Shortcut       |
|----------------------------|----------------|
| **Strings window**         | `Shift + F12`  |
| **Functions window**       | `Shift + F4`   |
| **Imports window**         | `Alt + F7`     |
| **Exports window**         | `Ctrl + E`     |
| **Jump to entry point**    | `Ctrl + E`     |
| **Rename label/function**  | `N`            |
| **Add comment**            | `;`            |
| **Hex/Text toggle**        | `H`            |
| **Follow call/jump**       | `Enter`        |
| **Go back**                | `Esc` / `Backspace` |
| **Graph view**             | `Space`        |
| **Hex view**               | `Tab`          |

---

## 🧭 Control Flow Arrow Colors

| Color      | Meaning                         | Instruction Types                            | Condition          |
|------------|----------------------------------|-----------------------------------------------|--------------------|
| 🔴 Red     | Unconditional jump               | `jmp`, `call`, `retn`                         | Always taken       |
| 🟢 Green   | Conditional true branch          | `je`, `jne`, `jg`, `jl`, `jz`, `jnz`, etc.    | If condition met   |
| 🔵 Blue    | Conditional false branch (fall)  | Same as above                                 | If condition fails |
| ⚫ Gray    | Linear/default flow               | Any normal instruction                        | Sequential flow    |

---

## 🔧 Common Assembly Instructions (x86/x64)

### 📦 Data Movement

| Instruction | Description                    | Example              |
|-------------|--------------------------------|----------------------|
| `mov`       | Copy value                     | `mov eax, 1`         |
| `lea`       | Load address                   | `lea eax, [ebx+4]`   |
| `push/pop`  | Stack operations               | `push ebp`, `pop eax`|
| `xchg`      | Swap values                    | `xchg eax, ebx`      |

### ➕ Arithmetic & Bitwise

| Instruction | Description                    | Example              |
|-------------|--------------------------------|----------------------|
| `add/sub`   | Addition/Subtraction           | `add eax, 1`         |
| `inc/dec`   | Increment/Decrement            | `inc eax`            |
| `imul/idiv` | Multiply/Divide (signed)       | `imul eax, ebx, 4`   |
| `and/or/xor`| Bitwise operations             | `xor eax, eax`       |
| `neg`       | Negate                         | `neg eax`            |

### 🔍 Comparisons & Conditions

| Instruction | Description                    | Example              |
|-------------|--------------------------------|----------------------|
| `cmp`       | Compare two values             | `cmp eax, 0`         |
| `test`      | Bitwise AND, sets flags        | `test eax, eax`      |

### 🔀 Jumps & Calls

| Instruction | Description                    | Example              |
|-------------|--------------------------------|----------------------|
| `jmp`       | Unconditional jump             | `jmp loc_401000`     |
| `call`      | Function call                  | `call printf`        |
| `ret`       | Return from function           | `ret`                |
| `je/jne`    | Jump if (not) equal            | `je loc_401020`      |
| `jg/jl`     | Jump if greater/less (signed)  | `jg loc_401030`      |
| `ja/jb`     | Jump if above/below (unsigned) | `ja loc_401040`      |

---

## 🔁 How Loops Appear in IDA (Graph View Style)

### While Loop
```asm
loc_401000:
  cmp eax, 5
  jge loc_401020    ; 🟢 jump if done
  inc eax
  jmp loc_401000    ; 🔴 unconditional loop
loc_401020:
