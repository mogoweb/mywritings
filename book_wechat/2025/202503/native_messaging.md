# 通过浏览器扩展获取本机 MAC 地址

在 Web 技术主导的 B/S 架构项目中，获取终端设备硬件信息（如 MAC 地址）的需求经常会碰到。尽管 Electron/CEF 等混合应用框架可通过系统级 API 轻松实现，但纯浏览器环境下的硬件信息获取则不那么容易。因为现代浏览器基于沙箱机制和隐私保护策略，严格禁止网页直接访问底层硬件资源。

但用户的需求不能不考虑，特别是在做商业项目时，这时就不得不给出方案，总结下来有如下方案：

* 扩展 JS API：比如以前在做广电 TVOS 系统，局方就制定了一个 NGB-H 的 JS API 扩展标准。标准里面定义了很多与硬件有关的 JS API，其中就包括获取 IP 地址、MAC 地址等。这种方案的缺点是需要修改浏览器内核，工作量比较大。而且 Web 应用不能运行在标准的浏览器上，会给开发和调试带来不便。
* 本地 http 服务：开发一个系统服务程序，该服务程序也同时是一个精简的 HTTP server，通过 RESTful API 提供硬件信息查询。web 应用通过 XMLHttpRequest 或 fetch 调用本地运行的 HTTP server，HTTP server 以 JSON 字符串的形式返回本机信息。这种方案需要在客户端安装一个服务，部署起来比较麻烦，而且需要考虑跨域的问题，需要特别小心。
* chrome 扩展 + 本地应用：chrome 扩展提供了一种 Native Messaging 机制，用于和本地应用进行交互。这里的本地应用可以是一个可执行程序，也可以是一个脚本。这种方案不需要修改浏览器内核，插件开发也相对比较简单，缺点是部署有些麻烦，需要将本地应用（脚本）部署到指定的位置。

这里就详细讨论第三种方案，以获取 MAC 为例，说明通过浏览器扩展获取本机 MAC 地址的方法。本示例以 Deepin 系统和 Chrome 浏览器为例，对于 Windows 和 Mac OS 脚本可能需要稍加修改。

## Native Messaging

Native Messaging 是一种允许浏览器扩展与本地应用程序（Native Host）进行安全通信的机制，其核心设计在于**突破浏览器沙箱限制**，实现跨进程的 JSON 消息交互。

* **浏览器扩展**：通过声明 `nativeMessaging` 权限，向浏览器注册通信需求  
* **Native Host**：本地可执行程序（如 qt 应用程序、Python 脚本），需在系统中注册路径和权限（通过 manifest 文件）  
* **消息通道**：基于标准输入输出（stdin/stdout）实现双向 JSON 数据传输

通过这种机制，开发者能在保证安全性的前提下，将浏览器功能扩展到本地系统层级，广泛应用于企业工具、硬件控制等复杂场景。

## 示例

本文的完成示例程序可以在 https://e.coding.net/mogoweb/qt-in-action/chrome-extensions-samples.git 获取。

### **一、项目结构**
```
native-messaging/
├── extension
│   ├── icon-128.png         # 插件图标
│   ├── main.html            # 插件界面
│   ├── main.js              # 插件脚本
│   └── manifest.json        # 插件配置文件
├── host/
│   ├── com.uniontech.browser.extension.getmac.json    # Native Host 配置文件
│   └── native-getmac        # 获取 MAC 地址，和插件通讯的 python 脚本

```

### **二、核心代码实现**

#### 1. **Chrome 插件部分**
**manifest.json** 
```json
{
  "manifest_version": 3,
  "name": "GetMac",
  "version": "1.0",
  "description": "Get host MAC address.",
  "action": {
    "default_title": "Get host MAC address",
    "default_popup": "main.html"
  },
  "icons": {
    "128": "icon-128.png"
  },
  "permissions": ["nativeMessaging"]
}
```

