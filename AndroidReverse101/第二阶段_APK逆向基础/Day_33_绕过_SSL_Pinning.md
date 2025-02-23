# **📜 Day 33: 绕过 SSL Pinning**

## **📌 学习目标**
✅ **掌握 SSL Pinning 的原理，理解为什么应用会校验证书**。  
✅ **学习如何使用 `Frida`、`Xposed`、`Burp Suite` 绕过 SSL Pinning**，拦截 HTTPS 请求。  
✅ **掌握 `Java.use` Hook Java 方法，动态修改 SSL 校验逻辑**。  
✅ **学习如何分析 `OkHttp`、`HttpURLConnection`、`SSLSocketFactory` 相关代码，找到 SSL Pinning 实现方式**。  
✅ **实战：使用 Frida 绕过 SSL Pinning，成功拦截 HTTPS 请求，抓取 API 数据！**  

---

# **1️⃣ 什么是 SSL Pinning？**
SSL Pinning 是一种 **防止中间人攻击（MITM）的安全机制**，Android 应用通过 SSL Pinning 绑定特定证书，使得 **仅受信任的证书才能建立 HTTPS 连接**，防止流量被劫持。

📌 **为什么要绕过 SSL Pinning？**
- **抓取 API 请求**
- **分析应用加密通信**
- **模拟 HTTP 请求，绕过认证**
- **拦截敏感数据，如 Token、密码、加密数据**

📌 **常见 SSL Pinning 实现方式**
| **方式** | **描述** | **绕过方式** |
|---------|--------|------------|
| **`OkHttp` 证书校验** | 使用 `CertificatePinner` 绑定证书 | Hook `check()` 方法 |
| **`HttpsURLConnection`** | 校验 `X509TrustManager` | Hook `checkServerTrusted()` |
| **`SSLSocketFactory`** | 直接校验证书 | Hook `createSocket()` |
| **JNI 层校验** | 通过 `libcrypto.so` 进行证书校验 | Hook `SSL_get_verify_result()` |

---

# **2️⃣ 分析应用的 SSL Pinning 代码**
## **✅ 1. 反编译 APK**
📌 **使用 `jadx` 反编译 APK**
```bash
jadx -d output/ app.apk
```
📌 **查找 `CertificatePinner` 关键字**
```bash
grep -r "CertificatePinner" output/
```
📌 **示例 Java 代码**
```java
OkHttpClient client = new OkHttpClient.Builder()
    .certificatePinner(new CertificatePinner.Builder()
        .add("api.example.com", "sha256/abcdef123456")
        .build())
    .build();
```
📌 **查找 `checkServerTrusted()`**
```bash
grep -r "checkServerTrusted" output/
```
📌 **示例 Java 代码**
```java
public void checkServerTrusted(X509Certificate[] chain, String authType) throws CertificateException {
    if (!chain[0].equals(VALID_CERT)) {
        throw new CertificateException("SSL Pinning Failed!");
    }
}
```

---

# **3️⃣ 使用 Frida 绕过 SSL Pinning**
## **✅ 1. Hook `OkHttp` 证书校验**
📌 **Frida Hook**
```js
Java.perform(function() {
    var CertificatePinner = Java.use("okhttp3.CertificatePinner");
    CertificatePinner.check.overload("java.lang.String", "java.util.List").implementation = function(hostname, peerCertificates) {
        console.log("[*] Bypassing SSL Pinning for: " + hostname);
        return;
    };
});
```
📌 **运行 Frida**
```bash
frida -U -n com.example.app -e "..."
```

---

## **✅ 2. Hook `checkServerTrusted()`**
📌 **Frida Hook**
```js
Java.perform(function() {
    var TrustManager = Java.use("javax.net.ssl.X509TrustManager");
    TrustManager.checkServerTrusted.implementation = function(chain, authType) {
        console.log("[*] Bypassing checkServerTrusted()");
        return;
    };
});
```

---

## **✅ 3. Hook `SSLSocketFactory`**
📌 **Frida Hook**
```js
Java.perform(function() {
    var SSLSocketFactory = Java.use("javax.net.ssl.SSLSocketFactory");
    SSLSocketFactory.createSocket.overload("java.net.Socket", "java.io.InputStream", "boolean").implementation = function(s, i, b) {
        console.log("[*] Bypassing SSL Pinning");
        return this.createSocket(s, i, b);
    };
});
```

