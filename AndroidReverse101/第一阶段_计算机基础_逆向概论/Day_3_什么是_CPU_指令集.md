# **📜 Day 3: 什么是 CPU 指令集？**
  
## **📌 学习目标**
✅ 了解 **CPU 指令集** 的概念，及其在不同架构（x86、ARM、Smali）中的应用。  
✅ 掌握 **x86（CISC）vs. ARM（RISC）vs. Smali（Android DEX 指令）** 的不同特点。  
✅ 掌握 **ARMv7 (ARM32) vs. ARMv8 (ARM64)** 指令的区别。  
✅ 理解 **C 语言与汇编语言（x86、ARM、Smali）的转换关系**。  
✅ 通过实际代码示例，熟悉不同 CPU 指令集的基本用法。  

---

# **1️⃣ 什么是 CPU 指令集？**
**CPU 指令集（Instruction Set Architecture, ISA）** 是 CPU 运行的软件指令集合，决定了 CPU 如何解析和执行二进制代码。  

🔹 主要作用：
- 控制 **数据存取、运算、逻辑操作** 等计算任务。  
- 影响 CPU 的**性能、功耗、兼容性**。  
- 各大 CPU 制造商采用不同的 **指令架构**。  

🔹 **三种主要 CPU 指令集**
| **架构** | **特点** | **应用场景** |
|---------|--------|------------|
| **x86（CISC）** | 复杂指令集，可变长指令，支持强大的寻址模式 | PC、服务器（Intel、AMD） |
| **ARM（RISC）** | 精简指令集，固定指令长度，低功耗高性能 | 移动设备、嵌入式（高通、苹果、华为） |
| **Smali（DEX 字节码）** | Android DEX 虚拟机指令，面向 Java/ART | Android 逆向工程 |

---

# **2️⃣ x86 指令集**
**x86（CISC）** 是 **Intel 和 AMD** 推动的架构，主要用于 PC 和服务器。  
🔹 **特点：**
- **指令长度不固定**（1-15 字节），比 ARM 复杂。  
- **多模式寻址**，支持更复杂的操作。  

### **🔹 x86 汇编示例**
```assembly
section .text
global _start

_start:
    mov eax, 5      ; 把 5 赋值给 eax
    add eax, 10     ; eax = eax + 10
    sub eax, 2      ; eax = eax - 2
    mov ebx, eax    ; 把 eax 复制到 ebx
    int 0x80        ; 触发系统调用
```

🔹 **特点：**
- `mov eax, 5`：数据传输指令。  
- `add eax, 10`：加法运算。  
- `int 0x80`：Linux 下的系统调用。  

---

# **3️⃣ ARM 指令集**
**ARM（RISC）** 适用于移动设备，采用 **精简指令集（RISC）**。  

🔹 **ARM 特点**
- **指令长度固定**（4 字节）。  
- **寄存器数量多**（减少对内存访问，提高性能）。  
- **低功耗高效能**，适用于移动设备。  

### **🔹 ARM 指令 vs. x86 指令**
| **操作** | **x86 指令（CISC）** | **ARM 指令（RISC）** |
|---------|-----------------|----------------|
| 赋值 | `mov eax, 5` | `MOV R0, #5` |
| 加法 | `add eax, 10` | `ADD R0, R0, #10` |
| 读取内存 | `mov eax, [ebx]` | `LDR R0, [R1]` |
| 存储到内存 | `mov [ebx], eax` | `STR R0, [R1]` |

---

# **4️⃣ ARMv7 (32 位) vs. ARMv8 (64 位)**
🔹 **ARMv7（ARM32）**
```assembly
.global _start
_start:
    MOV R0, #5      ; 赋值 5 给 R0
    ADD R0, R0, #3  ; R0 = R0 + 3
    LDR R1, [R2]    ; 读取 R2 指向的内存到 R1
    STR R1, [R3]    ; 存储 R1 到 R3 指向的内存
    B _start        ; 无限循环
```

🔹 **ARMv8（ARM64）**
```assembly
.global _start
_start:
    MOV X0, #5      ; 赋值 5 给 X0
    ADD X0, X0, #3  ; X0 = X0 + 3
    LDR X1, [X2]    ; 读取 X2 指向的内存到 X1
    STR X1, [X3]    ; 存储 X1 到 X3 指向的内存
    B _start        ; 无限循环
```

📌 **区别：**
- ARMv7 使用 `R0-R15`，ARMv8 使用 `X0-X30`。  
- ARMv8 指令支持 64 位数据计算，提高计算能力。  

---

# **5️⃣ Smali 汇编语言（Android DEX 指令）**
**Smali 是 Android DEX 的汇编语言，相当于 Java 字节码的汇编版本。**  

### **🔹 Smali 代码示例**
```smali
.method public static sum(II)I
    .registers 3
    add-int v0, p0, p1
    return v0
.end method
```

🔹 **Smali 关键点**
| **指令** | **作用** |
|---------|--------|
| `add-int v0, p0, p1` | p0 + p1 结果存入 v0 |
| `return v0` | 返回计算结果 |

### **🔹 Smali 破解示例**
```smali
.method public isVip()Z
    .registers 2
    const/4 v0, 0x1  # 让所有用户变成 VIP
    return v0
.end method
```

---

# **6️⃣ C 语言 vs. 汇编（x86/ARM/Smali）**
### **🔹 C 代码**
```c
int sum(int a, int b) {
    return a + b;
}
```

### **🔹 x86 汇编**
```assembly
sum:
    mov eax, edi
    add eax, esi
    ret
```

### **🔹 ARM 汇编**
```assembly
sum:
    ADD R0, R0, R1
    BX LR
```

### **🔹 Smali 代码**
```smali
.method public static sum(II)I
    .registers 3
    add-int v0, p0, p1
    return v0
.end method
```

---

# **🛠 实战任务**
1️⃣ **检查 Android 设备的 CPU 架构**
```bash
cat /proc/cpuinfo
```

2️⃣ **运行 x86 汇编**
```bash
nasm -f elf64 test.asm
ld -o test test.o
./test
```

3️⃣ **反编译 Android APK**
```bash
apktool d myapp.apk -o output_dir
```

---

# **📚 参考资料**
📌 **ARM 指令集**
- ARM 官方文档：[https://developer.arm.com/documentation](https://developer.arm.com/documentation)  

📌 **x86 指令集**
- Intel 手册：[https://software.intel.com/en-us/articles/intel-sdm](https://software.intel.com/en-us/articles/intel-sdm)  

📌 **Smali 教程**
- Smali 指南：[https://github.com/JesusFreke/smali](https://github.com/JesusFreke/smali)  

---

🔥 **任务完成后，你将掌握：**  
✅ x86、ARM、Smali 指令集的区别。  
✅ ARM32 vs. ARM64 的演变。  
✅ 运行和修改 Smali 代码。  

🚀 **下一步（Day 4）**：**进制转换：为什么 16 进制很重要？** 🎯  