# **📜 Day 22: 如何反编译 APK**

## **📌 学习目标**
✅ **理解 Android APK 反编译的核心概念**，掌握不同的反编译工具和方法。  
✅ **学习如何提取 `classes.dex` 并转换回 Java 代码**，掌握 `jadx`、`baksmali`、`dex2jar` 等工具。  
✅ **掌握 `AndroidManifest.xml` 反编译、资源文件提取、Smali 代码分析**。  
✅ **学习 APK 反编译后的修改技巧，包括绕过加密、修改逻辑、重打包**。  
✅ **掌握 SO 共享库（Native 层）的反编译分析，使用 `IDA Pro` 和 `Ghidra` 进行逆向工程**。  
✅ **实战：反编译 APK，提取 Java 代码，分析 SO 文件，修改逻辑，重新打包运行！**  

---

# **1️⃣ 什么是 APK 反编译？**
APK 反编译是指将编译后的 APK 文件 **转换回可读代码**，从而：
- 提取 Java 代码（`classes.dex` → `.java`）。
- 查看资源文件（布局 XML、图片、字符串）。
- 分析 `AndroidManifest.xml` 组件和权限信息。
- 逆向分析 SO 共享库（ELF 文件），找出敏感逻辑和加密算法。
- 修改 APP 逻辑，移除广告、破解 VIP 限制等。

---

# **2️⃣ APK 反编译的三种方法**
| **方法** | **适用场景** | **工具** |
|---------|----------|------|
| **Java 反编译** | 还原 Java 代码 | `jadx`, `dex2jar` |
| **Smali 反编译** | 修改 Dalvik 字节码 | `apktool`, `smali` |
| **Native 逆向** | 逆向分析 `.so` 文件 | `IDA Pro`, `Ghidra` |

---

# **3️⃣ 提取 APK 代码**
## **✅ 1. 提取 APK**
📌 **获取已安装 APK**
```bash
adb shell pm list packages | grep example
adb shell pm path com.example.app
adb pull /data/app/com.example.app-1/base.apk .
```

## **✅ 2. 提取 `classes.dex`**
```bash
unzip base.apk classes.dex
```

---

# **4️⃣ Java 代码反编译**
## **✅ 1. 使用 `jadx`**
📌 **安装 `jadx`**
```bash
git clone https://github.com/skylot/jadx.git
cd jadx
./gradlew dist
```
📌 **反编译 DEX**
```bash
jadx -d output/ base.apk
```
📌 **查看反编译代码**
```bash
cat output/com/example/MainActivity.java
```
示例输出：
```java
package com.example;

public class MainActivity {
    public void onCreate() {
        System.out.println("Hello, Reverse Engineering!");
    }
}
```

---

## **✅ 2. 使用 `dex2jar` + `JD-GUI`**
📌 **安装 `dex2jar`**
```bash
wget https://github.com/pxb1988/dex2jar/releases/download/v2.0/dex-tools-2.0.zip
unzip dex-tools-2.0.zip
```
📌 **转换 DEX 为 JAR**
```bash
./d2j-dex2jar.sh classes.dex
```
📌 **使用 JD-GUI 打开**
```bash
jd-gui classes-dex2jar.jar
```

---

# **5️⃣ Smali 代码反编译**
## **✅ 1. 使用 `apktool`**
📌 **安装 `apktool`**
```bash
wget https://raw.githubusercontent.com/iBotPeaches/Apktool/master/scripts/linux/apktool
chmod +x apktool
```
📌 **反编译 APK**
```bash
apktool d base.apk -o output/
```
📌 **查看 Smali 代码**
```bash
cat output/smali/com/example/MainActivity.smali
```
示例：
```smali
.method public onCreate()V
    .locals 1
    const-string v0, "Hello, Reverse Engineering!"
    invoke-static {v0}, Ljava/lang/System;->println(Ljava/lang/String;)V
.end method
```

---

