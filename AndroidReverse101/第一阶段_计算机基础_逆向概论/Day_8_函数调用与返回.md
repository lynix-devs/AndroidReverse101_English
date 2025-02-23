# **📜 Day 8: 函数调用与返回（ARM32 & ARM64）**

## **📌 学习目标**
✅ **理解 ARM32 和 ARM64 体系架构下的函数调用与返回机制**。  
✅ **掌握 ARM 函数调用的参数传递方式**（寄存器 & 栈）。  
✅ **学习如何在汇编中调用和返回函数**，并与 C 语言进行交互。  
✅ **掌握 ARM 过程调用约定（AAPCS）**，理解 `BL`, `BX`, `RET` 指令的作用。  
✅ **通过调试和反汇编分析 ARM 汇编中的函数调用方式**。

---

# **1️⃣ 函数调用的基本原理**
### **🔹 什么是函数调用？**
函数调用（Function Call）是程序执行过程中 **跳转到另一个代码段执行特定任务，并在执行完毕后返回** 的过程。  
在 ARM 体系结构中，函数调用主要涉及：
1. **传递参数（Registers & Stack）**
2. **保存返回地址（Link Register, LR）**
3. **执行函数代码**
4. **返回到调用方**

---

# **2️⃣ ARM32 vs. ARM64 函数调用约定**
### **🔹 ARM32（AAPCS 约定）**
在 ARM32（AArch32）下：
- **前 4 个参数存放在 R0-R3 寄存器**，剩余参数存放在栈中。
- **返回值存放在 R0（整数）或 R0-R1（64 位整数 / 浮点数）**。
- **调用函数前，`BL` 指令会自动存储返回地址到 `LR（R14）`**。

### **🔹 ARM64（AAPCS 约定）**
在 ARM64（AArch64）下：
- **前 8 个参数存放在 X0-X7 寄存器**，其余参数存入栈中。
- **返回值存放在 X0（整数）或 X0-X1（64 位整数 / 浮点数）**。
- **调用函数时，`BL` 指令会自动存储返回地址到 `X30（LR）`**。

---

# **3️⃣ 函数调用指令**
| **指令** | **作用** | **ARM32 示例** | **ARM64 示例** | **等价 C 代码** |
|--------|------|-------------|-------------|-------------|
| `BL`  | 调用子函数 | `BL func` | `BL func` | `func();` |
| `BX`  | 返回调用者 | `BX LR` | N/A | `return;` |
| `RET` | 返回调用者 | N/A | `RET` | `return;` |

---

# **4️⃣ ARM32 & ARM64 函数调用示例**
### **🔹 ARM32 示例**
```assembly
.global _start

_start:
    MOV R0, #5      ; 传递参数 a = 5
    MOV R1, #3      ; 传递参数 b = 3
    BL add_numbers  ; 调用 add_numbers(R0, R1)
    B _start        ; 无限循环

add_numbers:
    ADD R0, R0, R1  ; R0 = R0 + R1
    BX LR           ; 返回调用者
```

**等价 C 代码**：
```c
int add_numbers(int a, int b) {
    return a + b;
}

int main() {
    int result = add_numbers(5, 3);
    while (1);
}
```

---

### **🔹 ARM64 示例**
```assembly
.global _start

_start:
    MOV X0, #5      ; 传递参数 a = 5
    MOV X1, #3      ; 传递参数 b = 3
    BL add_numbers  ; 调用 add_numbers(X0, X1)
    B _start        ; 无限循环

add_numbers:
    ADD X0, X0, X1  ; X0 = X0 + X1
    RET             ; 返回调用者
```

**等价 C 代码**：
```c
long add_numbers(long a, long b) {
    return a + b;
}

int main() {
    long result = add_numbers(5, 3);
    while (1);
}
```

---

# **5️⃣ 复杂函数调用示例**
### **🔹 传递多个参数**
ARM32：
```assembly
MOV R0, #5
MOV R1, #3
MOV R2, #2
MOV R3, #4
BL complex_func
```

ARM64：
```assembly
MOV X0, #5
MOV X1, #3
MOV X2, #2
MOV X3, #4
BL complex_func
```

等价 C 代码：
```c
int complex_func(int a, int b, int c, int d);
complex_func(5, 3, 2, 4);
```

---

### **🔹 递归函数调用（阶乘）**
**ARM32**
```assembly
.global factorial

factorial:
    CMP R0, #1
    BLE end_factorial
    PUSH {R0, LR}
    SUB R0, R0, #1
    BL factorial
    POP {R1, LR}
    MUL R0, R0, R1
end_factorial:
    BX LR
```

**ARM64**
```assembly
.global factorial

factorial:
    CMP X0, #1
    BLE end_factorial
    STP X0, LR, [SP, #-16]!
    SUB X0, X0, #1
    BL factorial
    LDP X1, LR, [SP], #16
    MUL X0, X0, X1
end_factorial:
    RET
```

等价 C 代码：
```c
int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}
```

---

# **6️⃣ 栈管理**
**ARM32**
```assembly
PUSH {R4, LR}   ; 保存寄存器
POP {R4, PC}    ; 还原并返回
```

**ARM64**
```assembly
STP X29, X30, [SP, #-16]!  ; 保护 X29, X30
LDP X29, X30, [SP], #16    ; 恢复 X29, X30
```

---

# **🛠 实战任务**
### **✅ 1. 编写并运行 ARM 汇编**
```bash
as -o arm.o arm.s
ld -o arm arm.o
./arm
```

### **✅ 2. 反编译 ARM ELF**
```bash
objdump -d arm_binary | head -n 20
```

### **✅ 3. 逆向分析 Android APK**
```bash
apktool d app.apk -o output
vim output/smali/com/example/Main.smali
```

---

# **📚 参考资料**
📌 **ARM 官方文档**
- `ARMv7 指令集`：[https://developer.arm.com/documentation](https://developer.arm.com/documentation)  
- `ARMv8 (AArch64) 指令集`：[https://developer.arm.com/documentation](https://developer.arm.com/documentation)  

📌 **逆向工程**
- `Ghidra`：[https://ghidra-sre.org/](https://ghidra-sre.org/)  
- `IDA Pro`：[https://hex-rays.com/](https://hex-rays.com/)  

---

🔥 **任务完成后，你将掌握：**  
✅ **ARM32 & ARM64 函数调用和返回机制**  
✅ **如何在 ARM 平台编写和调试函数调用**  
✅ **ARM 汇编在逆向工程中的实际应用**  

🚀 **下一步（Day 9）**：**Android CPU 架构解析！** 🎯