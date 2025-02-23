# **📜 Day 13: Android APP 目录结构解析**

## **📌 学习目标**
✅ **理解 Android APP 目录结构**，掌握 `/data/data` 目录中的重要文件。  
✅ **学习 Android APP 的存储方式**，包括 **SharedPreferences、SQLite、内部 & 外部存储**。  
✅ **掌握 Android APK 安装后的文件分布**，学习 **应用数据备份、迁移、逆向分析方法**。  
✅ **分析 `AndroidManifest.xml` 对文件存储路径的影响**。  
✅ **实战提取 Android APP 数据，进行逆向分析 & 调试**。  

---

# **1️⃣ Android APP 目录结构**
Android APP 安装后，其数据主要存放在 `/data/data/<package_name>/` 目录下（需 Root 访问）。

📌 **查看所有已安装 APP 的数据目录**
```bash
adb shell ls /data/data/
```

📌 **示例：某 APP 的目录结构**
```
/data/data/com.example.app/
├── cache/                  # 缓存文件（临时数据）
├── code_cache/             # ART 编译缓存
├── databases/              # SQLite 数据库存储目录
│   ├── user.db             # 主要数据库文件
│   ├── user.db-shm         # SQLite 共享内存文件
│   ├── user.db-wal         # SQLite WAL 日志文件
├── files/                  # 应用存储的文件（默认目录）
│   ├── logs/               # 日志文件
│   ├── config.json         # 配置文件
├── shared_prefs/           # SharedPreferences（XML 格式存储）
│   ├── settings.xml        # APP 设置存储
│   ├── user_data.xml       # 用户数据
├── lib/                    # Native C/C++ so 库
│   ├── libnative.so        # 本地共享库
├── app_webview/            # WebView 存储数据
├── app_flutter/            # Flutter APP 资源
└── cache/                  # 临时缓存文件
```

📌 **查看特定 APP 的目录**
```bash
adb shell ls -l /data/data/com.example.app/
```

---

# **2️⃣ 目录作用解析**
| **目录** | **作用** | **存储方式** | **是否可访问** |
|---------|---------|-------------|-------------|
| **`cache/`** | 临时缓存数据 | 文件 | ✅ 可读写（应用内） |
| **`code_cache/`** | ART 编译缓存 | OAT 文件 | 🚫 仅 Root 访问 |
| **`databases/`** | APP 的 SQLite 数据库 | SQLite3 | 🚫 仅 Root 访问 |
| **`files/`** | APP 存储的文件 | 普通文件 | ✅ 可读写（应用内） |
| **`shared_prefs/`** | 共享存储，存储应用设置 | XML | 🚫 仅 Root 访问 |
| **`lib/`** | 存储 Native .so 库 | ELF 文件 | 🚫 仅 Root 访问 |
| **`app_webview/`** | WebView 本地存储 | Cache | 🚫 仅 Root 访问 |
| **`cache/`** | 临时缓存 | Cache | ✅ 可读写（应用内） |

📌 **查看数据库文件**
```bash
adb shell ls /data/data/com.example.app/databases/
```

📌 **查看 SharedPreferences**
```bash
adb shell cat /data/data/com.example.app/shared_prefs/settings.xml
```

---

# **3️⃣ Android APP 存储方式**
### **✅ 1. 内部存储（Internal Storage）**
**APP 私有存储，其他 APP 不能访问（无 Root 权限）。**
```java
File file = new File(getFilesDir(), "config.json");
```
📌 **查看 APP 内部存储**
```bash
adb shell run-as com.example.app ls -l /data/data/com.example.app/files/
```

---

### **✅ 2. 外部存储（External Storage）**
**可被其他 APP 访问（需要权限 `WRITE_EXTERNAL_STORAGE`）。**
```java
File file = new File(Environment.getExternalStorageDirectory(), "myfile.txt");
```
📌 **查看外部存储文件**
```bash
adb shell ls -l /sdcard/Android/data/com.example.app/
```

---

