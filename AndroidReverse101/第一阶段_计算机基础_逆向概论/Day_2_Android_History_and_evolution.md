# **📜 Day 2: The History and Evolution of Android Reverse Engineering**  

## **📌 Learning Objectives**  
✅ Understand **the evolution of Android reverse engineering**, from early APK cracking to modern app protection techniques.  
✅ Master **the cat-and-mouse game of APK protection vs. cracking**, and grasp the development of hardening and de-hardening.  
✅ Learn how to compare **old vs. new Android security mechanisms** through practical examples.  
✅ Explore **modern Android anti-debugging techniques** to prepare for reverse engineering.  

---

## **📖 Key Topics**  

### **1️⃣ The Evolution of Android Reverse Engineering**  
The development of Android reverse engineering can be divided into the following stages:  

| **Stage** | **Timeframe** | **Characteristics** |
|---------|--------|---------|
| **Early Cracking Era** | 2008 - 2012 | Simple APK structure; modifying `smali` files could bypass verification. |
| **Obfuscation & Signature Protection** | 2012 - 2015 | Developers adopted `ProGuard` for code obfuscation to prevent direct decompilation. |
| **Hardening & Dynamic Protection** | 2015 - 2018 | Companies like 360, Tencent, and Baidu introduced hardening solutions using code extraction and dynamic loading. |
| **AI & Advanced Security** | 2019 - Present | Android introduced `Play Protect` and `SafetyNet`, increasing reverse engineering difficulty. |

---

### **2️⃣ Early Android Reverse Engineering (2008 - 2012)**
#### **Example 1: Cracking VIP Membership**
Around 2010, many apps used simple `if (isVip) {}` logic to check VIP status.  

**🔹 Early Cracking Method**
- Decompile APK using `Apktool`:  
  ```bash
  apktool d my_app.apk -o output_dir
-Modify smali code to alter the isVip() method:
  ```smali
  .method public isVip()Z
    .registers 2
    const/4 v0, 0x1  # Force all users to become VIP
    return v0
.end method
  ```
- Repack and sign:
  ```bash
  apktool b output_dir -o my_hacked_app.apk
  java -jar signapk.jar cert.pem key.pk8 my_hacked_app.apk signed.apk
  ```
- Install the modified APK to bypass VIP restrictions.

---

### **The Rise of Android App Protection (2012 - 2018)**  
🔹 **Code Obfuscation (ProGuard & R8)**  
- `ProGuard` became popular, making code unreadable:
  ```java
  public class a {
      public void b() { System.out.println("Hello!"); }
  }
  ```
  **After decompilation：**
  ```java
  public class a {
      public void a() { System.out.println("Hello!"); }
  }
  ```

🔹 **Hardening Techniques (Dex Encryption & Code Extraction)**
- Code was no longer stored directly in classes.dex but loaded dynamically at runtime.
- Example: After 360 hardening, classes.dex might become libshell.so, requiring a dump to restore.
🔹 **Anti-Dynamic Analysis (Anti-Debugging & Anti-Frida)**  
- Early hooking tools (e.g., Frida) could easily modify app logic, but modern apps detect Frida:
  ```java
  public static boolean isFridaDetected() {
      return Class.forName("frida.Agent") != null;
  }
  ```

---

### **4️⃣ Modern Android Anti-Debugging (2018 - Present)**
🔹 **SafetyNet & Play Protect**  
- Google introduced Play Protect in Android 8.0+, using AI to detect malicious apps, preventing Root and Hook.
- SafetyNet checks for emulators, rooted devices, and modified systems. 

🔹 **Modern Anti-RE Techniques**
| **Security Tech** | **Function** |
|-------------|---------|
| **Code Obfuscation (ProGuard/R8)** | Makes decompiled code unreadable |
| **DEX Encryption** | Renders `classes.dex` unreadable |
| **ShellCode Dynamic Loading** | Code is decrypted at runtime, not stored in dex |
| **Anti-Debugging** | Detects debuggers (e.g., GDB, Frida) and kills the process |
| **Anti-Hook** | Detects `Frida` or `Xposed` injection |

---

## **🛠 Hands-on Tasks**
1️⃣ **Analyze an early APK**
- Download an APK from ~2012 (e.g., an old UC Browser version).
- Decompile it using apktool and inspect AndroidManifest.xml and smali code.
- Try modifying the isVip() method to bypass verification. 

2️⃣ **Analyze a modern App**
- Choose a modern app (e.g., WeChat, TikTok, Taobao) and decompile it with jadx.
- Check if ProGuard or R8 obfuscation is used.
- Test if Frida can hook the app’s logic.

---

## **📚 References**
📌 **Android Reverse Engineering Tools**
- `Apktool`：[https://github.com/iBotPeaches/Apktool](https://github.com/iBotPeaches/Apktool)  
- `Frida`：[https://frida.re](https://frida.re)  
- `Ghidra`：[https://ghidra-sre.org](https://ghidra-sre.org)  
- `Jadx`：[https://github.com/skylot/jadx](https://github.com/skylot/jadx)  
- `Xposed`：[https://repo.xposed.info/module/de.robv.android.xposed.installer](https://repo.xposed.info/module/de.robv.android.xposed.installer)  

📌 **Related Articles**
- 《Android Reverse Engineering: From Beginner to Expert》  
- Google Play Protect：[https://support.google.com/googleplay/answer/2812853](https://support.google.com/googleplay/answer/2812853)  
- Reverse Engineering Forum：[https://reverseengineering.stackexchange.com](https://reverseengineering.stackexchange.com)  

---

🔥 **After completing this module, you will:**  
✅ Understand Android reverse engineering history and key technological advancements.
✅ Be familiar with modern Android security strategies.
✅ Be able to decompile APKs using apktool and analyze their structure.

🚀 Next (Day 3): **What is a CPU Instruction Set? 🎯** 
