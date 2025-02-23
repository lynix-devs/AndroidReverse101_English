# **📜 Day 16: 反汇编工具介绍**

## **📌 学习目标**
✅ **理解反汇编的基本概念**，掌握如何将二进制文件转换为可读的汇编代码。  
✅ **熟悉主流反汇编工具**（IDA Pro、Ghidra、Radare2、objdump）的功能与使用场景。  
✅ **学习如何分析 Android ELF 文件、DEX 文件、Native 共享库（.so）**。  
✅ **掌握基于 Frida / Xposed 的动态分析方法**，Hook 关键函数。  
✅ **通过实际操作，提取、反编译、分析 Android 应用的二进制代码**。  

---

# **1️⃣ 反汇编基础**
### **🔹 什么是反汇编？**
反汇编（Disassembly）是指将 **机器码（Binary Code）转换为汇编代码（Assembly Code）**，从而理解程序的运行逻辑。

📌 **示例**
| **二进制指令** | **ARM 汇编** | **等价 C 代码** |
|-------------|-----------|-----------|
| `E3A00005` | `MOV R0, #5` | `int a = 5;` |
| `E0801001` | `ADD R1, R0, R1` | `b = a + b;` |
| `EB000001` | `BL function` | `function();` |

📌 **查看 ELF 文件架构**
```bash
file libnative.so
```
示例输出：
```
libnative.so: ELF 64-bit LSB shared object, ARM aarch64
```

---

# **2️⃣ 反汇编工具**
## **✅ 1. objdump（Linux 自带）**
📌 **反汇编 ELF 文件**
```bash
objdump -d libnative.so | head -n 20
```
📌 **解析 ELF 段信息**
```bash
readelf -h libnative.so
```

---

## **✅ 2. IDA Pro（交互式反汇编器）**
📌 **安装 IDA Free**
```bash
wget https://out7.hex-rays.com/files/idafree83_linux.run
chmod +x idafree83_linux.run
./idafree83_linux.run
```
📌 **打开 ELF 文件**
1. **File → Open → 选择 `libnative.so`**
2. **选择 CPU 架构（ARM/ARM64）**
3. **开始分析，查看函数表、字符串、交叉引用（XREF）**

📌 **快捷键**
| **操作** | **快捷键** |
|---------|---------|
| 交叉引用 | `X` |
| 切换汇编/C 代码 | `F5` |
| 查找字符串 | `Shift + F12` |

---

## **✅ 3. Ghidra（NSA 开源工具）**
📌 **安装 Ghidra**
```bash
wget https://ghidra-sre.org/ghidra_10.1.5_PUBLIC_20231005.zip
unzip ghidra_10.1.5_PUBLIC_20231005.zip
cd ghidra_10.1.5
./ghidraRun
```
📌 **分析 ELF 文件**
1. **File → New Project → Import `libnative.so`**
2. **选择 ARM 处理器**
3. **双击函数，查看反汇编结果**
4. **使用 `Decompiler` 还原 C 代码**

📌 **Ghidra 常用快捷键**
| **操作** | **快捷键** |
|---------|---------|
| 查找字符串 | `Ctrl + Shift + F` |
| 交叉引用 | `Ctrl + Shift + X` |
| 反编译 | `F4` |

---

## **✅ 4. Radare2（开源 CLI 反汇编工具）**
📌 **安装 Radare2**
```bash
git clone https://github.com/radareorg/radare2.git
cd radare2
sys/install.sh
```
📌 **反汇编 ELF**
```bash
r2 -AA libnative.so
```
📌 **常用命令**
| **命令** | **作用** |
|---------|---------|
| `aa` | 自动分析 |
| `afl` | 显示所有函数 |
| `pdf @ main` | 反汇编 `main()` |
| `izz` | 查找字符串 |

---

# **3️⃣ 逆向分析 Android ELF**
## **✅ 1. 提取 libnative.so**
```bash
adb shell run-as com.example.app cat /data/app/com.example.app/lib/arm64/libnative.so > libnative.so
```

## **✅ 2. 解析 ELF 头**
```bash
readelf -h libnative.so
```

## **✅ 3. 反汇编**
```bash
objdump -d libnative.so | head -n 20
```

## **✅ 4. Hook ELF 运行时行为**
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
📌 **执行**
```bash
frida -U -n com.example.app -e "..."
```

---

# **4️⃣ Hook DEX 代码**
📌 **使用 Frida Hook DEX 运行时**
```js
Java.perform(function() {
    var DexClassLoader = Java.use("dalvik.system.DexClassLoader");
    DexClassLoader.loadClass.implementation = function(name) {
        console.log("Hooked Dex Load: " + name);
        return this.loadClass(name);
    };
});
```

📌 **执行**
```bash
frida -U -n com.example.app -e "..."
```

---

# **🛠 实战任务**
### **✅ 1. 使用 `objdump` 反汇编 ELF**
```bash
objdump -d libnative.so | head -n 20
```
### **✅ 2. 使用 IDA Pro/Ghidra 反编译 ELF**
1. **导入 `libnative.so`**
2. **查找字符串**
3. **分析关键函数**
### **✅ 3. Hook `open()` 调用**
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
### **✅ 4. Hook `DexClassLoader`**
```js
Java.perform(function() {
    var DexClassLoader = Java.use("dalvik.system.DexClassLoader");
    DexClassLoader.loadClass.implementation = function(name) {
        console.log("Hooked Dex Load: " + name);
        return this.loadClass(name);
    };
});
```

---

# **📚 参考资料**
📌 **反汇编工具**
- `IDA Pro`：[https://hex-rays.com/](https://hex-rays.com/)  
- `Ghidra`：[https://ghidra-sre.org/](https://ghidra-sre.org/)  
- `Radare2`：[https://github.com/radareorg/radare2](https://github.com/radareorg/radare2)  

📌 **Android ELF 逆向**
- `Frida`：[https://frida.re](https://frida.re/)  
- `objdump`：[https://sourceware.org/binutils/docs/binutils/](https://sourceware.org/binutils/docs/binutils/)  

---

🔥 **任务完成后，你将掌握：**  
✅ **主流反汇编工具的使用（IDA, Ghidra, objdump, Radare2）**  
✅ **如何解析 ELF 文件，分析 Android Native 代码**  
✅ **如何使用 Frida/Xposed 进行动态分析**  

🚀 **下一步（Day 17）**：**ELF 文件解析！** 🎯