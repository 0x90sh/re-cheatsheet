# 🧠 IDA Pro Cheat Sheet for Static Analysis

A concise reference for reverse engineering with IDA Pro.

---

## 📁 Navigation Shortcuts

| Action                          | Shortcut       |
|--------------------------------|----------------|
| **Strings window**             | `Shift + F12`  |
| **Functions window**           | `Shift + F4`   |
| **Imports window**             | `Alt + F7`     |
| **Exports window**             | `Ctrl + E`     |
| **Jump to entry point**        | `Ctrl + E`     |
| **Jump to main() via list**    | `Shift + F4` → type `main` |
| **Set bookmark**               | `Alt + M`      |
| **Go to bookmark**             | `Ctrl + M`     |
| **Rename label/function**      | `N`            |
| **Add comment**                | `;`            |
| **Hex/Text toggle**            | `H`            |
| **Follow call/jump**           | `Enter`        |
| **Go back**                    | `Esc` / `Backspace` |
| **Graph view**                 | `Space`        |
| **Hex view**                   | `Tab`          |

---

## 🧽 Control Flow Arrow Colors

| Color           | Meaning                                    | Instruction Types                                          | Condition / Flow                              |
|-----------------|--------------------------------------------|------------------------------------------------------------|-----------------------------------------------|
| 🟢 Green        | Conditional jump (true branch)             | `je`, `jne`, `jg`, `jl`, `ja`, `jb`, `jz`, `jnz`, etc.     | Branch taken if condition is satisfied        |
| 🔴 Red          | Conditional jump (false branch)            | Same as above (the “not taken” path)                      | Branch taken if condition is **not** satisfied |
| 🔵 Blue         | Unconditional jump                         | `jmp`, (sometimes `call` / `ret`, depends on IDA settings) | Always taken (direct transfer of execution)   |
| ⚫ Gray / Black | Linear (default) flow / fall-through        | Any normal instruction (no jump)                          | Continues sequentially to the next instruction |

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
  jge loc_401020    ; 🧲 jump if done
  inc eax
  jmp loc_401000    ; 🔴 unconditional loop
loc_401020:
```

### For Loop
```asm
loc_401000:
  mov ecx, 0
loc_401003:
  cmp ecx, 10
  jge loc_401010    ; 🧲 exit condition
  ; loop body
  inc ecx
  jmp loc_401003    ; 🔴 loop back
loc_401010:
```

### Do-While Loop
```asm
loc_401000:
  ; loop body
  inc eax
  cmp eax, 5
  jl loc_401000     ; 🧲 backward jump if still looping
```

---

## 📊 Program Exit Paths

| Path        | What It Does                                 | Clean?                  |
|-------------|----------------------------------------------|--------------------------|
| `exit()`    | Ends process immediately                     | ❌ Skips CRT cleanup     |
| `_cexit()`  | Calls `atexit`, destructors, then returns    | ✅ Full CRT cleanup      |
| `retn` only | Manual stack cleanup, no CRT exit used       | ⚠️ Depends on context    |

---

## 🧬 What Happens Before `main()`

🔹 **Actual Entry Point ≠ `main()`**  
The OS starts your program at a function like `start:` or `WinMainCRTStartup`.  
This is **compiler-generated** setup code, not your own.

🔹 **CRT Startup Routine Tasks**  
Prepares the environment:
- Exception handling (`SetUnhandledExceptionFilter`)
- IO, heap, `stdin`/`stdout`
- `argv`, `envp`
- C++ global constructors

➡️ Then it calls your actual `main()` function.

---

## 🔢 Registers Overview (x86_64)

| Register  | Common Usage                           | Notes                                |
|-----------|----------------------------------------|--------------------------------------|
| `rax`     | Return value / accumulator             | Used for return from functions       |
| `rbx`     | Base pointer (callee-saved)            | Often used to preserve values        |
| `rcx`     | First integer argument (Windows x64)   | Also used in string operations       |
| `rdx`     | Second argument                        | Common in API calls                  |
| `rsi`     | Source index                           | Used in string/memory copy           |
| `rdi`     | Destination index / first argument     | Important in Windows APIs            |
| `rbp`     | Base pointer                           | Used to reference stack frames       |
| `rsp`     | Stack pointer                          | Always tracks current stack location |
| `r8`-`r9` | Additional function args                | Windows: 3rd and 4th param           |
| `r10`-`r15`| General purpose                        | Compiler- or manually-used registers |

### Notes:
- Registers may be used **temporarily**, **for arguments**, or **saved across calls**.
- Windows x64 calling convention uses: `rcx`, `rdx`, `r8`, `r9` for the first 4 arguments.
- `rsp` must always be 16-byte aligned before a `call` instruction.
- Callee-saved: `rbx`, `rbp`, `rsi`, `rdi`, `r12–r15`.
- Caller-saved (volatile): `rax`, `rcx`, `rdx`, `r8–r11`.
