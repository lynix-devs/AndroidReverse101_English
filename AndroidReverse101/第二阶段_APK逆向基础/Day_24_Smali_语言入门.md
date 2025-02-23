# **📜 Day 24: Smali 语言入门**

## **📌 学习目标**
✅ **理解 Smali 语言的基本语法和指令**，掌握 Smali 代码的编写与修改。  
✅ **学习如何反编译 DEX 文件，获取 Smali 代码，修改并重新编译**。  
✅ **掌握 Android 方法调用、寄存器操作、逻辑控制等核心 Smali 指令**。  
✅ **学习如何修改 Smali 代码，绕过 VIP 认证、修改返回值等**。  
✅ **实战：修改 Smali 代码，绕过应用限制，实现破解功能！**  

---

# **1️⃣ 什么是 Smali 语言？**
Smali 是 Android 的 **Dalvik 虚拟机（DVM）** 使用的汇编语言，  
主要用于反编译 DEX 文件，修改应用逻辑并重新打包。

Smali 代码通常以 `.smali` 文件形式存储，每个类对应一个 `.smali` 文件。  
例如：
```
smali/com/example/MainActivity.smali
```

---

# **2️⃣ Smali 代码结构**
Smali 代码类似汇编语言，主要包含：
- **类定义**
- **方法定义**
- **寄存器操作**
- **逻辑控制**
- **API 调用**

📌 **基本结构**
```smali
.class public Lcom/example/MainActivity;
.super Landroid/app/Activity;

.method public onCreate()V
    .locals 1
    const-string v0, "Hello Smali!"
    invoke-static {v0}, Ljava/lang/System;->println(Ljava/lang/String;)V
    return-void
.end method
```
- `.class public` → 定义类
- `.super` → 继承 `Activity`
- `.method public onCreate()V` → 定义 `onCreate` 方法
- `.locals 1` → 分配 1 个寄存器 `v0`
- `const-string v0, "Hello Smali!"` → 加载字符串到 `v0`
- `invoke-static {v0}, Ljava/lang/System;->println(Ljava/lang/String;)V` → 调用 `System.out.println`
- `return-void` → 返回

---

# **3️⃣ Smali 关键指令**
## **✅ 1. 变量与寄存器**
Smali 使用 `v` 开头的变量存储数据：
```smali
.locals 2
const/4 v0, 1
const/16 v1, 100
```
- `v0 = 1`
- `v1 = 100`

## **✅ 2. 方法调用**
📌 **调用静态方法**
```smali
invoke-static {}, Lcom/example/Utils;->getFlag()Ljava/lang/String;
```
📌 **调用实例方法**
```smali
invoke-virtual {p0}, Lcom/example/User;->getName()Ljava/lang/String;
```
📌 **调用构造方法**
```smali
invoke-direct {p0}, Lcom/example/MainActivity;-><init>()V
```

## **✅ 3. 条件判断**
📌 **等于判断**
```smali
if-eq v0, v1, :label_true
```
📌 **大于判断**
```smali
if-gt v0, v1, :label_greater
```
📌 **跳转**
```smali
:label_true
    const-string v0, "True!"
    goto :label_end

:label_greater
    const-string v0, "Greater!"

:label_end
    return-void
```

---

# **4️⃣ 反编译 APK，修改 Smali**
## **✅ 1. 反编译 APK**
```bash
apktool d app.apk -o output/
```
## **✅ 2. 查找 VIP 方法**
```bash
grep -r "isVIP" output/smali/
```
📌 **修改 `isVIP()`**
```smali
.method public isVIP()Z
    .locals 1
    const/4 v0, 0x1
    return v0
.end method
```
## **✅ 3. 重新打包**
```bash
apktool b output -o modded.apk
```
## **✅ 4. 重新签名**
```bash
jarsigner -verbose -keystore my.keystore modded.apk alias_name
adb install modded.apk
```

---

# **5️⃣ Hook Smali 运行时**
## **✅ 1. Hook `isVIP()`**
```js
Java.perform(function() {
    var MainActivity = Java.use("com.example.MainActivity");
    MainActivity.isVIP.implementation = function() {
        return true;
    };
});
```
## **✅ 2. 运行 Frida**
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
### **✅ 3. 重新打包 & 安装**
```bash
apktool b output -o modded.apk
jarsigner -verbose -keystore my.keystore modded.apk alias_name
adb install modded.apk
```
### **✅ 4. Hook Smali 方法**
```js
Java.perform(function() {
    var MainActivity = Java.use("com.example.MainActivity");
    MainActivity.isVIP.implementation = function() {
        return true;
    };
});
```

---

# **📚 参考资料**
📌 **Smali 语法**
- `Smali 参考文档`：[https://github.com/JesusFreke/smali](https://github.com/JesusFreke/smali)  
- `Smali 逆向指南`：[https://www.androidreversing.com/smali](https://www.androidreversing.com/smali)  

📌 **APK 反编译**
- `APKTool`：[https://github.com/iBotPeaches/Apktool](https://github.com/iBotPeaches/Apktool)  

📌 **运行时 Hook**
- `Frida`：[https://frida.re](https://frida.re)  

---

🔥 **任务完成后，你将掌握：**  
✅ **如何编写和修改 Smali 代码，修改 Android 应用逻辑**  
✅ **如何反编译 APK，修改 Smali 代码并重新打包**  
✅ **如何 Hook 运行时 Smali 方法，实现动态修改**  

🚀 **下一步（Day 25）**：**Smali 代码修改实验！** 🎯