# **📜 Day 41: 解密加固 APK（初级）**

## **📌 学习目标**
✅ **掌握加固 APK 的解密原理，理解加固的工作机制**。  
✅ **学习如何使用 `Frida`、`Xposed`、`GDB` 进行 DEX 提取和解密**。  
✅ **掌握 `DumpDex`、`gdb`、`IDA Pro` 等工具进行解密分析**。  
✅ **实战：解密一个 360 加固 APK，提取原始 DEX 并进行反编译！**  

---

# **1️⃣ 加固 APK 的解密原理**
加固的主要目的是 **加密 DEX 文件**，防止静态分析：
- **加密 DEX**
  - `classes.dex` 经过 AES/RSA 加密存储在 `assets` 或 `lib` 目录中。
- **运行时解密**
  - 加固壳 `libjiagu.so` 在运行时解密并加载 DEX。
- **Anti-Dump 机制**
  - 使用 `ptrace()` 保护，防止 Dump 内存。

📌 **解密思路**
1. **找到 DEX 加载时机**
2. **Hook `DexClassLoader`，Dump 解密后的 DEX**
3. **使用 `gdb` 或 `Frida` 提取 DEX**

---

# **2️⃣ 检测 APK 是否加密**
## **✅ 1. 解压 APK 查看文件**
📌 **解压 APK**
```bash
unzip app.apk -d output/
```
📌 **检查 `classes.dex` 是否存在**
```bash
ls output/classes.dex
```
📌 **如果 `classes.dex` 不存在，说明 DEX 可能被加密**

---

## **✅ 2. 使用 `APKiD` 检测加密**
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
 - 360加固 detected
 - DexClassLoader used
```

---

## **✅ 3. 运行时检查 DEX 加载**
📌 **使用 Frida 监听 DEX 加载**
```js
Java.perform(function() {
    var DexClassLoader = Java.use("dalvik.system.DexClassLoader");
    DexClassLoader.loadClass.overload("java.lang.String").implementation = function(className) {
        console.log("[*] Loading class: " + className);
        return this.loadClass(className);
    };
});
```
📌 **运行 Frida**
```bash
frida -U -n com.example.app -e "..."
```
📌 **如果发现 DEX 被动态加载，则需要 Dump 运行时内存！**

---

# **3️⃣ 使用 Frida DumpDex 提取 DEX**
## **✅ 1. 安装 `Frida DumpDex`**
📌 **下载 DumpDex**
```bash
git clone https://github.com/hluwa/FRIDA-DEXDump
cd FRIDA-DEXDump
```
📌 **运行 `DumpDex`**
```bash
frida -U -n com.example.app -e "DexDump.dump()"
```
📌 **提取解密 DEX**
```bash
adb pull /sdcard/dump.dex
```
📌 **反编译 `dump.dex`**
```bash
jadx -d output/ dumped.dex
```

---

# **4️⃣ 手动解密加密 DEX**
## **✅ 1. 使用 `gdb` Dump 进程内存**
📌 **附加到进程**
```bash
gdb -p $(pidof com.example.app)
```
📌 **查找 DEX 加载地址**
```bash
info proc mappings
```
📌 **Dump 进程内存**
```bash
dump memory dumped.dex 0x12345678 0x87654321
```
📌 **反编译**
```bash
jadx -d output/ dumped.dex
```

---

# **5️⃣ 逆向 `libjiagu.so` 解密 DEX**
## **✅ 1. 提取 `libjiagu.so`**
📌 **查找加密库**
```bash
adb shell ls /data/app/com.example.app/lib/arm64/
```
📌 **提取 so**
```bash
adb pull /data/app/com.example.app/lib/arm64/libjiagu.so .
```

## **✅ 2. 使用 IDA Pro 分析**
📌 **步骤**
1. **加载 `libjiagu.so`**
2. **搜索 `AES_decrypt()` 或 `DexClassLoader`**
3. **Patch 代码，让其直接 dump 解密数据**

---

# **6️⃣ 绕过 Anti-Dump 机制**
## **✅ 1. Hook `ptrace()`**
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

## **✅ 2. Hook `isDebuggerConnected()`**
📌 **Frida Hook**
```js
Java.perform(function() {
    var Debug = Java.use("android.os.Debug");
    Debug.isDebuggerConnected.implementation = function() {
        return false;
    };
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

# **🛠 实战任务**
### **✅ 1. 检测 APK 是否加密**
```bash
apkid app.apk
ls output/classes.dex
```
### **✅ 2. 运行 Frida 监听 DEX 加载**
```js
Java.perform(function() {
    var DexClassLoader = Java.use("dalvik.system.DexClassLoader");
    DexClassLoader.loadClass.overload("java.lang.String").implementation = function(className) {
        console.log("[*] Loading class: " + className);
        return this.loadClass(className);
    };
});
```
### **✅ 3. Dump DEX**
```bash
frida -U -n com.example.app -e "DexDump.dump()"
adb pull /sdcard/dump.dex
jadx -d output/ dumped.dex
```
### **✅ 4. 使用 `gdb` Dump 进程**
```bash
gdb -p $(pidof com.example.app)
info proc mappings
dump memory dumped.dex 0x12345678 0x87654321
```
### **✅ 5. 逆向 `libjiagu.so`**
```bash
adb pull /data/app/com.example.app/lib/arm64/libjiagu.so .
```
1. **使用 IDA Pro 反编译**
2. **搜索 `AES_decrypt()`**
3. **Patch 代码，Dump 解密后的 DEX**
### **✅ 6. 绕过 Anti-Dump**
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

---

# **📚 参考资料**
📌 **加密检测**
- `APKiD`：[https://github.com/rednaga/APKiD](https://github.com/rednaga/APKiD)  

📌 **DEX Dump**
- `Frida DumpDex`：[https://github.com/hluwa/FRIDA-DEXDump](https://github.com/hluwa/FRIDA-DEXDump)  

📌 **逆向分析**
- `IDA Pro`：[https://hex-rays.com/](https://hex-rays.com/)  
- `Ghidra`：[https://ghidra-sre.org/](https://ghidra-sre.org/)  

---

🔥 **任务完成后，你将掌握：**  
✅ **如何检测 APK 是否加密**  
✅ **如何使用 `Frida` 提取解密 DEX**  
✅ **如何使用 `gdb` Dump 内存中的 DEX**  
✅ **如何逆向 `libjiagu.so`，分析解密流程**  
✅ **如何绕过 Anti-Dump 机制，成功解密 APK**  

🚀 **下一步（Day 42）**：**解密加固 APK（进阶）！** 🎯