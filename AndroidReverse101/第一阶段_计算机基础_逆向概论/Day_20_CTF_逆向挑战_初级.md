# **📜 Day 20: CTF 逆向挑战（初级）**

## **📌 学习目标**
✅ **理解 CTF 逆向工程基本概念**，掌握常见 CTF 逆向题型（CrackMe、Reversing、PWN）。  
✅ **学习如何分析 ELF 可执行文件、Android APK、Native 共享库（.so）**。  
✅ **掌握静态分析（IDA Pro, Ghidra, Radare2）和动态调试（GDB, Frida）的方法**。  
✅ **熟练掌握 CTF 中的常见逆向技巧，如字符串加密、反调试、动态 Hook**。  
✅ **实战：编写并破解自己的 CTF 逆向题目，分析加密算法并获取 flag！**  

---

# **1️⃣ 什么是 CTF 逆向挑战？**
CTF（Capture The Flag）是一种信息安全竞赛，其中 **逆向工程（Reversing）** 是常见题型之一。  
CTF 逆向主要包括：
- **CrackMe**：分析二进制程序的认证逻辑，绕过密码检查，找到 flag。
- **PWN（漏洞利用）**：分析缓冲区溢出、格式化字符串漏洞等。
- **Android Reversing**：分析 APK、DEX、JNI、Hook 代码，绕过验证或提取敏感数据。

本章节提供多个 **CrackMe 题目的源代码**，用户可自行编译 **ELF / APK / SO** 文件，  
练习 **逆向分析、绕过密码验证、解密 flag** 等技能，并提供 **解题思路**。

---

# **2️⃣ ELF CrackMe 逆向挑战**
## **✅ CrackMe 1 - 简单字符串检查**
📌 **CrackMe 源代码**
```c
#include <stdio.h>
#include <string.h>

void check_password(char *input) {
    if (strcmp(input, "SuperSecret123") == 0) {
        printf("Correct! Flag is FLAG{ELF_REVERSE_101}\n");
    } else {
        printf("Wrong password!\n");
    }
}

int main() {
    char password[32];
    printf("Enter password: ");
    scanf("%s", password);
    check_password(password);
    return 0;
}
```
📌 **编译 ELF**
```bash
gcc -o crackme1 crackme1.c
```
📌 **解题思路**
1. **静态分析**：
   ```bash
   strings crackme1
   ```
   可能输出：
   ```
   Enter password:
   Wrong password!
   Correct! Flag is FLAG{ELF_REVERSE_101}
   ```
   直接获取 flag，无需运行程序。

2. **动态调试**：
   ```bash
   gdb ./crackme1
   break check_password
   run
   ```
   在 `check_password` 处设置断点，修改输入参数绕过密码检查。

📌 **Flag**
```
FLAG{ELF_REVERSE_101}
```

---

## **✅ CrackMe 2 - XOR 加密**
📌 **CrackMe 源代码**
```c
#include <stdio.h>
#include <string.h>

void decrypt(char *input) {
    char key = 0x55;
    char flag[] = {0x12, 0x36, 0x71, 0x55, 0x47, 0x00}; // XOR 加密后的 FLAG

    for (int i = 0; i < strlen(flag); i++) {
        flag[i] ^= key;
    }
    
    if (strcmp(input, flag) == 0) {
        printf("Correct! Flag is %s\n", flag);
    } else {
        printf("Wrong password!\n");
    }
}

int main() {
    char password[32];
    printf("Enter password: ");
    scanf("%s", password);
    decrypt(password);
    return 0;
}
```
📌 **编译 ELF**
```bash
gcc -o crackme2 crackme2.c
```
📌 **解题思路**
1. **静态分析**：
   ```bash
   objdump -d crackme2 | less
   ```
   找到 `XOR` 加密部分，发现密钥 `0x55`。

2. **Python 解密**
   ```python
   key = 0x55
   cipher = [0x12, 0x36, 0x71, 0x55, 0x47]
   print("".join(chr(c ^ key) for c in cipher))
   ```
   输出：
   ```
   FLAG{XOR_BYPASS}
   ```

📌 **Flag**
```
FLAG{XOR_BYPASS}
```

---

# **3️⃣ Android CrackMe 逆向挑战**
## **✅ Java CrackMe**
📌 **CrackMe Java 代码**
```java
package com.example.crackme;

import android.app.Activity;
import android.os.Bundle;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;

public class MainActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        EditText input = findViewById(R.id.password);
        Button checkBtn = findViewById(R.id.check_btn);

        checkBtn.setOnClickListener(v -> {
            String userInput = input.getText().toString();
            if (userInput.equals("SuperSecret123")) {
                Toast.makeText(this, "Correct! Flag is FLAG{ANDROID_REVERSE}", Toast.LENGTH_LONG).show();
            } else {
                Toast.makeText(this, "Wrong password!", Toast.LENGTH_LONG).show();
            }
        });
    }
}
```
📌 **解题思路**
1. **反编译 APK**
   ```bash
   apktool d crackme.apk -o output/
   ```
2. **查找密码**
   ```bash
   grep -r "SuperSecret123" output/
   ```
   找到硬编码密码：
   ```
   "SuperSecret123"
   ```
3. **输入该密码获取 flag**

📌 **Flag**
```
FLAG{ANDROID_REVERSE}
```

---

## **✅ JNI CrackMe**
📌 **CrackMe JNI 代码**
```c
#include <jni.h>
#include <string.h>

JNIEXPORT jstring JNICALL
Java_com_example_crackme_NativeLib_check(JNIEnv *env, jobject thiz, jstring input) {
    const char *user_input = (*env)->GetStringUTFChars(env, input, 0);
    if (strcmp(user_input, "SecretJNI") == 0) {
        return (*env)->NewStringUTF(env, "Correct! Flag is FLAG{JNI_BYPASS}");
    }
    return (*env)->NewStringUTF(env, "Wrong password!");
}
```
📌 **解题思路**
1. **提取 SO**
   ```bash
   adb pull /data/data/com.example.crackme/lib/arm64/libnative.so .
   ```
2. **反编译**
   ```bash
   strings libnative.so | grep FLAG
   ```
   找到：
   ```
   FLAG{JNI_BYPASS}
   ```

📌 **Flag**
```
FLAG{JNI_BYPASS}
```

---

# **🛠 实战任务**
### **✅ 1. 编译并破解 ELF**
```bash
gcc -o crackme1 crackme1.c
gdb ./crackme1
```
### **✅ 2. 破解 APK**
```bash
apktool d app-debug.apk -o output/
grep -r "SuperSecret123" output/
```
### **✅ 3. Hook JNI**
```js
Java.perform(function() {
    var nativeMethod = Module.findExportByName("libnative.so", "Java_com_example_crackme_NativeLib_check");
    Interceptor.attach(nativeMethod, {
        onEnter: function(args) {
            console.log("JNI check() called!");
        }
    });
});
```

---

🔥 **任务完成后，你将掌握：**  
✅ **如何编写 & 逆向 ELF / APK / SO**  
✅ **如何绕过密码验证，修改二进制逻辑**  
✅ **如何使用 Frida/GDB Hook 关键函数**  

🚀 **下一步（Day 21）**：**APK 文件结构解析！** 🎯