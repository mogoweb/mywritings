# 哭死，几年前重金购买的 AI PC，竟然还比不上免费的云端 GPU 的算力

五年前，那个时候大模型还没有爆发，还没有 AI PC 的概念，但 AI 的苗头已经开始显现。正好那个时候工作比较清闲，就决定学一学 AI。记得当时学的是《机器学习》和《深度学习》这两本书。为了能够更好地实现，就花了将近两万块钱，买了一台带独立显卡的台式机。在这之前，我买电脑都是用的集成显卡，因为我不玩游戏，也没有机会做 3D 建模相关的工作。当时买的电脑配置如下：

* 处理器：Intel Core i9-9900K
* 显卡：NVIDIA GeForce RTX 2080 Ti（11GB显存）
* 内存：32GB
* 硬盘：256 GB SSD + 1TB HDD

但是，我的 AI 之路并没有那么顺利。当时死磕《机器学习》和《深度学习》这两本书，但实在磕不动，太难了，简直是看天书，一个个字都认识，但合在一起就不知道啥意思。本来，我对自己的学习能力还挺有信心的，但这次真的被打击到了。这种理论学习，没有极高的智商加持，真的没法入门。难怪现在那些 AI 天才，刚毕业就能拿到百万年薪。

之后有一段时间，我都没怎么碰 AI。后来，随着 ChatGPT 的爆火，我也想通了，搞不懂 AI 理论没有关系，我可以做应用啊。现在的大模型都提供了 API，集成到自己的产品中，也挺好。

如果是集成大模型，其实对于电脑的要求并不高，但是如果自己部署开源模型，那就需要比较好一点的机器了。但我悲哀的发现，我当年花重金购买的台式机，已经严重落伍了，甚至还不如现在免费的云端 GPU 算力。

这几天无意中浏览到腾讯推出了 Cloud Studio 在线应用开发平台，除了可以开发 C/C++、Java 等应用外，还可以免费使用云端 GPU 来做 AI 开发。当然，免费版本有时长限制，不过每个月 1 万小时，还是挺良心的。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202501/images/tencent_cloud_01.png)

使用 Cloud Studio 之前需要注册腾讯云账号，这个就不赘述了。登录后，点击上图中的 “前往专业版” 按钮，就可以进入 “云端 IDE” 界面。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202501/images/tencent_cloud_02.png)

然后再点击 “免费创建 GPU 空间”。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202501/images/tencent_cloud_03.png)

一般情况下，我们可以选择模板创建空间，当然，如果你希望一切由自己控制，也可以 “直接创建”。这里我选择模板。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202501/images/tencent_cloud_04.png)

这里模板很多，请根据需求选择，我希望使用 ollama 部署 DeepSeek 开源大模型，所以选择了 “Ollama” 模板。出现”空间规格”的选择界面。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202501/images/tencent_cloud_05.png)

可以看到，免费版本提供了 16 GB+ 的显存，8 核 CPU 和 32 GB 的内存，真的是超级强。这里没有列出硬盘空间大小，后来我进入到实例，查了一下，总共提供大约 50 GB 的空间。

```
(base) root@VM-9-56-ubuntu:/workspace# df
Filesystem     1K-blocks     Used Available Use% Mounted on
overlay         51290592 26192784  22459984  54% /
tmpfs              65536        0     65536   0% /dev
tmpfs           16041936        0  16041936   0% /sys/fs/cgroup
/dev/vdb        51290592 26192784  22459984  54% /etc/.hai
tmpfs           16041936       24  16041912   1% /dev/shm
tmpfs           16041936       12  16041924   1% /proc/driver/nvidia
/dev/vda2       30856076 19287576  10215316  66% /usr/bin/nvidia-smi
tmpfs            3208388      852   3207536   1% /run/nvidia-persistenced/socket
udev            15991728        0  15991728   0% /dev/nvidia0
tmpfs           16041936        0  16041936   0% /proc/asound
tmpfs           16041936        0  16041936   0% /proc/acpi
tmpfs           16041936        0  16041936   0% /proc/scsi
tmpfs           16041936        0  16041936   0% /sys/firmware
```

除掉系统所占用的空间，大小勉强够用。

接下来显示所创建的实例。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202501/images/tencent_cloud_06.png)

点击其中的一个实例，可以看到我们非常熟悉的界面，一个浏览器中仿真的 Visual Studio Code 编辑器。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202501/images/tencent_cloud_07.png)

试用了一下，和在本地使用 VS Code 没什么区别，也能安装各种插件。

如果要操作创建的主机实例，官方没有提供 ssh 之类的登录方法。但我们可以在 VS Code 中打开中端，比如我要用 ollama 安装 DeepSeek 大模型：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202501/images/tencent_cloud_08.png)

还可以在 VS Code 中创建多个终端。DeepkSeek 大模型安装好后，就可以对话了。

如果你想学习 AI 相关开发，完全不需要购买昂贵的 AI PC，只需要一台普通的电脑就可以。现在的大模型越来越大，个人消费级电脑也没法去进行模型训练或模型调优，还不如直接使用云端 GPU。如果觉得免费版本不够用，也可以付费，相当于租一台高性能的 GPU 电脑。这样可以避免过几年后，电脑落伍的尴尬。