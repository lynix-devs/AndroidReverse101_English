# **📜 Day 38: 游戏破解基础**

## **📌 学习目标**
✅ **掌握 Android 游戏的基本架构，理解游戏的内存管理 & 反作弊机制**。  
✅ **学习如何使用 `Frida`、`Cheat Engine`、`GameGuardian` 进行游戏修改**。  
✅ **学习如何 Hook 关键函数，如 `金币系统`、`攻击力计算`，实现游戏修改**。  
✅ **掌握如何分析 `libgame.so`，逆向游戏逻辑，修改关键变量**。  
✅ **实战：修改游戏内金币数量、攻击力、解锁 VIP 角色！**  

---

# **1️⃣ Android 游戏的架构**
📌 **常见游戏引擎**
| **游戏引擎** | **特点** | **使用案例** |
|------------|--------|--------|
| **Unity3D** | 使用 `C#`，核心代码在 `libil2cpp.so` | 王者荣耀、崩坏3 |
| **Unreal Engine** | 使用 `C++`，代码在 `libUE4.so` | PUBG Mobile |
| **Cocos2d-x** | 使用 `C++`，代码在 `libgame.so` | 小型 2D 游戏 |
| **原生 Android** | 直接使用 `Java/Kotlin` | 休闲类小游戏 |

📌 **游戏内常见变量**
| **变量** | **用途** | **存储位置** |
|--------|--------|--------|
| `金币` | 影响游戏内购买 | 存储在 `SharedPreferences` 或 `内存` |
| `攻击力` | 影响战斗伤害 | 存储在 `内存` |
| `血量` | 影响角色生存 | 存储在 `内存` |
| `经验值` | 用于升级 | 存储在 `内存` |

---

# **2️⃣ 修改游戏数据**
## **✅ 1. 使用 GameGuardian 修改金币**
📌 **步骤**
1. **安装 GameGuardian**
2. **运行游戏，打开 GameGuardian**
3. **搜索金币数值**
4. **修改为 `9999999`**
5. **验证修改是否生效**

📌 **注意**
- **浮点数修改**：部分游戏使用 `float` 代替 `int`，搜索时需尝试 `float`。
- **加密存储**：部分游戏会对数据进行加密，如 `金币 = (真实值 + 1234) * 2`。

---

## **✅ 2. 使用 Cheat Engine 修改攻击力**
📌 **步骤**
1. **在 PC 上运行 Android 模拟器**
2. **使用 Cheat Engine 连接模拟器**
3. **搜索当前攻击力**
4. **修改攻击力为 `9999`**
5. **在游戏中攻击敌人，观察是否生效**

---

## **✅ 3. 使用 Frida Hook 修改游戏逻辑**
### **📌 Hook 金币增加方法**
📌 **游戏原始代码**
```java
public void addCoins(int amount) {
    this.coins += amount;
}
```
📌 **Frida Hook**
```js
Java.perform(function() {
    var GameClass = Java.use("com.example.game.GameManager");
    GameClass.addCoins.implementation = function(amount) {
        console.log("[*] Hooked addCoins, original: " + amount);
        this.addCoins(9999999);
    };
});
```
📌 **运行 Frida**
```bash
frida -U -n com.example.game -e "..."
```

---

# **3️⃣ 逆向 `libgame.so` 进行修改**
## **✅ 1. 提取 `libgame.so`**
📌 **查找 so 文件**
```bash
adb shell ls /data/app/com.example.game/lib/arm64/
```
📌 **提取 so**
```bash
adb pull /data/app/com.example.game/lib/arm64/libgame.so .
```

## **✅ 2. 反编译 so 文件**
📌 **使用 IDA Pro**
1. **加载 `libgame.so`**
2. **搜索 `金币相关函数`**
3. **找到 `ADD COINS` 相关逻辑**
4. **修改 `MOV R0, #9999999`**

📌 **使用 Ghidra**
1. **加载 `libgame.so`**
2. **查看 `gameManager` 相关函数**
3. **修改 `金币增加逻辑`**

---

# **4️⃣ 绕过游戏的反作弊**
## **✅ 1. 绕过 `ptrace()` 反调试**
📌 **Frida Hook**
```js
Interceptor.attach(Module.findExportByName(null, "ptrace"), {
    onEnter: function(args) {
        console.log("[*] Bypassing ptrace anti-debugging");
        args[0] = 0;
    }
});
```

## **✅ 2. 绕过 `isRooted()` 检测**
📌 **Frida Hook**
```js
Java.perform(function() {
    var SecurityUtils = Java.use("com.example.game.SecurityUtils");
    SecurityUtils.isRooted.implementation = function() {
        console.log("[*] Bypassing root detection");
        return false;
    };
});
```

---

# **5️⃣ 伪造服务器数据**
## **✅ 1. 拦截 & 修改 API 响应**
📌 **修改金币 API 返回值**
```js
Interceptor.attach(Module.findExportByName("libokhttp.so", "ssl_read"), {
    onLeave: function(retval) {
        console.log("[*] Intercepted API response: " + Memory.readUtf8String(retval));
        var fakeResponse = '{"coins": 9999999}';
        Memory.writeUtf8String(retval, fakeResponse);
    }
});
```

---

# **🛠 实战任务**
### **✅ 1. 提取 `libgame.so`**
```bash
adb pull /data/app/com.example.game/lib/arm64/libgame.so .
```
### **✅ 2. 使用 GameGuardian 修改金币**
1. **搜索当前金币数量**
2. **修改为 `9999999`**
### **✅ 3. 使用 Frida Hook 金币系统**
```js
Java.perform(function() {
    var GameClass = Java.use("com.example.game.GameManager");
    GameClass.addCoins.implementation = function(amount) {
        this.addCoins(9999999);
    };
});
```
### **✅ 4. 使用 IDA Pro 修改 `libgame.so`**
1. **搜索 `金币增加函数`**
2. **修改 `MOV R0, #9999999`**
### **✅ 5. 绕过反作弊**
```js
Interceptor.attach(Module.findExportByName(null, "ptrace"), {
    onEnter: function(args) {
        args[0] = 0;
    }
});
```
### **✅ 6. 伪造 API 响应**
```js
Interceptor.attach(Module.findExportByName("libokhttp.so", "ssl_read"), {
    onLeave: function(retval) {
        var fakeResponse = '{"coins": 9999999}';
        Memory.writeUtf8String(retval, fakeResponse);
    }
});
```

---

# **📚 参考资料**
📌 **游戏破解工具**
- `GameGuardian`：[https://gameguardian.net/](https://gameguardian.net/)  
- `Cheat Engine`：[https://cheatengine.org/](https://cheatengine.org/)  

📌 **动态 Hook**
- `Frida`：[https://frida.re](https://frida.re)  

📌 **逆向分析**
- `IDA Pro`：[https://hex-rays.com/](https://hex-rays.com/)  
- `Ghidra`：[https://ghidra-sre.org/](https://ghidra-sre.org/)  

---

🔥 **任务完成后，你将掌握：**  
✅ **如何使用 `GameGuardian`、`Cheat Engine` 修改游戏数据**  
✅ **如何 Hook `addCoins()` 方法，修改游戏金币**  
✅ **如何使用 IDA Pro 修改 `libgame.so`，破解游戏逻辑**  
✅ **如何绕过游戏反作弊，成功 Hook 关键函数**  

🚀 **下一步（Day 39）**：**反反调试！** 🎯