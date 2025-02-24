# **📱 OWASP Uncrackable Android Level 1 - Writeup 🚀**

今天我们来挑战 **OWASP Uncrackable Android Level 1**！🎯 

这个应用可以在 [OWASP MSTG GitHub 仓库](https://github.com/OWASP/owasp-mstg/tree/master/Crackmes) 中找到。

也可以在本仓库中找到：[Crackmes/Android/Level_01/UnCrackable-Level1.apk](UnCrackable-Level1.apk)。

---

## **🔍 1. 逆向分析**
### **💡 初步探索**
首先，我用 **Jadx-GUI** 反编译 APK，看看 `MainActivity` 里藏着什么秘密 🕵️‍♂️。

`MainActivity` 代码如下：
```java
package sg.vantagepoint.uncrackable1;

import android.app.Activity;
import android.app.AlertDialog;
import android.content.DialogInterface;
import android.os.Bundle;
import android.view.View;
import android.widget.EditText;
import owasp.mstg.uncrackable1.R;
import sg.vantagepoint.a.b;
import sg.vantagepoint.a.c;

/* loaded from: classes.dex */
public class MainActivity extends Activity {
    private void a(String str) {
        AlertDialog create = new AlertDialog.Builder(this).create();
        create.setTitle(str);
        create.setMessage("This is unacceptable. The app is now going to exit.");
        create.setButton(-3, "OK", new DialogInterface.OnClickListener() { // from class: sg.vantagepoint.uncrackable1.MainActivity.1
            @Override // android.content.DialogInterface.OnClickListener
            public void onClick(DialogInterface dialogInterface, int i) {
                System.exit(0);
            }
        });
        create.setCancelable(false);
        create.show();
    }

    @Override // android.app.Activity
    protected void onCreate(Bundle bundle) {
        if (c.a() || c.b() || c.c()) {
            a("Root detected!");
        }
        if (b.a(getApplicationContext())) {
            a("App is debuggable!");
        }
        super.onCreate(bundle);
        setContentView(R.layout.activity_main);
    }

    public void verify(View view) {
        String str;
        String obj = ((EditText) findViewById(R.id.edit_text)).getText().toString();
        AlertDialog create = new AlertDialog.Builder(this).create();
        if (a.a(obj)) {
            create.setTitle("Success!");
            str = "This is the correct secret.";
        } else {
            create.setTitle("Nope...");
            str = "That's not it. Try again.";
        }
        create.setMessage(str);
        create.setButton(-3, "OK", new DialogInterface.OnClickListener() { // from class: sg.vantagepoint.uncrackable1.MainActivity.2
            @Override // android.content.DialogInterface.OnClickListener
            public void onClick(DialogInterface dialogInterface, int i) {
                dialogInterface.dismiss();
            }
        });
        create.show();
    }
}
```

打开代码后，发现 `onCreate` 里有两个保护机制：
1. **🔐 Root 检测** —— 检查设备是否被 root。
2. **🐛 Debug 检测** —— 检测应用是否在调试模式下运行。

除此之外，`verify` 方法用于检查用户输入的密钥是否合法，核心代码是：
```java
if (a.a(obj)) {
```
这里的 `a` 类没有出现在 imports 里，所以它应该就在 `sg.vantagepoint.uncrackable1` 这个包里。

### **🔑 密钥计算逻辑**

`a.a` 方法的代码如下：
```java
package sg.vantagepoint.uncrackable1;

import android.util.Base64;
import android.util.Log;

/* loaded from: classes.dex */
public class a {
    public static boolean a(String str) {
        byte[] bArr;
        byte[] bArr2 = new byte[0];
        try {
            bArr = sg.vantagepoint.a.a.a(b("8d127684cbc37c17616d806cf50473cc"), Base64.decode("5UJiFctbmgbDoLXmpL12mkno8HT4Lv8dlat8FxR2GOc=", 0));
        } catch (Exception e) {
            Log.d("CodeCheck", "AES error:" + e.getMessage());
            bArr = bArr2;
        }
        return str.equals(new String(bArr));
    }

    public static byte[] b(String str) {
        int length = str.length();
        byte[] bArr = new byte[length / 2];
        for (int i = 0; i < length; i += 2) {
            bArr[i / 2] = (byte) ((Character.digit(str.charAt(i), 16) << 4) + Character.digit(str.charAt(i + 1), 16));
        }
        return bArr;
    }
}
```

### **🔑 运行时计算的密钥**
应用不会在代码里硬编码密钥，而是 **动态计算**：
- `sg.vantagepoint.a.a.a` 方法使用 **AES 加密**。
- 第一个参数是密钥，第二个参数是被加密的字符串。

### **🔐 AES 加密逻辑**

`sg.vantagepoint.a.a` 类的代码如下：
```java
package sg.vantagepoint.a;

import javax.crypto.Cipher;
import javax.crypto.spec.SecretKeySpec;

/* loaded from: classes.dex */
public class a {
    public static byte[] a(byte[] bArr, byte[] bArr2) {
        SecretKeySpec secretKeySpec = new SecretKeySpec(bArr, "AES/ECB/PKCS7Padding");
        Cipher cipher = Cipher.getInstance("AES");
        cipher.init(2, secretKeySpec);
        return cipher.doFinal(bArr2);
    }
}
```

---

## **🎭 2. 绕过保护机制**
有很多办法能搞定这个挑战：
✅ **Root 设备 + Hook**（但要绕过 Root 检测）
✅ **Debug 模式 + 直接读取内存**（需要绕过 Root & Debug 检测）
✅ **提取相关类，放进新应用里计算密钥**
✅ **写个 Python 脚本，离线计算密钥**

这次我们用 **Xposed Hook** 来绕过检测，并把密钥偷出来 😈。

---

## **🛠️ 3. Xposed Hook 绕过 Root 检测**
### **🚀 运行 APK**
在 Root 设备上运行后，App 直接报错 ❌，说明 **Root 检测生效了**。

### **🔬 找到 Root 检测逻辑**
`MainActivity.onCreate` 里调用了这些方法：
- `c.a()`
- `c.b()`
- `c.c()`
- `b.a()`

如果检测到 Root，就会执行 `MainActivity.a()`，然后……Boom 💥，App 崩了！

### **🩹 绕过 Root 检测**
有两种方式：
1. **🛠️ 修改 APK（Smali Patch）**
   - 用 `apktool` 反编译 APK
   - 改 `smali` 代码绕过检测
   - 重新打包并签名
2. **🐍 用 Xposed Hook（更优雅！）**
   - 直接 Hook `MainActivity.a()`，让它 **永远返回 false**。

---

## **💣 4. 提取密钥**
密钥是通过 AES 计算的，我们可以 **Hook 计算方法** 并打印密钥 ✨。

### **💻 Hook 代码**
```java
package owasp.uncrackable;

import de.robv.android.xposed.IXposedHookLoadPackage;
import de.robv.android.xposed.XC_MethodHook;
import de.robv.android.xposed.XposedBridge;
import de.robv.android.xposed.XposedHelpers;
import de.robv.android.xposed.callbacks.XC_LoadPackage;

public class OwaspHook implements IXposedHookLoadPackage {
    @Override
    public void handleLoadPackage(XC_LoadPackage.LoadPackageParam loadPackageParam) throws Throwable {
        if (!loadPackageParam.packageName.equalsIgnoreCase("sg.vantagepoint.uncrackable1")) {
            return;
        }

        XposedBridge.log("🚀 HOOKED: " + loadPackageParam.packageName);

        // 🎭 绕过 Root 检测
        XposedHelpers.findAndHookMethod("sg.vantagepoint.uncrackable1.MainActivity",
                loadPackageParam.classLoader, "a", String.class, 
                new XC_MethodHook() {
                    @Override
                    protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                        param.setResult(false); // 直接返回 false，绕过 Root 检测 ✅
                    }
                });

        // 🔓 Hook AES 方法，偷密钥
        XposedHelpers.findAndHookMethod("sg.vantagepoint.a.a",
                loadPackageParam.classLoader, "a", byte[].class, byte[].class,
                new XC_MethodHook() {
                    @Override
                    protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                        byte[] secretBytes = (byte[]) param.getResult();
                        String secretString = new String(secretBytes);
                        XposedBridge.log("🔑 SECRET: " + secretString);
                    }
                });
    }
}
```

---

## **🎬 5. 运行 Hook**
### **🚀 操作步骤**
1. **安装 Xposed 框架**（建议用 EdXposed 适配 Android 8+）。
2. **在 Xposed 模块中启用 Hook**。
3. **重启设备**，启动 **Uncrackable Level 1**。
4. **打开日志监听**：
   ```bash
   adb logcat -s Xposed
   ```
5. **输入任意字符测试**，密钥就会乖乖出现在日志里：
   ```
   🔑 SECRET: <解密后的密钥>
   ```
   
6. **输入密钥**，成功解锁应用！🎉
    ```
    🔑 SECRET: I want to believe
    ```

---

## **🏆 6. 最终结果**
🎉 成功绕过 Root 检测，并成功拿到了应用的密钥！

🔹 **静态分析** 定位 Root 检测 & AES 逻辑  
🔹 **动态 Hook** 绕过检测并 dump 出密钥  
🔹 **完美破解 Uncrackable Level 1！** 🎯  
