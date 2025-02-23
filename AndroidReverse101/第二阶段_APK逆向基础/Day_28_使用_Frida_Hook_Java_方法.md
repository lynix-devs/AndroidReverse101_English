# **📜 Day 28: 使用 Frida Hook Java 方法**

## **📌 学习目标**
✅ **掌握 Frida Hook Java 方法的基本原理**，实现动态修改应用行为。  
✅ **学习如何 Hook Java 方法，修改返回值、打印函数参数、绕过安全校验。**  
✅ **掌握 `Java.use`、`Java.choose`、`Interceptor.attach` 等 Frida API 进行 Hook**。  
✅ **学习如何动态修改应用逻辑，如绕过 VIP 认证、拦截网络请求、查看敏感数据。**  
✅ **实战：使用 Frida Hook Android 方法，动态修改应用逻辑！**  

---

# **1️⃣ 什么是 Frida Hook？**
Frida 是一种 **动态调试工具**，可用于 **Hook Java / Native 方法**，修改应用逻辑。  
Frida 允许 **在应用运行时注入代码**，拦截方法调用，甚至 **修改方法参数 & 返回值**。

📌 **常见 Hook 场景**
- **绕过 VIP 认证**（`isVIP()` 方法）
- **绕过登录校验**（修改 `checkLogin()` 方法）
- **拦截网络请求**（修改 `HttpURLConnection` 方法）
- **解密数据**（Hook `decrypt()` 方法）

---

# **2️⃣ 安装 & 运行 Frida**
## **✅ 1. 安装 Frida**
📌 **在 PC 端安装 Frida**
```bash
pip install frida
pip install frida-tools
```
📌 **在 Android 设备上安装 Frida Server**
```bash
adb push frida-server /data/local/tmp/
adb shell chmod +x /data/local/tmp/frida-server
adb shell /data/local/tmp/frida-server &
```
📌 **确认 Frida 运行正常**
```bash
frida -U -n com.example.app
```
---

# **3️⃣ Hook Java 方法**
## **✅ 1. Hook `isVIP()` 绕过会员认证**
📌 **原始 Java 代码**
```java
public boolean isVIP() {
    return false;
}
```
📌 **Frida Hook**
```js
Java.perform(function() {
    var MainActivity = Java.use("com.example.MainActivity");
    MainActivity.isVIP.implementation = function() {
        console.log("[*] Hooked isVIP(), returning true");
        return true;
    };
});
```
📌 **运行 Frida**
```bash
frida -U -n com.example.app -e "..."
```

---

## **✅ 2. Hook `checkLogin()` 绕过登录验证**
📌 **原始 Java 代码**
```java
public boolean checkLogin(String username, String password) {
    return username.equals("admin") && password.equals("123456");
}
```
📌 **Frida Hook**
```js
Java.perform(function() {
    var Auth = Java.use("com.example.Auth");
    Auth.checkLogin.implementation = function(username, password) {
        console.log("[*] Intercepted login attempt - Username: " + username + ", Password: " + password);
        return true;
    };
});
```
📌 **运行 Frida**
```bash
frida -U -n com.example.app -e "..."
```

---

## **✅ 3. Hook 网络请求**
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

## **✅ 4. Hook AES 解密**
📌 **原始 Java 代码**
```java
public String decrypt(String ciphertext) {
    return AES.decrypt(ciphertext, "secret_key");
}
```
📌 **Frida Hook**
```js
Java.perform(function() {
    var Crypto = Java.use("com.example.Crypto");
    Crypto.decrypt.implementation = function(ciphertext) {
        console.log("[*] Intercepted decryption - Ciphertext: " + ciphertext);
        var result = this.decrypt(ciphertext);
        console.log("[*] Decryption result: " + result);
        return result;
    };
});
```
📌 **运行 Frida**
```bash
frida -U -n com.example.app -e "..."
```

---

# **4️⃣ 遍历 Java 类 & 方法**
📌 **查找 Java 类**
```js
Java.enumerateLoadedClasses({
    onMatch: function(className) {
        console.log(className);
    },
    onComplete: function() {
        console.log("Done!");
    }
});
```
📌 **查找方法**
```js
Java.perform(function() {
    var MainActivity = Java.use("com.example.MainActivity");
    console.log(MainActivity.class.getDeclaredMethods());
});
```

---

# **5️⃣ 绕过反 Hook 机制**
部分应用有 **Frida 检测 & 反 Hook 机制**，常见绕过方式：
- **Hook `detectFrida()`**
- **Hook `ptrace()` 反调试**
- **Patch `Frida` 检测代码**

📌 **Hook `detectFrida()`**
```js
Java.perform(function() {
    var AntiFrida = Java.use("com.example.AntiFrida");
    AntiFrida.detectFrida.implementation = function() {
        console.log("[*] Bypassing Frida detection");
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
### **✅ 1. Hook `isVIP()`**
```js
Java.perform(function() {
    var MainActivity = Java.use("com.example.MainActivity");
    MainActivity.isVIP.implementation = function() {
        return true;
    };
});
```
### **✅ 2. Hook `checkLogin()`**
```js
Java.perform(function() {
    var Auth = Java.use("com.example.Auth");
    Auth.checkLogin.implementation = function(username, password) {
        return true;
    };
});
```
### **✅ 3. Hook AES 解密**
```js
Java.perform(function() {
    var Crypto = Java.use("com.example.Crypto");
    Crypto.decrypt.implementation = function(ciphertext) {
        console.log("[*] Decryption result: " + this.decrypt(ciphertext));
        return this.decrypt(ciphertext);
    };
});
```
### **✅ 4. 绕过 Frida 检测**
```js
Java.perform(function() {
    var AntiFrida = Java.use("com.example.AntiFrida");
    AntiFrida.detectFrida.implementation = function() {
        return false;
    };
});
```
### **✅ 5. 运行 Frida**
```bash
frida -U -n com.example.app -e "..."
```

---

# **📚 参考资料**
📌 **Frida 官方文档**
- `Frida`：[https://frida.re](https://frida.re)  

📌 **Frida Hook 教程**
- `Android Hook 教程`：[https://www.androidreversing.com/](https://www.androidreversing.com/)  

📌 **绕过 Frida 检测**
- `Frida Detection Bypass`：[https://frida.re/docs/android/](https://frida.re/docs/android/)  

---

🔥 **任务完成后，你将掌握：**  
✅ **如何使用 Frida Hook Android 方法，修改应用逻辑**  
✅ **如何拦截 & 修改网络请求，解密敏感数据**  
✅ **如何绕过 Frida 检测，成功 Hook 目标应用**  

🚀 **下一步（Day 29）**：**Frida Hook 实战！** 🎯