### **✅ 3. SharedPreferences（轻量级存储）**
**存储简单的键值对数据（XML 格式）。**
```java
SharedPreferences prefs = getSharedPreferences("settings", MODE_PRIVATE);
prefs.edit().putString("username", "admin").apply();
```
📌 **查看 SharedPreferences**
```bash
adb shell cat /data/data/com.example.app/shared_prefs/settings.xml
```

---

### **✅ 4. SQLite 数据库**
**存储结构化数据**
```java
SQLiteDatabase db = openOrCreateDatabase("user.db", MODE_PRIVATE, null);
db.execSQL("CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, name TEXT)");
```
📌 **Dump SQLite 数据库**
```bash
adb shell "sqlite3 /data/data/com.example.app/databases/user.db 'SELECT * FROM users;'"
```

---

# **4️⃣ 逆向分析 APP 目录**
## **✅ 1. 提取 APK 资源**
```bash
adb pull /data/app/com.example.app/base.apk .
apktool d base.apk -o output/
```
**修改 `AndroidManifest.xml` 以查看文件存储权限**
```xml
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```
重新打包：
```bash
apktool b output -o modded.apk
jarsigner -verbose -keystore my.keystore modded.apk alias_name
adb install modded.apk
```

---

## **✅ 2. 获取数据库数据**
📌 **Dump SQLite 数据库**
```bash
adb shell "sqlite3 /data/data/com.example.app/databases/user.db .dump"
```
📌 **复制数据库到本地**
```bash
adb pull /data/data/com.example.app/databases/user.db .
sqlite3 user.db
```

---

## **✅ 3. Hook SharedPreferences**
📌 **使用 Frida Hook 读取 SharedPreferences**
```js
Java.perform(function() {
    var SharedPreferences = Java.use("android.app.SharedPreferencesImpl");
    SharedPreferences.getString.implementation = function(key, defValue) {
        console.log("Hooked SharedPreferences: " + key);
        return this.getString(key, defValue);
    };
});
```
执行：
```bash
frida -U -n com.example.app -e "..."
```

---

## **✅ 4. 拦截文件存取**
📌 **使用 Xposed Hook `open` 读取 `config.json`**
```java
findAndHookMethod("java.io.FileInputStream", "open", String.class, new XC_MethodHook() {
    @Override
    protected void beforeHookedMethod(MethodHookParam param) {
        Log.d("Xposed", "File opened: " + param.args[0]);
    }
});
```

---

# **🛠 实战任务**
### **✅ 1. 查询某应用的存储路径**
```bash
adb shell ls -l /data/data/com.example.app/
```
### **✅ 2. 提取 SQLite 数据**
```bash
adb pull /data/data/com.example.app/databases/user.db .
sqlite3 user.db
```
### **✅ 3. 修改 SharedPreferences**
```bash
adb shell cat /data/data/com.example.app/shared_prefs/settings.xml
adb shell "echo '<string name=\"is_premium\">true</string>' >> /data/data/com.example.app/shared_prefs/settings.xml"
```
### **✅ 4. Hook 文件访问**
```js
Java.perform(function() {
    var File = Java.use("java.io.File");
    File.getAbsolutePath.implementation = function() {
        console.log("File Accessed: " + this.getAbsolutePath());
        return this.getAbsolutePath();
    };
});
```

---

# **📚 参考资料**
📌 **Android 存储机制**
- `官方存储指南`：[https://developer.android.com/training/data-storage](https://developer.android.com/training/data-storage)  
- `Android 数据库存储`：[https://developer.android.com/training/data-storage/sqlite](https://developer.android.com/training/data-storage/sqlite)  

📌 **逆向工程**
- `Frida Hook 文件存储`：[https://frida.re](https://frida.re)  
- `SQLite 逆向分析`：[https://github.com/sqlitebrowser/sqlitebrowser](https://github.com/sqlitebrowser/sqlitebrowser)  

---

🔥 **任务完成后，你将掌握：**  
✅ **Android APP 目录结构，及其存储机制**  
✅ **如何获取 SQLite 数据、SharedPreferences**  
✅ **如何使用 Frida/Xposed Hook 文件访问**  

🚀 **下一步（Day 14）**：**APK 是如何加载的？** 🎯