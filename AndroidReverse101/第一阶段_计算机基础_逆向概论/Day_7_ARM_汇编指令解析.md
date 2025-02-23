# **📜 Day 7: ARM 汇编指令解析（ARM32 & ARM64）**

## **📌 学习目标**
✅ **深入理解 ARM 指令集（ARM32 & ARM64）**，掌握各种常见指令的使用方式。  
✅ **学习 ARM 指令的分类**，包括 **数据处理、加载/存储、分支跳转、条件执行、系统调用**。  
✅ **对比 ARM 汇编与 C 语言**，并提供等价的 C 代码示例。  
✅ **编写和运行 ARM 汇编代码**，进行基本的调试和分析。  
✅ **理解 ARM 汇编指令在逆向工程和漏洞利用中的应用**。

---

# **1️⃣ ARM 汇编基础**
### **🔹 为什么学习 ARM 汇编？**
- **移动端主流架构**（Android、iOS、嵌入式设备）。  
- **低功耗高性能**，采用 **RISC 设计**，广泛用于服务器、智能手机等设备。  
- **逆向工程 & 安全研究必备技能**（如 APK 破解、调试分析等）。  

### **🔹 ARM 处理器架构**
| **架构** | **位数** | **寄存器数** | **指令长度** | **代表 CPU** |
|---------|------|-------|----------|-----------|
| ARMv7（ARM32） | 32 位 | 16 个（R0-R15） | 4 字节（Thumb 为 2 字节） | Cortex-A7, Cortex-A15 |
| ARMv8（ARM64） | 64 位 | 31 个（X0-X30） | 4 字节 | Cortex-A53, Cortex-A72 |

---

# **2️⃣ ARM 指令集分类**
### **✅ 1. 数据处理指令**
数据处理指令用于 **算术运算、逻辑运算、数据移动**。

| **指令** | **作用** | **ARM32 示例** | **ARM64 示例** | **等价 C 代码** |
|--------|------|-------------|-------------|-------------|
| `MOV`  | 赋值 | `MOV R0, #5` | `MOV X0, #5` | `int a = 5;` |
| `ADD`  | 加法 | `ADD R0, R0, #10` | `ADD X0, X0, #10` | `a = a + 10;` |
| `SUB`  | 减法 | `SUB R0, R0, #2` | `SUB X0, X0, #2` | `a = a - 2;` |
| `MUL`  | 乘法 | `MUL R0, R1, R2` | `MUL X0, X1, X2` | `a = b * c;` |
| `AND`  | 按位与 | `AND R0, R0, R1` | `AND X0, X0, X1` | `a = a & b;` |
| `ORR`  | 按位或 | `ORR R0, R0, R1` | `ORR X0, X0, X1` | `a = a | b;` |
| `EOR`  | 按位异或 | `EOR R0, R0, R1` | `EOR X0, X0, X1` | `a = a ^ b;` |
| `LSL`  | 左移 | `LSL R0, R1, #2` | `LSL X0, X1, #2` | `a = b << 2;` |
| `LSR`  | 右移 | `LSR R0, R1, #2` | `LSR X0, X1, #2` | `a = b >> 2;` |

**🔹 示例：**
```assembly
MOV R0, #10  ; R0 = 10
ADD R0, R0, #5  ; R0 = R0 + 5
SUB R1, R0, #2  ; R1 = R0 - 2
MUL R2, R0, R1  ; R2 = R0 * R1
```

等价的 **C 代码**：
```c
int a = 10;
a = a + 5;
int b = a - 2;
int c = a * b;
```

---

## **✅ 2. 加载 / 存储指令**
ARM 采用 **Load-Store 架构**，所有的内存访问必须使用 **LDR（加载）和 STR（存储）**。

| **指令** | **作用** | **ARM32 示例** | **ARM64 示例** | **等价 C 代码** |
|--------|------|-------------|-------------|-------------|
| `LDR`  | 读取内存 | `LDR R0, [R1]` | `LDR X0, [X1]` | `a = *ptr;` |
| `STR`  | 存储到内存 | `STR R0, [R1]` | `STR X0, [X1]` | `*ptr = a;` |

**🔹 示例：**
```assembly
LDR R0, =0x1000  ; 读取内存地址 0x1000
LDR R1, [R0]     ; 读取 R0 指向的内存值
STR R1, [R2]     ; 存储 R1 的值到 R2 指向的内存
```

等价的 **C 代码**：
```c
int *ptr = (int *)0x1000;
int a = *ptr;
ptr2 = &a;
*ptr2 = a;
```

---

## **✅ 3. 分支与控制流**
ARM 使用 **条件跳转（B, BL, BX）** 和 **循环（CMP + Bxx）** 来实现控制流。

| **指令** | **作用** | **ARM32 示例** | **ARM64 示例** | **等价 C 代码** |
|--------|------|-------------|-------------|-------------|
| `B`    | 无条件跳转 | `B loop` | `B loop` | `goto loop;` |
| `BL`   | 过程调用 | `BL function` | `BL function` | `function();` |
| `CMP`  | 比较 | `CMP R0, R1` | `CMP X0, X1` | `if (a == b) {...}` |
| `BEQ`  | 等于跳转 | `BEQ label` | `B.EQ label` | `if (a == b) goto label;` |
| `BNE`  | 不等跳转 | `BNE label` | `B.NE label` | `if (a != b) goto label;` |

**🔹 示例：**
```assembly
CMP R0, R1    ; 比较 R0 和 R1
BEQ equal     ; 如果相等，跳转到 equal
BNE not_equal ; 如果不相等，跳转到 not_equal
B loop        ; 继续循环
```

等价的 **C 代码**：
```c
if (a == b) {
    goto equal;
} else {
    goto not_equal;
}
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

🚀 **下一步（Day 8）**：**函数调用与返回！** 🎯