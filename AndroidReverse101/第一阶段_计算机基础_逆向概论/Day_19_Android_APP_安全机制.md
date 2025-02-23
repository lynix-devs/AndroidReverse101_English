# **📜 Day 19: Android APP 安全机制**

## **📌 学习目标**
✅ **理解 Android 安全机制**，包括 **应用沙盒（Sandbox）、SELinux、签名验证、Root 检测** 等。  
✅ **掌握 Android APP 代码保护方式**，如 **混淆（ProGuard）、加固、动态调试检测**。  
✅ **学习 Android 存储安全**，避免 **数据泄露（SharedPreferences、SQLite、文件存储）**。  
✅ **掌握动态分析技巧**，绕过 **Root 检测、反调试、加固保护**，提高逆向能力。  
✅ **实战：分析并绕过 Android APP 的安全防护**。

---

# **1️⃣ Android 应用沙盒（Sandbox）**
**🔹 什么是应用沙盒？**
- 每个 Android APP 运行在 **独立的 Linux 进程** 中，具有 **独立的 UID/GID**。
- **不同应用的文件、数据、进程默认不能互相访问**。

📌 **查看某应用的 UID**
```bash
adb shell dumpsys package com.example.app | grep userId=
```
示例输出：
```
userId=10123
```
👉 **每个 APP 都有唯一的 UID，防止数据泄露。**

📌 **尝试访问其他 APP 目录**
```bash
adb shell ls /data/data/com.other.app/
```
**如果未 Root，访问会被拒绝！**

---

# **2️⃣ SELinux（强制访问控制）**
SELinux（Security-Enhanced Linux）用于限制 **进程间通信、访问系统资源**。

📌 **查看 SELinux 状态**
```bash
adb shell getenforce
```
示例输出：
```
Enforcing  # 说明 SELinux 已启用
```
📌 **禁用 SELinux（Root 权限）**
```bash
adb shell setenforce 0
```

📌 **查看 SELinux 拒绝的访问**
```bash
adb shell dmesg | grep avc
```

---

# **3️⃣ Android 签名验证**
Android APP 在安装时需要 **数字签名** 以确保应用完整性。

📌 **检查 APK 签名**
```bash
apksigner verify --print-certs app.apk
```
示例输出：
```
Signer #1 certificate DN: CN=Developer, O=Example Corp, C=US
```
📌 **绕过签名校验**
在 `AndroidManifest.xml` 添加：
```xml
<uses-permission android:name="android.permission.INSTALL_PACKAGES"/>
```
然后重新签名：
```bash
zipalign -v 4 modded.apk aligned.apk
apksigner sign --ks my.keystore --out signed.apk aligned.apk
adb install signed.apk
```

---

# **4️⃣ Root 检测**
许多应用 **检测设备是否 Root** 以防止被逆向分析。

📌 **常见 Root 检测方法**
1. **检查 su**
```java
File f = new File("/system/bin/su");
if (f.exists()) {
    Log.d("RootCheck", "Device is Rooted!");
}
```

2. **检测 Root 进程**
```bash
adb shell ps -A | grep magiskd
```
示例输出：
```
magiskd       1023  567   123456K fg    00000000 S magiskd
```

📌 **绕过 Root 检测**
**使用 Frida Hook `File.exists()`**
```js
Java.perform(function() {
    var File = Java.use("java.io.File");
    File.exists.implementation = function() {
        return false;
    };
});
```

📌 **执行 Frida**
```bash
frida -U -n com.example.app -e "..."
```

---

# **5️⃣ 代码混淆 & 加固**
### **✅ 1. ProGuard 混淆**
```proguard
-dontobfuscate
-keep class com.example.** { *; }
```
📌 **反混淆 DEX**
```bash
jadx -d output/ app.apk
```

### **✅ 2. 加固**
加固（如 `360加固`, `腾讯加固`）用于 **隐藏代码逻辑，防止逆向**。

📌 **检查 APK 是否加固**
```bash
unzip -l app.apk | grep classes
```
如果 `classes.dex` 被拆分或替换，则 **APK 可能被加固**。

📌 **绕过加固（Frida Dump DEX）**
```bash
frida -U -n com.example.app -e "Java.perform(function() {
    Java.use('dalvik.system.BaseDexClassLoader').loadClass.implementation = function(name) {
        console.log('Load Class: ' + name);
    };
});"
```

---

# **6️⃣ Android 存储安全**
### **✅ 1. 避免明文存储**
❌ **不安全**
```java
SharedPreferences prefs = getSharedPreferences("settings", MODE_PRIVATE);
prefs.edit().putString("password", "123456").apply();
```
📌 **查看存储数据**
```bash
adb shell cat /data/data/com.example.app/shared_prefs/settings.xml
```
**✅ 解决方案：使用加密存储**
```java
EncryptedSharedPreferences encryptedPrefs = EncryptedSharedPreferences.create(
    "settings",
    MasterKeys.getOrCreate(MasterKeys.AES256_GCM_SPEC),
    context,
    EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
    EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
);
encryptedPrefs.edit().putString("password", "123456").apply();
```

---

# **7️⃣ 逆向绕过安全机制**
## **✅ 1. 绕过 Root 检测**
```js
Java.perform(function() {
    var File = Java.use("java.io.File");
    File.exists.implementation = function() {
        return false;
    };
});
```

## **✅ 2. 绕过反调试**
```js
Java.perform(function() {
    var Debug = Java.use("android.os.Debug");
    Debug.isDebuggerConnected.implementation = function() {
        return false;
    };
});
```

## **✅ 3. Dump DEX**
```js
Java.perform(function() {
    var DexClassLoader = Java.use("dalvik.system.DexClassLoader");
    DexClassLoader.loadClass.implementation = function(name) {
        console.log("Hooked Dex Load: " + name);
        return this.loadClass(name);
    };
});
```

---

# **🛠 实战任务**
### **✅ 1. 检查应用的 UID**
```bash
adb shell dumpsys package com.example.app | grep userId=
```
### **✅ 2. 禁用 SELinux**
```bash
adb shell setenforce 0
```
### **✅ 3. Hook `File.exists()` 绕过 Root 检测**
```js
Java.perform(function() {
    var File = Java.use("java.io.File");
    File.exists.implementation = function() {
        return false;
    };
});
```
### **✅ 4. Dump DEX**
```js
Java.perform(function() {
    var DexClassLoader = Java.use("dalvik.system.DexClassLoader");
    DexClassLoader.loadClass.implementation = function(name) {
        console.log("Hooked Dex Load: " + name);
        return this.loadClass(name);
    };
});
```

---

# **📚 参考资料**
📌 **Android 安全机制**
- `SELinux`：[https://source.android.com/security/selinux](https://source.android.com/security/selinux)  
- `ProGuard`：[https://developer.android.com/studio/build/shrink-code](https://developer.android.com/studio/build/shrink-code)  
- `Android 签名机制`：[https://developer.android.com/studio/publish/app-signing](https://developer.android.com/studio/publish/app-signing)  

📌 **逆向分析**
- `Frida`：[https://frida.re](https://frida.re)  
- `Xposed Hook 教程`：[https://github.com/rovo89/XposedBridge](https://github.com/rovo89/XposedBridge)  

---

🔥 **任务完成后，你将掌握：**  
✅ **Android 沙盒机制、SELinux、安全存储方法**  
✅ **如何绕过 Root 检测、签名校验、反调试**  
✅ **如何 Dump DEX、分析代码混淆与加固**  

🚀 **下一步（Day 20）**：**CTF 逆向挑战（初级）！** 🎯