**main.js** 
```javascript
function connect() {
  const hostName = 'com.uniontech.browser.extension.getmac';
  appendMessage('Connecting to native messaging host <b>' + hostName + '</b>');
  port = chrome.runtime.connectNative(hostName);
  port.onMessage.addListener(onNativeMessage);
  port.onDisconnect.addListener(onDisconnected);
  updateUiState();
}

document.addEventListener('DOMContentLoaded', function () {
  document.getElementById('connect-button').addEventListener('click', connect);
  document
    .getElementById('send-message-button')
    .addEventListener('click', sendNativeMessage);
  updateUiState();
});
```

#### 2. **本地 python 程序**
**native-getmac** 
```python
import struct
import sys
import threading
import queue as Queue
import uuid

def get_mac_address():
  mac_num = uuid.getnode()  # 获取48位整型地址
  mac_hex = [
      f'{(mac_num >> shift) & 0xff:02x}'  # 按字节分割并转十六进制
        for shift in range(0, 48, 8)        # 从高位到低位处理
    ][::-1]  # 反转顺序以符合人类阅读习惯
  return ":".join(mac_hex)

# Helper function that sends a message to the webapp.
def send_message(message):
   # Write message size.
  sys.stdout.buffer.write(struct.pack('I', len(message)))
  # Write the message itself.
  sys.stdout.write(message)
  sys.stdout.flush()

# Thread that reads messages from the webapp.
def read_thread_func(queue):
  message_number = 0
  while 1:
    # Read the message length (first 4 bytes).
    text_length_bytes = sys.stdin.buffer.read(4)

    if len(text_length_bytes) == 0:
      if queue:
        queue.put(None)
      sys.exit(0)

    # Unpack message length as 4 byte integer.
    text_length = struct.unpack('@I', text_length_bytes)[0]

    # Read the text (JSON object) of the message.
    text = sys.stdin.buffer.read(text_length).decode('utf-8')

    if text == '{"text":"exit"}':
      break

    if queue:
      queue.put(text)
    else:
      # In headless mode just send an echo message back.
      send_message('{"echo": %s}' % text)

def main():
  
  send_message('"Mac: ' + get_mac_address() + '"')
  read_thread_func(None)
  sys.exit(0)

if __name__ == '__main__':
  main()
```

#### 3. **Native Host 配置**
**com.uniontech.browser.extension.getmac.json** 
```json
{
  "name": "com.uniontech.browser.extension.getmac",
  "description": "Browser Native Messaging API to get MAC address",
  "path": "HOST_PATH",
  "type": "stdio",
  "allowed_origins": ["chrome-extension://npjblgiigcihbbfekfndobpbfibgldee/"]
}
```

注意，这个 json 中的 HOST_PATH 会通过 install_host.sh 脚本替换为实际的地址。

### **三、注册与部署**

  ```bash
  ./install_host.sh
  ```

  这个脚本实际上是将 com.uniontech.browser.extension.getmac.json 文件中的 HOST_PATH 替换成实际地址，并复制到 `~/.config/google-chrome/NativeMessagingHosts/` 目录。

### **四、功能测试**
1. 加载插件到 Chrome（`chrome://extensions` → 开发者模式 → 加载已解压的扩展程序）
2. 点击插件，点击 connect 按钮。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202503/images/native_messaging_01.png)

在上面的界面中，我们还可以在插件中给本地应用发送消息，并接收本地应用返回的消息。

## 小结

本文探讨了一种通过 Chrome 插件和 Native Messaging 机制获取 MAC 地址的方法。通过修改本机 native 应用，我们还可以获取更多的硬件信息，甚至可以做更多的硬件控制。这种方法不需要修改浏览器内核，在 Linux、Windows 等系统都适用，甚至在 firefox 等浏览器上也有类似的机制，适应面广。

希望本文提供的方法对大家有用，如果有更好的方案，欢迎交流。