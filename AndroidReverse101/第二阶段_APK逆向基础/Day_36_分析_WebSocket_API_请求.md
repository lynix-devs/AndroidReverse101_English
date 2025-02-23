# **📜 Day 36: 分析 WebSocket & API 请求**

## **📌 学习目标**
✅ **掌握 WebSocket 与 REST API 的基本原理，理解它们在 Android 应用中的作用**。  
✅ **学习如何使用 `Burp Suite`、`Mitmproxy`、`Wireshark` 拦截 & 分析 API 请求**。  
✅ **学习如何使用 `Frida` Hook WebSocket & API 代码，拦截数据并修改请求**。  
✅ **掌握如何分析 HTTP/HTTPS/WebSocket 协议，实现数据篡改 & 重放**。  
✅ **实战：拦截 WebSocket & API 通信，逆向分析数据交互流程！**  

---

# **1️⃣ 什么是 WebSocket & API 请求？**
Android 应用通常使用 **WebSocket** 或 **HTTP API** 进行数据通信：
- **WebSocket**：实时通信协议，适用于聊天、直播、推送等。
- **REST API**（HTTP API）：客户端与服务器间的常见通信方式，适用于数据请求、身份验证等。

📌 **常见 API 交互方式**
| **协议** | **端口** | **特点** |
|---------|--------|--------|
| **HTTP** | 80 | 明文传输，可拦截 & 修改 |
| **HTTPS** | 443 | 加密传输，需绕过 SSL Pinning |
| **WebSocket** | 80/443 | 实时通信，持续连接 |
| **WSS（加密 WebSocket）** | 443 | 需绕过 SSL Pinning |

📌 **常见的 API 请求**
```java
OkHttpClient client = new OkHttpClient();
Request request = new Request.Builder()
    .url("https://api.example.com/user")
    .build();
Response response = client.newCall(request).execute();
```

📌 **WebSocket 连接**
```java
WebSocket webSocket = client.newWebSocket(
    new Request.Builder().url("wss://chat.example.com/socket").build(),
    new WebSocketListener() {
        @Override
        public void onMessage(WebSocket webSocket, String text) {
            Log.d("WebSocket", "Received: " + text);
        }
    }
);
```

---

# **2️⃣ 拦截 & 分析 API 请求**
## **✅ 1. 使用 Burp Suite 拦截 HTTP API**
📌 **步骤**
1. **启动 Burp Suite**
2. **设置代理**
   - 监听端口 `8080`
   - 进入 `Proxy` -> `Intercept` 关闭 `Intercept`
3. **在 Android 设备上配置代理**
   - 代理地址填写 **PC IP**
   - 端口 **8080**
4. **安装 Burp 证书**
   ```bash
   adb push burp_cert.der /sdcard/
   ```
5. **抓取 API 请求**

📌 **未绕过 SSL Pinning**
- 证书错误，无法解密 HTTPS 数据。

📌 **成功绕过**
- 可看到 API 请求数据，如 `Token`、`User ID`、`Session Key`。

---

## **✅ 2. 使用 Wireshark 监听 WebSocket 流量**
📌 **监听 Android 设备网络**
```bash
tcpdump -i any port 443 -w ws_traffic.pcap
```
📌 **在 Wireshark 解析**
```bash
wireshark -r ws_traffic.pcap
```
📌 **过滤 WebSocket**
```
websocket
```
📌 **分析数据**
- 找到 WebSocket 连接 `wss://chat.example.com/socket`
- 右键 `Follow WebSocket Stream`，查看完整数据交互

---

## **✅ 3. 使用 `Frida` Hook API 请求**
📌 **Hook `OkHttp` API 请求**
```js
Java.perform(function() {
    var Request = Java.use("okhttp3.Request");
    Request.body.implementation = function() {
        console.log("[*] Intercepted API Request: " + this.body());
        return this.body();
    };
});
```
📌 **运行 Frida**
```bash
frida -U -n com.example.app -e "..."
```

---