## **✅ 2. 修改 Smali 代码**
📌 **修改 `MainActivity.smali`**
```smali
.method public onCreate()V
    .locals 1
    const-string v0, "Hacked by Reverse Engineer!"
    invoke-static {v0}, Ljava/lang/System;->println(Ljava/lang/String;)V
.end method
```
📌 **重新打包**
```bash
apktool b output -o modded.apk
```
📌 **重新签名**
```bash
jarsigner -verbose -keystore my.keystore modded.apk alias_name
```
📌 **安装 APK**
```bash
adb install modded.apk
```

---

# **6️⃣ 解析 SO 共享库（Native 层）**
📌 **提取 SO 文件**
```bash
adb pull /data/data/com.example.app/lib/arm64/libnative.so .
```
📌 **查看 ELF 结构**
```bash
readelf -h libnative.so
```
📌 **查看导出函数**
```bash
nm -D libnative.so
```

---

## **✅ 1. 使用 IDA Pro 分析 SO**
📌 **步骤**
1. **打开 IDA Pro**
2. **选择 `libnative.so` 并设置架构（ARM/ARM64）**
3. **分析代码，查找关键函数**
4. **查找字符串**
   - `Shift + F12` 查看所有字符串
   - `X` 查看交叉引用

📌 **示例**
```c
if (strcmp(input, "SecretKey") == 0) {
    return "FLAG{IDA_NATIVE_REVERSE}";
}
```
**思路**：搜索 `"SecretKey"` 字符串，找到关键验证逻辑，修改二进制或 Hook 该函数。

---

## **✅ 2. 使用 Ghidra 反编译 SO**
📌 **步骤**
1. **打开 Ghidra**
2. **新建工程，导入 `libnative.so`**
3. **选择处理器架构（ARM/ARM64）**
4. **使用 `Decompiler` 还原 C 代码**
5. **查找 `strcmp()` 调用，分析逻辑**

📌 **示例**
```c
bool check_password(char* input) {
    return strcmp(input, "SuperSecretKey") == 0;
}
```
**思路**：可以使用 Frida 修改 `strcmp()` 返回值，实现绕过验证。

---

## **✅ 3. Hook SO 运行时修改代码**
📌 **Hook `check_password()`**
```js
Java.perform(function() {
    var nativeFunc = Module.findExportByName("libnative.so", "check_password");
    Interceptor.attach(nativeFunc, {
        onEnter: function(args) {
            console.log("check_password called!");
            args[0] = ptr("HackedPassword");
        }
    });
});
```
📌 **运行 Frida**
```bash
frida -U -n com.example.app -e "..."
```

---

# **🛠 实战任务**
### **✅ 1. 提取 `classes.dex`**
```bash
unzip base.apk classes.dex
```
### **✅ 2. 反编译 Java**
```bash
jadx -d output/ base.apk
```
### **✅ 3. 反编译 Smali**
```bash
apktool d base.apk -o output/
cat output/smali/com/example/MainActivity.smali
```
### **✅ 4. 修改 VIP 方法**
```smali
.method public isVIP()Z
    const/4 v0, 0x1
    return v0
.end method
```
### **✅ 5. 解析 SO**
```bash
readelf -h libnative.so
nm -D libnative.so
```
### **✅ 6. 使用 IDA Pro 分析**
- 打开 `libnative.so`
- 查找 `strcmp()` 调用
- 修改返回值

---

# **📚 参考资料**
📌 **APK 反编译**
- `IDA Pro`：[https://hex-rays.com/](https://hex-rays.com/)  
- `Ghidra`：[https://ghidra-sre.org/](https://ghidra-sre.org/)  
- `Frida`：[https://frida.re](https://frida.re)  

---

🔥 **任务完成后，你将掌握：**  
✅ **如何反编译 Java & Smali 代码**  
✅ **如何逆向分析 Native SO 代码**  
✅ **如何使用 IDA Pro、Frida Hook、Ghidra 进行逆向调试**  

🚀 **下一步（Day 23）**：**DEX 文件结构解析！** 🎯