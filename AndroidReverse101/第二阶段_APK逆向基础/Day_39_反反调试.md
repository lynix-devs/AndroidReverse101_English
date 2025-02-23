# **📜 Day 39: 反反调试**

## **📌 学习目标**
✅ **掌握 Android 应用的常见反调试（Anti-Debugging）机制，理解其工作原理**。  
✅ **学习如何绕过 `ptrace()`、`TracerPid` 检测，避免调试被中断**。  
✅ **掌握 Frida、Xposed、GDB 等工具，绕过动态调试检测**。  
✅ **学习如何修改 `libnative.so` 逆向 ELF 反调试代码**。  
✅ **实战：绕过 `ptrace()`、`syscall` 监控，成功调试目标应用！**  

---

# **1️⃣ 什么是反调试？**
Android 应用通常会使用 **反调试技术（Anti-Debugging）**，防止调试器（如 GDB、Frida）分析应用逻辑。

📌 **常见反调试检测方式**
| **方法** | **原理** | **绕过方法** |
|---------|--------|------------|
| **`ptrace()`** | 防止进程被调试 | Hook `ptrace()`，返回 `0` |
| **`TracerPid` 监测** | 读取 `/proc/self/status` 判断是否被调试 | Hook `read()` 修改返回值 |
| **`getppid()` 监测** | 检测父进程是否为调试器 | Hook `getppid()`，返回 `0` |
| **`isDebuggerConnected()`** | 调用 `Debug.isDebuggerConnected()` 检测调试状态 | Hook 方法，返回 `false` |
| **`syscall` 监控** | 监视系统调用 | Hook `syscall`，修改 `ptrace` 调用 |
| **`anti-Frida` 检测** | 扫描 Frida 进程 & 端口 | 隐藏 Frida 进程，Hook `open()` |

---

# **2️⃣ 反编译 & 分析反调试代码**
## **✅ 1. 反编译 APK**
📌 **使用 `jadx` 反编译**
```bash
jadx -d output/ app.apk
```
📌 **查找反调试代码**
```bash
grep -r "Debug" output/
grep -r "ptrace" output/
grep -r "TracerPid" output/
```
📌 **示例 Java 代码**
```java
public boolean isDebuggerConnected() {
    return android.os.Debug.isDebuggerConnected();
}
```
📌 **示例 `ptrace()` 调用**
```java
System.loadLibrary("native-lib");
public native void antiDebug();
```
📌 **Native 层 `ptrace` 代码**
```c
#include <sys/ptrace.h>
void antiDebug() {
    if (ptrace(PTRACE_TRACEME, 0, 0, 0) == -1) {
        exit(1);
    }
}
```

---

# **3️⃣ 使用 Frida 绕过反调试**
## **✅ 1. Hook `ptrace()`**
📌 **Frida Hook**
```js
Interceptor.attach(Module.findExportByName(null, "ptrace"), {
    onEnter: function(args) {
        console.log("[*] Bypassing ptrace");
        args[0] = 0;
    },
    onLeave: function(retval) {
        retval.replace(0);
    }
});
```
📌 **运行 Frida**
```bash
frida -U -n com.example.app -e "..."
```

---

## **✅ 2. Hook `isDebuggerConnected()`**
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

---

## **✅ 3. Hook `TracerPid` 读取**
📌 **Frida Hook**
```js
Interceptor.attach(Module.findExportByName(null, "read"), {
    onEnter: function(args) {
        var fd = args[0].toInt32();
        var buf = args[1];
        if (fd == "/proc/self/status") {
            console.log("[*] Bypassing TracerPid");
            Memory.writeUtf8String(buf, "TracerPid:\t0\n");
        }
    }
});
```

---

# **4️⃣ 使用 Xposed 绕过反调试**
📌 **Hook `isDebuggerConnected()`**
```java
XposedHelpers.findAndHookMethod("android.os.Debug", lpparam.classLoader, "isDebuggerConnected",
    new XC_MethodReplacement() {
        @Override
        protected Object replaceHookedMethod(MethodHookParam param) {
            return false;
        }
    });
```

