# 配置 UOS/deepin 系统远程桌面，实现多台电脑协同办公

由于开发工作的需要，我的办公桌上目前有多台电脑。一台是 i7 配置的电脑，运行 UOS V20 系统，作为主力办公电脑，负责处理企业微信、OA 等任务，并偶尔进行代码编译和验证软件在 UOS V20 系统下的兼容性；另一台 i7 配置的电脑则安装了 deepin V23 系统，作为主力开发机，用于编译应用；还有一台电脑运行的是 V25 系统，主要用于测试应用在该系统下的兼容性。此外，我还有两台迷你主机，一台搭载兆芯 CPU，运行 UOS V20 系统，由于 CPU 性能有限，通常用来进行一些应用测试；另一台则搭载龙芯 CPU，用来验证应用在龙芯系统上的表现。

这五台设备是常驻设备，有时为了验证 ARM 系统上的问题，还会临时添加其他设备。如此众多的电脑，如果每台都配备显示器、键盘和鼠标，办公桌几乎无法容纳。所以，我使用一台显示器和一套键盘鼠标来管理这五到六台机器，这就需要依赖远程桌面技术来便捷地管理和操作其他电脑。

对于普通的命令行操作，使用 ssh 连接即可满足需求，但大部分情况下，我需要使用图形界面。因此，我尝试使用远程桌面功能来访问这些设备。

远程桌面常用的协议有 RDP，例如在 UOS 或 deepin 系统中可以安装 xrdp 包来实现。然而，经过使用发现，这种方式常常会出现黑屏问题，尤其是在远程计算机已经登录的情况下。于是，我寻找到了另一种方案：结合 X11 和 VNC 技术。

下面是配置的详细步骤，我们首先从被控制端的电脑开始：

## **安装x11vnc**
```bash
# 更新软件仓库
sudo apt update

# 安装x11vnc
sudo apt install x11vnc -y
```

由于设备在局域网内供自己使用，为了简化配置，我选择不设置 VNC 密码。

## **创建系统服务文件**
```bash
# 创建服务配置文件
sudo vi /lib/systemd/system/x11vnc.service
```

将以下内容粘贴到文件中：

```plain
[Unit]
Description="x11vnc"
Requires=display-manager.service
After=display-manager.service

[Service]
ExecStart=/usr/bin/x11vnc -auth guess -loop -forever -safer -shared -ultrafilexfer -bg -o /var/log/x11vnc.log
ExecStop=/usr/bin/killall x11vnc

[Install]
WantedBy=multi-user.target
```

**参数说明**：

+ `-auth guess`：自动检测X11认证文件
+ `-forever`：保持服务持续运行
+ `-shared`：允许多客户端同时连接

## **启用并启动服务**
```bash
# 重新加载systemd配置
sudo systemctl daemon-reload

# 设置开机自启
sudo systemctl enable x11vnc

# 立即启动服务
sudo systemctl start x11vnc

# 验证服务状态
sudo systemctl status x11vnc
```

若输出显示`active (running)`则表示配置成功。

```
● x11vnc.service - "x11vnc"
   Loaded: loaded (/lib/systemd/system/x11vnc.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2025-04-01 20:49:48 CST; 9s ago
 Main PID: 5117 (x11vnc)
    Tasks: 2 (limit: 4915)
   Memory: 21.7M
   CGroup: /system.slice/x11vnc.service
           ├─5117 /usr/bin/x11vnc -auth guess -loop -forever -safer -shared -ultrafilexfer -bg -o /var/log/x11
           └─5118 /usr/bin/x11vnc -auth guess -loop -forever -safer -shared -ultrafilexfer -bg -o /var/log/x11
```

接下来配置主控端。在主控端，VNC 客户端有多种选择，我选择了我一直在使用的 **Remmina 远程桌面**，它可以从 UOS 应用商店中下载安装。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202504/images/remote_desktop_01.png)

安装完成后，打开 Remmina 远程桌面，点击左上角的加号 (+) 图标，创建一个新的连接配置文件。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202504/images/remote_desktop_02.png)

在配置界面中，选择 Remmina VNC 插件作为协议，填写被控端计算机的 IP 地址，并输入该计算机系统的用户名和密码。完成设置后，点击保存并连接，就可以看到被控端的桌面了。

Remmina 支持同时管理多个会话，并可随时切换会话，这使得管理多台电脑变得更加高效和便捷。

这样，你就能够顺利地在一台显示器、键盘和鼠标上管理多台设备，实现高效办公。

你还有其他的方案吗？欢迎留言讨论！
