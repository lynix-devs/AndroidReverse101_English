# **📜 Day 9: Android CPU 架构解析**

## **📌 学习目标**
✅ 了解 **Android 设备的 CPU 架构**，包括 **ARM32、ARM64、x86、RISC-V**。  
✅ 掌握 **ARMv7、ARMv8（AArch64）及其主要特性**，理解 **big.LITTLE** 架构。  
✅ 熟悉 **Android 设备的 CPU 兼容性**，学习如何判断 APK 适用于哪种架构。  
✅ 通过 **实战分析 Android 设备 CPU 信息**，获取 `/proc/cpuinfo` 解析设备架构。  
✅ 理解不同 CPU 架构的 **逆向工程、性能优化、漏洞分析的影响**。  

---

# **1️⃣ Android 平台的主流 CPU 架构**
目前，Android 设备主要采用以下 CPU 架构：
| **架构** | **指令集** | **常见处理器** | **适用设备** |
|---------|----------|--------------|------------|
| **ARMv7 (32-bit)** | ARM32 | Cortex-A7, Cortex-A15 | 旧款 Android 手机、IoT 设备 |
| **ARMv8 (64-bit)** | ARM64 (AArch64) | Cortex-A53, Cortex-A76 | 现代 Android 旗舰手机 |
| **x86 / x86_64** | CISC | Intel Atom, AMD Ryzen Embedded | Android 模拟器、少量平板 |
| **RISC-V** | RISC | 未来趋势 | 低功耗设备、实验性 Android 平台 |

---

# **2️⃣ ARMv7 vs. ARMv8 (AArch64)**
| **特性** | **ARMv7 (32-bit)** | **ARMv8 (64-bit)** |
|---------|----------------|----------------|
| **指令集** | ARM32 (AArch32) | ARM64 (AArch64) |
| **最大内存支持** | 4GB | 16EB（大幅提升） |
| **通用寄存器** | 16 个（R0-R15） | 31 个（X0-X30） |
| **指令集优化** | 依赖 `Thumb-2` | 精简、提高能效 |
| **NEON SIMD 支持** | 可选 | 内置支持 |
| **性能** | 低功耗，较慢 | 高效，高吞吐量 |
| **兼容性** | 不能运行 64 位应用 | 向下兼容 ARMv7 代码 |

**🔹 示例：ARM32 vs. ARM64 汇编**
```assembly
; ARM32 版本
MOV R0, #5
ADD R0, R0, #3

; ARM64 版本
MOV X0, #5
ADD X0, X0, #3
```
📌 **ARM64 版本的寄存器更宽，计算能力更强！**

---

# **3️⃣ big.LITTLE 架构**
### **🔹 什么是 big.LITTLE？**
big.LITTLE 是 ARM 开发的一种 **异构多核架构**，将 **高性能核心（big）** 与 **节能核心（LITTLE）** 结合，达到 **性能与功耗的平衡**。

| **核心类型** | **特点** | **常见核心** |
|------------|--------|-------------|
| **big 核心** | 高性能，适合重负载任务 | Cortex-A76, Cortex-X1 |
| **LITTLE 核心** | 低功耗，适合后台任务 | Cortex-A55 |

📌 **常见 big.LITTLE 组合**
| **CPU 组合** | **示例处理器** |
|------------|------------|
| 4× big + 4× LITTLE | Snapdragon 865 (4× Cortex-A77 + 4× Cortex-A55) |
| 1× big + 3× middle + 4× LITTLE | Snapdragon 8 Gen 1 (1× Cortex-X2 + 3× Cortex-A710 + 4× Cortex-A510) |

**🔹 big.LITTLE 如何工作？**
- 当 **设备空闲** 时，任务运行在 **LITTLE 核心**，节省功耗。
- 当 **用户运行大型应用** 时，任务切换到 **big 核心**，提高性能。

**💡 逆向工程影响**：  
- **不同核心运行不同任务**，调试时要注意 CPU 负载的变化。  
- **big 核心更适合跑高负载逆向分析，如 Frida、Ghidra。**

