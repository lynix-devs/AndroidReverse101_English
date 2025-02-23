# **📜 Day 25: Smali 代码修改实验**

## **📌 学习目标**
✅ **掌握 Smali 代码修改技巧**，学习如何修改 Android 应用逻辑。  
✅ **学习如何修改方法返回值、去除 VIP 认证、绕过登录验证、解锁隐藏功能。**  
✅ **掌握 Smali 代码的常见修改方式，如 `if` 语句篡改、字符串替换、修改 `invoke` 方法调用。**  
✅ **通过 `apktool` 反编译 APK，修改 Smali 代码，并重新打包 & 安装。**  
✅ **实战：修改 `isVIP()` 方法，绕过会员认证，去除广告，修改应用文字！**  

---

# **1️⃣ Smali 代码修改基础**
## **✅ 1. Smali 方法返回值修改**
**目标**：将 `isVIP()` 方法的返回值从 `false` 改为 `true`。

📌 **反编译 APK**
```bash
apktool d app.apk -o output/
```
📌 **查找 `isVIP()` 方法**
```bash
grep -r "isVIP" output/smali/
```
📌 **原始 Smali 代码**
```smali
.method public isVIP()Z
    .locals 1
    const/4 v0, 0x0  # 0 = false
    return v0
.end method
```
📌 **修改 Smali 代码**
```smali
.method public isVIP()Z
    .locals 1
    const/4 v0, 0x1  # 1 = true
    return v0
.end method
```
📌 **重新打包**
```bash
apktool b output -o modded.apk
```
📌 **签名 & 安装**
```bash
jarsigner -verbose -keystore my.keystore modded.apk alias_name
adb install modded.apk
```

---

## **✅ 2. 绕过登录验证**
📌 **查找 `checkLogin()` 方法**
```bash
grep -r "checkLogin" output/smali/
```
📌 **原始 Smali 代码**
```smali
.method public checkLogin(Ljava/lang/String;Ljava/lang/String;)Z
    .locals 2
    invoke-static {p1, p2}, Lcom/example/AuthUtils;->verify(Ljava/lang/String;Ljava/lang/String;)Z
    move-result v0
    return v0
.end method
```
📌 **修改 Smali 代码**
```smali
.method public checkLogin(Ljava/lang/String;Ljava/lang/String;)Z
    .locals 1
    const/4 v0, 0x1  # 直接返回 true
    return v0
.end method
```
📌 **重新打包 & 安装**
```bash
apktool b output -o modded.apk
jarsigner -verbose -keystore my.keystore modded.apk alias_name
adb install modded.apk
```

---

## **✅ 3. 去除广告**
📌 **查找 `showAd()` 方法**
```bash
grep -r "showAd" output/smali/
```
📌 **原始 Smali 代码**
```smali
.method public showAd()V
    .locals 1
    const-string v0, "com.example.ads.AdsManager"
    invoke-static {v0}, Ljava/lang/Class;->forName(Ljava/lang/String;)Ljava/lang/Class;
    return-void
.end method
```
📌 **修改 Smali 代码**
```smali
.method public showAd()V
    .locals 0
    return-void  # 直接返回，不显示广告
.end method
```

---

## **✅ 4. 修改应用文本**
📌 **查找 `strings.xml`**
```bash
grep -r "VIP" output/res/values/strings.xml
```
📌 **修改文本**
```xml
<string name="vip_member">超级会员</string>
```
📌 **重新打包 & 安装**
```bash
apktool b output -o modded.apk
adb install modded.apk
```

---

# **2️⃣ Hook Smali 运行时**
## **✅ 1. Hook `checkLogin()`**
📌 **Frida Hook**
```js
Java.perform(function() {
    var AuthUtils = Java.use("com.example.AuthUtils");
    AuthUtils.verify.implementation = function(user, pass) {
        return true; // 绕过登录
    };
});
```
📌 **运行 Frida**
```bash
frida -U -n com.example.app -e "..."
```

---

# **🛠 实战任务**
### **✅ 1. 反编译 APK**
```bash
apktool d app.apk -o output/
```
### **✅ 2. 修改 VIP 方法**
```smali
.method public isVIP()Z
    const/4 v0, 0x1
    return v0
.end method
```
### **✅ 3. 绕过登录**
```smali
.method public checkLogin(Ljava/lang/String;Ljava/lang/String;)Z
    const/4 v0, 0x1
    return v0
.end method
```
### **✅ 4. 去除广告**
```smali
.method public showAd()V
    return-void
.end method
```
### **✅ 5. 修改文本**
```xml
<string name="vip_member">超级会员</string>
```
### **✅ 6. 重新打包 & 安装**
```bash
apktool b output -o modded.apk
adb install modded.apk
```

---

# **📚 参考资料**
📌 **Smali 修改**
- `Smali 参考文档`：[https://github.com/JesusFreke/smali](https://github.com/JesusFreke/smali)  

📌 **APK 反编译**
- `APKTool`：[https://github.com/iBotPeaches/Apktool](https://github.com/iBotPeaches/Apktool)  

📌 **运行时 Hook**
- `Frida`：[https://frida.re](https://frida.re)  

---

🔥 **任务完成后，你将掌握：**  
✅ **如何修改 Smali 代码，绕过 VIP、登录验证**  
✅ **如何去除广告，修改应用文本**  
✅ **如何 Hook 运行时，动态修改应用行为**  

🚀 **下一步（Day 26）**：**APK 重新打包 & 签名！** 🎯