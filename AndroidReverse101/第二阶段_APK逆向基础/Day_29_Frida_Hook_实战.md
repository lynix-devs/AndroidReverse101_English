# **📜 Day 29: Frida Hook 实战**

## **📌 学习目标**
✅ **掌握 Frida 高级 Hook 技巧**，实战拦截、修改、绕过安全机制。  
✅ **学习如何 Hook `JNI` 层 Native 方法，修改 C/C++ 代码行为。**  
✅ **掌握 `Java.use`、`Java.choose`、`Interceptor.attach` 进行高级 Hook**。  
✅ **学习如何动态修改应用逻辑，如绕过 SSL Pinning、拦截支付接口、分析加密算法。**  
✅ **实战：使用 Frida Hook 目标方法，修改加密数据、绕过反调试机制、拦截 API 请求！**  

---

# **1️⃣ 什么是 Frida Hook 实战？**
Frida 允许我们 **动态修改 Android 应用逻辑**，在不反编译 APK 的情况下实现：
- **绕过 VIP / 登录认证**
- **拦截 HTTPS 请求**
- **绕过 SSL Pinning**
- **修改加密算法**
- **Hook JNI 层函数**

---

# **2️⃣ Hook Java 层 - 绕过支付验证**
📌 **原始 Java 代码**
```java
public boolean checkPayment(String userId, int amount) {
    return Server.verify(userId, amount);
}
```
📌 **Frida Hook**
```js
Java.perform(function() {
    var Payment = Java.use("com.example.Payment");
    Payment.checkPayment.implementation = function(userId, amount) {
        console.log("[*] Bypassing payment check for User: " + userId);
        return true;
    };
});
```
📌 **运行 Frida**
```bash
frida -U -n com.example.app -e "..."
```

---

# **3️⃣ Hook Native 层 - 修改 `JNI` 返回值**
📌 **原始 C 代码**
```c
JNIEXPORT jint JNICALL
Java_com_example_NativeLib_verify(JNIEnv *env, jobject thiz, jint code) {
    return code == 1234;
}
```
📌 **Frida Hook**
```js
Interceptor.attach(Module.findExportByName("libnative.so", "Java_com_example_NativeLib_verify"), {
    onEnter: function(args) {
        console.log("[*] Hooked JNI function - Original Code: " + args[2].toInt32());
        args[2] = ptr(9999);  // 修改传入参数
    },
    onLeave: function(retval) {
        console.log("[*] Modifying return value!");
        retval.replace(1);  // 强制返回 true
    }
});
```
📌 **运行 Frida**
```bash
frida -U -n com.example.app -e "..."
```

---

# **4️⃣ Hook SSL Pinning - 绕过 HTTPS 证书校验**
📌 **原始 Java 代码**
```java
public void checkSSLCert(X509Certificate cert) throws CertificateException {
    if (!cert.isValid()) {
        throw new CertificateException("SSL Pinning Failed!");
    }
}
```
📌 **Frida Hook**
```js
Java.perform(function() {
    var SSLChecker = Java.use("com.example.SSLChecker");
    SSLChecker.checkSSLCert.implementation = function(cert) {
        console.log("[*] Bypassing SSL Pinning...");
        return;
    };
});
```
📌 **运行 Frida**
```bash
frida -U -n com.example.app -e "..."
```

---

# **5️⃣ Hook 网络请求 - 修改 API 响应**
📌 **原始 Java 代码**
```java
URL url = new URL("https://api.example.com");
HttpURLConnection conn = (HttpURLConnection) url.openConnection();
```
📌 **Frida Hook**
```js
Java.perform(function() {
    var URL = Java.use("java.net.URL");
    URL.openConnection.implementation = function() {
        console.log("[*] Intercepted HTTP request to: " + this.toString());
        return this.openConnection();
    };
});
```

---