---

# **4️⃣ Android APK 架构兼容性**
Android 平台上的 APK 需要匹配设备 CPU 架构。APK 支持的 CPU 架构信息存储在 `lib/` 目录，例如：
```
lib/arm64-v8a/       # 64-bit ARM
lib/armeabi-v7a/     # 32-bit ARM
lib/x86/             # 32-bit x86
lib/x86_64/          # 64-bit x86
```
📌 **判断 Android 设备支持的 CPU 架构**
```bash
adb shell getprop ro.product.cpu.abi
```
示例输出：
```
arm64-v8a
```
📌 **查看 Android `/proc/cpuinfo`**
```bash
adb shell cat /proc/cpuinfo
```
示例输出：
```
Processor    : ARMv8 Processor rev 3
BogoMIPS    : 38.40
Features    : fp asimd evtstrm aes pmull sha1 sha2 crc32
CPU architecture: 8
```

📌 **逆向分析 APK 目标架构**
```bash
unzip -l myapp.apk | grep lib/
```
示例输出：
```
lib/arm64-v8a/libnative.so
lib/armeabi-v7a/libnative.so
```
👉 说明此 APK **同时支持 ARM64 和 ARM32 设备**。

---

# **5️⃣ 逆向工程 & 漏洞分析**
不同 CPU 架构影响逆向分析：
1. **ARM64 指令更简洁**，但 **寄存器 X0-X30 增加复杂性**。
2. **big.LITTLE 调度影响动态分析**，调试时需关注 CPU 核心的任务分配情况。
3. **漏洞利用**：
   - **ROP 攻击（Return Oriented Programming）**：x86 上常见，ARM64 由于 `PAC` 保护难度更大。
   - **指令集转换**：ARM 支持 **AArch32 & AArch64**，部分应用仍使用 ARM32 指令，可导致兼容性问题。

📌 **示例：使用 GDB 读取 `/proc/cpuinfo`**
```bash
adb shell su
gdbserver :5039 --attach <pid>
```
然后在 GDB 中：
```gdb
p *(char(*)[100])0x7ffd8c3b0000
```
📌 **分析 CPU 指令**
```bash
objdump -d libnative.so | head -n 20
```
---

# **🛠 实战任务**
### **✅ 1. 检查 Android 设备 CPU 架构**
```bash
adb shell getprop ro.product.cpu.abi
```
### **✅ 2. 解析 `/proc/cpuinfo`**
```bash
adb shell cat /proc/cpuinfo
```
### **✅ 3. 检查 APK 兼容的 CPU 架构**
```bash
unzip -l myapp.apk | grep lib/
```
### **✅ 4. 反编译 Android 共享库**
```bash
objdump -d libnative.so | head -n 20
```
---

# **📚 参考资料**
📌 **ARM 官方文档**
- `ARMv7 指令集`：[https://developer.arm.com/documentation](https://developer.arm.com/documentation)  
- `ARMv8 (AArch64) 指令集`：[https://developer.arm.com/documentation](https://developer.arm.com/documentation)  

📌 **big.LITTLE 参考**
- `ARM big.LITTLE 技术`：[https://developer.arm.com/solutions/mobile](https://developer.arm.com/solutions/mobile)  

📌 **Android 逆向工程**
- `Ghidra`：[https://ghidra-sre.org/](https://ghidra-sre.org/)  
- `IDA Pro`：[https://hex-rays.com/](https://hex-rays.com/)  

---

🔥 **任务完成后，你将掌握：**  
✅ **Android 设备 CPU 架构（ARM32, ARM64, x86, RISC-V）**  
✅ **如何分析 Android 设备的 `/proc/cpuinfo` 获取 CPU 信息**  
✅ **如何判断 APK 适用于哪种 CPU 架构**  
✅ **big.LITTLE 架构的工作原理及其影响**  

🚀 **下一步（Day 10）**：**Dalvik vs. ART 运行时解析！** 🎯