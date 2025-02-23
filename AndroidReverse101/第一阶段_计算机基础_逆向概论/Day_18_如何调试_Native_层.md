# **📜 Day 18: 如何调试 Native 层**

## **📌 学习目标**
✅ **掌握 Android Native 层的调试方法**，包括 GDB、LLDB、Frida 远程调试。  
✅ **学习如何加载、分析和调试 Native 共享库（.so）**。  
✅ **掌握使用 GDB 进行 Native 断点调试的方法**，追踪 JNI 调用流程。  
✅ **理解 Frida 在 Native Hook 中的应用**，动态修改程序行为。  
✅ **实战：使用 GDB + Frida Hook `libnative.so`，调试 Native 方法调用**。  

---

# **1️⃣ 为什么需要调试 Native 层？**
Android 应用通常包含 Native 代码（C/C++ 编写），这些代码以 **共享库（.so）** 形式存在。  
Native 代码通常用于：
- **JNI（Java Native Interface）调用**（如音视频处理、算法库）。
- **反调试 & 反 Hook**（常见于加固应用）。
- **高性能计算（如加密/解密、数学运算）**。

---

# **2️⃣ 准备工作**
📌 **获取目标应用的进程 ID（PID）**
```bash
adb shell ps -A | grep com.example.app
```
示例输出：
```
u0_a123    1234  ...  com.example.app
```
👉 **记住 `1234` 这个进程 ID！**

📌 **获取应用的 Native 库**
```bash
adb shell ls /data/app/com.example.app/lib/arm64/
```
示例输出：
```
libnative.so
```
📌 **将 `libnative.so` 拷贝到本地**
```bash
adb pull /data/app/com.example.app/lib/arm64/libnative.so .
```

---

# **3️⃣ 使用 GDB 进行 Native 调试**
### **✅ 1. 安装 `gdbserver`**
```bash
adb shell "cp /system/bin/gdbserver /data/local/tmp/"
adb shell chmod +x /data/local/tmp/gdbserver
```

### **✅ 2. 让 `gdbserver` 附加到目标进程**
```bash
adb shell su -c "/data/local/tmp/gdbserver :12345 --attach 1234"
```
（`1234` 为目标进程 ID）

📌 **确认 `gdbserver` 在监听**
```bash
adb shell netstat -tulnp | grep 12345
```

---

### **✅ 3. 在本地启动 GDB**
📌 **将 Android NDK 提供的 GDB 拷贝到本地**
```bash
export PATH=$PATH:/path/to/android-ndk/prebuilt/linux-x86_64/bin
```

📌 **连接远程 `gdbserver`**
```bash
gdb-multiarch
```
然后在 GDB 中执行：
```gdb
target remote <设备 IP>:12345
```

📌 **加载目标库**
```gdb
add-symbol-file libnative.so 0x0000000000001000
```

---

# **4️⃣ 设置断点并调试**
📌 **列出所有符号**
```gdb
info functions
```

📌 **在 `native_func` 处设置断点**
```gdb
break native_func
```

📌 **开始调试**
```gdb
continue
```

📌 **查看寄存器**
```gdb
info registers
```

📌 **单步执行**
```gdb
stepi
```

📌 **打印内存**
```gdb
x/10xw $sp
```

---

# **5️⃣ 使用 Frida 进行动态调试**
### **✅ 1. Hook Native 函数**
📌 **Hook `libnative.so` 中的 `native_func`**
```js
var lib = Module.findExportByName("libnative.so", "native_func");
Interceptor.attach(lib, {
    onEnter: function(args) {
        console.log("native_func called!");
        console.log("Arg1: " + args[0].toInt32());
    },
    onLeave: function(retval) {
        console.log("native_func returned: " + retval.toInt32());
    }
});
```

📌 **执行 Frida**
```bash
frida -U -n com.example.app -e "..."
```

---

### **✅ 2. 修改返回值**
📌 **改变 `native_func` 的返回值**
```js
Interceptor.attach(lib, {
    onLeave: function(retval) {
        retval.replace(999);
    }
});
```

📌 **执行**
```bash
frida -U -n com.example.app -e "..."
```

---

# **6️⃣ 逆向 JNI 调用**
📌 **查看 JNI 方法表**
```bash
readelf -s libnative.so | grep Java_
```
示例输出：
```
00001234 FUNC GLOBAL DEFAULT Java_com_example_app_NativeLib_nativeMethod
```

📌 **动态 Hook JNI**
```js
var nativeMethod = Module.findExportByName("libnative.so", "Java_com_example_app_NativeLib_nativeMethod");
Interceptor.attach(nativeMethod, {
    onEnter: function(args) {
        console.log("JNI nativeMethod called!");
    }
});
```

---

# **🛠 实战任务**
### **✅ 1. 启动 `gdbserver`**
```bash
adb shell su -c "/data/local/tmp/gdbserver :12345 --attach 1234"
```
### **✅ 2. 连接 GDB**
```bash
gdb-multiarch
target remote <设备 IP>:12345
```
### **✅ 3. Hook `native_func`**
```js
Interceptor.attach(Module.findExportByName("libnative.so", "native_func"), {
    onEnter: function(args) {
        console.log("Called!");
    }
});
```
### **✅ 4. 修改返回值**
```js
Interceptor.attach(Module.findExportByName("libnative.so", "native_func"), {
    onLeave: function(retval) {
        retval.replace(999);
    }
});
```

---

# **📚 参考资料**
📌 **GDB 调试**
- `GDB 官方文档`：[https://sourceware.org/gdb/](https://sourceware.org/gdb/)  

📌 **Frida 动态调试**
- `Frida 官方文档`：[https://frida.re](https://frida.re)  

📌 **Android Native 逆向**
- `Android NDK 调试`：[https://developer.android.com/ndk](https://developer.android.com/ndk)  

---

🔥 **任务完成后，你将掌握：**  
✅ **如何使用 GDB 调试 Android Native 代码**  
✅ **如何使用 Frida Hook Native 层函数**  
✅ **如何修改 ELF 运行时行为**  

🚀 **下一步（Day 19）**：**Android APP 安全机制解析！** 🎯