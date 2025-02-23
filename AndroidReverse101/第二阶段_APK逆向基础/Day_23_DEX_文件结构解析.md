# **📜 Day 23: DEX 文件结构解析**

## **📌 学习目标**
✅ **深入理解 DEX（Dalvik Executable）文件格式**，掌握其结构、字段和解析方法。  
✅ **学习如何手动解析 `classes.dex`，分析 DEX 头部、方法表、类定义等信息。**  
✅ **掌握 `dexdump`、`JEB`、`baksmali` 等工具，分析 DEX 结构与 Smali 代码。**  
✅ **学习如何使用 `Hex Editor` 解析 DEX 文件头，修改方法名称或 Magic 头部信息。**  
✅ **实战：解析 `classes.dex`，提取 Java 类、方法，分析 DEX 结构，修改 Smali 代码！**  

---

# **1️⃣ 什么是 DEX 文件？**
DEX（Dalvik Executable）是 **Android 虚拟机（Dalvik/ART）执行的字节码格式**，  
用于存放 Java 类、方法、字段等信息。

📌 **提取 `classes.dex`**
```bash
unzip base.apk classes.dex
```
📌 **查看 DEX 文件类型**
```bash
file classes.dex
```
示例输出：
```
classes.dex: Dalvik dex file version 038
```

---

# **2️⃣ DEX 文件结构**
DEX 文件由多个部分组成：
| **偏移** | **结构** | **作用** |
|---------|---------|------|
| `0x00` | **DEX Header** | DEX 文件头，包含 Magic 版本、大小等 |
| `0x20` | **String IDs** | 存放字符串索引表 |
| `0x40` | **Type IDs** | 存放类型索引表 |
| `0x60` | **Proto IDs** | 方法原型（参数 & 返回值） |
| `0x80` | **Field IDs** | 字段（成员变量）信息 |
| `0xA0` | **Method IDs** | 方法索引表 |
| `0xC0` | **Class Defs** | Java 类定义表 |
| `0xE0` | **Data** | 代码 & 常量数据 |

📌 **使用 `hexdump` 查看 DEX 头**
```bash
hexdump -C classes.dex | head -n 20
```
示例：
```
00000000  64 65 78 0a 30 33 38 00  63 72 61 63 6b 65 64 00  |dex.038.cracked.|
00000010  70 00 00 00 4c 00 00 00  00 00 00 00 00 00 00 00  |p...L...........|
```
🔹 **Magic 头**：`64 65 78 0A 30 33 38 00` → `dex\n038\0`（DEX 版本 038）

---

# **3️⃣ 解析 DEX 头部**
📌 **使用 `dexdump` 解析 DEX 头**
```bash
dexdump -f classes.dex
```
示例输出：
```
DEX file header:
magic          : dex\n038\0
checksum       : 0x12345678
fileSize       : 0x004C0000 (49152 bytes)
headerSize     : 0x00000070
endianTag      : 0x12345678
linkSize       : 0x00000000
linkOff        : 0x00000000
```

📌 **修改 Magic 头（伪加密）**
```bash
hexedit classes.dex
```
修改 `dex\n038\0` → `crk\n038\0`，可导致部分工具无法解析该 DEX。

---

# **4️⃣ 解析 Java 类与方法**
## **✅ 1. 提取所有类**
📌 **使用 `dexdump`**
```bash
dexdump -l classes.dex | grep "Class descriptor"
```
示例输出：
```
Class descriptor  : 'Lcom/example/MainActivity;'
```

## **✅ 2. 提取方法**
📌 **使用 `dexdump`**
```bash
dexdump -d classes.dex | grep "invoke-static"
```
示例输出：
```
invoke-static {v0}, Lcom/example/MainActivity;->isVIP()Z
```
表示 `isVIP()` 方法返回 `boolean` 类型。

---

# **5️⃣ 使用 JEB 解析 DEX**
📌 **安装 JEB 反编译器**
```bash
jeb -c classes.dex
```
📌 **查看 `isVIP()` 方法**
```java
public boolean isVIP() {
    return false;
}
```
**思路**：将 `return false;` 修改为 `return true;`，绕过 VIP 逻辑。

---

# **6️⃣ Smali 代码分析**
📌 **反编译 DEX**
```bash
baksmali d classes.dex -o smali_output/
```
📌 **查看 `isVIP.smali`**
```smali
.method public isVIP()Z
    .registers 1
    const/4 v0, 0x0
    return v0
.end method
```
📌 **修改 VIP 方法**
```smali
.method public isVIP()Z
    .registers 1
    const/4 v0, 0x1
    return v0
.end method
```
📌 **重新打包**
```bash
smali a smali_output/ -o new_classes.dex
```
📌 **替换 DEX**
```bash
zip -r new_app.apk new_classes.dex
```
📌 **重新签名**
```bash
jarsigner -verbose -keystore my.keystore new_app.apk alias_name
adb install new_app.apk
```

---

# **7️⃣ Hook DEX 运行时修改方法**
📌 **Hook `isVIP()` 方法**
```js
Java.perform(function() {
    var MainActivity = Java.use("com.example.MainActivity");
    MainActivity.isVIP.implementation = function() {
        return true;
    };
});
```
📌 **运行 Frida**
```bash
frida -U -n com.example.app -e "..."
```

---

# **🛠 实战任务**
### **✅ 1. 解析 DEX**
```bash
dexdump -f classes.dex
```
### **✅ 2. 提取所有类**
```bash
dexdump -l classes.dex | grep "Class descriptor"
```
### **✅ 3. 反编译 DEX**
```bash
jadx -d output/ classes.dex
```
### **✅ 4. 修改 VIP 方法**
```smali
.method public isVIP()Z
    const/4 v0, 0x1
    return v0
.end method
```
### **✅ 5. 重新打包 & 安装**
```bash
smali a smali_output/ -o new_classes.dex
zip -r new_app.apk new_classes.dex
adb install new_app.apk
```
### **✅ 6. Hook VIP 方法**
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
📌 **DEX 解析**
- `DEX 规范`：[https://source.android.com/docs/core/runtime/dex-format](https://source.android.com/docs/core/runtime/dex-format)  
- `JEB 反编译器`：[https://www.pnfsoftware.com/jeb/](https://www.pnfsoftware.com/jeb/)  

📌 **DEX 反编译**
- `baksmali`：[https://github.com/JesusFreke/smali](https://github.com/JesusFreke/smali)  
- `jadx`：[https://github.com/skylot/jadx](https://github.com/skylot/jadx)  

📌 **运行时 Hook**
- `Frida`：[https://frida.re](https://frida.re)  

---

🔥 **任务完成后，你将掌握：**  
✅ **如何解析 DEX 结构，分析 Java 类、方法**  
✅ **如何反编译 Smali 代码，修改方法逻辑**  
✅ **如何 Hook DEX 方法，修改运行时行为**  

🚀 **下一步（Day 24）**：**Smali 语言入门！** 🎯