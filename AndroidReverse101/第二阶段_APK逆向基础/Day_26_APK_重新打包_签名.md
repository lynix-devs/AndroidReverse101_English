# **📜 Day 26: APK 重新打包 & 签名**

## **📌 学习目标**
✅ **掌握 APK 反编译后如何正确重新打包 & 签名**，确保 APK 可正常安装运行。  
✅ **学习如何使用 `apktool` 重新打包已修改的 APK 文件，并使用 `zipalign` 优化**。  
✅ **掌握 APK 签名机制，使用 `apksigner` 和 `jarsigner` 生成 & 签名 APK**。  
✅ **学习如何绕过 Android 的签名验证机制，使用 `Lucky Patcher` 或 `Frida` 绕过签名检查**。  
✅ **实战：对修改后的 APK 重新打包、签名，并成功安装到 Android 设备！**  

---

# **1️⃣ 为什么需要重新打包 & 签名？**
当我们使用 `apktool` 反编译 APK 并修改 `Smali` 代码后，APK 需要 **重新打包** 并 **签名**，否则：
- Android 系统不会允许安装未签名的 APK
- 部分应用有 **签名校验** 机制，可能会拒绝运行

---

# **2️⃣ APK 重新打包**
📌 **使用 `apktool` 反编译 APK**
```bash
apktool d app.apk -o output/
```
📌 **修改 Smali 代码**
```smali
.method public isVIP()Z
    const/4 v0, 0x1
    return v0
.end method
```
📌 **重新打包 APK**
```bash
apktool b output -o modded.apk
```
📌 **优化 APK 结构**
```bash
zipalign -v 4 modded.apk aligned.apk
```

---

# **3️⃣ APK 签名**
## **✅ 1. 生成签名密钥**
如果没有密钥，需要生成一个：
```bash
keytool -genkey -v -keystore my.keystore -alias myalias -keyalg RSA -keysize 2048 -validity 10000
```
示例输出：
```
Enter keystore password: android
Re-enter new password: android
What is your first and last name?
  [Unknown]:  Android Reverse Engineer
What is the name of your organization?
  [Unknown]:  Reverse Team
...
```

## **✅ 2. 使用 `jarsigner` 签名**
```bash
jarsigner -verbose -keystore my.keystore aligned.apk myalias
```

## **✅ 3. 使用 `apksigner` 签名**
```bash
apksigner sign --ks my.keystore --out signed.apk aligned.apk
```

## **✅ 4. 安装已签名 APK**
```bash
adb install signed.apk
```

---

# **4️⃣ 绕过签名校验**
某些应用有 **签名校验**，即使修改 & 重新签名 APK 也无法运行，解决方案：
- **修改 Smali 代码，跳过签名校验**
- **使用 Frida Hook 运行时绕过签名检查**
- **使用 Lucky Patcher 移除签名验证**

## **✅ 1. 修改 `Signature Check`**
📌 **查找 `getPackageInfo`**
```bash
grep -r "getPackageInfo" output/smali/
```
📌 **原始 Smali 代码**
```smali
invoke-virtual {p1}, Landroid/content/pm/PackageManager;->getPackageInfo(Ljava/lang/String;I)Landroid/content/pm/PackageInfo;
```
📌 **修改返回值**
```smali
const/4 v0, 0x0
return v0
```

## **✅ 2. 使用 Frida 绕过签名校验**
```js
Java.perform(function() {
    var PackageManager = Java.use("android.content.pm.PackageManager");
    PackageManager.getPackageInfo.implementation = function(pkg, flags) {
        console.log("Bypassing signature check for:", pkg);
        return this.getPackageInfo(pkg, flags);
    };
});
```
📌 **运行 Frida**
```bash
frida -U -n com.example.app -e "..."
```

## **✅ 3. 使用 Lucky Patcher**
- 打开 Lucky Patcher  
- 选择 APK  
- 选择 **"移除签名校验"**  
- 重新安装 APK

---

# **5️⃣ 实战任务**
### **✅ 1. 反编译 & 修改 APK**
```bash
apktool d app.apk -o output/
```
### **✅ 2. 重新打包**
```bash
apktool b output -o modded.apk
```
### **✅ 3. 优化 APK**
```bash
zipalign -v 4 modded.apk aligned.apk
```
### **✅ 4. 签名 APK**
```bash
apksigner sign --ks my.keystore --out signed.apk aligned.apk
```
### **✅ 5. 安装 APK**
```bash
adb install signed.apk
```
### **✅ 6. 绕过签名校验**
```js
Java.perform(function() {
    var PackageManager = Java.use("android.content.pm.PackageManager");
    PackageManager.getPackageInfo.implementation = function(pkg, flags) {
        return this.getPackageInfo(pkg, flags);
    };
});
```

---

# **📚 参考资料**
📌 **APK 反编译 & 签名**
- `APKTool`：[https://github.com/iBotPeaches/Apktool](https://github.com/iBotPeaches/Apktool)  
- `APKSIGNER`：[https://developer.android.com/studio/command-line/apksigner](https://developer.android.com/studio/command-line/apksigner)  

📌 **绕过签名校验**
- `Frida`：[https://frida.re](https://frida.re)  
- `Lucky Patcher`：[https://lucky-patcher.net/](https://lucky-patcher.net/)  

---

🔥 **任务完成后，你将掌握：**  
✅ **如何重新打包 & 签名 APK，确保修改后的 APK 可安装运行**  
✅ **如何绕过应用的签名校验，成功运行修改后的应用**  
✅ **如何使用 Frida Hook 运行时，绕过签名验证机制**  

🚀 **下一步（Day 27）**：**动态调试入门！** 🎯