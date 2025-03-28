### **📜 Day 1: What is Reverse Engineering**

#### **📌 Learning Objectives**  
✅ Understand what **reverse engineering** is and its application scenarios.  
✅ Understand the difference between **reverse engineering** and **forward engineering**.  
✅ Familiarize yourself with basic concepts of **software reverse engineering**, such as **static analysis** and **dynamic analysis**.  
✅ Learn about common reverse engineering tools and set up the basic environment.  

---

#### **📖 Knowledge Points**  

### **1️⃣ What is Reverse Engineering?**  
Reverse engineering (RE) is the process of analyzing the **structure, function, and implementation principles** of an existing system to understand how it works.  

🔹 The essence of reverse engineering is **disassembly and understanding**. It is not only applicable to software but also widely used in **hardware, network protocols, security research, AI model analysis, etc.**  

### **2️⃣ Application Scenarios of Reverse Engineering**
| **Field**      | **Applications** |
|----------------|--------------------------------------------------|
| **Software Analysis** | Reverse engineering apps, software cracking, reverse protocols, API debugging |
| **Security Research** | Malware analysis, vulnerability mining, virus analysis, web security testing |
| **Hardware Analysis** | Chip analysis, PCB design, IoT device cracking |
| **AI Reverse Engineering** | AI model analysis, weight extraction, inference optimization |
| **Game Reverse Engineering** | Game cheat development, resource extraction, network packet analysis |

**🌟 Case 1: Software Cracking**  
- You downloaded a **paid app** but don't have purchase rights. You can use reverse engineering to analyze its **verification logic** and try to bypass it.  

**🌟 Case 2: Protocol Analysis**  
- An app only supports official clients to access the API. You can reverse engineer its **API structure** and then write your own code to call it.  

### **3️⃣ Reverse Engineering vs. Forward Engineering**
|  | **Forward Engineering** | **Reverse Engineering** |
|------|----------------|----------------|
| **Thinking Approach** | Design and build | Disassemble and analyze |
| **Goal** | Develop products from 0 to 1 | Understand existing products |
| **Application Scenarios** | Software development, system design | Cracking, vulnerability mining, optimization |
| **Tools** | IDEs (VS Code, Android Studio) | Decompilers, Debuggers |

### **4️⃣ Static Analysis vs. Dynamic Analysis**
|  | **Static Analysis** | **Dynamic Analysis** |
|-------------|---------------------------------|---------------------------------|
| **Method** | Directly view file code, structure | Run the program, monitor behavior |
| **Tools** | Decompilers (jadx, Ghidra) | Debuggers (Frida, GDB, LLDB) |
| **Advantages** | Quick analysis, avoids triggering anti-debugging | Real runtime environment, observe dynamically |
| **Disadvantages** | Sometimes can't directly see the running logic | May trigger anti-debugging protection |

---

#### **🛠 Practical Tasks**
1️⃣ **Install Basic Reverse Engineering Tools**  
🔹 Download and install **jadx** (for decompiling APKs), and analyze the `classes.dex` structure.  
🔹 Install **Frida** (for hooking), and try hooking a simple Python script.  
🔹 Install **Ghidra / IDA Free**, open and view an ELF file.  

2️⃣ **Analyze a Simple APK**  
- Download any **APK file** (e.g., `Calculator.apk`).  
- Use `jadx` to decompile the APK and view the `MainActivity.java` code.  

---

#### **📚 Reference Materials**
📌 **Android Reverse Engineering Tools**  
- `jadx`: [https://github.com/skylot/jadx](https://github.com/skylot/jadx)  
- `Frida`: [https://frida.re](https://frida.re)  
- `Ghidra`: [https://ghidra-sre.org](https://ghidra-sre.org)  
- `APKTool`: [https://github.com/iBotPeaches/Apktool](https://github.com/iBotPeaches/Apktool)  

📌 **Recommended Reading**  
- "Android Software Security and Reverse Analysis"  
- "The Art of Reverse Engineering"  
- Reverse engineering blog: [https://reverseengineering.stackexchange.com](https://reverseengineering.stackexchange.com)  

---

🔥 **After completing the tasks, you will master:**  
✅ Core concepts of reverse engineering  
✅ Differences between reverse engineering and forward engineering  
✅ Installation and use of basic tools  

🚀 **Next step (Day 2)**: **History and Development of Android Reverse Engineering** 🎯  
