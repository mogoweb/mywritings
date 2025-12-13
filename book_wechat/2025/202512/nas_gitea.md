# 绿联 NAS（DH4300 Plus）上部署私有 Git 仓库

在《[老登的新玩具：NAS](https://mp.weixin.qq.com/s/5kFWFXAn_c6eEFumQO4kRg)》一文中，我简单介绍了新购置的 NAS 设备。最初入手 NAS 的主要目的，其实很单纯——备份照片，同时作为家庭影视库使用。前段时间也确实“物尽其用”，补完了不少经典电影和美剧。

但副作用也很明显：一连几周，公众号几乎停更，技术研究也被搁置到了一旁。好在待看片单已经清空，现在终于可以把注意力重新拉回到正事上，认真思考一下 NAS 还能发挥哪些价值。

NAS 的用途很多，而我首先想到的，就是用它来搭建一个私有的 Git 代码仓库。

目前市面上的 Git 托管服务非常成熟，国外有 GitHub、GitLab，国内也有 Gitee、Coding 等选择。这些平台大多提供免费的私有仓库，稳定性、可用性和运维保障都相当成熟。从“省心”的角度来看，似乎并没有自己部署 Git 服务的必要。

但问题也恰恰出在这里。

对于个人用户而言，这类托管平台通常都会对仓库容量设定上限，常见的限制是 1G，github 比较大气一些，也不超过 10 GB。而我目前基于 Chromium 进行浏览器产品开发，即便不导入完整的 Chromium Git 历史，仅保留源码本身，加上构建所需的一系列二进制工具，整体体积也早已超过 10 GB。如果要导入完整的 Chromium Git 历史，那容量要直奔 100G 了。

所以，`mojo-browser` 项目只能长期存放在本地开发机上，既不便于多设备协作，也缺乏可靠的备份机制。

现在有了大容量的 NAS，貌似有了新的解决方案。一方面，NAS 的存储空间足够充裕，完全可以承载这类“重量级”代码仓库；另一方面，NAS 本身可以配置定期自动备份，无论是硬盘故障还是误操作，都能多一层数据安全保障。

## Git 软件选择

市面上成熟的 Git 服务器实现并不少，我最先考虑的是 GitLab。一方面是因为在日常工作中在使用，对其功能比较熟悉；另一方面，GitLab 在权限管理、CI/CD、审计和生态完整性方面确实非常强大。

但在进一步调研之后，我很快意识到：**GitLab 并不适合部署在 NAS 这类轻量级设备上**。

GitLab 并不是一个“单体服务”，而是一整套复杂的服务集群，核心组件就包括 PostgreSQL、Redis、Sidekiq、Gitaly、Prometheus 等。这些组件对 CPU 性能、内存容量以及磁盘 IOPS 都比较敏感，哪怕只是小规模自用，也很难算得上“轻量”。

官方给出的硬件要求如下：

| 项目  | 官方最低要求   | 实际推荐         |
| --- | -------- | ------------ |
| CPU | 4 核      | 4–8 核        |
| 内存  | **4 GB** | **8 GB 及以上** |
| 硬盘  | 10 GB 以上 | 20–50 GB     |

而我使用的绿联 DH4300 Plus 搭载的是 ARM 架构的 RK3588C 处理器，8 核 8 线程，主频 1.8 GHz，内存为 8 GB。在这台设备上运行 GitLab，大概率会出现响应迟缓、资源占用过高，甚至稳定性问题。

何况这台 NAS 还要进行照片备份，音视频播放，不能让 Git 服务吃掉太多的内存和 CPU。

综合权衡之后，我最终选择了 **Gitea**。相比 GitLab，Gitea 架构极为简洁，单一二进制即可运行，对 CPU 和内存的需求都要低得多；同时在代码托管、权限管理、Webhook、基础 CI 集成等方面，已经完全能够满足个人和小团队的开发需求。

## 安装 Gitea

绿联 NAS 原生支持 Docker，因此这里直接采用 Docker 部署 的方式，这是目前最简单、也最稳定的安装方案。

首先进入 NAS 的 Docker 管理界面，在镜像仓库中搜索 gitea，选择下载量最多、维护最活跃的官方镜像进行下载。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202512/images/nas_gitea_01.png)

如果在拉取镜像过程中速度较慢或失败，可以为 Docker 配置镜像加速器，例如：https://docker.1ms.run。

镜像下载完成后，基于该镜像创建 Gitea 容器。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202512/images/nas_gitea_02.png)

在容器设置中，建议勾选 “自动重启”，以确保 NAS 重启或 Docker 服务异常恢复后，Gitea 能够自动启动。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202512/images/nas_gitea_03.png)

其余选项保持默认即可，无需额外修改。这里需要特别留意端口映射关系，后续访问和 Git 操作会用到该端口。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202512/images/nas_gitea_04.png)

容器启动后，点击容器右侧的 `快速访问`，即可进入 Gitea 的初始化配置页面。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202512/images/nas_gitea_05.png)

在大多数个人使用场景下，直接采用默认配置即可，点击 “立即安装” 完成初始化。需要注意的是：

> 第一个注册的用户将自动成为 Gitea 的系统管理员。

安装完成后，即可看到熟悉的 Gitea 管理界面：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202512/images/nas_gitea_06.png)

这里需要特别说明一个容易踩到的问题。

绿联 NAS 提供的内网穿透服务，本质上只支持浏览器访问，并且需要经过登录认证。当尝试通过外网地址进行 Git 克隆时，无论是 SSH 还是 HTTPS，都会失败，例如：

```bash
$ git clone git@app-39639-mojo.cn24.ugdocker.link:mogoweb/mojo-browser.git mojo-browser-gitea
正克隆到 'mojo-browser-gitea'...
ssh: connect to host app-39639-mojo.cn24.ugdocker.link port 22: Connection refused
致命错误：无法读取远程仓库。

请确认您有正确的访问权限并且仓库存在。
$ git clone https://app-39639-mojo.cn24.ugdocker.link/mogoweb/mojo-browser.git mojo-browser-gitea
正克隆到 'mojo-browser-gitea'...
致命错误：无法访问 'https://app-39639-mojo.cn24.ugdocker.link/mogoweb/mojo-browser.git/'：Failed to connect to mojo.cn24.ug.link port 80 after 10562 ms: Could not connect to server
```

这是由于内网穿透并未对 Git 协议（SSH / HTTP）提供直通支持所导致的。

解决办法也很简单：在局域网环境下，直接使用 NAS 的内网 IP 地址和映射端口进行访问，例如：

```
$ git clone http://192.168.3.7:39639/mogoweb/mojo-browser.git mojo-browser-gitea
```

至此，一个运行在绿联 NAS 上的私有 Git 仓库就已经成功部署完成了。

NAS 远不只是一个“存照片和电影的盒子”，围绕开发和基础设施还有不少值得折腾的玩法，后续如果有新的实践，也会继续记录。敬请关注，欢迎交流。
