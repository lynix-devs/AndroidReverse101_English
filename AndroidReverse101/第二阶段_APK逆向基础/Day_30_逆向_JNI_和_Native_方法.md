# **📜 Day 30: 逆向 JNI 和 Native 方法**

## **📌 学习目标**
✅ **掌握 Android 中 JNI（Java Native Interface）的工作原理**，理解 Java 调用 C/C++ 代码的方式。  
✅ **学习如何逆向分析 Native 层（so 文件）代码，掌握 `Ghidra`、`IDA Pro`、`objdump` 等工具。**  
✅ **学习如何 Hook JNI 方法，修改参数、拦截返回值、绕过安全校验。**  
✅ **掌握 `Frida` 进行动态分析，修改 Native 方法返回值、绕过反调试机制。**  
✅ **实战：逆向分析 `libnative.so`，修改 `checkLicense()` 方法，绕过应用授权验证！**  

---

# **1️⃣ 什么是 JNI（Java Native Interface）？**
JNI（Java Native Interface）是 **Java 和 C/C++ 之间的桥梁**，Android APP 通过 JNI 调用 **Native 层（C/C++）代码**，通常用于：
- **提高性能**（计算密集型任务）
- **调用系统 API**
- **代码加密保护**（防止直接反编译 Java 代码）
- **使用第三方库**

📌 **JNI 调用流程**
```java
public native boolean checkLicense(String key);
static {
    System.loadLibrary("native-lib");
}
```
📌 **C 代码**
```c
JNIEXPORT jboolean JNICALL
Java_com_example_NativeLib_checkLicense(JNIEnv *env, jobject thiz, jstring key) {
    const char* key_str = (*env)->GetStringUTFChars(env, key, 0);
    if (strcmp(key_str, "VALID_LICENSE") == 0) {
        return JNI_TRUE;
    }
    return JNI_FALSE;
}
```

---

# **2️⃣ 提取 Native 库**
## **✅ 1. 提取 `libnative.so`**
📌 **查找 SO 文件**
```bash
adb shell ls /data/app/com.example.app/lib/arm64/
```
📌 **提取 SO**
```bash
adb pull /data/app/com.example.app/lib/arm64/libnative.so .
```
📌 **查看文件信息**
```bash
file libnative.so
```
示例输出：
```
libnative.so: ELF 64-bit LSB shared object, ARM aarch64
```

---

# **3️⃣ 反编译 Native 层**
## **✅ 1. 使用 `objdump` 查看 SO 符号**
```bash
objdump -T libnative.so | grep "Java_com_example"
```
示例输出：
```
0000000000001120 T Java_com_example_NativeLib_checkLicense
```

## **✅ 2. 使用 `strings` 查找关键字符串**
```bash
strings libnative.so | grep "VALID_LICENSE"
```
示例输出：
```
VALID_LICENSE
INVALID_LICENSE
```

## **✅ 3. 使用 `IDA Pro` 逆向 SO**
📌 **步骤**
1. **打开 IDA Pro**
2. **选择 `libnative.so` 并设置架构（ARM64）**
3. **找到 `checkLicense` 函数**
4. **查看字符串 `VALID_LICENSE` 交叉引用**
5. **修改 `strcmp()` 逻辑**

📌 **修改 `strcmp()`**
```c
if (strcmp(key, "VALID_LICENSE") == 0) {
    return JNI_TRUE;
}
```
📌 **Patch 修改**
- **直接改为 `return JNI_TRUE`**
- **绕过密钥验证**

---

# **4️⃣ 使用 Frida Hook JNI**
## **✅ 1. Hook `checkLicense()` 方法**
📌 **Frida Hook**
```js
Interceptor.attach(Module.findExportByName("libnative.so", "Java_com_example_NativeLib_checkLicense"), {
    onEnter: function(args) {
        console.log("[*] Hooked JNI function - License Key: " + Memory.readUtf8String(args[2]));
        args[2] = Memory.allocUtf8String("VALID_LICENSE");
    },
    onLeave: function(retval) {
        console.log("[*] Modifying return value!");
        retval.replace(1);
    }
});
```
📌 **运行 Frida**
```bash
frida -U -n com.example.app -e "..."
```

---

# **5️⃣ 绕过 Native 反调试**
某些应用会检测调试器（GDB / Frida），常见绕过方法：
- **Hook `ptrace()`**
- **修改 `anti_debug` 逻辑**

📌 **Hook `ptrace()`**
```js
Interceptor.attach(Module.findExportByName(null, "ptrace"), {
    onEnter: function(args) {
        console.log("[*] Bypassing ptrace anti-debugging");
        args[0] = 0;
    }
});
```

📌 **绕过 `isDebuggerConnected()`**
```js
Java.perform(function() {
    var Debug = Java.use("android.os.Debug");
    Debug.isDebuggerConnected.implementation = function() {
        return false;
    };
});
```

---

# **🛠 实战任务**
### **✅ 1. 提取 SO**
```bash
adb pull /data/app/com.example.app/lib/arm64/libnative.so .
```
### **✅ 2. 使用 `objdump` 查看符号**
```bash
objdump -T libnative.so | grep "Java_com_example"
```
### **✅ 3. 逆向 `checkLicense()`**
```bash
strings libnative.so | grep "VALID_LICENSE"
```
### **✅ 4. 使用 Frida Hook `checkLicense()`**
```js
Interceptor.attach(Module.findExportByName("libnative.so", "Java_com_example_NativeLib_checkLicense"), {
    onEnter: function(args) {
        args[2] = Memory.allocUtf8String("VALID_LICENSE");
    },
    onLeave: function(retval) {
        retval.replace(1);
    }
});
```
### **✅ 5. 绕过反调试**
```js
Interceptor.attach(Module.findExportByName(null, "ptrace"), {
    onEnter: function(args) {
        args[0] = 0;
    }
});
```
### **✅ 6. 运行 Frida**
```bash
frida -U -n com.example.app -e "..."
```

---

# **📚 参考资料**
📌 **Native 逆向**
- `IDA Pro`：[https://hex-rays.com/](https://hex-rays.com/)  
- `Ghidra`：[https://ghidra-sre.org/](https://ghidra-sre.org/)  

📌 **Frida Hook**
- `Frida`：[https://frida.re](https://frida.re)  

📌 **绕过反调试**
- `Frida Anti-Debugging Bypass`：[https://frida.re/docs/android/](https://frida.re/docs/android/)  

---

🔥 **任务完成后，你将掌握：**  
✅ **如何提取 & 逆向分析 Native SO 代码**  
✅ **如何使用 Frida Hook JNI 方法，修改应用逻辑**  
✅ **如何绕过 Native 反调试，成功 Hook 目标应用**  

🚀 **下一步（Day 31）**：**Xposed 入门！** 🎯