# **3️⃣ WebSocket 数据拦截 & 修改**
## **✅ 1. Hook `WebSocketListener.onMessage()`**
📌 **拦截 WebSocket 消息**
```js
Java.perform(function() {
    var WebSocketListener = Java.use("okhttp3.WebSocketListener");
    WebSocketListener.onMessage.overload("okhttp3.WebSocket", "java.lang.String").implementation = function(ws, msg) {
        console.log("[*] Intercepted WebSocket Message: " + msg);
        return this.onMessage(ws, msg);
    };
});
```
📌 **修改 WebSocket 消息**
```js
Java.perform(function() {
    var WebSocket = Java.use("okhttp3.WebSocket");
    WebSocket.send.implementation = function(message) {
        console.log("[*] Sending WebSocket Message: " + message);
        if (message.includes("old_value")) {
            message = message.replace("old_value", "new_value");
        }
        return this.send(message);
    };
});
```

---

# **4️⃣ 绕过 SSL Pinning**
## **✅ 1. Hook `OkHttp`**
📌 **Frida Hook**
```js
Java.perform(function() {
    var CertificatePinner = Java.use("okhttp3.CertificatePinner");
    CertificatePinner.check.overload("java.lang.String", "java.util.List").implementation = function(hostname, peerCertificates) {
        console.log("[*] Bypassing SSL Pinning for: " + hostname);
        return;
    };
});
```

## **✅ 2. Hook `checkServerTrusted()`**
📌 **Frida Hook**
```js
Java.perform(function() {
    var TrustManager = Java.use("javax.net.ssl.X509TrustManager");
    TrustManager.checkServerTrusted.implementation = function(chain, authType) {
        console.log("[*] Bypassing SSL Pinning");
        return;
    };
});
```

---

# **5️⃣ API 伪造 & 重放**
## **✅ 1. 使用 `Postman` 发送 API 请求**
📌 **复制 API 请求**
- 在 `Burp Suite` 找到 API 请求
- 复制 `Headers` 和 `Body`
- 在 `Postman` 发送请求

## **✅ 2. 伪造 WebSocket 消息**
📌 **使用 WebSocket Client**
```python
import websocket

ws = websocket.WebSocket()
ws.connect("wss://chat.example.com/socket")
ws.send('{"type": "message", "content": "Hello!"}')
print(ws.recv())
```

---

# **🛠 实战任务**
### **✅ 1. 使用 Burp Suite 拦截 API**
```bash
adb push burp_cert.der /sdcard/
```
### **✅ 2. Hook `OkHttp` API 请求**
```js
Java.perform(function() {
    var Request = Java.use("okhttp3.Request");
    Request.body.implementation = function() {
        console.log("[*] Intercepted API Request: " + this.body());
        return this.body();
    };
});
```
### **✅ 3. Hook WebSocket**
```js
Java.perform(function() {
    var WebSocketListener = Java.use("okhttp3.WebSocketListener");
    WebSocketListener.onMessage.overload("okhttp3.WebSocket", "java.lang.String").implementation = function(ws, msg) {
        console.log("[*] Intercepted WebSocket Message: " + msg);
        return this.onMessage(ws, msg);
    };
});
```
### **✅ 4. 绕过 SSL Pinning**
```js
Java.perform(function() {
    var CertificatePinner = Java.use("okhttp3.CertificatePinner");
    CertificatePinner.check.overload("java.lang.String", "java.util.List").implementation = function(hostname, peerCertificates) {
        return;
    };
});
```
### **✅ 5. 伪造 API 请求**
```python
import requests
response = requests.post("https://api.example.com/vip", data={"user": "admin"})
print(response.text)
```
### **✅ 6. 伪造 WebSocket 消息**
```python
import websocket
ws = websocket.WebSocket()
ws.connect("wss://chat.example.com/socket")
ws.send('{"type": "message", "content": "Hello!"}')
print(ws.recv())
```

---

# **📚 参考资料**
📌 **网络分析**
- `Burp Suite`：[https://portswigger.net/burp](https://portswigger.net/burp)  
- `Wireshark`：[https://www.wireshark.org/](https://www.wireshark.org/)  

📌 **Frida Hook**
- `Frida`：[https://frida.re](https://frida.re)  

---

🔥 **任务完成后，你将掌握：**  
✅ **如何拦截 & 分析 API / WebSocket 通信**  
✅ **如何 Hook 网络请求，拦截 & 修改 API 数据**  
✅ **如何绕过 SSL Pinning，抓取加密流量**  

🚀 **下一步（Day 37）**：**破解应用限制（实战）！** 🎯