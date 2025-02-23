# **📜 Day 27: 动态调试入门**

## **📌 学习目标**
✅ **掌握动态调试（Dynamic Debugging）的基本概念**，了解与静态分析的区别。  
✅ **学习如何使用 `adb logcat` 监控应用日志，分析应用行为。**  
✅ **掌握 `gdb`、`lldb`、`Frida`、`Xposed` 等调试工具，动态修改应用逻辑。**  
✅ **学习如何设置断点、Hook 关键方法、修改函数返回值、绕过反调试保护。**  
✅ **实战：使用 Frida Hook 目标方法，动态修改返回值 & 绕过登录认证！**  

---

# **1️⃣ 什么是动态调试？**
动态调试（Dynamic Debugging）是一种 **在应用运行时分析和修改代码行为** 的技术，  
不同于静态分析（直接反编译代码），它通过 **Hook 方法、设置断点、监控日志** 等方式实时修改代码执行。

📌 **动态调试 vs. 静态分析**
| **方法** | **优点** | **缺点** |
|---------|--------|-------|
| **静态分析** | 直接查看代码结构 | 不能实时修改行为 |
| **动态调试** | 运行时修改代码逻辑 | 需要附加调试器，部分应用有反调试机制 |

---

# **2️⃣ 监控 Android 应用日志**
## **✅ 1. 使用 `adb logcat`**
📌 **过滤特定应用日志**
```bash
adb logcat -s "com.example.app"
```
📌 **查看 `System.out.println()` 输出**
```bash
adb logcat | grep "Hello Debugging"
```
📌 **查看崩溃日志**
```bash
adb logcat *:E
```

## **✅ 2. 使用 `logcat` 监控函数调用**
📌 **插入 `Log.d` 语句**
```smali
invoke-static {}, Landroid/util/Log;->d(Ljava/lang/String;Ljava/lang/String;)I
```
📌 **过滤特定关键字**
```bash
adb logcat -s "LoginActivity"
```

---

# **3️⃣ 使用 Frida 进行动态调试**
## **✅ 1. 安装 Frida**
📌 **在 Android 设备上安装 Frida**
```bash
adb push frida-server /data/local/tmp/
adb shell chmod +x /data/local/tmp/frida-server
adb shell /data/local/tmp/frida-server &
```
📌 **在 PC 端连接 Frida**
```bash
frida -U -n com.example.app -i
```

## **✅ 2. Hook 方法 & 修改返回值**
📌 **Hook `isVIP()`**
```js
Java.perform(function() {
    var MainActivity = Java.use("com.example.MainActivity");
    MainActivity.isVIP.implementation = function() {
        console.log("Bypassing VIP check...");
        return true;
    };
});
```
📌 **运行**
```bash
frida -U -n com.example.app -e "..."
```

## **✅ 3. Hook `getPackageInfo()` 绕过签名校验**
```js
Java.perform(function() {
    var PackageManager = Java.use("android.content.pm.PackageManager");
    PackageManager.getPackageInfo.implementation = function(pkg, flags) {
        console.log("Bypassing signature check for:", pkg);
        return this.getPackageInfo(pkg, flags);
    };
});
```

---

# **4️⃣ 使用 GDB / LLDB 调试 Native 层**
## **✅ 1. 附加到进程**
📌 **获取进程 ID**
```bash
adb shell ps | grep com.example.app
```
📌 **附加 GDB**
```bash
gdb -p <pid>
```

## **✅ 2. 设置断点**
📌 **设置 `strcmp()` 断点**
```gdb
b strcmp
```
📌 **继续执行**
```gdb
c
```

---

# **5️⃣ 绕过反调试**
某些应用会检测调试器（GDB / Frida），常见绕过方法：
- **修改 `ptrace()` 调用**
- **Patch 反调试 Smali 代码**
- **使用 `Frida` Patch `anti-debug` 方法**

📌 **Hook `ptrace()` 绕过调试检测**
```js
Interceptor.attach(Module.findExportByName(null, "ptrace"), {
    onEnter: function(args) {
        console.log("Bypassing ptrace anti-debugging");
        args[0] = 0;
    }
});
```

---

# **🛠 实战任务**
### **✅ 1. 监控应用日志**
```bash
adb logcat -s "com.example.app"
```
### **✅ 2. Hook `isVIP()`**
```js
Java.perform(function() {
    var MainActivity = Java.use("com.example.MainActivity");
    MainActivity.isVIP.implementation = function() {
        return true;
    };
});
```
### **✅ 3. Hook `getPackageInfo()` 绕过签名校验**
```js
Java.perform(function() {
    var PackageManager = Java.use("android.content.pm.PackageManager");
    PackageManager.getPackageInfo.implementation = function(pkg, flags) {
        return this.getPackageInfo(pkg, flags);
    };
});
```
### **✅ 4. 绕过 `ptrace()` 反调试**
```js
Interceptor.attach(Module.findExportByName(null, "ptrace"), {
    onEnter: function(args) {
        args[0] = 0;
    }
});
```

---

# **📚 参考资料**
📌 **动态调试**
- `Frida`：[https://frida.re](https://frida.re)  
- `GDB`：[https://sourceware.org/gdb/](https://sourceware.org/gdb/)  
- `LLDB`：[https://lldb.llvm.org/](https://lldb.llvm.org/)  

📌 **绕过反调试**
- `Android Anti-Debugging`：[https://www.androidreversing.com/](https://www.androidreversing.com/)  

---

🔥 **任务完成后，你将掌握：**  
✅ **如何使用 `adb logcat` 监控日志，分析应用行为**  
✅ **如何使用 Frida Hook 目标方法，动态修改返回值**  
✅ **如何使用 GDB/LLDB 进行 Native 调试，分析 C/C++ 代码**  
✅ **如何绕过反调试机制，成功 Hook 关键方法**  

🚀 **下一步（Day 28）**：**使用 Frida Hook Java 方法！** 🎯