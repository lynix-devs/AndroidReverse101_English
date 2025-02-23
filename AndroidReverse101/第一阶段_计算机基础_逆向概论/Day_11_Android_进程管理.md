# **📜 Day 11: Android 进程管理**

## **📌 学习目标**
✅ **理解 Android 进程管理的工作方式**，包括 **Zygote 进程、App 进程、System Server 进程**。  
✅ **掌握 Android 进程的生命周期**，学习 **前台进程、后台进程、服务进程、缓存进程的调度策略**。  
✅ **学习 Android 的 OOM（Out Of Memory）管理机制**，理解 **进程优先级与杀死策略**。  
✅ **分析 /proc 目录下的进程信息**，通过 `ps`, `top`, `dumpsys activity` 命令获取进程状态。  
✅ **学习如何 Hook Android 进程管理机制**，进行应用持久化 & 逆向调试。

---

# **1️⃣ Android 进程管理概述**
在 Android 中，应用程序 **通常不会直接创建进程**，而是由 **Zygote** 负责孵化。  

| **进程类型** | **作用** | **示例** |
|------------|------|------|
| **Zygote** | 负责 Fork 其他进程 | `/system/bin/app_process` |
| **System Server** | 管理系统服务 | `system_server` 进程 |
| **App 进程** | 运行应用 | `com.example.app` |
| **Native 进程** | 运行 C/C++ 代码 | `surfaceflinger`, `mediaserver` |

---

# **2️⃣ 关键进程解析**
## **✅ 1. Zygote 进程**
**Zygote 是 Android 进程的起点**，其主要作用是：
- **预加载类库 & 资源**，减少应用启动时间。
- **Fork 子进程**，所有 App 进程均由 Zygote 复制而来。

📌 **查看 Zygote 进程**
```bash
adb shell ps -A | grep zygote
```
输出示例：
```
zygote64   1234  567   123456K fg    00000000 S zygote64
zygote     1235  567   123456K fg    00000000 S zygote
```

📌 **Zygote 如何 Fork 进程？**
```java
public static void main(String[] argv) {
    ZygoteServer zygoteServer = new ZygoteServer();
    while (true) {
        ZygoteConnection connection = zygoteServer.acceptCommandPeer();
        connection.runOnce();
    }
}
```
📌 **逆向分析 Zygote**
```bash
strings /system/bin/app_process
```

---

## **✅ 2. System Server 进程**
**System Server 进程** 负责管理 Android 系统服务，如 **AMS（ActivityManagerService）**, **PMS（PackageManagerService）**。

📌 **查看 System Server 进程**
```bash
adb shell ps -A | grep system_server
```
输出：
```
system    1356  567   456789K fg    00000000 S system_server
```

📌 **System Server 关键代码**
```java
public static void main(String[] args) {
    SystemServer server = new SystemServer();
    server.run();
}
```

📌 **分析 System Server 中的 AMS**
```bash
adb shell dumpsys activity
```

---

## **✅ 3. 应用进程**
应用进程通常由 **Zygote Fork**，负责执行应用代码。

📌 **查看当前运行的 App 进程**
```bash
adb shell ps -A | grep com.example.app
```

📌 **查看应用进程详情**
```bash
adb shell dumpsys meminfo com.example.app
```

📌 **杀死应用进程**
```bash
adb shell am force-stop com.example.app
```

---

# **3️⃣ 进程优先级 & OOM 机制**
Android 采用 **OOM（Out Of Memory）优先级管理机制**，根据进程的重要性决定 **是否杀死进程**。

| **优先级** | **进程类型** | **是否可杀死** |
|-----------|----------|-----------|
| **0（最高）** | 前台进程（前台 Activity） | ❌ 不能杀死 |
| **1** | 可见进程（后台 Activity） | ❌ 通常保留 |
| **2** | 服务进程（后台 Service） | ✅ 低内存时杀死 |
| **3** | 后台进程（不可见 Activity） | ✅ 低内存时杀死 |
| **4（最低）** | 缓存进程（长期未使用的 App） | ✅ 优先被杀死 |

📌 **查看进程 OOM 级别**
```bash
adb shell cat /proc/1234/oom_adj
```
输出：
```
0   # 前台进程，不会被杀死
6   # 后台进程，低内存时可能被杀死
15  # 缓存进程，优先被杀死
```

📌 **调整进程 OOM 级别**
```bash
adb shell echo -17 > /proc/1234/oom_adj
```
👉 **可用于保护进程，防止被系统杀死（需 Root）**。

---

# **4️⃣ Android 进程管理 API**
## **✅ 1. ActivityManager**
```java
ActivityManager am = (ActivityManager) getSystemService(Context.ACTIVITY_SERVICE);
List<ActivityManager.RunningAppProcessInfo> processes = am.getRunningAppProcesses();
for (ActivityManager.RunningAppProcessInfo process : processes) {
    Log.d("Process", "PID: " + process.pid + " Name: " + process.processName);
}
```

## **✅ 2. 监听进程状态**
```java
ProcessLifecycleOwner.get().getLifecycle().addObserver(new LifecycleObserver() {
    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    public void onAppBackgrounded() {
        Log.d("Process", "App moved to background!");
    }
});
```

## **✅ 3. 让进程持久运行**
```java
startForegroundService(new Intent(this, MyService.class));
```

---

# **5️⃣ 逆向分析 & Hook 进程**
## **✅ 1. Hook Android 进程调度**
```bash
frida -U -n system_server -e "Interceptor.attach(Module.findExportByName(null, 'fork'), { onEnter: function(args) { console.log('fork called'); }})"
```

## **✅ 2. 绕过进程杀死**
```bash
adb shell setprop persist.sys.background_process_limit 0
```

## **✅ 3. 限制某进程 CPU 使用**
```bash
taskset -p 0x01 1234
```

---

# **🛠 实战任务**
### **✅ 1. 检查 Android 设备上的进程**
```bash
adb shell ps -A
```
### **✅ 2. 解析 Zygote 进程**
```bash
adb shell ps -A | grep zygote
```
### **✅ 3. 获取 App 进程的内存信息**
```bash
adb shell dumpsys meminfo com.example.app
```
### **✅ 4. 监听进程状态**
```java
ProcessLifecycleOwner.get().getLifecycle().addObserver(new LifecycleObserver() {
    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    public void onAppBackgrounded() {
        Log.d("Process", "App moved to background!");
    }
});
```

---

# **📚 参考资料**
📌 **Android 进程管理**
- `Zygote 进程`：[https://source.android.com/devices/tech/dalvik/zygote](https://source.android.com/devices/tech/dalvik/zygote)  
- `AMS 进程调度`：[https://developer.android.com/guide/components/activities/process-lifecycle](https://developer.android.com/guide/components/activities/process-lifecycle)  

📌 **Android 逆向**
- `Frida`：[https://frida.re](https://frida.re)  
- `Android 进程 Hook`：[https://github.com/lasting-yang/AndroidReverseStudy](https://github.com/lasting-yang/AndroidReverseStudy)  

---

🔥 **任务完成后，你将掌握：**  
✅ **Android 进程管理的核心机制**  
✅ **如何查看 & 调试 Android 进程**  
✅ **如何 Hook Android 进程管理，进行持久化与逆向分析**  

🚀 **下一步（Day 12）**：**Android 权限机制解析！** 🎯