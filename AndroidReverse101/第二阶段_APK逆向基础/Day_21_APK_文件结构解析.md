# **📜 Day 21: APK 文件结构解析**

## **📌 学习目标**
✅ **深入理解 APK 文件的结构**，掌握各个组件的作用和解析方法。  
✅ **学习如何使用 `apktool`、`jadx`、`aapt` 解析 APK，查看 AndroidManifest.xml、DEX 文件、资源文件等。**  
✅ **掌握 APK 反编译与重打包技术，学习如何修改代码、绕过安全机制。**  
✅ **分析 `AndroidManifest.xml` 权限声明、入口 Activity、组件结构。**  
✅ **实战：手动拆解 APK，提取 `classes.dex`，反编译 Java 代码，修改资源文件。**  

---

# **1️⃣ 什么是 APK 文件？**
APK（Android Package）是 Android 应用的安装包，**本质上是一个 ZIP 压缩包**，包含所有 **应用代码、资源、配置文件**。

📌 **查看 APK 文件结构**
```bash
unzip -l app.apk
```
示例输出：
```
Archive:  app.apk
  Length      Date    Time    Name
---------  ---------- -----   ----
   123456  2024-02-22 12:34   AndroidManifest.xml
   234567  2024-02-22 12:34   classes.dex
   345678  2024-02-22 12:34   lib/arm64-v8a/libnative.so
   456789  2024-02-22 12:34   res/
   567890  2024-02-22 12:34   META-INF/
```
---

# **2️⃣ APK 主要结构解析**
| **文件** | **作用** |
|---------|---------|
| **AndroidManifest.xml** | 应用的核心配置文件（权限、组件、入口点） |
| **classes.dex** | Dalvik Executable 文件（应用的字节码） |
| **lib/** | 存放 `*.so` 共享库（C/C++ 编写的 Native 代码） |
| **res/** | 存放图片、布局、字符串等资源 |
| **META-INF/** | 存放签名、证书信息 |
| **assets/** | 存放未编译的资源，如字体、数据库、配置文件 |
| **resources.arsc** | 存放资源 ID，优化后的资源文件 |

---

# **3️⃣ 解析 AndroidManifest.xml**
AndroidManifest.xml 是 **Android 应用的配置文件**，包含：
- **应用权限**
- **四大组件（Activity、Service、BroadcastReceiver、ContentProvider）**
- **入口 Activity**
- **签名信息**

📌 **使用 `aapt` 查看 Manifest**
```bash
aapt dump badging app.apk
```
示例输出：
```
package: name='com.example.app' versionCode='1' versionName='1.0'
sdkVersion:'29'
targetSdkVersion:'33'
launchable-activity: name='com.example.MainActivity'
uses-permission: name='android.permission.INTERNET'
```

📌 **使用 `apktool` 反编译 Manifest**
```bash
apktool d app.apk -o output/
cat output/AndroidManifest.xml
```
示例：
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.app">
    <uses-permission android:name="android.permission.INTERNET"/>
    <application android:label="Example App">
        <activity android:name="com.example.MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
    </application>
</manifest>
```

---

# **4️⃣ 解析 DEX（Dalvik Executable）**
DEX 文件是 **Android 虚拟机（ART/Dalvik）执行的字节码**，存放 Java 代码编译后的可执行文件。

📌 **提取 DEX**
```bash
unzip app.apk classes.dex
```
📌 **使用 `jadx` 反编译 DEX**
```bash
jadx -d output/ classes.dex
```
📌 **查看 DEX 结构**
```bash
dexdump classes.dex | less
```
📌 **反编译 Java 代码**
```bash
jadx -d output/ app.apk
```
打开 `output/com/example/MainActivity.java`，可以看到：
```java
package com.example;

public class MainActivity {
    public void onCreate() {
        System.out.println("Hello, Android Reverse Engineering!");
    }
}
```

---

# **5️⃣ 解析 Native 共享库（lib/*.so）**
某些 APK **包含 C/C++ 代码**，存放于 `lib/` 目录下。

📌 **查看 SO 文件**
```bash
file lib/arm64-v8a/libnative.so
```
示例输出：
```
libnative.so: ELF 64-bit LSB shared object, ARM aarch64
```
📌 **反编译 SO**
```bash
strings lib/arm64-v8a/libnative.so | less
```
📌 **Hook SO**
```js
Java.perform(function() {
    var nativeFunc = Module.findExportByName("libnative.so", "native_func");
    Interceptor.attach(nativeFunc, {
        onEnter: function(args) {
            console.log("native_func called!");
        }
    });
});
```

---

# **6️⃣ 修改 APK 并重新打包**
## **✅ 1. 修改资源文件**
📌 **替换 APP 图标**
```bash
cp new_icon.png output/res/drawable/ic_launcher.png
```
📌 **修改 APP 名称**
编辑 `output/res/values/strings.xml`：
```xml
<string name="app_name">Hacked App</string>
```

## **✅ 2. 修改 Java 代码**
📌 **反编译并修改 `MainActivity.java`**
```java
public void onCreate() {
    System.out.println("Hacked App Launched!");
}
```

## **✅ 3. 重新打包**
```bash
apktool b output -o modded.apk
```

## **✅ 4. 重新签名**
```bash
jarsigner -verbose -keystore my.keystore modded.apk alias_name
```

## **✅ 5. 安装 APK**
```bash
adb install modded.apk
```

---

# **🛠 实战任务**
### **✅ 1. 提取 DEX**
```bash
unzip app.apk classes.dex
```
### **✅ 2. 反编译 Java**
```bash
jadx -d output/ app.apk
```
### **✅ 3. 查看 AndroidManifest.xml**
```bash
apktool d app.apk -o output/
cat output/AndroidManifest.xml
```
### **✅ 4. Hook SO**
```js
Java.perform(function() {
    var nativeFunc = Module.findExportByName("libnative.so", "native_func");
    Interceptor.attach(nativeFunc, {
        onEnter: function(args) {
            console.log("native_func called!");
        }
    });
});
```
### **✅ 5. 修改并重新打包**
```bash
apktool b output -o modded.apk
```
### **✅ 6. 重新签名**
```bash
jarsigner -verbose -keystore my.keystore modded.apk alias_name
```
### **✅ 7. 安装修改后的 APK**
```bash
adb install modded.apk
```

---

# **📚 参考资料**
📌 **APK 解析**
- `APKTool`：[https://github.com/iBotPeaches/Apktool](https://github.com/iBotPeaches/Apktool)  
- `AAPT`：[https://developer.android.com/studio/command-line/aapt2](https://developer.android.com/studio/command-line/aapt2)  

📌 **DEX 解析**
- `JADX`：[https://github.com/skylot/jadx](https://github.com/skylot/jadx)  
- `smali 代码反编译`：[https://github.com/JesusFreke/smali](https://github.com/JesusFreke/smali)  

📌 **Native 逆向**
- `Frida`：[https://frida.re](https://frida.re)  
- `Ghidra`：[https://ghidra-sre.org/](https://ghidra-sre.org/)  

---

🔥 **任务完成后，你将掌握：**  
✅ **如何解析 APK 文件，分析 Android 代码、资源、Manifest**  
✅ **如何修改 APK 资源、Java 代码，并重新打包**  
✅ **如何 Hook Native SO 文件，修改程序逻辑**  

🚀 **下一步（Day 22）**：**如何反编译 DEX 文件！** 🎯