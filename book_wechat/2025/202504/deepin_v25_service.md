# deepin V25 磐石系统下添加系统服务

在上一篇文章《[配置 UOS/deepin 系统远程桌面，实现多台电脑协同办公](https://mp.weixin.qq.com/s/wZnTJJuoJzsXZBAglv2NVQ)》中介绍了在国产系统 UOS/deepin 下添加 VNC 服务，实现远程桌面访问，但是将该方法应用到 deepin V25 上时，却碰到了问题。这是由于 V25 系统引入了磐石系统，这是一种不可变文件系统，根文件系统通常是只读的。即使拥有 root 权限，也无法将文件写入根文件系统。比如执行以下命令:

```
sudo vi /lib/systemd/system/x11vnc.service
```

最后保存时却提示：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202504/images/deepin_v25_service_01.png)

查了一下资料，发现 systemd 有用户级服务（user service）的概念，即在用户 HOME 目录下创建服务文件，然后通过 `systemctl --user` 命令来管理。

下面说明一下通过用户级 systemd 服务添加 x11vnc 服务的步骤。

1. ​创建用户服务单元文件

在用户目录下新建.service文件：~/.config/systemd/user/x11vnc.service。内容如下：

```
[Unit]
Description="x11vnc"
After=display-manager.service

[Service]

ExecStart=/usr/bin/x11vnc -auth guess -loop -forever -safer -shared -ultrafilexfer -bg
ExecStop=/usr/bin/killall x11vnc

[Install]
WantedBy=default.target
```
2. ​启用并管理服务

执行以下命令激活用户级服务：

```
systemctl --user daemon-reload
systemctl --user enable --now x11vnc.service  # 启用服务
systemctl --user start x11vnc.service  # 启动服务
systemctl --user status x11vnc.service  # 检查服务状态
```

此方法无需 root 权限，服务生命周期与用户会话绑定。

不过，这种方法有一个缺点，就是系统启动后，需要登录进去，这个服务才会启动。这也不难理解，因为服务是绑定在用户会话上的，不登录进去，服务自然也就无法启动。

有没有更好的方法呢？后来还是被我找到了，磐石系统虽然根文件系统是只读的，但允许在 /etc/systemd/system/ 下添加服务文件，因为这些目录位于独立可写分区。

接下来的操作就非常简单了，直接在 /etc/systemd/system/ 下创建 x11vnc.service 文件即可。

```
sudo vi /etc/systemd/system/x11vnc.service
```

将如下内容粘贴到文件中：

```
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

接下来启用并启动系统服务：

```
# 重新加载systemd配置
sudo systemctl daemon-reload
 
# 设置开机自启
sudo systemctl enable x11vnc
 
# 立即启动服务
sudo systemctl start x11vnc
 
# 验证服务状态
sudo systemctl status x11vnc
```

这种方法和[上一篇文章](https://mp.weixin.qq.com/s/wZnTJJuoJzsXZBAglv2NVQ)方法一样，区别就在于创建的 x11vnc.service 文件位置不同。

至此，问题得到解决。