📌 **Hook `getppid()`**
```java
XposedHelpers.findAndHookMethod("android.os.Process", lpparam.classLoader, "getParentPid",
    new XC_MethodReplacement() {
        @Override
        protected Object replaceHookedMethod(MethodHookParam param) {
            return 0;
        }
    });
```

---

# **5️⃣ 逆向 `libnative.so` 进行修改**
## **✅ 1. 提取 `libnative.so`**
📌 **查找 so 文件**
```bash
adb shell ls /data/app/com.example.app/lib/arm64/
```
📌 **提取 so**
```bash
adb pull /data/app/com.example.app/lib/arm64/libnative.so .
```

## **✅ 2. 反编译 `libnative.so`**
📌 **使用 IDA Pro**
1. **加载 `libnative.so`**
2. **搜索 `ptrace` 调用**
3. **修改 `CMP R0, -1` 为 `CMP R0, 0`**

📌 **使用 Ghidra**
1. **加载 `libnative.so`**
2. **搜索 `antiDebug()`**
3. **修改 `ptrace` 逻辑**

---

# **6️⃣ 绕过 Frida 反检测**
部分应用会检测 Frida 进程：
📌 **检测 Frida 进程**
```bash
ps -A | grep frida
```
📌 **Hook `open()` 隐藏 Frida**
```js
Interceptor.attach(Module.findExportByName(null, "open"), {
    onEnter: function(args) {
        var path = Memory.readUtf8String(args[0]);
        if (path.includes("frida")) {
            console.log("[*] Bypassing Frida detection");
            args[0] = Memory.allocUtf8String("/dev/null");
        }
    }
});
```

---

# **🛠 实战任务**
### **✅ 1. 反编译 & 分析 APK**
```bash
jadx -d output/ app.apk
grep -r "ptrace" output/
grep -r "TracerPid" output/
```
### **✅ 2. Hook `ptrace()`**
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
### **✅ 3. Hook `isDebuggerConnected()`**
```js
Java.perform(function() {
    var Debug = Java.use("android.os.Debug");
    Debug.isDebuggerConnected.implementation = function() {
        return false;
    };
});
```
### **✅ 4. Hook `TracerPid`**
```js
Interceptor.attach(Module.findExportByName(null, "read"), {
    onEnter: function(args) {
        var fd = args[0].toInt32();
        var buf = args[1];
        if (fd == "/proc/self/status") {
            Memory.writeUtf8String(buf, "TracerPid:\t0\n");
        }
    }
});
```
### **✅ 5. 逆向 `libnative.so`**
```bash
adb pull /data/app/com.example.app/lib/arm64/libnative.so .
```
1. **使用 IDA Pro 或 Ghidra 反编译**
2. **修改 `ptrace(PTRACE_TRACEME, 0, 0, 0)` 返回值**
3. **重新打包 & 运行 APK**

---

# **📚 参考资料**
📌 **反调试绕过**
- `Frida`：[https://frida.re](https://frida.re)  
- `Xposed`：[http://repo.xposed.info/](http://repo.xposed.info/)  

📌 **逆向分析**
- `IDA Pro`：[https://hex-rays.com/](https://hex-rays.com/)  
- `Ghidra`：[https://ghidra-sre.org/](https://ghidra-sre.org/)  

---

🔥 **任务完成后，你将掌握：**  
✅ **如何绕过 `ptrace()` 反调试**  
✅ **如何 Hook `TracerPid`，隐藏调试器**  
✅ **如何修改 `libnative.so`，逆向 ELF 反调试代码**  
✅ **如何绕过 Frida 检测，隐藏 Hook 进程**  

🚀 **下一步（Day 40）**：**Android 加固原理！** 🎯