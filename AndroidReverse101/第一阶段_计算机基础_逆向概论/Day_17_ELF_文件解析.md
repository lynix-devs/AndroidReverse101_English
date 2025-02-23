# **📜 Day 17: ELF 文件解析**

## **📌 学习目标**
✅ **理解 ELF（Executable and Linkable Format）文件格式**，掌握其头部结构、节（Section）、程序头（Program Header）。  
✅ **掌握如何解析 ELF 文件**，使用 `readelf`、`objdump`、`hexdump` 等工具进行分析。  
✅ **学习如何从 ELF 文件提取符号表、函数地址、动态链接信息**，理解 SO 共享库的结构。  
✅ **掌握 ELF 逆向分析技术**，包括静态分析和动态调试（GDB, Frida Hook）。  
✅ **分析 Android 中的 ELF（`libnative.so`）文件，掌握 Hook 和劫持技巧**。  

---

# **1️⃣ ELF 文件基础**
### **🔹 什么是 ELF？**
ELF（Executable and Linkable Format）是一种 **Linux 和 Android** 下的可执行文件格式，包括：
- **可执行文件（Executable）**
- **共享库（Shared Object, .so）**
- **核心转储（Core Dump）**

📌 **查看 ELF 文件**
```bash
file libnative.so
```
示例输出：
```
libnative.so: ELF 64-bit LSB shared object, ARM aarch64
```

📌 **解析 ELF 结构**
```bash
readelf -h libnative.so
```

---

# **2️⃣ ELF 头部解析**
### **🔹 ELF 头部（ELF Header）**
ELF 头部包含 ELF 文件的基本信息，如 **架构、入口点、段表位置** 等。

📌 **查看 ELF 头**
```bash
readelf -h libnative.so
```
示例输出：
```
ELF Header:
  Magic:   7f 45 4c 46  ...
  Class:   ELF64
  Data:    2's complement, little endian
  Entry point address: 0x0000000000001234
```

📌 **关键字段**
| **字段** | **含义** |
|---------|---------|
| `Magic` | ELF 文件标识（7F 45 4C 46 -> .ELF） |
| `Class` | ELF32 / ELF64 |
| `Data` | 小端/大端存储 |
| `Entry point` | 程序入口地址 |

---

# **3️⃣ 程序头（Program Header）**
程序头（Program Header）定义了 **可执行文件的加载方式**，指定 **代码段、数据段、动态链接信息**。

📌 **查看程序头**
```bash
readelf -l libnative.so
```
示例输出：
```
Program Headers:
  Type           Offset   VirtAddr   PhysAddr   FileSiz  MemSiz Flags Align
  LOAD           0x000000 0x000000  0x000000  0x1234   0x2000 R E 0x1000
```

📌 **关键字段**
| **字段** | **含义** |
|---------|---------|
| `Type` | LOAD（可加载段）, DYNAMIC（动态段）, NOTE（调试信息） |
| `VirtAddr` | 代码段/数据段的虚拟地址 |
| `Flags` | R（读）, W（写）, X（执行） |

---

# **4️⃣ 节表（Section Header）**
节表（Section Header）描述了 ELF 文件的 **各个节（代码段、数据段、符号表等）**。

📌 **查看节表**
```bash
readelf -S libnative.so
```
示例输出：
```
Section Headers:
  [Nr] Name          Type         Addr      Off    Size   ES Flg Lk Inf Al
  [ 1] .text         PROGBITS     0x000010  0x0010 0x1000 00  AX  0   0 16
  [ 2] .data         PROGBITS     0x002000  0x2000 0x2000 00  WA  0   0 16
```

📌 **关键段**
| **节名** | **作用** |
|---------|---------|
| `.text` | 代码段（只读、可执行） |
| `.data` | 数据段（读写） |
| `.rodata` | 只读数据段 |
| `.bss` | 未初始化数据 |

---

# **5️⃣ 符号表（Symbol Table）**
符号表（Symbol Table）存储了 **函数名、变量名与地址映射关系**，用于调试和动态链接。

📌 **查看符号表**
```bash
readelf -s libnative.so | grep " func"
```
示例输出：
```
  1234: 00000000  123 FUNC  GLOBAL DEFAULT  UND printf
```

📌 **反查符号**
```bash
nm -D libnative.so
```

---

# **6️⃣ 动态链接信息**
📌 **查看共享库依赖**
```bash
ldd libnative.so
```
示例输出：
```
libc.so => /lib/libc.so
libm.so => /lib/libm.so
```

📌 **查看动态段**
```bash
readelf -d libnative.so
```

---

# **7️⃣ 逆向 ELF**
## **✅ 1. 解析 ELF 头**
```bash
readelf -h libnative.so
```
## **✅ 2. 解析符号表**
```bash
nm -D libnative.so
```
## **✅ 3. 反汇编 ELF**
```bash
objdump -d libnative.so | head -n 20
```
## **✅ 4. Hook ELF 运行时**
📌 **使用 Frida Hook `open()`**
```js
Java.perform(function() {
    var libc = Module.findExportByName(null, "open");
    Interceptor.attach(libc, {
        onEnter: function(args) {
            console.log("File Opened: " + Memory.readUtf8String(args[0]));
        }
    });
});
```

---

# **🛠 实战任务**
### **✅ 1. 提取 ELF**
```bash
adb shell run-as com.example.app cat /data/app/com.example.app/lib/arm64/libnative.so > libnative.so
```
### **✅ 2. 解析 ELF**
```bash
readelf -h libnative.so
readelf -S libnative.so
readelf -s libnative.so
```
### **✅ 3. 反汇编**
```bash
objdump -d libnative.so | head -n 20
```
### **✅ 4. Hook `open()`**
```js
Java.perform(function() {
    var libc = Module.findExportByName(null, "open");
    Interceptor.attach(libc, {
        onEnter: function(args) {
            console.log("File Opened: " + Memory.readUtf8String(args[0]));
        }
    });
});
```

---

# **📚 参考资料**
📌 **ELF 解析**
- `ELF 格式官方文档`：[https://man7.org/linux/man-pages/man5/elf.5.html](https://man7.org/linux/man-pages/man5/elf.5.html)  
- `readelf 手册`：[https://sourceware.org/binutils/docs/binutils/readelf.html](https://sourceware.org/binutils/docs/binutils/readelf.html)  

📌 **逆向分析**
- `Frida Hook ELF`：[https://frida.re](https://frida.re)  
- `objdump`：[https://sourceware.org/binutils/docs/binutils/](https://sourceware.org/binutils/docs/binutils/)  

---

🔥 **任务完成后，你将掌握：**  
✅ **如何解析 ELF 头、程序头、符号表**  
✅ **如何反汇编 ELF 并分析 Native 代码**  
✅ **如何使用 Frida/Xposed Hook ELF 运行时行为**  

🚀 **下一步（Day 18）**：**如何调试 Native 层？** 🎯