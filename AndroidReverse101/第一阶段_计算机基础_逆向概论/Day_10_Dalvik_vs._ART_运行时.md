# **📜 Day 10: Dalvik vs. ART 运行时解析**

## **📌 学习目标**
✅ 理解 **Dalvik VM** 和 **ART（Android Runtime）** 的区别及其对 Android 应用的影响。  
✅ 掌握 **DEX（Dalvik Executable）文件格式**，理解 **字节码执行方式**。  
✅ 学习 **ART 运行时如何优化 Android 性能**，如 **AOT（Ahead-Of-Time）编译** 和 **JIT（Just-In-Time）编译**。  
✅ 通过 **分析 Dalvik & ART 运行时的行为**，理解它们对逆向工程和性能优化的影响。  
✅ 实践 **Dalvik 与 ART 运行时的调试与逆向分析**。

---

# **1️⃣ 什么是 Dalvik 和 ART？**
Android 应用的代码通常以 **Java/Kotlin** 编写，编译后生成 **DEX（Dalvik Executable）** 文件，在 Android 设备上运行。  
**运行 DEX 代码的环境** 就是 **Dalvik VM 或 ART 运行时**。

| **运行时** | **特性** | **Android 版本** |
|---------|----------------|----------------|
| **Dalvik VM** | 解释执行，JIT 编译，基于寄存器 | **Android 2.2 - 4.4** |
| **ART（Android Runtime）** | AOT + JIT 编译，运行时优化，减少 CPU 负担 | **Android 5.0+** |

📌 **简而言之**：
- **Dalvik**：类似传统 Java 虚拟机，每次运行都需要解释代码，影响性能。  
- **ART**：引入 **AOT 编译**（安装时编译为机器码），大幅提升性能。  

---

# **2️⃣ Dalvik 与 ART 的核心区别**
| **特性** | **Dalvik VM** | **ART 运行时** |
|---------|------------|------------|
| **代码执行方式** | 解释执行 + JIT 编译 | AOT 编译 + JIT 编译 |
| **启动速度** | 快（直接执行 DEX） | 稍慢（首次安装时编译 OAT） |
| **运行时性能** | 低（每次运行需解释代码） | 高（已编译为本机代码） |
| **内存占用** | 低（仅加载需要的字节码） | 高（额外存储 OAT 文件） |
| **电池消耗** | 较高（频繁解释代码） | 较低（减少运行时编译） |
| **GC（垃圾回收）** | 分步 GC（影响 UI 流畅度） | 并发 GC（更流畅） |
| **调试支持** | Smali 级 Hook | 机器码级 Hook（更难调试） |

---

# **3️⃣ Dalvik 运行原理**
### **🔹 DEX（Dalvik Executable）格式**
Dalvik 运行时执行 **DEX 字节码**，与传统 Java `.class` 文件不同：
- **寄存器架构**（不像 JVM 的栈架构）。
- **更少的指令集**，更适合移动设备。

📌 **反编译 DEX**
```bash
dexdump classes.dex
```
示例输出：
```
Dex file version 035
magic: dex\n035
class_defs_size: 5
```

📌 **反汇编 Smali**
```bash
baksmali d classes.dex -o output/
```

---

# **4️⃣ ART 运行原理**
### **🔹 AOT（Ahead-of-Time 编译）**
ART 运行时在 **安装应用时** 预编译 `.dex → .oat`（本机代码），减少运行时开销。

📌 **查看 OAT 文件**
```bash
ls /data/dalvik-cache/
```

📌 **反编译 OAT**
```bash
oatdump --oat-file=/data/dalvik-cache/arm64/system@framework@boot.art
```

### **🔹 JIT（Just-In-Time 编译）**
- Android 7.0+ 在 **AOT 基础上增加 JIT**，动态优化热点代码，提高运行效率。
- 代码执行过程中，根据 **运行情况** 进行 **热点优化**，提升应用响应速度。

---

# **5️⃣ 逆向工程中的影响**
### **🔹 Dalvik 逆向分析**
- Dalvik **基于 DEX**，可直接反编译 **Smali** 代码。
- 通过 `smali/baksmali` 轻松修改 `.dex` 代码：
```bash
apktool d myapp.apk
vim smali/com/example/MainActivity.smali
```
- 修改后重新打包：
```bash
apktool b myapp
jarsigner -verbose -keystore my.keystore myapp/dist/myapp.apk alias_name
```

### **🔹 ART 逆向分析**
- ART **直接执行 OAT 机器码**，无法简单修改 Smali 代码。
- 需要使用 **Frida / GDB / IDA Pro** 进行动态调试：
```bash
frida -U -n myapp -e "Interceptor.attach(Module.findExportByName(null, 'open'), {onEnter: function(args) { console.log('open called'); }})"
```
- ART 使用 **AOT 编译**，脱壳更困难：
```bash
dd if=/data/app/com.example-1/oat/arm64/base.odex of=/sdcard/base.odex
```

---

# **6️⃣ Android 设备上的 Dalvik vs. ART**
📌 **检查 Android 设备使用的运行时**
```bash
adb shell getprop persist.sys.dalvik.vm.lib.2
```
输出：
```
libart.so
```
👉 说明该设备使用 **ART 运行时**。

📌 **查看 OAT 目录**
```bash
adb shell ls /data/dalvik-cache
```
示例输出：
```
arm64/data@app@com.example-1@base.apk@classes.dex
```
📌 **检查应用的执行模式**
```bash
adb shell getprop dalvik.vm.execution-mode
```
可能的输出：
```
jit
aot
```

---

# **🛠 实战任务**
### **✅ 1. 检查 Android 设备使用的运行时**
```bash
adb shell getprop persist.sys.dalvik.vm.lib.2
```
### **✅ 2. 解析 DEX 文件**
```bash
dexdump classes.dex
baksmali d classes.dex -o output/
```
### **✅ 3. 反编译 ART OAT 文件**
```bash
oatdump --oat-file=/data/dalvik-cache/arm64/system@framework@boot.art
```
### **✅ 4. Hook ART 运行时**
```bash
frida -U -n myapp -e "Interceptor.attach(Module.findExportByName(null, 'open'), {onEnter: function(args) { console.log('open called'); }})"
```

---

# **📚 参考资料**
📌 **Android 运行时**
- `ART 介绍`：[https://source.android.com/devices/tech/dalvik](https://source.android.com/devices/tech/dalvik)  
- `DEX 结构`：[https://source.android.com/devices/tech/dalvik/dex-format](https://source.android.com/devices/tech/dalvik/dex-format)  

📌 **逆向工程**
- `Smali / Baksmali`：[https://github.com/JesusFreke/smali](https://github.com/JesusFreke/smali)  
- `Frida`：[https://frida.re](https://frida.re)  
- `oatdump`：[https://developer.android.com/ndk/guides/other_build_systems](https://developer.android.com/ndk/guides/other_build_systems)  

---

🔥 **任务完成后，你将掌握：**  
✅ **Dalvik 和 ART 运行时的核心区别**  
✅ **如何解析 DEX、OAT 文件，进行应用逆向分析**  
✅ **如何检查 Android 设备的运行时环境**  
✅ **如何利用 Frida / GDB Hook ART 运行时**  

🚀 **下一步（Day 11）**：**Android 进程管理解析！** 🎯