# **📜 Day 32: 破解 VIP 限制**

## **📌 学习目标**
✅ **掌握如何通过静态 & 动态分析破解 Android 应用的 VIP 限制**。  
✅ **学习如何使用 `jadx`、`apktool` 反编译 APK，查找 VIP 认证逻辑**。  
✅ **学习如何使用 `Frida`、`Xposed` Hook VIP 方法，动态修改应用逻辑**。  
✅ **掌握修改 Smali 代码 & 重新打包签名，实现永久 VIP 破解**。  
✅ **实战：破解 VIP 会员系统，解锁高级功能！**  

---

# **1️⃣ 分析 VIP 限制**
大多数 Android 应用的 VIP 认证逻辑有以下几种方式：
- **本地存储判断**
  - 直接在 SharedPreferences / SQLite 数据库中存储 VIP 状态。
- **API 请求服务器验证**
  - 服务器返回 `is_vip=true`，应用根据返回值解锁 VIP。
- **JNI / SO 层认证**
  - VIP 逻辑被隐藏在 `libnative.so`，通过 JNI 调用。
- **License Key 校验**
  - 应用需要输入 Key 或 Token 进行验证。

---

# **2️⃣ 静态分析 VIP 方法**
## **✅ 1. 反编译 APK**
📌 **使用 `apktool` 反编译**
```bash
apktool d app.apk -o output/
```
📌 **查找 VIP 相关关键字**
```bash
grep -r "vip" output/smali/
```
📌 **示例 VIP 方法**
```smali
.method public isVIP()Z
    .locals 1
    const/4 v0, 0x0  # 0 = false (非 VIP)
    return v0
.end method
```
📌 **修改 VIP 逻辑**
```smali
.method public isVIP()Z
    .locals 1
    const/4 v0, 0x1  # 1 = true (VIP)
    return v0
.end method
```
📌 **重新打包 & 签名**
```bash
apktool b output -o modded.apk
apksigner sign --ks my.keystore --out signed.apk modded.apk
adb install signed.apk
```

---

# **3️⃣ 动态 Hook VIP 方法**
## **✅ 1. Hook `isVIP()` 方法**
📌 **使用 Frida**
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

## **✅ 2. 使用 Xposed Hook VIP**
📌 **Xposed Hook**
```java
XposedHelpers.findAndHookMethod("com.example.MainActivity", lpparam.classLoader, "isVIP",
    new XC_MethodReplacement() {
        @Override
        protected Object replaceHookedMethod(MethodHookParam param) {
            return true;
        }
    });
```
📌 **启用 Xposed 模块 & 重启**

---

# **4️⃣ 绕过服务器 VIP 校验**
📌 **分析 API**
```bash
adb logcat -s "OkHttp" | grep "vip"
```
📌 **Hook `HttpURLConnection`，修改返回值**
```js
Java.perform(function() {
    var URL = Java.use("java.net.URL");
    URL.openConnection.implementation = function() {
        console.log("[*] Intercepted HTTP request to: " + this.toString());
        return this.openConnection();
    };
});
```
📌 **拦截 API 响应**
```js
Interceptor.attach(Module.findExportByName("libokhttp.so", "ssl_read"), {
    onLeave: function(retval) {
        console.log("[*] Original Response: " + Memory.readUtf8String(retval));
        var fakeResponse = '{"is_vip": true}';
        Memory.writeUtf8String(retval, fakeResponse);
    }
});
```

---

# **5️⃣ 绕过 JNI 层 VIP 校验**
📌 **查找 JNI 方法**
```bash
objdump -T libnative.so | grep "Java_com_example"
```
📌 **Frida Hook JNI**
```js
Interceptor.attach(Module.findExportByName("libnative.so", "Java_com_example_NativeLib_isVIP"), {
    onLeave: function(retval) {
        retval.replace(1);
    }
});
```

---

# **🛠 实战任务**
### **✅ 1. 反编译 & 修改 APK**
```bash
apktool d app.apk -o output/
grep -r "vip" output/smali/
apktool b output -o modded.apk
apksigner sign --ks my.keystore --out signed.apk modded.apk
adb install signed.apk
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
### **✅ 3. Hook API VIP 认证**
```js
Interceptor.attach(Module.findExportByName("libokhttp.so", "ssl_read"), {
    onLeave: function(retval) {
        var fakeResponse = '{"is_vip": true}';
        Memory.writeUtf8String(retval, fakeResponse);
    }
});
```
### **✅ 4. Hook JNI VIP 校验**
```js
Interceptor.attach(Module.findExportByName("libnative.so", "Java_com_example_NativeLib_isVIP"), {
    onLeave: function(retval) {
        retval.replace(1);
    }
});
```

---

# **📚 参考资料**
📌 **APK 反编译**
- `APKTool`：[https://github.com/iBotPeaches/Apktool](https://github.com/iBotPeaches/Apktool)  
- `JADX`：[https://github.com/skylot/jadx](https://github.com/skylot/jadx)  

📌 **动态 Hook**
- `Frida`：[https://frida.re](https://frida.re)  
- `Xposed`：[http://repo.xposed.info/](http://repo.xposed.info/)  

📌 **JNI 逆向**
- `IDA Pro`：[https://hex-rays.com/](https://hex-rays.com/)  
- `Ghidra`：[https://ghidra-sre.org/](https://ghidra-sre.org/)  

---

🔥 **任务完成后，你将掌握：**  
✅ **如何分析 & 破解 VIP 限制，解锁高级功能**  
✅ **如何使用 Smali 代码修改 VIP 认证逻辑**  
✅ **如何使用 Frida / Xposed Hook VIP 方法，动态绕过 VIP 校验**  
✅ **如何拦截 & 修改服务器返回的 VIP 认证信息**  
✅ **如何绕过 JNI 层 VIP 逻辑，实现完全破解**  

🚀 **下一步（Day 33）**：**绕过 SSL Pinning！** 🎯