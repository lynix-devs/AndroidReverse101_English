# **📜 Day 60: 深入分析 CTF 逆向挑战**

## **📌 学习目标**
✅ **掌握 CTF 逆向工程的常见题型，包括 CrackMe、动态分析、加密算法、反调试等**。  
✅ **学习如何使用 IDA Pro、Ghidra、Frida、GDB、Radare2 进行 CTF 逆向分析**。  
✅ **掌握 ELF/DEX 可执行文件结构，提取关键函数 & 解密算法**。  
✅ **实战：完成多个 CTF 逆向挑战，成功获取 Flag！**  

---

# **1️⃣ CTF 逆向工程常见题型**
📌 **CTF 逆向工程题通常涉及以下类型**
| **题型** | **描述** | **常见解法** |
|---------|--------|------------|
| **CrackMe** | 找到正确的输入，绕过密码验证 | 静态分析 + Hook `strcmp()` |
| **ELF 二进制分析** | 分析 Linux ELF 执行逻辑，找到 Flag | `strings` + `objdump` + `GDB` |
| **加密算法破解** | 逆向 AES、RSA、Base64 等加密算法 | 静态分析 + 动态调试 |
| **反调试 & 反虚拟机检测** | 绕过 `ptrace()`、`syscall` 检测 | `Frida` Hook + Patch |
| **Android DEX 逆向** | 分析 `classes.dex`，提取 Flag | `jadx` + `Frida` + `Xposed` |

---

# **2️⃣ 经典 CTF 逆向 CrackMe 题目**
## **✅ 1. CrackMe 示例**
📌 **CrackMe C 代码**
```c
#include <stdio.h>
#include <string.h>

int main() {
    char input[100];
    printf("Enter the password: ");
    scanf("%s", input);
    if (strcmp(input, "SuperSecret") == 0) {
        printf("Correct! Flag: FLAG{CrackMe_Solved}\n");
    } else {
        printf("Wrong password!\n");
    }
    return 0;
}
```
📌 **编译 ELF**
```bash
gcc -o crackme crackme.c
```
📌 **分析 CrackMe**
```bash
strings crackme | grep FLAG
objdump -d crackme | less
gdb ./crackme
```

## **✅ 2. 使用 `GDB` 动态分析**
📌 **运行 CrackMe**
```bash
gdb ./crackme
break *main
run
```
📌 **修改输入**
```bash
set $eax = 0
continue
```
📌 **成功获取 Flag**
```
FLAG{CrackMe_Solved}
```

---

# **3️⃣ ELF 逆向分析**
## **✅ 1. 检查 ELF 结构**
📌 **查看 ELF 头**
```bash
readelf -h crackme
```
📌 **查找字符串**
```bash
strings crackme | grep FLAG
```
📌 **反汇编**
```bash
objdump -d crackme | grep strcmp
```

## **✅ 2. 使用 IDA Pro 逆向**
📌 **步骤**
1. **加载 `crackme` 到 IDA Pro**
2. **查找 `strcmp()` 调用**
3. **修改 `jne` 为 `jmp`，绕过密码检查**

---

# **4️⃣ 加密算法破解**
## **✅ 1. 逆向 Base64 加密**
📌 **示例加密代码**
```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

int main() {
    char key[] = "MySecretKey";
    char flag[] = "FLAG{Encrypted}";
    for (int i = 0; i < strlen(flag); i++) {
        flag[i] ^= key[i % strlen(key)];
    }
    printf("Encrypted: %s\n", flag);
}
```
📌 **解密 XOR**
```python
key = "MySecretKey"
enc = "Encrypted_String"
flag = "".join(chr(ord(enc[i]) ^ ord(key[i % len(key)])) for i in range(len(enc)))
print(flag)
```

---

# **5️⃣ Android DEX 逆向**
## **✅ 1. 反编译 DEX**
📌 **使用 `jadx` 反编译 APK**
```bash
jadx -d output/ app.apk
```
📌 **查找 `FLAG{}`**
```bash
grep -r "FLAG{" output/
```

## **✅ 2. Hook `checkPassword()`**
📌 **Frida Hook**
```js
Java.perform(function() {
    var MainActivity = Java.use("com.example.MainActivity");
    MainActivity.checkPassword.implementation = function(input) {
        console.log("[*] Hooked password check: " + input);
        return true;
    };
});
```
📌 **运行 Frida**
```bash
frida -U -n com.example.app -e "..."
```

---

# **6️⃣ 绕过反调试 & 反 VM**
## **✅ 1. Hook `ptrace()`**
📌 **Frida Hook**
```js
Interceptor.attach(Module.findExportByName(null, "ptrace"), {
    onEnter: function(args) {
        args[0] = 0;
    },
    onLeave: function(retval) {
        retval.replace(0);
    }
});
```

## **✅ 2. 修改 `isDebuggerConnected()`**
📌 **Frida Hook**
```js
Java.perform(function() {
    var Debug = Java.use("android.os.Debug");
    Debug.isDebuggerConnected.implementation = function() {
        return false;
    };
});
```

---

# **🛠 实战任务**
### **✅ 1. 逆向 CrackMe**
```bash
strings crackme | grep FLAG
objdump -d crackme | grep strcmp
gdb ./crackme
```
### **✅ 2. 逆向 ELF**
```bash
readelf -h crackme
strings crackme | grep FLAG
objdump -d crackme
```
### **✅ 3. 破解 Base64 加密**
```python
key = "MySecretKey"
enc = "Encrypted_String"
flag = "".join(chr(ord(enc[i]) ^ ord(key[i % len(key)])) for i in range(len(enc)))
print(flag)
```
### **✅ 4. 逆向 DEX**
```bash
jadx -d output/ app.apk
grep -r "FLAG{" output/
```
### **✅ 5. Hook `checkPassword()`**
```js
Java.perform(function() {
    var MainActivity = Java.use("com.example.MainActivity");
    MainActivity.checkPassword.implementation = function(input) {
        return true;
    };
});
```
### **✅ 6. 绕过反调试**
```js
Interceptor.attach(Module.findExportByName(null, "ptrace"), {
    onEnter: function(args) {
        args[0] = 0;
    },
    onLeave: function(retval) {
        retval.replace(0);
    }
});
```

---

# **📚 参考资料**
📌 **ELF 逆向**
- `GDB`：[https://www.gnu.org/software/gdb/](https://www.gnu.org/software/gdb/)  
- `IDA Pro`：[https://hex-rays.com/](https://hex-rays.com/)  

📌 **DEX 逆向**
- `jadx`：[https://github.com/skylot/jadx](https://github.com/skylot/jadx)  
- `Frida`：[https://frida.re](https://frida.re)  

📌 **CTF 逆向**
- `CTF Writeups`：[https://ctftime.org/](https://ctftime.org/)  

---

🔥 **任务完成后，你将掌握：**  
✅ **如何破解 CrackMe 题目，绕过密码验证**  
✅ **如何分析 ELF 二进制文件，提取 Flag**  
✅ **如何逆向加密算法，解密加密数据**  
✅ **如何使用 Frida Hook Android DEX，修改应用逻辑**  
✅ **如何绕过反调试 & 反虚拟机检测，成功调试目标程序**  

🚀 **下一步（Day 70）**：**逆向挖掘 0Day 漏洞！** 🎯