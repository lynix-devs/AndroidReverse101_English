# **📜 Day 37: 破解应用限制（实战）**

## **📌 学习目标**
✅ **掌握如何绕过 Android 应用的限制，如强制更新、区域限制、功能锁定等**。  
✅ **学习如何使用 `Frida`、`Xposed`、`Smali` 逆向分析 & 破解应用限制**。  
✅ **学习如何修改 `isForceUpdate()`、`isFeatureLocked()` 方法，解锁被限制的功能**。  
✅ **掌握 API 拦截 & 修改，绕过服务器返回的限制信息**。  
✅ **实战：破解 VIP 会员、绕过区域限制、解锁隐藏功能！**  

---

# **1️⃣ 识别应用的限制**
Android 应用通常会 **限制用户访问某些功能**，常见方式：
- **强制更新**
  - 通过 `isForceUpdate()` 方法强制用户更新应用。
- **区域限制**
  - 服务器返回 `geo_blocked=true`，阻止特定国家访问。
- **功能锁定**
  - 会员功能仅对 `isVIP=true` 用户开放。

📌 **示例 Java 代码**
```java
public boolean isForceUpdate() {
    return true;  // 强制更新
}

public boolean isFeatureLocked() {
    return !user.isVIP();  // 仅 VIP 可用
}
```

---

# **2️⃣ 静态分析 & 修改 APK**
## **✅ 1. 反编译 APK**
📌 **使用 `jadx` 反编译**
```bash
jadx -d output/ app.apk
```
📌 **查找限制代码**
```bash
grep -r "isForceUpdate" output/
grep -r "isFeatureLocked" output/
```
📌 **示例 `isForceUpdate()` 方法**
```java
public boolean isForceUpdate() {
    return true;
}
```

## **✅ 2. 修改 Smali 代码**
📌 **找到 Smali 代码**
```bash
grep -r "isForceUpdate" output/smali/
```
📌 **修改 Smali**
```smali
.method public isForceUpdate()Z
    .locals 1
    const/4 v0, 0x0  # 0 = 不强制更新
    return v0
.end method
```

📌 **重新打包 APK**
```bash
apktool b output -o modded.apk
apksigner sign --ks my.keystore --out signed.apk modded.apk
adb install signed.apk
```

---

# **3️⃣ 使用 Frida 绕过应用限制**
## **✅ 1. Hook `isForceUpdate()`**
📌 **Frida Hook**
```js
Java.perform(function() {
    var MainActivity = Java.use("com.example.MainActivity");
    MainActivity.isForceUpdate.implementation = function() {
        console.log("[*] Bypassing force update");
        return false;
    };
});
```
📌 **运行 Frida**
```bash
frida -U -n com.example.app -e "..."
```

## **✅ 2. Hook `isFeatureLocked()`**
📌 **Frida Hook**
```js
Java.perform(function() {
    var FeatureManager = Java.use("com.example.FeatureManager");
    FeatureManager.isFeatureLocked.implementation = function() {
        console.log("[*] Unlocking feature");
        return false;
    };
});
```

---

# **4️⃣ 绕过区域限制**
## **✅ 1. 服务器 API 返回 `geo_blocked=true`**
📌 **API 返回**
```json
{
    "geo_blocked": true
}
```
📌 **Hook API 响应**
```js
Interceptor.attach(Module.findExportByName("libokhttp.so", "ssl_read"), {
    onLeave: function(retval) {
        console.log("[*] Intercepted API response: " + Memory.readUtf8String(retval));
        var fakeResponse = '{"geo_blocked": false}';
        Memory.writeUtf8String(retval, fakeResponse);
    }
});
```

---

# **5️⃣ Xposed 破解应用限制**
📌 **Hook `isForceUpdate()`**
```java
XposedHelpers.findAndHookMethod("com.example.MainActivity", lpparam.classLoader, "isForceUpdate",
    new XC_MethodReplacement() {
        @Override
        protected Object replaceHookedMethod(MethodHookParam param) {
            return false;
        }
    });
```

📌 **Hook `isFeatureLocked()`**
```java
XposedHelpers.findAndHookMethod("com.example.FeatureManager", lpparam.classLoader, "isFeatureLocked",
    new XC_MethodReplacement() {
        @Override
        protected Object replaceHookedMethod(MethodHookParam param) {
            return false;
        }
    });
```

---

# **6️⃣ 伪造 API 请求**
📌 **修改 HTTP 请求**
```js
Java.perform(function() {
    var Request = Java.use("okhttp3.Request");
    Request.body.implementation = function() {
        console.log("[*] Modifying API Request: " + this.body());
        return this.body().replace("user=free", "user=vip");
    };
});
```

📌 **伪造 WebSocket 消息**
```python
import websocket
ws = websocket.WebSocket()
ws.connect("wss://chat.example.com/socket")
ws.send('{"type": "feature_unlock", "status": "enabled"}')
print(ws.recv())
```

---

# **🛠 实战任务**
### **✅ 1. 反编译 & 修改 APK**
```bash
apktool d app.apk -o output/
grep -r "isForceUpdate" output/smali/
apktool b output -o modded.apk
apksigner sign --ks my.keystore --out signed.apk modded.apk
adb install signed.apk
```
### **✅ 2. Hook `isForceUpdate()`**
```js
Java.perform(function() {
    var MainActivity = Java.use("com.example.MainActivity");
    MainActivity.isForceUpdate.implementation = function() {
        return false;
    };
});
```
### **✅ 3. Hook `isFeatureLocked()`**
```js
Java.perform(function() {
    var FeatureManager = Java.use("com.example.FeatureManager");
    FeatureManager.isFeatureLocked.implementation = function() {
        return false;
    };
});
```
### **✅ 4. 伪造 API 响应**
```js
Interceptor.attach(Module.findExportByName("libokhttp.so", "ssl_read"), {
    onLeave: function(retval) {
        var fakeResponse = '{"geo_blocked": false}';
        Memory.writeUtf8String(retval, fakeResponse);
    }
});
```
### **✅ 5. 伪造 API 请求**
```js
Java.perform(function() {
    var Request = Java.use("okhttp3.Request");
    Request.body.implementation = function() {
        return this.body().replace("user=free", "user=vip");
    };
});
```
### **✅ 6. 伪造 WebSocket 消息**
```python
import websocket
ws = websocket.WebSocket()
ws.connect("wss://chat.example.com/socket")
ws.send('{"type": "feature_unlock", "status": "enabled"}')
print(ws.recv())
```

---

# **📚 参考资料**
📌 **Android 逆向**
- `jadx`：[https://github.com/skylot/jadx](https://github.com/skylot/jadx)  
- `apktool`：[https://github.com/iBotPeaches/Apktool](https://github.com/iBotPeaches/Apktool)  

📌 **动态 Hook**
- `Frida`：[https://frida.re](https://frida.re)  
- `Xposed`：[http://repo.xposed.info/](http://repo.xposed.info/)  

📌 **API 伪造**
- `Mitmproxy`：[https://mitmproxy.org/](https://mitmproxy.org/)  

---

🔥 **任务完成后，你将掌握：**  
✅ **如何修改 APK，绕过 `isForceUpdate()` 限制**  
✅ **如何使用 `Frida` Hook 方法，动态解锁功能**  
✅ **如何使用 `Xposed` 持久 Hook 方法，修改应用逻辑**  
✅ **如何拦截 & 伪造 API 响应，绕过区域限制**  

🚀 **下一步（Day 38）**：**游戏破解基础！** 🎯