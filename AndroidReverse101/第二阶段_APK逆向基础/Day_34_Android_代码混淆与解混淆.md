# **📜 Day 34: Android 代码混淆与解混淆**

## **📌 学习目标**
✅ **掌握 Android 代码混淆（Obfuscation）的基本概念，理解为什么应用进行混淆**。  
✅ **学习 ProGuard、R8、DexGuard 等混淆工具的工作原理**，分析其对逆向的影响。  
✅ **掌握代码解混淆技术，包括 `jadx`、`procyon`、`bytecode viewer` 等工具**。  
✅ **学习如何手动还原混淆代码、恢复类/方法/变量名，并进行代码重构分析**。  
✅ **实战：分析一个混淆后的 APK，进行解混淆并恢复可读的代码！**  

---

# **1️⃣ 什么是 Android 代码混淆？**
Android 开发者通常使用 **混淆工具** 对代码进行保护，使逆向分析变得困难，主要目的：
- **防止逆向分析**
- **提高应用安全性**
- **减少 APK 体积**
- **保护核心算法**

📌 **常见的混淆工具**
| **工具** | **特点** | **是否默认启用** |
|---------|--------|------------|
| **ProGuard** | 代码缩减 & 重命名 | ✅ 默认启用 |
| **R8** | 代替 ProGuard，支持 D8 | ✅ 默认启用 |
| **DexGuard** | 付费，支持加密 & 动态混淆 | ❌ 需要单独购买 |
| **Jiagu (360加固)** | 国内常见的加固方案 | ❌ 需手动配置 |

📌 **混淆对代码的影响**
| **原始代码** | **混淆后代码** |
|------------|------------|
| `public void checkLogin()` | `public void a()` |
| `private boolean isVIP()` | `private boolean b()` |
| `String username = "admin";` | `String a = "admin";` |

---

# **2️⃣ 检测 APK 是否混淆**
## **✅ 1. 使用 `jadx` 反编译**
📌 **反编译 APK**
```bash
jadx -d output/ app.apk
```
📌 **查看类 & 方法名**
```bash
ls output/sources/com/example/
```
如果类名是 `a`, `b`, `c`，则表示已被混淆。

---

# **3️⃣ 代码解混淆**
## **✅ 1. 还原 ProGuard 混淆**
ProGuard & R8 会生成 `mapping.txt` 文件，存储混淆映射关系。

📌 **示例 `mapping.txt`**
```
com.example.MainActivity -> a
checkLogin() -> b
isVIP() -> c
```
📌 **还原代码**
```bash
proguard retrace mapping.txt stacktrace.txt
```

---

## **✅ 2. 使用 `bytecode viewer` 进行解混淆**
📌 **步骤**
1. 下载 `Bytecode Viewer`：[https://github.com/Konloch/bytecode-viewer](https://github.com/Konloch/bytecode-viewer)
2. 打开 APK，选择 `Procyon` 反编译器
3. **查看混淆后的代码**
4. **自动恢复部分变量 & 方法名**

---

## **✅ 3. 手动恢复类 & 方法名**
📌 **示例：混淆后的 Smali 代码**
```smali
.method public b()V
    .locals 1
    const-string v0, "Login Success"
    invoke-static {v0}, Landroid/util/Log;->d(Ljava/lang/String;Ljava/lang/String;)V
    return-void
.end method
```
📌 **手动修改 & 还原**
```smali
.method public checkLogin()V
    .locals 1
    const-string v0, "Login Success"
    invoke-static {v0}, Landroid/util/Log;->d(Ljava/lang/String;Ljava/lang/String;)V
    return-void
.end method
```

---

## **✅ 4. 使用 `deobfuscator` 进行自动解混淆**
📌 **运行 `deobfuscator`**
```bash
java -jar deobfuscator.jar -input app.apk -output output.apk
```
📌 **自动分析 & 还原类名**
```bash
grep -r "MainActivity" output/
```

---

# **4️⃣ 逆向 ProGuard 混淆代码**
## **✅ 1. 获取 `mapping.txt`**
如果 `mapping.txt` 不可用，可尝试使用 **`deobfuscator`** 进行自动分析：
```bash
java -jar deobfuscator.jar -input app.apk -output output.apk
```

## **✅ 2. 使用 `jadx` 分析代码**
```bash
jadx -d output/ app.apk
grep -r "a()" output/
```

## **✅ 3. 手动恢复类名**
```bash
mv output/sources/a.java output/sources/MainActivity.java
```

---

# **5️⃣ 绕过 R8/DexGuard 加密**
## **✅ 1. 分析 Dex 文件**
📌 **使用 `dexdump` 查看 Dex 结构**
```bash
dexdump -d classes.dex | grep "MainActivity"
```
📌 **手动解混淆**
```bash
baksmali disassemble classes.dex -o smali/
```

---

# **6️⃣ 破解 360 加固 & 其他保护**
某些应用使用 **360 加固**，导致无法直接反编译：
📌 **查找 360 加固签名**
```bash
grep -r "qihoo" output/
```
📌 **使用 `unpackers` 工具**
```bash
java -jar Qihoo360Unpacker.jar -i app.apk -o unpacked.apk
```
📌 **重新反编译**
```bash
jadx -d output/ unpacked.apk
```

---

# **🛠 实战任务**
### **✅ 1. 反编译 APK**
```bash
jadx -d output/ app.apk
ls output/sources/com/example/
```
### **✅ 2. 查找混淆代码**
```bash
grep -r "a()" output/
```
### **✅ 3. 还原 `mapping.txt`**
```bash
proguard retrace mapping.txt stacktrace.txt
```
### **✅ 4. 使用 `bytecode viewer`**
- 打开 `app.apk`
- 选择 `Procyon`
- 自动恢复代码
### **✅ 5. 破解 360 加固**
```bash
java -jar Qihoo360Unpacker.jar -i app.apk -o unpacked.apk
jadx -d output/ unpacked.apk
```

---

# **📚 参考资料**
📌 **代码解混淆**
- `jadx`：[https://github.com/skylot/jadx](https://github.com/skylot/jadx)  
- `bytecode viewer`：[https://github.com/Konloch/bytecode-viewer](https://github.com/Konloch/bytecode-viewer)  

📌 **绕过 ProGuard**
- `ProGuard Retrace`：[https://www.guardsquare.com/en/products/proguard](https://www.guardsquare.com/en/products/proguard)  

📌 **破解加固**
- `360加固 Unpacker`：[https://github.com/bunq/xposed-anti360](https://github.com/bunq/xposed-anti360)  

---

🔥 **任务完成后，你将掌握：**  
✅ **如何检测 APK 是否被混淆**  
✅ **如何使用 `jadx`、`bytecode viewer` 进行解混淆**  
✅ **如何还原 `mapping.txt`，恢复原始类名 & 方法名**  
✅ **如何破解 360 加固，提取未加密的 DEX 文件**  

🚀 **下一步（Day 35）**：**逆向加密算法（MD5、AES、RSA）！** 🎯