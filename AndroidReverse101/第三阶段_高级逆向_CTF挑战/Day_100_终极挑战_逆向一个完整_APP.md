## **📜 Day 100: 终极挑战 - 逆向一个完整 APP**

## **📌 学习目标**
✅ **掌握 Android 逆向工程的三大核心技能：静态分析、抓包分析、动态分析**。  
✅ **学会使用 `jadx`、`IDA Pro`、`Frida`、`Xposed`、`Wireshark`、`Mitmproxy` 等工具进行完整逆向分析**。  
✅ **学习如何绕过登录认证、Hook 关键方法、解密存储数据、拦截 API 通信、破解 VIP 限制**。  
✅ **深入分析 Native 层，提取 `libnative.so`，绕过 `ptrace()` 反调试**。  
✅ **最终成功提取 `Flag`，完成 Android 逆向 100 天的终极挑战！**  

---

## **📜 1. 逆向三板斧**
在 Android 逆向分析中，**最常用的方法有三种**：
1. **静态分析（Static Analysis）**  
   - 通过 **反编译 APK**，直接阅读代码，找到关键逻辑，例如 **登录验证、加密算法、API URL**。
2. **抓包分析（Network Analysis）**  
   - 通过 **Wireshark、Burp Suite、Mitmproxy** 监控和拦截网络通信，伪造 API 响应，分析 `Token` 机制。
3. **动态调试（Dynamic Analysis）**  
   - 通过 **Frida/Xposed Hook**，修改运行时逻辑，例如 **绕过登录验证、伪造 VIP 会员、修改 SharedPreferences**。

📌 **完整逆向流程**
1. **提取目标 APP**
2. **静态分析：解包 APK，阅读代码**
3. **抓包分析：拦截 API 通信**
4. **动态分析：Hook 关键方法，修改应用行为**
5. **绕过安全机制，提取最终 `Flag`**
6. **验证 & 复盘，撰写完整 `Writeup`**

---

## **📜 2. 静态分析：提取 & 反编译 APK**
### **✅ 1. 获取目标 APK**
📌 **从设备提取 APK**
```bash
adb shell pm list packages | grep target.app
adb shell pm path com.example.target.app
adb pull /data/app/com.example.target.app/base.apk .
```
📌 **解压 APK**
```bash
unzip base.apk -d output/
```
📌 **检查 APK 目录结构**
```bash
tree output/
```
**关键文件**
- `AndroidManifest.xml` → **权限 & 入口点**
- `classes.dex` → **DEX 字节码**
- `res/` → **资源文件**
- `assets/` → **可能存储加密数据**
- `lib/` → **可能包含 Native SO 库**

---

### **✅ 2. 反编译 APK**
📌 **使用 `jadx` 反编译 DEX**
```bash
jadx -d decompiled/ base.apk
```
📌 **搜索关键代码**
```bash
grep -r "login" decompiled/
grep -r "encrypt" decompiled/
grep -r "Flag{" decompiled/
```
📌 **分析代码**
- 找到 `checkLogin()` 方法，分析登录逻辑
- 查找 `AES 加密算法`，尝试解密存储数据
- 发现 `API 请求 URL`，为抓包做准备

---

### **✅ 3. 逆向加密算法**
📌 **示例 AES 加密代码**
```java
public static byte[] encryptAES(String key, String data) throws Exception {
    SecretKeySpec secretKey = new SecretKeySpec(key.getBytes(), "AES");
    Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
    cipher.init(Cipher.ENCRYPT_MODE, secretKey);
    return cipher.doFinal(data.getBytes());
}
```
📌 **Python 解密**
```python
from Crypto.Cipher import AES
import base64

key = b'Sixteen byte key'
cipher = AES.new(key, AES.MODE_ECB)
encrypted = cipher.encrypt(b"hello world!!!")
print(base64.b64encode(encrypted))
```

---

## **📜 3. 抓包分析：拦截 & 伪造 API**
### **✅ 1. 设置代理**
📌 **使用 Burp Suite 监听流量**
```bash
adb shell settings put global http_proxy 192.168.1.2:8080
```
📌 **安装 Burp 证书**
```bash
adb push burp_cert.der /sdcard/
```

### **✅ 2. 绕过 SSL Pinning**
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

### **✅ 3. 拦截 API 请求**
📌 **分析 `OkHttp`**
```bash
grep -r "okhttp3" decompiled/
```
📌 **Frida Hook**
```js
Java.perform(function() {
    var Request = Java.use("okhttp3.Request");
    Request.body.implementation = function() {
        console.log("[*] Intercepted API Request: " + this.body());
        return this.body();
    };
});
```

---

## **📜 4. 动态分析：Hook 关键方法**
### **✅ 1. 绕过登录认证**
📌 **查找 `checkLogin()`**
```bash
grep -r "checkLogin" decompiled/
```
📌 **Frida Hook**
```js
Java.perform(function() {
    var Auth = Java.use("com.example.app.Auth");
    Auth.checkLogin.implementation = function(username, password) {
        console.log("[*] Bypassing checkLogin()");
        return true;
    };
});
```

### **✅ 2. 破解 VIP 功能**
📌 **Hook `isVip()`**
```js
Java.perform(function() {
    var User = Java.use("com.example.app.User");
    User.isVip.implementation = function() {
        console.log("[*] Unlocking VIP features");
        return true;
    };
});
```

### **✅ 3. 修改 SharedPreferences**
📌 **Frida Hook**
```js
Java.perform(function() {
    var SharedPreferences = Java.use("android.content.SharedPreferences");
    SharedPreferences.getBoolean.implementation = function(key, defValue) {
        console.log("[*] Bypassing VIP check");
        return true;
    };
});
```

---

## **📜 5. 逆向 Native 层 & 提取 Flag**
### **✅ 1. 反编译 `libnative.so`**
📌 **使用 IDA Pro**
1. **加载 `libnative.so`**
2. **搜索 `flag{}`**
3. **修改 `strcmp()` 比较逻辑**

### **✅ 2. Hook `getFlag()`**
📌 **Frida Hook**
```js
Java.perform(function() {
    var Challenge = Java.use("com.example.app.Challenge");
    Challenge.getFlag.implementation = function() {
        console.log("[*] Extracting Flag!");
        return "FLAG{Final_Challenge_Solved}";
    };
});
```

---

# **🎯 挑战完成！你已成功掌握 Android 逆向工程！** 🚀