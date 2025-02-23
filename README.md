# 🔥 AndroidReverse101 | 100 天精通 Android 逆向工程

> - 更新时间：2025年2月23日
> - 作者：Evil0ctal
> - 编写工具：ChatGPT、Claude、DeepSeek
> - GitHub 仓库：[AndroidReverse101](https://github.com/Evil0ctal/AndroidReverse101)
> -  欢迎各位 Star、Fork，PR，一起交流和学习 Android 逆向！

📖 **从 0 到 1，系统化学习 Android 逆向，让学习变得有趣、好玩、易上手！**  
💡 **学习目标**：
1. **新手友好**，即使没有编程经验也能逐步上手。
2. **深入底层**，掌握 CPU 架构、汇编语言、ARM 指令集、Android 运行机制等知识。
3. **实战驱动**，每天一个知识点+实验，学完即能应用。
4. **破解与安全并重**，既能学习破解技巧，也能理解 Android 安全体系。

---

## **🚀 第一阶段：计算机基础 & 逆向概论（Day 1 - Day 20）**  
🔹 **目标**：理解计算机底层架构，掌握 16 进制、CPU 指令集、汇编语言、Android 运行原理。

| **Day** | **主题**                                               | **内容** |
|------|------------------------------------------------------|---------|
| 🏁 **Day 1** | [**什么是逆向工程？**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_1_什么是逆向工程.md) | 现实世界 vs. 软件世界，逆向的应用场景 |
| 🔍 **Day 2** | [**Android 逆向的历史 & 发展**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_2_Android_逆向的历史与发展.md) | 从早期 APK 破解到现代应用保护 |
| ⚙️ **Day 3** | [**什么是 CPU 指令集？**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_3_什么是_CPU_指令集.md) | CISC vs. RISC，为什么 Android 采用 ARM |
| 🔥 **Day 4** | [**进制转换：为什么 16 进制很重要？**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_4_进制转换_为什么16进制很重要.md) | 2 进制、10 进制、16 进制转换与应用 |
| 🏗 **Day 5** | [**汇编语言基础**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_5_汇编语言基础.md) | 汇编和机器码的关系，寄存器的作用 |
| 🏹 **Day 6** | [**x86 vs. ARM 汇编**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_6_x86_vs._ARM_汇编.md) | x86 指令与 ARM 指令的区别 |
| 📜 **Day 7** | [**ARM 汇编指令解析**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_7_ARM_汇编指令解析.md) | `MOV`、`ADD`、`SUB`、`LDR`、`STR` 指令 |
| 🚀 **Day 8** | [**函数调用与返回**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_8_函数调用与返回.md) | `BL`、`BX`、`CALL`、`RET` 指令解析 |
| 🏗 **Day 9** | [**Android CPU 架构解析**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_9_Android_CPU_架构解析.md) | ARMv7、ARMv8、ARM64 的区别 |
| 📦 **Day 10** | [**Dalvik vs. ART 运行时**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_10_Dalvik_vs._ART_运行时.md) | Android 的 Java 虚拟机如何执行代码 |
| 🔥 **Day 11** | [**Android 进程管理**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_11_Android_进程管理.md) | 什么是 Zygote 进程，APP 进程的生命周期 |
| 🚀 **Day 12** | [**Android 权限机制**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_12_Android_权限机制.md) | `AndroidManifest.xml` 里的权限如何影响应用安全？ |
| 📂 **Day 13** | [**Android APP 目录结构**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_13_Android_APP_目录结构.md) | `/data/data` 目录解析，APP 数据存储位置 |
| 🔍 **Day 14** | [**APK 是如何加载的？**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_14_APK_是如何加载的.md) | Android 进程如何解析 APK |
| 🛠 **Day 15** | [**手写 ARM 汇编代码（实验）**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_15_手写_ARM_汇编代码_实验.md) | 编写简单的 ARM 汇编程序，并运行 |
| 🔬 **Day 16** | [**反汇编工具介绍**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_16_反汇编工具介绍.md) | IDA Pro、Ghidra、objdump 等工具 |
| 🏴 **Day 17** | [**ELF 文件解析**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_17_ELF_文件解析.md) | `readelf` 解析 `so` 文件结构 |
| 🔥 **Day 18** | [**如何调试 Native 层？**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_18_如何调试_Native_层.md) | LLDB / GDB 调试 `so` 文件 |
| 🚀 **Day 19** | [**Android APP 安全机制**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_19_Android_APP_安全机制.md) | SELinux、应用沙盒、Root 检测 |
| 🛡 **Day 20** | [**CTF 逆向挑战（初级）**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_20_CTF_逆向挑战_初级.md) | 参加一个 Android 逆向 CTF 题目 |

---

## **🔍 第二阶段：APK 逆向基础（Day 21 - Day 50）**  
🔹 **目标**：掌握 APK 结构、DEX 反编译、Smali 语言、动态调试等核心技能。

| **Day** | **主题**                                                                  | **内容** |
|------|-------------------------------------------------------------------------|---------|
| 🛠 **Day 21** | [**APK 文件结构解析**](./AndroidReverse101/第二阶段_APK逆向基础/Day_21_APK_文件结构解析.md) | `AndroidManifest.xml`、资源文件、DEX 文件 |
| 🔄 **Day 22** | [**如何反编译 APK？**](./AndroidReverse101/第二阶段_APK逆向基础/Day_22_如何反编译_APK.md)                            | `jadx`、`apktool`、`baksmali` 介绍 |
| 📜 **Day 23** | [**DEX 文件结构解析**](./AndroidReverse101/第二阶段_APK逆向基础/Day_23_DEX_文件结构解析.md)                           | `ClassDefItem`、`MethodIdItem`、`StringIdItem` |
| 📦 **Day 24** | [**Smali 语言入门**](./AndroidReverse101/第二阶段_APK逆向基础/Day_24_Smali_语言入门.md)                           | Smali 代码结构、指令解析 |
| 📝 **Day 25** | [**Smali 代码修改实验**](./AndroidReverse101/第二阶段_APK逆向基础/Day_25_Smali_代码修改实验.md)                       | 手动修改 `smali` 代码，绕过 VIP 限制 |
| 🚀 **Day 26** | [**APK 重新打包 & 签名**](./AndroidReverse101/第二阶段_APK逆向基础/Day_26_APK_重新打包_签名.md)                       | `apktool` 修改 APK，重新打包并签名 |
| 🔍 **Day 27** | [**动态调试入门**](./AndroidReverse101/第二阶段_APK逆向基础/Day_27_动态调试入门.md)                                   | `logcat` 观察应用行为 |
| 🔬 **Day 28** | [**使用 Frida Hook Java 方法**](./AndroidReverse101/第二阶段_APK逆向基础/Day_28_使用_Frida_Hook_Java_方法.md)     | 修改 Java 方法返回值 |
| 🏹 **Day 29** | [**Frida Hook 实战**](./AndroidReverse101/第二阶段_APK逆向基础/Day_29_Frida_Hook_实战.md)                     | 绕过 Root 检测 |
| 💉 **Day 30** | [**逆向 JNI 和 Native 方法**](./AndroidReverse101/第二阶段_APK逆向基础/Day_30_逆向_JNI_和_Native_方法.md)           | 如何分析 `libnative.so` |
| 🔥 **Day 31** | [**Xposed 入门**](./AndroidReverse101/第二阶段_APK逆向基础/Day_31_Xposed_入门.md)                             | Hook Java 方法，修改应用行为 |
| 🚀 **Day 32** | [**实战：破解 VIP 限制**](./AndroidReverse101/第二阶段_APK逆向基础/Day_32_破解_VIP_限制.md)                          | Hook `isVip()` 方法，解锁 App 会员功能 |
| 🔗 **Day 33** | [**绕过 SSL Pinning**](./AndroidReverse101/第二阶段_APK逆向基础/Day_33_绕过_SSL_Pinning.md)                   | 破解 HTTPS 请求拦截，抓取 API 数据 |
| 🛡 **Day 34** | [**Android 代码混淆与解混淆**](./AndroidReverse101/第二阶段_APK逆向基础/Day_34_Android_代码混淆与解混淆.md)               | ProGuard、R8 的工作原理 |
| 🔍 **Day 35** | [**逆向加密算法（MD5、AES、RSA）**](./AndroidReverse101/第二阶段_APK逆向基础/Day_35_逆向加密算法_MD5_AES_RSA.md)          | 分析应用的加密逻辑 |
| 🔥 **Day 36** | [**分析 WebSocket & API 请求**](./AndroidReverse101/第二阶段_APK逆向基础/Day_36_分析_WebSocket_API_请求.md)       | 使用 Burp Suite 进行协议分析 |
| 🚀 **Day 37** | [**破解应用限制（实战）**](./AndroidReverse101/第二阶段_APK逆向基础/Day_37_破解应用限制_实战.md)                            | 绕过 `isForceUpdate()` 方法 |
| 🏹 **Day 38** | [**游戏破解基础**](./AndroidReverse101/第二阶段_APK逆向基础/Day_38_游戏破解基础.md)                                   | Hook `buyItem()`，模拟游戏内购 |
| 🔬 **Day 39** | [**反反调试**](./AndroidReverse101/第二阶段_APK逆向基础/Day_39_反反调试.md)                                       | 绕过 `ptrace()` 保护 |
| 🏴‍☠️ **Day 40** | [**Android 加固原理**](./AndroidReverse101/第二阶段_APK逆向基础/Day_40_Android_加固原理.md)                       | 360 加固、腾讯加固的工作方式 |
| 🔍 **Day 41** | [**解密加固 APK（初级）**](./AndroidReverse101/第二阶段_APK逆向基础/Day_41_解密加固_APK_初级.md)                        | 脱壳工具 Frida DumpDex |

---

## **🚀 第三阶段：高级逆向 & CTF 挑战（Day 51 - Day 100）**  
🔹 **目标**：深入研究 Android 加固与反加固、协议分析、漏洞挖掘。

| **Day** | **主题**                                                                                | **内容** |
|------|---------------------------------------------------------------------------------------|---------|
| 🔥 **Day 60** | [**深入分析 CTF 逆向挑战**](./AndroidReverse101/第三阶段_高级逆向_CTF挑战/Day_60_深入分析_CTF_逆向挑战.md)           | 分析高难度 APK |
| 🏴‍☠️ **Day 70** | [**逆向挖掘 0Day 漏洞**](./AndroidReverse101/第三阶段_高级逆向_CTF挑战/Day_70_逆向挖掘_0Day_漏洞.md)        | 逆向真实应用，寻找安全漏洞 |
| 🏆 **Day 100** | [**终极挑战：逆向一个完整 APP**](./AndroidReverse101/第三阶段_高级逆向_CTF挑战/Day_100_终极挑战_逆向一个完整_APP.md) | 还原加密算法，分析协议，破解 VIP |

---

🔥 **100 天后，你将具备完整的 Android 逆向能力！**  
🔓 **破解应用、分析安全漏洞、探索 Android 底层奥秘！** 🚀  

---

## **📚 附录：学习资源 & 工具推荐**

1. **书籍推荐**：
   - 《Android 安全攻防实战》
   - 《Android 逆向工程》
   - 《Android Hacker's Handbook》
   - 《Android 漏洞与逆向分析》
   - 《Android 安全攻防权威指南》

2. **工具推荐**：
    - 反编译工具：jadx、apktool、dex2jar
    - 动态调试工具：Frida、Xposed
    - 逆向工具：IDA Pro、Ghidra、Hopper
    - 调试器：LLDB、GDB
    - 加固工具：360 加固、腾讯加固、阿里加固
    - 逆向平台：Cuckoo、VirusTotal
    - 网络代理：Burp Suite、Charles、Fiddler、Wireshark、mitmproxy、reqable
    - 沙箱：DroidBox、AndroBugs、QARK

3. **学习网站**：
    - [Android Developers](https://developer.android.com/)
    - [Android 开发者官网](https://developer.android.google.cn/)
    - [Android 开发者博客](https://android-developers.googleblog.com/)
    - [Android Weekly](https://androidweekly.net/)
    - [Android Arsenal](https://android-arsenal.com/)
    - [AndroidX Tech](https://androidx.tech/)
    - [AndroidX Source](https://source.android.com/)

4. **社区 & 论坛**：
    - [XDA Developers](https://www.xda-developers.com/)
    - [Reddit - Android Dev](https://www.reddit.com/r/androiddev/)
    - [Reddit - Reverse Engineering](https://www.reddit.com/r/ReverseEngineering/)
    - [Reddit - Malware Analysis](https://www.reddit.com/r/Malware/)
    - [Reddit - CTF](https://www.reddit.com/r/securityCTF/)
    - [Reddit - NetSec](https://www.reddit.com/r/netsec/)
    - [Reddit - Hacking](https://www.reddit.com/r/hacking/)
    - [Reddit - HowToHack](https://www.reddit.com/r/HowToHack/)
    - [看雪安全论坛](https://bbs.kanxue.com/)
    - [吾爱破解论坛](https://www.52pojie.cn/)
    - [安全客](https://www.anquanke.com/)
    - [FreeBuf](https://www.freebuf.com/)
    - [SecWiki](https://www.sec-wiki.com/)
    - [安全脉搏](https://www.secpulse.com/)

5. **CTF 竞赛**：
    - [CTFTime](https://ctftime.org/)
    - [Hacker101 CTF](https://ctf.hacker101.com/)
    - [PicoCTF](https://picoctf.org/)
    - [OverTheWire](https://overthewire.org/wargames/)
    - [Hack The Box](https://www.hackthebox.eu/)
    - [Root Me](https://www.root-me.org/)
    - [VulnHub](https://www.vulnhub.com/)
    - [RingZer0 CTF](https://ringzer0ctf.com/)
    - [HackThisSite](https://www.hackthissite.org/)
    - [CTF365](https://ctf365.com/)

6. **安全会议**：
    - [DEF CON](https://www.defcon.org/)
    - [Black Hat](https://www.blackhat.com/)
    - [RSA Conference](https://www.rsaconference.com/)
    - [ShmooCon](https://shmoocon.org/)
    - [HITB Security Conference](https://conference.hitb.org/)
    - [CanSecWest](https://cansecwest.com/)
    - [Infiltrate](https://www.infiltratecon.com/)
    - [BSides](https://www.securitybsides.com/)

7. **赏金猎人平台**：
    - [HackerOne](https://www.hackerone.com/)
    - [Bugcrowd](https://www.bugcrowd.com/)
    - [Synack](https://www.synack.com/)
    - [Cobalt](https://cobalt.io/)
    - [YesWeHack](https://www.yeswehack.com/)
    - [Intigriti](https://www.intigriti.com/)
    - [Open Bug Bounty](https://www.openbugbounty.org/)

8. **安全博客**：
    - [Google Project Zero](https://googleprojectzero.blogspot.com/)
    - [360 安全研究](https://www.freebuf.com/author/360)
    - [腾讯安全应急响应中心](https://security.tencent.com/)
    - [阿里安全威胁情报中心](https://security.alibaba.com/)
    - [腾讯玄武实验室](https://xlab.tencent.com/)
    - [360 核心安全技术博客](https://blogs.360.cn/)
    - [FreeBuf](https://www.freebuf.com/)
    - [安全客](https://www.anquanke.com/)


## **📝 作者的话**

🔥 **Android 逆向工程是一门既有趣又具有挑战性的技能！**

🚀 **通过系统化学习，你将掌握逆向的核心技能，成为一名优秀的安全研究员！**

📚 **学无止境，持续学习，不断进步！**

👨‍💻 **加油！**




