# **📜 Day 35: 逆向加密算法（MD5、AES、RSA）**

## **📌 学习目标**
✅ **掌握常见加密算法（MD5、AES、RSA）的基本原理**，理解其在 Android 应用中的作用。  
✅ **学习如何通过静态分析 & 动态调试，破解加密算法，解密密文**。  
✅ **掌握 Frida Hook 方法，拦截 `encrypt()` 和 `decrypt()` 方法，直接获取加密/解密数据**。  
✅ **学习如何绕过服务器签名校验，伪造加密数据，模拟 API 请求**。  
✅ **实战：逆向分析一个 APK，获取密钥 & 破解加密通信！**  

---

# **1️⃣ 什么是加密算法？**
Android 应用通常使用 **加密算法** 保护用户数据 & API 交互，防止数据被篡改。  
📌 **常见加密算法**
| **算法** | **用途** | **是否可逆** |
|---------|--------|------------|
| **MD5** | 哈希校验 | ❌ 不可逆 |
| **AES** | 数据加密 | ✅ 可逆 |
| **RSA** | 非对称加密 | ✅ 可逆 |

📌 **常见加密 API**
```java
import java.security.*;
import javax.crypto.*;

MessageDigest md = MessageDigest.getInstance("MD5");
Cipher cipher = Cipher.getInstance("AES");
KeyPairGenerator keyGen = KeyPairGenerator.getInstance("RSA");
```

---

# **2️⃣ 分析 APK 的加密逻辑**
## **✅ 1. 反编译 APK**
📌 **使用 `jadx` 反编译**
```bash
jadx -d output/ app.apk
```
📌 **查找 `MD5`、`AES`、`RSA` 关键字**
```bash
grep -r "MD5" output/
grep -r "AES" output/
grep -r "RSA" output/
```
📌 **示例 Java 代码**
```java
public static String md5(String input) {
    try {
        MessageDigest md = MessageDigest.getInstance("MD5");
        md.update(input.getBytes());
        byte[] digest = md.digest();
        return bytesToHex(digest);
    } catch (Exception e) {
        return null;
    }
}
```

---

## **✅ 2. 使用 Frida 动态分析**
📌 **Hook `MessageDigest.getInstance("MD5")`**
```js
Java.perform(function() {
    var MessageDigest = Java.use("java.security.MessageDigest");
    MessageDigest.getInstance.implementation = function(algorithm) {
        console.log("[*] Hooked getInstance: " + algorithm);
        return this.getInstance(algorithm);
    };
});
```
📌 **运行 Frida**
```bash
frida -U -n com.example.app -e "..."
```

---

# **3️⃣ 逆向 MD5 加密**
📌 **示例 MD5 哈希**
```java
public static String md5(String input) {
    try {
        MessageDigest md = MessageDigest.getInstance("MD5");
        md.update(input.getBytes());
        byte[] digest = md.digest();
        return bytesToHex(digest);
    } catch (Exception e) {
        return null;
    }
}
```
📌 **破解 MD5**
```python
import hashlib
hash = hashlib.md5(b"password").hexdigest()
print(hash)
```

📌 **Hook MD5 计算**
```js
Java.perform(function() {
    var MessageDigest = Java.use("java.security.MessageDigest");
    MessageDigest.digest.implementation = function() {
        var result = this.digest();
        console.log("[*] Intercepted MD5: " + result);
        return result;
    };
});
```

---

# **4️⃣ 逆向 AES 加密**
📌 **示例 AES 加密**
```java
public static byte[] encryptAES(String key, String data) throws Exception {
    SecretKeySpec secretKey = new SecretKeySpec(key.getBytes(), "AES");
    Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
    cipher.init(Cipher.ENCRYPT_MODE, secretKey);
    return cipher.doFinal(data.getBytes());
}
```
📌 **Hook AES 加密**
```js
Java.perform(function() {
    var Cipher = Java.use("javax.crypto.Cipher");
    Cipher.doFinal.overload("[B").implementation = function(data) {
        console.log("[*] Intercepted AES Encrypt: " + data);
        return this.doFinal(data);
    };
});
```

📌 **解密 AES**
```python
from Crypto.Cipher import AES
import base64

key = b'Sixteen byte key'
cipher = AES.new(key, AES.MODE_ECB)
encrypted = cipher.encrypt(b"hello world!!!")
print(base64.b64encode(encrypted))
```

---

# **5️⃣ 逆向 RSA 加密**
📌 **示例 RSA 加密**
```java
public static byte[] encryptRSA(String data, PublicKey pubKey) throws Exception {
    Cipher cipher = Cipher.getInstance("RSA/ECB/PKCS1Padding");
    cipher.init(Cipher.ENCRYPT_MODE, pubKey);
    return cipher.doFinal(data.getBytes());
}
```
📌 **Hook RSA 加密**
```js
Java.perform(function() {
    var Cipher = Java.use("javax.crypto.Cipher");
    Cipher.doFinal.overload("[B").implementation = function(data) {
        console.log("[*] Intercepted RSA Encrypt: " + data);
        return this.doFinal(data);
    };
});
```

📌 **解密 RSA**
```python
from Crypto.PublicKey import RSA
from Crypto.Cipher import PKCS1_v1_5
import base64

private_key = RSA.importKey(open("private.pem").read())
cipher = PKCS1_v1_5.new(private_key)
decrypted = cipher.decrypt(base64.b64decode(encrypted_data), None)
print(decrypted)
```

---

# **6️⃣ 破解 API 加密校验**
某些应用在 API 请求时使用 AES/RSA 进行加密校验，必须破解：
📌 **拦截 API 请求**
```bash
adb logcat -s "OkHttp" | grep "encrypt"
```
📌 **Hook 网络加密**
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

# **🛠 实战任务**
### **✅ 1. 反编译 APK**
```bash
jadx -d output/ app.apk
grep -r "AES" output/
grep -r "RSA" output/
```
### **✅ 2. Hook MD5**
```js
Java.perform(function() {
    var MessageDigest = Java.use("java.security.MessageDigest");
    MessageDigest.digest.implementation = function() {
        console.log("[*] Intercepted MD5: " + this.digest());
        return this.digest();
    };
});
```
### **✅ 3. Hook AES**
```js
Java.perform(function() {
    var Cipher = Java.use("javax.crypto.Cipher");
    Cipher.doFinal.overload("[B").implementation = function(data) {
        console.log("[*] Intercepted AES Encrypt: " + data);
        return this.doFinal(data);
    };
});
```
### **✅ 4. Hook RSA**
```js
Java.perform(function() {
    var Cipher = Java.use("javax.crypto.Cipher");
    Cipher.doFinal.overload("[B").implementation = function(data) {
        console.log("[*] Intercepted RSA Encrypt: " + data);
        return this.doFinal(data);
    };
});
```
### **✅ 5. 拦截 API 加密**
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

# **📚 参考资料**
📌 **加密破解**
- `Frida`：[https://frida.re](https://frida.re)  
- `Crypto Python`：[https://pycryptodome.readthedocs.io/](https://pycryptodome.readthedocs.io/)  

---

🔥 **任务完成后，你将掌握：**  
✅ **如何分析 APK 内的加密算法**  
✅ **如何使用 Frida Hook 加密方法，解密数据**  
✅ **如何破解 API 加密校验，伪造加密数据**  

🚀 **下一步（Day 36）**：**分析 WebSocket & API 请求！** 🎯