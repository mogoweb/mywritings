# 龙芯迷你主机上安装 Deepin 操作系统

在《(龙芯迷你主机，用来办公怎么样？)[https://mp.weixin.qq.com/s/4WtgcfzNDiaq_9M-KG9G8A]》这篇文章中提到，新买的龙芯迷你主机出厂预装的是统信 UOS 操作系统。作为一个喜欢尝新的程序员，我在进行了基本的体验后，决定将系统换为 Deepin 系统。Deepin 系统是 UOS 操作系统的社区版本，也是开源版本。一般来说，社区版本软件更新更快，但稳定性上可能不如专业版本。

《(各种国产操作系统，一个 U 盘搞定)[https://mp.weixin.qq.com/s/Es6cnRQ65vKtljDtpvma3Q]》一文发出以后，有人留言问是否支持龙芯架构，当时由于手头没有设备，所以没法回答。这回有设备了，可以回答读者的提问，不行。我把制作的 Ventory 启动 U 盘插入这款龙芯迷你主机，然后选择从 U 盘启动，但根本不会进入 U 盘中的系统。还是老老实实用 balenaEtcher 烧写 U 盘吧。这里要提一句，Ventory 无法启动这台龙芯迷你主机，并不是架构的原因，也可能其它龙芯的主机可以用 Ventory 启动。比如同样是 ARM 架构，我用 Ventory 制作的启动 U 盘，可以安装飞腾芯片的主机，但是几款华为的 ARM 主机，怎么也无法进入启动界面。可能还是系统固件的兼容性问题。

Deepin V23 支持很多架构，这其中就包含龙芯。进入 Deepin 官网 https://www.deepin.org/zh/download/ 下载镜像。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202412/images/deepin_loongson_01.png)

选择龙芯架构的 iso 文件，然后使用 balenaEtcher 烧写 U 盘。接下来的安装过程就不赘述了，和其它架构的 Deepin 系统安装一模一样。

安装完毕之后，发现龙芯架构的 Deepin 还处于 preview 阶段，而 x86 架构的 Deepin 则已经发布正式版了。发现一点不足之处，Preview 版本的 Deepin 没有应用商店，只支持 apt 包安装，对我来说，这也不是大问题。

