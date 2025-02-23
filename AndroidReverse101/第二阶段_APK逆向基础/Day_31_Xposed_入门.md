# **📜 Day 31: Xposed 入门**

## **📌 学习目标**
✅ **掌握 Xposed 框架的基本原理，理解如何使用 Xposed Hook Android 应用**。  
✅ **学习 Xposed 框架安装方法，并成功运行 Hook 模块**。  
✅ **学习如何 Hook Java 方法，修改应用逻辑，如 VIP 认证、登录校验、绕过反调试**。  
✅ **掌握 Xposed 模块开发，创建自己的 Hook 插件**。  
✅ **实战：使用 Xposed Hook 目标应用，修改 `isVIP()` 方法，绕过授权验证！**  

---

# **1️⃣ 什么是 Xposed 框架？**
Xposed 是一个 **强大的 Android Hook 框架**，允许在不修改 APK 的情况下 **动态修改应用逻辑**。

📌 **Xposed vs. Frida**
| **特性** | **Xposed** | **Frida** |
|---------|----------|----------|
| 运行方式 | 需要 Root | 仅需 Frida Server |
| Hook 层 | Java 层 | Java & Native 层 |
| 作用范围 | 作用于整个系统 | 作用于特定进程 |
| 适用场景 | **长期修改/增强功能** | **临时调试/逆向分析** |

---

# **2️⃣ 安装 Xposed 框架**
## **✅ 1. 确保设备已 Root**
📌 **检查 Root 权限**
```bash
adb shell su -c "whoami"
```
如果输出 `root`，则表示已 Root。

