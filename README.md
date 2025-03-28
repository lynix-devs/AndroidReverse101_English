# AndroidReverse101 | Mastering Android Reverse Engineering in 100 Days

> - Last Updated: February 24, 2025
> - Author: Evil0ctal
> - GitHub Repository: [AndroidReverse101](https://github.com/Evil0ctal/AndroidReverse101)
> -  Contributions Welcome: Feel free to ⭐️ Star, Fork, and submit PRs to collaborate and learn Android reverse engineering together!

📖 **From Zero to One: A Systematic Approach to Learning Android Reverse Engineering—Making It Fun, Engaging, and Easy to Start!**  
💡 **Learning Goals**：
1. **Beginner-Friendly** – Designed for those with little to no programming experience.
2. **Deep Dive** – Learn CPU architecture, assembly language, ARM instruction sets, Android runtime mechanics, and more.
3. **Hands-on Approach** – Daily lessons + experiments, so you can immediately apply what you learn.
4. **Balancing Cracking & Security** – Understand both reverse engineering techniques and Android security mechanisms.

---

## **Phase 1: Computer Fundamentals & Reverse Engineering Basics (Day 1 - Day 20)**  
🔹 **Objective**：Understand low-level computer architecture, hexadecimal systems, CPU instruction sets, assembly language, and Android runtime principles.

| **Day** | **Topic**                                               | **Content** |
|------|------------------------------------------------------|---------|
| 🏁 **Day 1** | [**What is Reverse Engineering？**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_1_什么是逆向工程.md) | Real-world vs. software reverse engineering, use cases |
| 🔍 **Day 2** | [**History & Evolution of Android Reverse Engineering**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_2_Android_逆向的历史与发展.md) | From early APK cracking to modern app protection |
| ⚙️ **Day 3** | [**What is a CPU Instruction Set?**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_3_什么是_CPU_指令集.md) | CISC vs. RISC, why Android uses ARM |
| 🔥 **Day 4** | [**Hexadecimal Conversion: Why It Matters?**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_4_进制转换_为什么16进制很重要.md) | Binary, Decimal, Hexadecimal conversion & usage |
| 🏗 **Day 5** | [**Assembly Language Basics**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_5_汇编语言基础.md) | Relationship between assembly and machine code, registers |
| 🏹 **Day 6** | [**x86 vs. ARM Assembly**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_6_x86_vs._ARM_汇编.md) | Differences between x86 and ARM instructions |
| 📜 **Day 7** | [**ARM Assembly Instruction Analysis**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_7_ARM_汇编指令解析.md) | Analysis of MOV, ADD, SUB, LDR, STR instructions |
| 🚀 **Day 8** | [**Function Calls and Returns**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_8_函数调用与返回.md) | Analysis of BL, BX, CALL, RET instructions |
| 🏗 **Day 9** | [**Android CPU Architecture Analysis**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_9_Android_CPU_架构解析.md) |  Differences between ARMv7, ARMv8, ARM64 |
| 📦 **Day 10** | [**Dalvik vs. ART Runtime**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_10_Dalvik_vs._ART_运行时.md) | How Android's Java Virtual Machine executes code |
| 🔥 **Day 11** | [**Android Process Management**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_11_Android_进程管理.md) | Understanding the Zygote process and app process lifecycle |
| 🚀 **Day 12** | [**Android Permission Mechanism**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_12_Android_权限机制.md) | How permissions in AndroidManifest.xml affect app security |
| 📂 **Day 13** | [**Android APP Directory Structure**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_13_Android_APP_目录结构.md) | Analysis of the /data/data directory and app data storage locations |
| 🔍 **Day 14** | [**How is an APK Loaded？**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_14_APK_是如何加载的.md) | How Android processes and loads an APK |
| 🛠 **Day 15** | [**Writing ARM Assembly Code (Lab)**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_15_手写_ARM_汇编代码_实验.md) |  Writing ARM Assembly Code (Lab) |
| 🔬 **Day 16** | [**Introduction to Disassembling Tools**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_16_反汇编工具介绍.md) | Tools like IDA Pro, Ghidra, objdump, etc. |
| 🏴 **Day 17** | [**ELF File Analysis**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_17_ELF_文件解析.md) |Using readelf to analyze so file structure |
| 🔥 **Day 18** | [**How to Debug the Native Layer？**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_18_如何调试_Native_层.md) | Debugging so files with LLDB / GDB |
| 🚀 **Day 19** | [**Android APP Security Mechanisms**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_19_Android_APP_安全机制.md) | SELinux, app sandboxing, root detection |
| 🛡 **Day 20** | [**CTF Reverse Engineering Challenges (Beginner)**](./AndroidReverse101/第一阶段_计算机基础_逆向概论/Day_20_CTF_逆向挑战_初级.md) | Participate in an Android reverse engineering CTF challenge |

---

## **Phase 2: APK Reverse Engineering Basics (Day 21 - Day 50)**  
🔹 **Objective:** Learn APK structure, DEX decompilation, Smali language, dynamic debugging, and core reverse engineering techniques.

| **Day** | **Topic**                                                                  | **Content** |
|------|-------------------------------------------------------------------------|---------|
| 🛠 **Day 21** | [**Understanding APK File Structure**](./AndroidReverse101/第二阶段_APK逆向基础/Day_21_APK_文件结构解析.md) | AndroidManifest.xml, resource files, DEX files |
| 🔄 **Day 22** | [**How to Decompile an APK?**](./AndroidReverse101/第二阶段_APK逆向基础/Day_22_如何反编译_APK.md)                            | Tools like jadx, apktool, baksmali |
| 📜 **Day 23** | [**Understanding DEX File Structure**](./AndroidReverse101/第二阶段_APK逆向基础/Day_23_DEX_文件结构解析.md)                           | `ClassDefItem`、`MethodIdItem`、`StringIdItem` |
| 📦 **Day 24** | [**Introduction to Smali Language**](./AndroidReverse101/第二阶段_APK逆向基础/Day_24_Smali_语言入门.md)                           | Smali code structure, instruction analysis |
| 📝 **Day 25** | [**Smali Code Modification Lab**](./AndroidReverse101/第二阶段_APK逆向基础/Day_25_Smali_代码修改实验.md)                       | Manually modify smali code to bypass VIP restrictions |
| 🚀 **Day 26** | [**APK Repackaging & Signing**](./AndroidReverse101/第二阶段_APK逆向基础/Day_26_APK_重新打包_签名.md)                       | Modify APK with apktool, repack, and sign |
| 🔍 **Day 27** | [**Introduction to Dynamic Debugging**](./AndroidReverse101/第二阶段_APK逆向基础/Day_27_动态调试入门.md)                                   | Observe app behavior with logcat |
| 🔬 **Day 28** | [**Using Frida to Hook Java Methods**](./AndroidReverse101/第二阶段_APK逆向基础/Day_28_使用_Frida_Hook_Java_方法.md)     | Modify Java method return values |
| 🏹 **Day 29** | [**Frida Hook Practical**](./AndroidReverse101/第二阶段_APK逆向基础/Day_29_Frida_Hook_实战.md)                     | Bypass Root detection |
| 💉 **Day 30** | [**Reverse Engineering JNI and Native Methods**](./AndroidReverse101/第二阶段_APK逆向基础/Day_30_逆向_JNI_和_Native_方法.md)           | How to analyze libnative.so |
| 🔥 **Day 31** | [**Introduction to Xposed**](./AndroidReverse101/第二阶段_APK逆向基础/Day_31_Xposed_入门.md)                             | Hook Java methods and modify app behavior |
| 🚀 **Day 32** | [**Practical: Bypass VIP Restrictions**](./AndroidReverse101/第二阶段_APK逆向基础/Day_32_破解_VIP_限制.md)                          | Hook `isVip()` method and unlock app membership features |
| 🔗 **Day 33** | [**Bypass SSL Pinning**](./AndroidReverse101/第二阶段_APK逆向基础/Day_33_绕过_SSL_Pinning.md)                   | Crack HTTPS request interception and capture API data |
| 🛡 **Day 34** | [**Android Code Obfuscation and De-obfuscation**](./AndroidReverse101/第二阶段_APK逆向基础/Day_34_Android_代码混淆与解混淆.md)               | How ProGuard and R8 work |
| 🔍 **Day 35** | [**Reverse Engineering Encryption Algorithms (MD5, AES, RSA)**](./AndroidReverse101/第二阶段_APK逆向基础/Day_35_逆向加密算法_MD5_AES_RSA.md)          | Analyze app encryption logic |
| 🔥 **Day 36** | [**Analyze WebSocket & API Requests**](./AndroidReverse101/第二阶段_APK逆向基础/Day_36_分析_WebSocket_API_请求.md)       | Use Burp Suite for protocol analysis |
| 🚀 **Day 37** | [**Cracking App Restrictions (Practical)**](./AndroidReverse101/第二阶段_APK逆向基础/Day_37_破解应用限制_实战.md)                            | Bypass `isForceUpdate()` method |
| 🏹 **Day 38** | [**Game Cracking Basics**](./AndroidReverse101/第二阶段_APK逆向基础/Day_38_游戏破解基础.md)                                   | Hook `buyItem()`，to simulate in-app purchases |
| 🔬 **Day 39** | [**Anti-Anti-Debugging**](./AndroidReverse101/第二阶段_APK逆向基础/Day_39_反反调试.md)                                       | Bypass `ptrace()` protection |
| 🏴‍☠️ **Day 40** | [**Android Hardening Principles**](./AndroidReverse101/第二阶段_APK逆向基础/Day_40_Android_加固原理.md)                       | How 360 and Tencent hardening works |
| 🔍 **Day 41** | [**Decrypt Hardened APK (Beginner)**](./AndroidReverse101/第二阶段_APK逆向基础/Day_41_解密加固_APK_初级.md)                        | Use Frida DumpDex to unpackage |

---

## **🚀 Phase 3: Advanced Reverse Engineering & CTF Challenges (Day 51 - Day 100)**  
🔹 **Objective:** Deep dive into Android obfuscation, anti-reversing techniques, network security analysis, and vulnerability research

| **Day** | **主题**                                                                                | **内容** |
|------|---------------------------------------------------------------------------------------|---------|
| 🔥 **Day 60** | [**CTF Reverse Engineering Challenges**](./AndroidReverse101/第三阶段_高级逆向_CTF挑战/Day_60_深入分析_CTF_逆向挑战.md)           | Analyze high-difficulty APK challenges |
| 🏴‍☠️ **Day 70** | [**Finding 0-Day Vulnerabilities via Reverse Engineering**](./AndroidReverse101/第三阶段_高级逆向_CTF挑战/Day_70_逆向挖掘_0Day_漏洞.md)        | Reverse real-world apps to identify security flaws |
| 🏆 **Day 100** | [**Final Challenge: Reverse Engineering a Full Android App**](./AndroidReverse101/第三阶段_高级逆向_CTF挑战/Day_100_终极挑战_逆向一个完整_APP.md) | Decrypt algorithms, analyze protocols, and bypass VIP features|

---

🔥 **After 100 Days, You Will Have Mastered Android Reverse Engineering!**  
🔓 **Crack apps, analyze security vulnerabilities, and explore Android internals like never before!** 🚀  

---

## **🔍 Crackme Challenges**

> **Crackme challenges are common in reverse engineering, used to practice reverse engineering techniques.**

| **题目**                                             | **描述**                      | **难度** |
|----------------------------------------------------|-----------------------------|---------|
| [Crackme 1](./Crackmes/Android/Level_01/README.md) | There is a hidden secret string somewhere in this app, try to extract it.	 | ⭐️ |
| [Crackme 2](./Crackmes/Android/Level_02/README.md) | This app contains secrets, possibly traces of native code.      | ⭐️⭐️ |


---

## **📚 Appendix: Recommended Learning Resources & Tools**

1. **Book Recommendations:**
   - 《Android Security Attack and Defense》
   - 《Android Reverse Engineering》
   - 《Android Hacker's Handbook》
   - 《Android Vulnerabilities and Reverse Analysis》
   - 《The Android Security Attack and Defense Authority Guide》

2. **Tool Recommendations**
    - Decompiling Tools: jadx, apktool, dex2jar
    - Dynamic Debugging Tools: Frida, Xposed
    - Reverse Engineering Tools: IDA Pro, Ghidra, Hopper
    - Debuggers: LLDB, GDB
    - Hardening Tools: 360 Hardening, Tencent Hardening, Ali Hardening
    - Reverse Platforms: Cuckoo, VirusTotal
    - Network Proxies: Burp Suite, Charles, Fiddler, Wireshark, mitmproxy, reqable
    - Sandboxes: DroidBox, AndroBugs, QARK

3. **Learning Websites**
    - [Android Developers](https://developer.android.com/)
    - [Android 开发者官网](https://developer.android.google.cn/)
    - [Android 开发者博客](https://android-developers.googleblog.com/)
    - [Android Weekly](https://androidweekly.net/)
    - [Android Arsenal](https://android-arsenal.com/)
    - [AndroidX Tech](https://androidx.tech/)
    - [AndroidX Source](https://source.android.com/)

4. **Communities & Forums:**
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

5. **CTF Competitions**
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

6. **Security Conferences:**
    - [DEF CON](https://www.defcon.org/)
    - [Black Hat](https://www.blackhat.com/)
    - [RSA Conference](https://www.rsaconference.com/)
    - [ShmooCon](https://shmoocon.org/)
    - [HITB Security Conference](https://conference.hitb.org/)
    - [CanSecWest](https://cansecwest.com/)
    - [Infiltrate](https://www.infiltratecon.com/)
    - [BSides](https://www.securitybsides.com/)

7. **Bug Bounty Platforms:**
    - [HackerOne](https://www.hackerone.com/)
    - [Bugcrowd](https://www.bugcrowd.com/)
    - [Synack](https://www.synack.com/)
    - [Cobalt](https://cobalt.io/)
    - [YesWeHack](https://www.yeswehack.com/)
    - [Intigriti](https://www.intigriti.com/)
    - [Open Bug Bounty](https://www.openbugbounty.org/)

8. **Security Blogs**：
    - [Google Project Zero](https://googleprojectzero.blogspot.com/)
    - [360 安全研究](https://www.freebuf.com/author/360)
    - [腾讯安全应急响应中心](https://security.tencent.com/)
    - [阿里安全威胁情报中心](https://security.alibaba.com/)
    - [腾讯玄武实验室](https://xlab.tencent.com/)
    - [360 核心安全技术博客](https://blogs.360.cn/)
    - [FreeBuf](https://www.freebuf.com/)
    - [安全客](https://www.anquanke.com/)


## **📝 Author's Note**

🔥 **Android Reverse Engineering is a fun yet challenging skill!**

🚀 **With systematic learning, you'll master the core skills of reverse engineering and become an excellent security researcher!**

📚 **Learning is endless, keep learning and continuously improving!**

👨‍💻 **Good luck!**



