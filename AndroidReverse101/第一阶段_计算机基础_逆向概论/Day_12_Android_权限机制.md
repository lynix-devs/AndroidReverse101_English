# **📜 Day 12: Android 权限机制解析**

## **📌 学习目标**
✅ **理解 Android 权限管理机制**，包括 **普通权限、危险权限、特殊权限**。  
✅ **掌握 Android 6.0+ 运行时权限模型**，学习 **动态权限申请流程**。  
✅ **学习如何在 Android 设备上查询、修改、绕过权限检查**。  
✅ **逆向分析应用权限调用，了解 Frida / Xposed 如何 Hook 权限验证**。  
✅ **分析 SELinux 在 Android 权限管理中的作用**。

---

# **1️⃣ Android 权限模型概述**
Android 采用 **基于权限的安全模型**，应用在访问敏感数据（如相机、定位、电话）时，必须 **声明并获得权限**。

**📌 权限分类**
| **权限类型** | **描述** | **示例** |
|------------|------|------|
| **普通权限（Normal）** | 低风险，安装时自动授予 | `INTERNET`, `ACCESS_NETWORK_STATE` |
| **危险权限（Dangerous）** | 涉及隐私，用户需手动授权 | `READ_CONTACTS`, `CAMERA`, `LOCATION` |
| **特殊权限（Signature/Privileged）** | 仅系统应用或特定签名的应用可使用 | `MANAGE_EXTERNAL_STORAGE`, `SYSTEM_ALERT_WINDOW` |

📌 **查看所有 Android 权限**
```bash
adb shell pm list permissions
```

---

# **2️⃣ Android 6.0+ 运行时权限模型**
📌 **Android 6.0（API 23）引入运行时权限**，危险权限需要 **动态申请**。

**📌 示例：申请 `CAMERA` 权限**
```java
if (ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA) != PackageManager.PERMISSION_GRANTED) {
    ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.CAMERA}, 100);
}
```
**📌 监听权限结果**
```java
@Override
public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
    if (requestCode == 100 && grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
        Log.d("Permission", "Camera permission granted!");
    }
}
```

---

# **3️⃣ Android 权限管理命令**
📌 **查看某应用的已授予权限**
```bash
adb shell dumpsys package com.example.app | grep permission
```

📌 **授予/撤销应用权限（需 root）**
```bash
adb shell pm grant com.example.app android.permission.CAMERA
adb shell pm revoke com.example.app android.permission.CAMERA
```

📌 **查询当前启用的 SELinux 模式**
```bash
adb shell getenforce
```
输出：
```
Enforcing  # SELinux 启用
```

📌 **禁用 SELinux（Root 权限）**
```bash
adb shell setenforce 0
```

---

# **4️⃣ 绕过权限验证**
## **✅ 1. Hook 权限检查**
📌 **使用 Frida 绕过 `checkSelfPermission`**
```js
Java.perform(function() {
    var ActivityCompat = Java.use("androidx.core.app.ActivityCompat");
    ActivityCompat.checkSelfPermission.implementation = function(context, permission) {
        console.log("Bypassing checkSelfPermission: " + permission);
        return 0;  // 直接返回 PERMISSION_GRANTED
    };
});
```
📌 **Frida 命令执行**
```bash
frida -U -n com.example.app -e "..."
```

---

## **✅ 2. 修改 `AndroidManifest.xml` 以绕过权限**
**逆向分析 APK 并修改权限**
```bash
apktool d app.apk -o decompiled/
vim decompiled/AndroidManifest.xml
```
修改：
```xml
<uses-permission android:name="android.permission.CAMERA"/>
```
重新打包：
```bash
apktool b decompiled -o modded.apk
jarsigner -verbose -keystore my.keystore modded.apk alias_name
adb install modded.apk
```

---

## **✅ 3. 绕过运行时权限**
**使用 Xposed 模块 Hook `requestPermissions`**
```java
findAndHookMethod("android.app.Activity", lpparam.classLoader, "requestPermissions",
    String[].class, int.class, new XC_MethodHook() {
        @Override
        protected void beforeHookedMethod(MethodHookParam param) {
            Log.d("Xposed", "Bypassing requestPermissions");
            param.setResult(null);
        }
    }
);
```

---

# **5️⃣ Android SELinux 权限管理**
### **🔹 SELinux 作用**
SELinux 是 Android 的 **强制访问控制（MAC）** 机制：
- **Enforcing 模式**（默认）：阻止未经授权的访问。
- **Permissive 模式**：仅记录日志，不拦截访问。

📌 **查询 SELinux 策略**
```bash
adb shell dmesg | grep avc
```

📌 **Hook SELinux 以绕过权限**
```c
int selinux_android_setcon(const char *context) {
    return 0;  // 绕过 SELinux 访问控制
}
```

---

# **🛠 实战任务**
### **✅ 1. 查询设备上的所有权限**
```bash
adb shell pm list permissions
```
### **✅ 2. 查看某应用的权限**
```bash
adb shell dumpsys package com.example.app | grep permission
```
### **✅ 3. 逆向分析 APK 权限**
```bash
apktool d app.apk -o decompiled/
vim decompiled/AndroidManifest.xml
```
### **✅ 4. Hook `checkSelfPermission` 绕过权限检查**
```js
Java.perform(function() {
    var ActivityCompat = Java.use("androidx.core.app.ActivityCompat");
    ActivityCompat.checkSelfPermission.implementation = function(context, permission) {
        console.log("Bypassing checkSelfPermission: " + permission);
        return 0;
    };
});
```

---

# **📚 参考资料**
📌 **Android 权限文档**
- `官方权限指南`：[https://developer.android.com/guide/topics/permissions/](https://developer.android.com/guide/topics/permissions/)  
- `Android 运行时权限`：[https://developer.android.com/training/permissions/requesting](https://developer.android.com/training/permissions/requesting)  

📌 **逆向工程**
- `Frida Hook Android`：[https://frida.re](https://frida.re)  
- `Xposed Hook 教程`：[https://github.com/rovo89/XposedBridge](https://github.com/rovo89/XposedBridge)  

---

🔥 **任务完成后，你将掌握：**  
✅ **Android 权限模型（普通权限、危险权限、特殊权限）**  
✅ **如何查询、修改、绕过 Android 权限**  
✅ **如何使用 Frida / Xposed Hook Android 权限检查**  
✅ **SELinux 在 Android 安全中的作用**  

🚀 **下一步（Day 13）**：**Android APP 目录结构解析！** 🎯