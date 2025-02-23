# **📜 Day 15: 手写 ARM 汇编代码（实验）**

## **📌 学习目标**
✅ **学习 ARM32 & ARM64 汇编语法**，掌握基本指令集（MOV, ADD, SUB, LDR, STR, BL, CMP, B）。  
✅ **理解 ARM 寄存器结构**（通用寄存器、栈指针、程序计数器）。  
✅ **编写并运行 ARM 汇编代码**，使用 `as`（GNU Assembler）和 `ld` 进行汇编和链接。  
✅ **掌握 Linux 平台下 ARM 汇编的调用约定**，与 C 代码进行交互。  
✅ **调试 ARM 汇编程序**，使用 `gdb` 进行单步跟踪。  

---

# **1️⃣ ARM 汇编基础**
### **🔹 主要寄存器**
| **架构** | **通用寄存器** | **特殊寄存器** |
|---------|--------------|--------------|
| **ARM32** | R0 - R12 | SP (R13), LR (R14), PC (R15) |
| **ARM64** | X0 - X30 | SP, LR (X30), PC |

- `SP`（Stack Pointer）：栈指针
- `LR`（Link Register）：存储函数返回地址
- `PC`（Program Counter）：程序计数器

📌 **查看寄存器**
```bash
adb shell cat /proc/cpuinfo
```

---

# **2️⃣ 基本指令**
| **指令** | **作用** | **ARM32 示例** | **ARM64 示例** |
|--------|------|-------------|-------------|
| `MOV`  | 赋值 | `MOV R0, #5` | `MOV X0, #5` |
| `ADD`  | 加法 | `ADD R0, R0, #10` | `ADD X0, X0, #10` |
| `SUB`  | 减法 | `SUB R0, R0, #2` | `SUB X0, X0, #2` |
| `MUL`  | 乘法 | `MUL R0, R1, R2` | `MUL X0, X1, X2` |
| `LDR`  | 读取内存 | `LDR R0, [R1]` | `LDR X0, [X1]` |
| `STR`  | 存储内存 | `STR R0, [R1]` | `STR X0, [X1]` |
| `CMP`  | 比较 | `CMP R0, R1` | `CMP X0, X1` |
| `B`    | 无条件跳转 | `B label` | `B label` |
| `BL`   | 函数调用 | `BL func` | `BL func` |
| `RET`  | 返回 | `BX LR` | `RET` |

📌 **示例**
```assembly
MOV R0, #10
ADD R0, R0, #5
SUB R1, R0, #2
MUL R2, R0, R1
```

---

# **3️⃣ ARM 汇编代码示例**
### **✅ 1. Hello World (ARM32)**
```assembly
.global _start
.section .data
msg:    .asciz "Hello, ARM!\n"
len = . - msg

.section .text
_start:
    MOV R0, #1          @ 文件描述符 1（标准输出）
    LDR R1, =msg        @ 加载字符串地址
    LDR R2, =len        @ 加载字符串长度
    MOV R7, #4          @ 调用 write() 系统调用
    SWI 0               @ 触发系统调用

    MOV R7, #1          @ 调用 exit() 系统调用
    SWI 0
```
📌 **编译运行**
```bash
as -o hello.o hello.s
ld -o hello hello.o
./hello
```

---

### **✅ 2. 加法函数（ARM64）**
```assembly
.global add_numbers
add_numbers:
    ADD X0, X0, X1  @ X0 = X0 + X1
    RET             @ 返回
```
📌 **等价 C 代码**
```c
long add_numbers(long a, long b) {
    return a + b;
}
```

---

# **4️⃣ ARM 汇编调用 C 代码**
📌 **汇编代码（ARM64）**
```assembly
.global _start
_start:
    MOV X0, #5
    MOV X1, #3
    BL add_numbers
    B _start

.global add_numbers
add_numbers:
    ADD X0, X0, X1
    RET
```
📌 **等价 C 代码**
```c
long add_numbers(long a, long b) {
    return a + b;
}
```

📌 **编译运行**
```bash
as -o add.o add.s
ld -o add add.o
./add
```

---

# **5️⃣ 调试 ARM 汇编**
📌 **使用 GDB 调试**
```bash
gdb ./hello
```
📌 **常用调试命令**
```gdb
disassemble _start     # 反汇编
info registers         # 查看寄存器
break _start          # 设置断点
run                   # 运行程序
stepi                 # 单步执行
```

---

# **🛠 实战任务**
### **✅ 1. 编写并运行 Hello World**
```bash
as -o hello.o hello.s
ld -o hello hello.o
./hello
```
### **✅ 2. 编写并调用加法函数**
```bash
as -o add.o add.s
ld -o add add.o
./add
```
### **✅ 3. 调试 ARM 汇编**
```bash
gdb ./hello
```

---

# **📚 参考资料**
📌 **ARM 汇编指南**
- `ARM 官方文档`：[https://developer.arm.com/documentation](https://developer.arm.com/documentation)  
- `ARM 指令集`：[https://developer.arm.com/architectures/instruction-sets](https://developer.arm.com/architectures/instruction-sets)  

📌 **调试工具**
- `GDB 调试 ARM`：[https://sourceware.org/gdb/](https://sourceware.org/gdb/)  

---

🔥 **任务完成后，你将掌握：**  
✅ **如何编写 ARM 汇编代码并运行**  
✅ **如何与 C 代码交互**  
✅ **如何调试 ARM 汇编程序**  

🚀 **下一步（Day 16）**：**反汇编工具介绍！** 🎯