## **✅ 2. 安装 EdXposed**
EdXposed 是 Xposed 的增强版本，兼容 Android 8+。
📌 **下载 Magisk**
- 访问：[Magisk 官方](https://github.com/topjohnwu/Magisk)
- 安装 Magisk Manager

📌 **安装 EdXposed**
1. 下载 `EdXposed Installer.apk`
2. 在 Magisk 安装 `Riru` & `EdXposed`
3. 启动 EdXposed，确保安装成功

📌 **验证 Xposed 是否生效**
```bash
adb shell getprop | grep edxposed
```

---

# **3️⃣ 使用 Xposed Hook Java 方法**
## **✅ 1. Hook `isVIP()` 绕过会员认证**
📌 **原始 Java 代码**
```java
public boolean isVIP() {
    return false;
}
```
📌 **Xposed Hook**
```java
package com.example.hook;

import de.robv.android.xposed.IXposedHookLoadPackage;
import de.robv.android.xposed.callbacks.XC_LoadPackage;
import de.robv.android.xposed.XposedHelpers;

public class HookModule implements IXposedHookLoadPackage {
    public void handleLoadPackage(final XC_LoadPackage.LoadPackageParam lpparam) throws Throwable {
        if (!lpparam.packageName.equals("com.example.app")) {
            return;
        }
        XposedHelpers.findAndHookMethod("com.example.MainActivity", lpparam.classLoader, "isVIP",
            new XC_MethodReplacement() {
                @Override
                protected Object replaceHookedMethod(MethodHookParam param) {
                    return true;
                }
            });
    }
}
```

---

## **✅ 2. Hook 登录校验**
📌 **原始 Java 代码**
```java
public boolean checkLogin(String username, String password) {
    return username.equals("admin") && password.equals("123456");
}
```
📌 **Xposed Hook**
```java
XposedHelpers.findAndHookMethod("com.example.Auth", lpparam.classLoader, "checkLogin",
    String.class, String.class,
    new XC_MethodReplacement() {
        @Override
        protected Object replaceHookedMethod(MethodHookParam param) {
            return true;
        }
    });
```

---

## **✅ 3. Hook SSL Pinning**
📌 **原始 Java 代码**
```java
public void checkSSLCert(X509Certificate cert) throws CertificateException {
    if (!cert.isValid()) {
        throw new CertificateException("SSL Pinning Failed!");
    }
}
```
📌 **Xposed Hook**
```java
XposedHelpers.findAndHookMethod("com.example.SSLChecker", lpparam.classLoader, "checkSSLCert",
    X509Certificate.class, new XC_MethodReplacement() {
        @Override
        protected Object replaceHookedMethod(MethodHookParam param) {
            return;
        }
    });
```

---

## **✅ 4. Hook API 请求**
📌 **原始 Java 代码**
```java
URL url = new URL("https://api.example.com");
HttpURLConnection conn = (HttpURLConnection) url.openConnection();
```
📌 **Xposed Hook**
```java
XposedHelpers.findAndHookMethod("java.net.URL", lpparam.classLoader, "openConnection",
    new XC_MethodHook() {
        @Override
        protected void beforeHookedMethod(MethodHookParam param) {
            XposedBridge.log("[*] Hooked HTTP request to: " + param.thisObject.toString());
        }
    });
```

---

## **✅ 5. 绕过反调试**
📌 **原始 Java 代码**
```java
public boolean detectDebugger() {
    return android.os.Debug.isDebuggerConnected();
}
```
📌 **Xposed Hook**
```java
XposedHelpers.findAndHookMethod("android.os.Debug", lpparam.classLoader, "isDebuggerConnected",
    new XC_MethodReplacement() {
        @Override
        protected Object replaceHookedMethod(MethodHookParam param) {
            return false;
        }
    });
```

---

# **4️⃣ 编写 Xposed 模块**
## **✅ 1. 创建 Xposed Module**
📌 **新建 `AndroidManifest.xml`**
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.xposedmodule">
    <application android:allowBackup="false" android:label="Xposed Hook Module" android:theme="@style/Theme.AppCompat">
        <meta-data android:name="xposedmodule" android:value="true"/>
    </application>
</manifest>
```

## **✅ 2. 创建 `XposedInit.java`**
```java
package com.example.xposedmodule;

import de.robv.android.xposed.IXposedMod;
import de.robv.android.xposed.IXposedHookLoadPackage;
import de.robv.android.xposed.callbacks.XC_LoadPackage;

public class XposedInit implements IXposedHookLoadPackage {
    public void handleLoadPackage(final XC_LoadPackage.LoadPackageParam lpparam) throws Throwable {
        if (!lpparam.packageName.equals("com.example.app")) {
            return;
        }
        XposedBridge.log("[*] Hooking " + lpparam.packageName);
    }
}
```

## **✅ 3. 编译 & 安装**
📌 **打包 APK**
```bash
./gradlew assembleDebug
adb install app-debug.apk
```
📌 **启用 Xposed 模块**
- 打开 **EdXposed Installer**
- 选择 **模块**
- 勾选 **Xposed Hook Module**
- 重启设备

---

# **🛠 实战任务**
### **✅ 1. 安装 Xposed**
```bash
adb shell getprop | grep edxposed
```
### **✅ 2. Hook `isVIP()`**
```java
XposedHelpers.findAndHookMethod("com.example.MainActivity", lpparam.classLoader, "isVIP",
    new XC_MethodReplacement() {
        @Override
        protected Object replaceHookedMethod(MethodHookParam param) {
            return true;
        }
    });
```
### **✅ 3. 绕过 SSL Pinning**
```java
XposedHelpers.findAndHookMethod("com.example.SSLChecker", lpparam.classLoader, "checkSSLCert",
    X509Certificate.class, new XC_MethodReplacement() {
        @Override
        protected Object replaceHookedMethod(MethodHookParam param) {
            return;
        }
    });
```
### **✅ 4. Hook API 请求**
```java
XposedHelpers.findAndHookMethod("java.net.URL", lpparam.classLoader, "openConnection",
    new XC_MethodHook() {
        @Override
        protected void beforeHookedMethod(MethodHookParam param) {
            XposedBridge.log("[*] Hooked HTTP request to: " + param.thisObject.toString());
        }
    });
```

---

# **📚 参考资料**
📌 **Xposed 官方**
- `Xposed`：[http://repo.xposed.info/](http://repo.xposed.info/)  
- `EdXposed`：[https://github.com/ElderDrivers/EdXposed](https://github.com/ElderDrivers/EdXposed)  

🔥 **任务完成后，你将掌握：**  
✅ **如何使用 Xposed Hook Android 方法，修改应用逻辑**  
✅ **如何开发 Xposed 模块，实现长期 Hook 方案**  
✅ **如何绕过反调试、绕过 SSL Pinning**  

🚀 **下一步（Day 32）**：**实战：破解 VIP 限制！** 🎯