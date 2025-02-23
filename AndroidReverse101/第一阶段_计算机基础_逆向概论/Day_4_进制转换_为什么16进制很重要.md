# **📜 Day 4: 进制转换 - 为什么 16 进制很重要？**

## **📌 学习目标**
✅ 掌握 **二进制、十进制、十六进制** 之间的转换方法和公式。  
✅ 了解 **为什么 16 进制在计算机、汇编语言和逆向工程中如此重要**。  
✅ 认识 **ELF 可执行文件的结构**，并学会如何分析 ELF 文件的内容。  
✅ 通过 **实战案例** 掌握 16 进制在 **内存地址、指令编码、文件结构** 以及 **漏洞分析** 中的作用。  

---

# **1️⃣ 进制转换概述**
### **🔹 为什么需要进制转换？**
计算机内部使用 **二进制** 存储数据，但二进制数据可读性较差，因此 **十六进制（Hex）** 被广泛用于表示地址、机器码等。

| **进制** | **基数** | **表示范围** | **示例** |
|--------|------|---------|---------|
| **二进制（Binary）** | 2 | `0, 1` | `101010` |
| **十进制（Decimal）** | 10 | `0-9` | `42` |
| **十六进制（Hexadecimal）** | 16 | `0-9, A-F` | `0x2A` |

---

# **2️⃣ 进制转换方法**
### **🔹 二进制 ↔ 十进制**
✅ **二进制转十进制**（`101010₂` → `42₁₀`）：  
**计算公式**：
\[
(1 × 2^5) + (0 × 2^4) + (1 × 2^3) + (0 × 2^2) + (1 × 2^1) + (0 × 2^0) = 42
\]

✅ **十进制转二进制**（`42₁₀` → `101010₂`）：  
**方法**：
1. `42 ÷ 2 = 21` 余 `0`
2. `21 ÷ 2 = 10` 余 `1`
3. `10 ÷ 2 = 5` 余 `0`
4. `5 ÷ 2 = 2` 余 `1`
5. `2 ÷ 2 = 1` 余 `0`
6. `1 ÷ 2 = 0` 余 `1`
**结果：`101010₂`**  

---

# **3️⃣ ELF 可执行文件**
### **🔹 什么是 ELF？**
**ELF（Executable and Linkable Format）** 是 Linux 及类 Unix 系统使用的 **可执行文件格式**，类似于 Windows 的 PE（Portable Executable）文件格式。

ELF 文件可以是：
- **可执行程序**（如 `/bin/ls`）
- **共享库（.so 文件）**
- **核心转储（core dump）**

### **🔹 ELF 文件的内容**
ELF 文件由多个 **段（Segment）** 和 **节（Section）** 组成，主要结构如下：
| **ELF 结构** | **功能** |
|-------------|------------------------------|
| **ELF 头（ELF Header）** | 存储 ELF 文件的基本信息，如魔数、入口地址、段偏移等 |
| **程序头表（Program Header）** | 描述如何加载文件到内存（仅在可执行文件中存在） |
| **节头表（Section Header）** | 描述各个节（Sections），如 `.text`, `.data`, `.rodata` |

---

# **4️⃣ ELF 文件结构解析**
我们可以使用 `hexdump` 命令查看 ELF 文件的 16 进制内容：
```bash
hexdump -C /bin/ls | head -n 10
```
示例输出：
```
00000000  7f 45 4c 46  02 01 01 00  00 00 00 00  00 00 00 00  | .ELF........... |
```
- `7F 45 4C 46` 是 **ELF 魔数（Magic Number）**，用于标识 ELF 文件。  

我们还可以使用 `readelf` 分析 ELF 结构：
```bash
readelf -h /bin/ls
```
示例输出：
```
ELF Header:
  Magic:   7f 45 4c 46
  Class:                             ELF64
  Data:                              2's complement, little endian
  Entry point address:               0x401000
```
- `Entry point address: 0x401000`  表示程序的入口地址。

---

# **5️⃣ 为什么 ELF 和 16 进制很重要？**
✅ **ELF 使用 16 进制存储指令**
所有 CPU 指令最终都会被转换为 **16 进制机器码**：
```assembly
mov eax, 5   ; x86 汇编
B8 05 00 00 00 ; 对应的十六进制机器码
```
```assembly
MOV R0, #5   ; ARM 汇编
E3A00005     ; 对应的十六进制机器码
```

✅ **ELF 可执行文件包含大量 16 进制数据**
例如，使用 `objdump` 查看 ELF 代码段：
```bash
objdump -d /bin/ls | head -n 20
```
示例输出：
```
0000000000401130 <_start>:
  401130:   48 83 ec 08    sub    rsp,0x8
  401134:   48 8b 05 dd 2e  mov    rax,QWORD PTR [rip+0x2edd]
```
这些都是 **ELF 可执行文件的指令代码**，是 **16 进制** 机器码的直接映射。

---

# **6️⃣ 实战任务**
### **✅ 1. 进制转换练习**
```python
# 二进制 -> 十进制
print(int("101010", 2))  # 输出：42

# 十进制 -> 十六进制
print(hex(42))  # 输出：0x2a

# 十六进制 -> 二进制
print(bin(0x2A))  # 输出：0b101010
```

### **✅ 2. 使用 `hexdump` 分析 ELF 文件**
```bash
hexdump -C /bin/ls | head -n 10
```

### **✅ 3. 用 `readelf` 查看 ELF 头**
```bash
readelf -h /bin/ls
```

### **✅ 4. 反编译 ELF 可执行文件**
```bash
objdump -d /bin/ls | head -n 20
```

---

# **📚 参考资料**
📌 **进制转换工具**
- 进制转换网站：[https://www.rapidtables.com/convert/number/](https://www.rapidtables.com/convert/number/)  

📌 **ELF 相关**
- `readelf` 手册：[https://man7.org/linux/man-pages/man1/readelf.1.html](https://man7.org/linux/man-pages/man1/readelf.1.html)  
- `objdump` 手册：[https://man7.org/linux/man-pages/man1/objdump.1.html](https://man7.org/linux/man-pages/man1/objdump.1.html)  

📌 **CPU 指令**
- `x86 指令手册`：[https://www.felixcloutier.com/x86/](https://www.felixcloutier.com/x86/)  
- `ARM 指令手册`：[https://developer.arm.com/documentation](https://developer.arm.com/documentation)  

---

🔥 **任务完成后，你将掌握：**  
✅ **二进制、十进制、十六进制的转换方法和公式。**  
✅ **ELF 文件的基本结构，并学会使用 `readelf` 和 `objdump` 进行分析。**  
✅ **利用 hexdump 查看 ELF 头部，并解析关键字段。**  

🚀 **下一步（Day 5）**：**汇编语言基础！** 🎯  