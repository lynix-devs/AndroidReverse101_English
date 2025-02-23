# **📜 Day 40: Android 加固原理**

## **📌 学习目标**
✅ **掌握 Android 应用加固（Application Hardening）的基本原理**，理解主流加固技术。  
✅ **学习常见的 Android 加固方案，如 360 加固、腾讯乐固、爱加密等**。  
✅ **学习如何检测 APK 是否被加固，并分析加固后的 APK 结构**。  
✅ **掌握加固脱壳（Dump Dex）的方法，提取原始 DEX 进行分析**。  
✅ **实战：绕过加固，提取并分析未加固的应用代码！**  

---

# **1️⃣ 什么是 Android 加固？**
Android 加固（Application Hardening）是一种 **保护应用代码** 的技术，主要目的是 **防止逆向工程、反调试、静态分析**。  
**加固方式通常包括：**
1. **DEX 加密**：将 `classes.dex` 加密存储，运行时解密加载。
2. **壳（Shell）保护**：加固壳劫持 `Application` 启动过程，动态解密 DEX。
3. **反调试 & 反 Hook**：检测调试器 & Frida，阻止调试和 Hook。
4. **JNI 加密**：使用 `libshell.so` 进行代码保护，隐藏关键逻辑。
5. **动态代码加载**：通过 `DexClassLoader` 在运行时加载加密代码。

📌 **常见的加固服务**
| **加固方案** | **特点** | **脱壳难度** |
|------------|--------|--------|
| **360 加固** | 国外流行，DEX 加密，加载 `libjiagu.so` | 🔥🔥🔥 |
| **腾讯乐固** | 使用 `libturing.so` 加载解密的 DEX | 🔥🔥 |
| **爱加密** | DEX 加密 & 代码变形，部分代码 Native 实现 | 🔥🔥🔥🔥 |

---

# **2️⃣ 检测 APK 是否加固**
## **✅ 1. 查看 APK 结构**
📌 **解压 APK**
```bash
unzip app.apk -d output/
```
📌 **检查是否存在 `libjiagu.so` 或 `libturing.so`**
```bash
ls output/lib/arm64-v8a/
```
📌 **如果发现这些库，说明应用已被加固**
```
libjiagu.so
libturing.so
libshell.so
```

## **✅ 2. 使用 `APKiD` 识别加固方案**
📌 **安装 `APKiD`**
```bash
pip install apkid
```
📌 **分析 APK**
```bash
apkid app.apk
```
📌 **示例输出**
```
APKID results:
 - Tencent Legu detected
 - DexClassLoader used
 - Anti-Debugging techniques detected
```

## **✅ 3. 使用 `frida -U -n` 运行时检测**
📌 **运行 Frida**
```bash
frida -U -n com.example.app -e "console.log('[*] Running Frida detection')"
```
📌 **如果应用崩溃，说明有 Frida 反调试**

---

# **3️⃣ Android 加固脱壳（Dump DEX）**
## **✅ 1. 使用 `Frida DumpDex`**
📌 **安装 `DumpDex`**
```bash
git clone https://github.com/hluwa/FRIDA-DEXDump
cd FRIDA-DEXDump
```
📌 **运行 `DumpDex`**
```bash
frida -U -n com.example.app -e "DexDump.dump()"
```
📌 **提取 `dumped.dex`**
```bash
adb pull /sdcard/dump.dex
```
📌 **反编译 `dumped.dex`**
```bash
jadx -d output/ dumped.dex
```

---

## **✅ 2. 使用 `Xposed` Hook `DexClassLoader`**
📌 **Hook `loadClass`，拦截动态加载的 DEX**
```java
XposedHelpers.findAndHookMethod("dalvik.system.DexClassLoader", lpparam.classLoader, "loadClass",
    String.class, new XC_MethodHook() {
        @Override
        protected void afterHookedMethod(MethodHookParam param) {
            String className = (String) param.args[0];
            XposedBridge.log("[*] Loaded Class: " + className);
        }
    });
```

---

# **4️⃣ 绕过反调试 & 反 Frida**
## **✅ 1. Hook `isDebuggerConnected()`**
📌 **Frida Hook**
```js
Java.perform(function() {
    var Debug = Java.use("android.os.Debug");
    Debug.isDebuggerConnected.implementation = function() {
        console.log("[*] Bypassing isDebuggerConnected()");
        return false;
    };
});
```

## **✅ 2. Hook `ptrace()`**
📌 **Frida Hook**
```js
Interceptor.attach(Module.findExportByName(null, "ptrace"), {
    onEnter: function(args) {
        args[0] = 0;
    },
    onLeave: function(retval) {
        retval.replace(0);
    }
});
```

## **✅ 3. 绕过 `getppid()`**
📌 **Frida Hook**
```js
Interceptor.attach(Module.findExportByName(null, "getppid"), {
    onLeave: function(retval) {
        retval.replace(0);
    }
});
```

---

# **5️⃣ 手动脱壳（Dump 内存 DEX）**
## **✅ 1. 使用 `IDA Pro` 查找 `DexClassLoader`**
1. **打开 `libjiagu.so`**
2. **搜索 `DexClassLoader`**
3. **找到 `loadClass()` 并 Patch**

## **✅ 2. 使用 `gdb` Dump 进程内存**
📌 **附加到目标进程**
```bash
gdb -p $(pidof com.example.app)
```
📌 **查找 DEX 加载地址**
```bash
info proc mappings
```
📌 **Dump DEX**
```bash
dump memory dumped.dex 0x12345678 0x87654321
```

---

# **🛠 实战任务**
### **✅ 1. 检测 APK 是否加固**
```bash
apkid app.apk
ls output/lib/arm64-v8a/
```
### **✅ 2. 使用 `Frida DumpDex`**
```bash
frida -U -n com.example.app -e "DexDump.dump()"
adb pull /sdcard/dump.dex
jadx -d output/ dumped.dex
```
### **✅ 3. Hook `DexClassLoader`**
```java
XposedHelpers.findAndHookMethod("dalvik.system.DexClassLoader", lpparam.classLoader, "loadClass",
    String.class, new XC_MethodHook() {
        @Override
        protected void afterHookedMethod(MethodHookParam param) {
            XposedBridge.log("[*] Loaded Class: " + className);
        }
    });
```
### **✅ 4. Hook `ptrace()`**
```js
Interceptor.attach(Module.findExportByName(null, "ptrace"), {
    onEnter: function(args) {
        args[0] = 0;
    },
    onLeave: function(retval) {
        retval.replace(0);
    }
});
```
### **✅ 5. Dump 进程内存**
```bash
gdb -p $(pidof com.example.app)
info proc mappings
dump memory dumped.dex 0x12345678 0x87654321
```

---

# **📚 参考资料**
📌 **加固检测**
- `APKiD`：[https://github.com/rednaga/APKiD](https://github.com/rednaga/APKiD)  

📌 **脱壳工具**
- `Frida DumpDex`：[https://github.com/hluwa/FRIDA-DEXDump](https://github.com/hluwa/FRIDA-DEXDump)  

📌 **逆向分析**
- `IDA Pro`：[https://hex-rays.com/](https://hex-rays.com/)  
- `Ghidra`：[https://ghidra-sre.org/](https://ghidra-sre.org/)  

---

🔥 **任务完成后，你将掌握：**  
✅ **如何检测 APK 是否加固**  
✅ **如何使用 `Frida`、`Xposed` 绕过加固**  
✅ **如何提取原始 `DEX` 代码进行分析**  
✅ **如何手动 Dump 进程内存，破解加固保护**  

🚀 **下一步（Day 41）**：**解密加固 APK（初级）！** 🎯