# **6️⃣ Hook AES 加密 - 获取加密数据**
📌 **原始 Java 代码**
```java
public String encrypt(String plaintext) {
    return AES.encrypt(plaintext, "secret_key");
}
```
📌 **Frida Hook**
```js
Java.perform(function() {
    var Crypto = Java.use("com.example.Crypto");
    Crypto.encrypt.implementation = function(plaintext) {
        console.log("[*] Intercepted encryption - Plaintext: " + plaintext);
        var result = this.encrypt(plaintext);
        console.log("[*] Encrypted data: " + result);
        return result;
    };
});
```
📌 **运行 Frida**
```bash
frida -U -n com.example.app -e "..."
```

---

# **7️⃣ 绕过反调试检测**
📌 **原始 Java 代码**
```java
public boolean detectDebugger() {
    return android.os.Debug.isDebuggerConnected();
}
```
📌 **Frida Hook**
```js
Java.perform(function() {
    var Debug = Java.use("android.os.Debug");
    Debug.isDebuggerConnected.implementation = function() {
        console.log("[*] Bypassing debugger detection");
        return false;
    };
});
```

📌 **Hook `ptrace()` 反调试**
```js
Interceptor.attach(Module.findExportByName(null, "ptrace"), {
    onEnter: function(args) {
        console.log("[*] Bypassing ptrace anti-debugging");
        args[0] = 0;
    }
});
```

---

# **🛠 实战任务**
### **✅ 1. Hook `checkPayment()`**
```js
Java.perform(function() {
    var Payment = Java.use("com.example.Payment");
    Payment.checkPayment.implementation = function(userId, amount) {
        return true;
    };
});
```
### **✅ 2. Hook JNI**
```js
Interceptor.attach(Module.findExportByName("libnative.so", "Java_com_example_NativeLib_verify"), {
    onEnter: function(args) {
        args[2] = ptr(9999);
    },
    onLeave: function(retval) {
        retval.replace(1);
    }
});
```
### **✅ 3. Hook SSL Pinning**
```js
Java.perform(function() {
    var SSLChecker = Java.use("com.example.SSLChecker");
    SSLChecker.checkSSLCert.implementation = function(cert) {
        return;
    };
});
```
### **✅ 4. Hook API 请求**
```js
Java.perform(function() {
    var URL = Java.use("java.net.URL");
    URL.openConnection.implementation = function() {
        return this.openConnection();
    };
});
```
### **✅ 5. Hook AES 加密**
```js
Java.perform(function() {
    var Crypto = Java.use("com.example.Crypto");
    Crypto.encrypt.implementation = function(plaintext) {
        console.log("[*] Encrypted data: " + this.encrypt(plaintext));
        return this.encrypt(plaintext);
    };
});
```
### **✅ 6. 绕过反调试**
```js
Java.perform(function() {
    var Debug = Java.use("android.os.Debug");
    Debug.isDebuggerConnected.implementation = function() {
        return false;
    };
});
```
### **✅ 7. 运行 Frida**
```bash
frida -U -n com.example.app -e "..."
```

---

# **📚 参考资料**
📌 **Frida Hook 教程**
- `Frida`：[https://frida.re](https://frida.re)  
- `Frida Android Hook`：[https://www.androidreversing.com/](https://www.androidreversing.com/)  

📌 **绕过 SSL Pinning**
- `Bypassing SSL Pinning`：[https://github.com/httptoolkit/android-ssl-trustkiller](https://github.com/httptoolkit/android-ssl-trustkiller)  

📌 **绕过反调试**
- `Frida Anti-Debugging Bypass`：[https://frida.re/docs/android/](https://frida.re/docs/android/)  

---

🔥 **任务完成后，你将掌握：**  
✅ **如何使用 Frida Hook Java & JNI 方法，修改应用逻辑**  
✅ **如何拦截 & 修改 API 请求，绕过 SSL Pinning**  
✅ **如何绕过 Frida 检测 & 反调试机制，成功 Hook 目标应用**  

🚀 **下一步（Day 30）**：**逆向 JNI 和 Native 方法！** 🎯