---

# **4️⃣ 使用 Xposed 绕过 SSL Pinning**
📌 **Xposed Hook**
```java
XposedHelpers.findAndHookMethod("javax.net.ssl.X509TrustManager", lpparam.classLoader, "checkServerTrusted",
    X509Certificate[].class, String.class, new XC_MethodReplacement() {
        @Override
        protected Object replaceHookedMethod(MethodHookParam param) {
            return;
        }
    });
```

---

# **5️⃣ 使用 Burp Suite 拦截流量**
📌 **步骤**
1. **在 Burp Suite 设置代理**
   - 监听端口 `8080`
   - 设置 `Intercept Off`
2. **在 Android 设备上安装 Burp 证书**
   ```bash
   adb push burp_cert.der /sdcard/
   ```
3. **配置 WiFi 代理**
   - 进入 `WiFi 设置`
   - 代理地址填写 PC IP，端口 `8080`
4. **尝试抓包**
   - **未绕过 SSL Pinning** → 证书错误
   - **成功绕过** → 可看到 API 请求数据

---

# **6️⃣ 绕过 Native 层 SSL Pinning**
某些应用会在 Native 层（C/C++）进行证书校验，如 `libcrypto.so`。
📌 **Frida Hook `SSL_get_verify_result()`**
```js
Interceptor.attach(Module.findExportByName("libcrypto.so", "SSL_get_verify_result"), {
    onLeave: function(retval) {
        console.log("[*] Bypassing Native SSL Pinning");
        retval.replace(0);
    }
});
```

---

# **🛠 实战任务**
### **✅ 1. 查找 SSL Pinning 代码**
```bash
jadx -d output/ app.apk
grep -r "CertificatePinner" output/
grep -r "checkServerTrusted" output/
```
### **✅ 2. Hook `OkHttp`**
```js
Java.perform(function() {
    var CertificatePinner = Java.use("okhttp3.CertificatePinner");
    CertificatePinner.check.overload("java.lang.String", "java.util.List").implementation = function(hostname, peerCertificates) {
        return;
    };
});
```
### **✅ 3. Hook `checkServerTrusted()`**
```js
Java.perform(function() {
    var TrustManager = Java.use("javax.net.ssl.X509TrustManager");
    TrustManager.checkServerTrusted.implementation = function(chain, authType) {
        return;
    };
});
```
### **✅ 4. Hook `SSLSocketFactory`**
```js
Java.perform(function() {
    var SSLSocketFactory = Java.use("javax.net.ssl.SSLSocketFactory");
    SSLSocketFactory.createSocket.overload("java.net.Socket", "java.io.InputStream", "boolean").implementation = function(s, i, b) {
        return this.createSocket(s, i, b);
    };
});
```
### **✅ 5. Hook Native SSL Pinning**
```js
Interceptor.attach(Module.findExportByName("libcrypto.so", "SSL_get_verify_result"), {
    onLeave: function(retval) {
        retval.replace(0);
    }
});
```
### **✅ 6. 运行 Frida**
```bash
frida -U -n com.example.app -e "..."
```

---

# **📚 参考资料**
📌 **Frida Hook**
- `Frida`：[https://frida.re](https://frida.re)  

📌 **Xposed Hook**
- `Xposed`：[http://repo.xposed.info/](http://repo.xposed.info/)  

📌 **Burp Suite**
- `Burp Suite`：[https://portswigger.net/burp](https://portswigger.net/burp)  

📌 **Native SSL Pinning**
- `libcrypto.so Hook`：[https://frida.re/docs/android/](https://frida.re/docs/android/)  

---

🔥 **任务完成后，你将掌握：**  
✅ **如何分析 & 绕过 SSL Pinning，成功抓取 HTTPS 数据**  
✅ **如何使用 Frida Hook SSL Pinning 代码**  
✅ **如何使用 Xposed 绕过 `checkServerTrusted()`**  
✅ **如何 Hook Native 层 `libcrypto.so` 绕过证书校验**  

🚀 **下一步（Day 34）**：**Android 代码混淆与解混淆！** 🎯