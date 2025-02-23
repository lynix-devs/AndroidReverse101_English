# 🔥 AndroidReverse101 | 100 天精通 Android 逆向工程 🔥
📖 **从 0 到 1，系统化学习 Android 逆向，让学习变得有趣、好玩、易上手！**  
💡 **学习目标**：
1. **新手友好**，即使没有编程经验也能逐步上手。
2. **深入底层**，掌握 CPU 架构、汇编语言、ARM 指令集、Android 运行机制等知识。
3. **实战驱动**，每天一个知识点+实验，学完即能应用。
4. **破解与安全并重**，既能学习破解技巧，也能理解 Android 安全体系。

---

## **🚀 第一阶段：计算机基础 & 逆向概论（Day 1 - Day 20）**  
🔹 **目标**：理解计算机底层架构，掌握 16 进制、CPU 指令集、汇编语言、Android 运行原理。

| **Day** | **主题** | **内容** |
|------|-------------|---------|
| 🏁 **Day 1** | **什么是逆向工程？** | 现实世界 vs. 软件世界，逆向的应用场景 |
| 🔍 **Day 2** | **Android 逆向的历史 & 发展** | 从早期 APK 破解到现代应用保护 |
| ⚙️ **Day 3** | **什么是 CPU 指令集？** | CISC vs. RISC，为什么 Android 采用 ARM |
| 🔥 **Day 4** | **进制转换：为什么 16 进制很重要？** | 2 进制、10 进制、16 进制转换与应用 |
| 🏗 **Day 5** | **汇编语言基础** | 汇编和机器码的关系，寄存器的作用 |
| 🏹 **Day 6** | **x86 vs. ARM 汇编** | x86 指令与 ARM 指令的区别 |
| 📜 **Day 7** | **ARM 汇编指令解析** | `MOV`、`ADD`、`SUB`、`LDR`、`STR` 指令 |
| 🚀 **Day 8** | **函数调用与返回** | `BL`、`BX`、`CALL`、`RET` 指令解析 |
| 🏗 **Day 9** | **Android CPU 架构解析** | ARMv7、ARMv8、ARM64 的区别 |
| 📦 **Day 10** | **Dalvik vs. ART 运行时** | Android 的 Java 虚拟机如何执行代码 |
| 🔥 **Day 11** | **Android 进程管理** | 什么是 Zygote 进程，APP 进程的生命周期 |
| 🚀 **Day 12** | **Android 权限机制** | `AndroidManifest.xml` 里的权限如何影响应用安全？ |
| 📂 **Day 13** | **Android APP 目录结构** | `/data/data` 目录解析，APP 数据存储位置 |
| 🔍 **Day 14** | **APK 是如何加载的？** | Android 进程如何解析 APK |
| 🛠 **Day 15** | **手写 ARM 汇编代码（实验）** | 编写简单的 ARM 汇编程序，并运行 |
| 🔬 **Day 16** | **反汇编工具介绍** | IDA Pro、Ghidra、objdump 等工具 |
| 🏴 **Day 17** | **ELF 文件解析** | `readelf` 解析 `so` 文件结构 |
| 🔥 **Day 18** | **如何调试 Native 层？** | LLDB / GDB 调试 `so` 文件 |
| 🚀 **Day 19** | **Android APP 安全机制** | SELinux、应用沙盒、Root 检测 |
| 🛡 **Day 20** | **CTF 逆向挑战（初级）** | 参加一个 Android 逆向 CTF 题目 |

---

## **🔍 第二阶段：APK 逆向基础（Day 21 - Day 50）**  
🔹 **目标**：掌握 APK 结构、DEX 反编译、Smali 语言、动态调试等核心技能。

| **Day** | **主题** | **内容** |
|------|-------------|---------|
| 🛠 **Day 21** | **APK 文件结构解析** | `AndroidManifest.xml`、资源文件、DEX 文件 |
| 🔄 **Day 22** | **如何反编译 APK？** | `jadx`、`apktool`、`baksmali` 介绍 |
| 📜 **Day 23** | **DEX 文件结构解析** | `ClassDefItem`、`MethodIdItem`、`StringIdItem` |
| 📦 **Day 24** | **Smali 语言入门** | Smali 代码结构、指令解析 |
| 📝 **Day 25** | **Smali 代码修改实验** | 手动修改 `smali` 代码，绕过 VIP 限制 |
| 🚀 **Day 26** | **APK 重新打包 & 签名** | `apktool` 修改 APK，重新打包并签名 |
| 🔍 **Day 27** | **动态调试入门** | `logcat` 观察应用行为 |
| 🔬 **Day 28** | **使用 Frida Hook Java 方法** | 修改 Java 方法返回值 |
| 🏹 **Day 29** | **Frida Hook 实战** | 绕过 Root 检测 |
| 💉 **Day 30** | **逆向 JNI 和 Native 方法** | 如何分析 `libnative.so` |
| 🔥 **Day 31** | **Xposed 入门** | Hook Java 方法，修改应用行为 |
| 🚀 **Day 32** | **实战：破解 VIP 限制** | Hook `isVip()` 方法，解锁 App 会员功能 |
| 🔗 **Day 33** | **绕过 SSL Pinning** | 破解 HTTPS 请求拦截，抓取 API 数据 |
| 🛡 **Day 34** | **Android 代码混淆与解混淆** | ProGuard、R8 的工作原理 |
| 🔍 **Day 35** | **逆向加密算法（MD5、AES、RSA）** | 分析应用的加密逻辑 |
| 🔥 **Day 36** | **分析 WebSocket & API 请求** | 使用 Burp Suite 进行协议分析 |
| 🚀 **Day 37** | **破解应用限制（实战）** | 绕过 `isForceUpdate()` 方法 |
| 🏹 **Day 38** | **游戏破解基础** | Hook `buyItem()`，模拟游戏内购 |
| 🔬 **Day 39** | **反反调试** | 绕过 `ptrace()` 保护 |
| 🏴‍☠️ **Day 40** | **Android 加固原理** | 360 加固、腾讯加固的工作方式 |
| 🔍 **Day 41** | **解密加固 APK（初级）** | 脱壳工具 Frida DumpDex |

---

## **🚀 第三阶段：高级逆向 & CTF 挑战（Day 51 - Day 100）**  
🔹 **目标**：深入研究 Android 加固与反加固、协议分析、漏洞挖掘。

| **Day** | **主题** | **内容** |
|------|-------------|---------|
| 🔥 **Day 60** | **深入分析 CTF 逆向挑战** | 分析高难度 APK |
| 🏴‍☠️ **Day 70** | **逆向挖掘 0Day 漏洞** | 逆向真实应用，寻找安全漏洞 |
| 🏆 **Day 100** | **终极挑战：逆向一个完整 APP** | 还原加密算法，分析协议，破解 VIP |

---

🔥 **100 天后，你将具备完整的 Android 逆向能力！**  
🔓 **破解应用、分析安全漏洞、探索 Android 底层奥秘！** 🚀  
