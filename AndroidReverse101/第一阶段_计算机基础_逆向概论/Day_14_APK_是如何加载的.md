# **📜 Day 14: APK 是如何加载的**

## **📌 学习目标**
✅ **理解 Android 应用的加载流程**，从 APK 安装到 Dalvik/ART 运行的全流程。  
✅ **掌握 APK 解析、DEX 加载、OAT 转换、类加载器（ClassLoader）等核心机制**。  
✅ **学习如何在 Android 设备上分析 APK 运行时行为**，掌握 `pm`, `am`, `dumpsys package` 命令。  
✅ **研究动态加载技术（DexClassLoader, PathClassLoader）及其在逆向工程中的应用**。  
✅ **使用 Frida/Xposed Hook APK 加载流程，进行安全分析与反调试**。  

---

# **1️⃣ APK 加载流程概述**
APK（Android Package）是 **Android 应用的安装包**，它的加载过程涉及多个关键步骤：

1️⃣ **APK 解析**：解析 `AndroidManifest.xml` 以确定权限、组件信息。  
2️⃣ **DEX 优化**：将 `classes.dex` 转换为 OAT/ART 文件，提升运行效率。  
3️⃣ **类加载**：使用 `ClassLoader` 机制加载 Java 类。  
4️⃣ **资源管理**：解析 `resources.arsc` 以加载 UI 资源。  
5️⃣ **Native 代码加载**：加载 `lib/` 目录下的 `.so` 本地库。  
6️⃣ **执行 `Application.onCreate()`**，启动应用。

---

# **2️⃣ APK 解析**
### **✅ 1. AndroidManifest.xml 解析**
📌 **查看已安装 APK 的 `AndroidManifest.xml`**
```bash
adb shell dumpsys package com.example.app | grep android.intent.action.MAIN
```
📌 **反编译 APK 以查看 `AndroidManifest.xml`**
```bash
apktool d app.apk -o output/
cat output/AndroidManifest.xml
```
📌 **修改 `AndroidManifest.xml` 并重新打包**
```bash
apktool b output -o modded.apk
jarsigner -verbose -keystore my.keystore modded.apk alias_name
adb install modded.apk
```

---

# **3️⃣ DEX 加载与 OAT 优化**
### **✅ 1. DEX 加载**
Android 使用 Dalvik/ART 运行时加载 `classes.dex`，执行 Java 代码。

📌 **查看 APK 内的 DEX 文件**
```bash
unzip -l app.apk | grep classes.dex
```
📌 **反编译 DEX**
```bash
jadx -d output/ app.apk
```

### **✅ 2. ART OAT 预编译**
📌 **Android 5.0+ 设备将 DEX 转换为 OAT 文件**
```bash
adb shell ls /data/dalvik-cache/
```
📌 **反编译 OAT**
```bash
oatdump --oat-file=/data/dalvik-cache/arm64/system@framework@boot.art
```

---

# **4️⃣ 类加载机制**
Android 使用 **类加载器（ClassLoader）** 加载 DEX 代码：
| **类加载器** | **作用** | **示例** |
|------------|------|------|
| **PathClassLoader** | 加载应用自带的 DEX 文件 | `system/lib/` |
| **DexClassLoader** | 运行时动态加载 DEX | `data/data/com.example.app/dex/` |

📌 **示例：动态加载 DEX**
```java
DexClassLoader loader = new DexClassLoader("/sdcard/test.dex", "/data/data/com.example.app/dex/",
        null, getClassLoader());
Class<?> clazz = loader.loadClass("com.example.MyClass");
Method method = clazz.getMethod("test");
method.invoke(null);
```
📌 **查看 ClassLoader**
```bash
adb shell dumpsys package com.example.app | grep class
```

---

# **5️⃣ 逆向分析 APK 加载**
## **✅ 1. Dump DEX 文件**
**使用 Frida 提取运行时加载的 DEX**
```js
Java.perform(function() {
    var DexClassLoader = Java.use("dalvik.system.DexClassLoader");
    DexClassLoader.loadClass.implementation = function(name) {
        console.log("Loading class: " + name);
        return this.loadClass(name);
    };
});
```
执行：
```bash
frida -U -n com.example.app -e "..."
```

---

## **✅ 2. Hook `loadClass`**
**拦截 `PathClassLoader` 加载类**
```js
Java.perform(function() {
    var PathClassLoader = Java.use("dalvik.system.PathClassLoader");
    PathClassLoader.loadClass.overload('java.lang.String').implementation = function(name) {
        console.log("Hooked class loading: " + name);
        return this.loadClass(name);
    };
});
```

---

## **✅ 3. 解析已安装 APK**
📌 **列出所有已安装应用**
```bash
adb shell pm list packages -f
```
📌 **提取 APK**
```bash
adb shell pm path com.example.app
adb pull /data/app/com.example.app/base.apk .
```
📌 **反编译**
```bash
apktool d base.apk -o output/
```

---

## **✅ 4. Hook `Application.onCreate()`**
**使用 Xposed 劫持 `Application.onCreate()`**
```java
findAndHookMethod("android.app.Application", "onCreate", new XC_MethodHook() {
    @Override
    protected void afterHookedMethod(MethodHookParam param) {
        Log.d("Xposed", "Application onCreate() called!");
    }
});
```

---

# **🛠 实战任务**
### **✅ 1. 解析 `AndroidManifest.xml`**
```bash
apktool d app.apk -o output/
cat output/AndroidManifest.xml
```
### **✅ 2. Dump 运行时加载的 DEX**
```js
Java.perform(function() {
    var DexClassLoader = Java.use("dalvik.system.DexClassLoader");
    DexClassLoader.loadClass.implementation = function(name) {
        console.log("Loading class: " + name);
        return this.loadClass(name);
    };
});
```
### **✅ 3. Hook `Application.onCreate()`**
```java
findAndHookMethod("android.app.Application", "onCreate", new XC_MethodHook() {
    @Override
    protected void afterHookedMethod(MethodHookParam param) {
        Log.d("Xposed", "Application onCreate() called!");
    }
});
```
---

# **📚 参考资料**
📌 **APK 加载**
- `官方文档`：[https://developer.android.com/guide/components](https://developer.android.com/guide/components)  
- `ClassLoader 解析`：[https://source.android.com/devices/tech/dalvik/dex-format](https://source.android.com/devices/tech/dalvik/dex-format)  

📌 **逆向分析**
- `Frida Hook Android`：[https://frida.re](https://frida.re)  
- `Xposed Hook 教程`：[https://github.com/rovo89/XposedBridge](https://github.com/rovo89/XposedBridge)  

---

🔥 **任务完成后，你将掌握：**  
✅ **APK 从安装到运行的加载机制**  
✅ **DEX/OAT 优化过程与 Hook 技术**  
✅ **如何分析、修改 APK 以进行逆向调试**  

🚀 **下一步（Day 15）**：**手写 ARM 汇编代码（实验）！** 🎯