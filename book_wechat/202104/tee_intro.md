<!--
 * @Author: your name
 * @Date: 2021-04-21 11:06:45
 * @LastEditTime: 2021-04-30 17:14:37
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: /mywritings/book_wechat/202104/tee_intro.md
-->
# 聊一聊可信执行环境

可信执行环境（TrustedExecution Environment, TEE）是一个与智能设备安全相关的话题。在开展这个话题之前，先看一个经济学上的话题：

> 某个汽车公司生产的一款汽车，设计有点问题，如果汽车发生追尾，汽车的油箱就会爆炸，很容易造成车里人的伤亡。这时如果汽车公司在油箱旁边加一块挡板，这块挡板只需要16元，就能大大降低伤亡的数字。汽车公司知道这个情况，但他们算过一笔账，如果每辆汽车都加一块16元的挡板，成本就会增加很多，超过了他们对意外的赔偿，所以他们宁愿赔偿那些伤亡者，也不愿加这块挡板。

很多人看到这儿，肯定会骂这些无良商家，为了追求利润置人命于不顾，简直没有道德底线。但经济学家并不这么看：

> 汽车制造商要不要给汽车加一块挡板，表面上看是制造商自己的决定，但其实最终是消费者的决定。我们要明白，要提高汽车的安全性能，加一块挡板是可以的，换一种材质、刹车设计、安全气囊都是可以的。但这里加一点，那里加一点，汽车的总价就上升了。这些加起来，都会成为汽车的成本，由消费者承担。
> 
> 消费者接受不接受呢？消费者愿意把他们最后一元钱放到安全性能上呢，还是汽车的功能上？还是放到汽车外形的美观上？不同的消费者有不同的选择。

这肯定会颠覆我们的认知，其实只要仔细想一想，确实是这么一个道理。比如 **沃尔沃** 主打安全，是公认的安全做得最好的汽车品牌，但卖得并不好。校车承载的是祖国的花朵，我们也不会把校车打造得像坦克那么坚固。如果我们说生命是无价的，安全性是我们不惜一切代价都应该追求的目标，那我们就再也不会在马路上见到我们今天见到的那些汽车了。汽车和飞机等现代化交通工具也不会发明出来。

回到手机安全上来，如果我们问消费者，是否需要一个安全的手机，大家肯定会回答需要。但如果我们追加一个条件：为了手机安全，大家可能需要额外花费 1000 元，还会有多少人愿意呢？

所以说，手机安全看似非常重要，但很多情况下也是可以舍弃的。更有甚者，如果安全影响到便利性，用户还会想方设法牺牲安全性。想想多少人曾经 root 过 Android 系统、越狱 IOS 系统，长期以来，很多人的 Windows 系统都是无密码直接进入。

现在的手机厂商很少以安全作为卖点，就是这个道理。

然而随着移动通信和互联网技术的飞速发展，智能设备在各个领域扮演着越来越重要的角色。购物、支付、炒股、自动驾驶 ...，智能设备上各种应用不断涌现，对智能设备安全的要求也越来越高。如果一些黑客能够破解智能设备的root权限，进而盗取用户数据或其他关键信息，造成用户数据的泄露或滥用，背锅的也会是厂商，所以系统厂商也在默默发力，致力于提高系统安全。

下面先盘点以下智能设备在安全方面采取了哪些措施，最后引出**可信执行环境**。

#### 富执行环境（Rich Execution Environment，REE)

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202104/images/tee_intro_01.png)

REE 通常就是指运行着 Android、IOS、Linux等系统的智能设备，其特点是操作系统提供系统安全性：

* 应用隔离，各个应用程序只能访问自己的数据
* 权限管理，普通用户和应用程序对系统数据的访问受限
* 通用，功能丰富，但应用程序和操作系统均有可能引入漏洞
* 开放，可扩展

但这样的系统，由于 OS 代码极其庞大和复杂，存在系统漏洞也是难免。一个系统发布之后，经常会收到系统安全补丁，就是因为系统过于复杂，很难进行全面的安全检验和认证。此外，这些系统还有一个很大的漏洞，那就是系统中的特权用户（比如 Android、Linux 系统的 root）。特权用户拥有所有数据的访问权，这就意味着如果黑客攻击系统获得特权用户权限，那整个系统就毫无秘密可言。目前的攻击手段一般是通过应用程序的漏洞，获取特权用户权限，进而控制整个系统。

#### 虚拟化技术实现隔离

虚拟化技术是指在同一台计算机上通过 hypervisor 虚拟出多个包括完整虚拟机系统镜像，每个虚拟机拥有独立的操作系统和硬件资源。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202104/images/tee_intro_02.png)

从图中可以看到，这种虚拟方案依然是基于软件的隔离，缺乏硬件安全性。整个系统的安全突破口在于 Hypervisor。 Hypervisor 可以访问所有 OS 的数据，进而可以访问所有 App 的数据，恶意软件可通过控制 hypervisor 窃取各种密钥。当然这种架构最大的问题还在于系统资源消耗大，一般用在服务器上。

#### 安全元件（Secure Element，SE）实现安全平台

安全元件（Secure Element）简称 SE，通常以芯片或 SD 卡等形式提供。为防止外部恶意解析攻击，保护数据安全，在芯片中具有加密/解密逻辑电路。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202104/images/tee_intro_03.png)

SE 具备极强的安全等级，但对外提供的接口和功能极其有限。首先 CPU 性能极低，无法处理大量数据，其次 SE 与智能设备通常采用非常缓慢的串口连接。SE 的应用场景有限，主要关注于保护内部密钥，一般用于对安全要求很高的场景。

#### 可信执行环境（Trusted Execution Environment，TEE）

为了给移动设备提供一个安全的运行环境， ARM 从 ARMv6 的架构开始引入了 TrustZone 技术。TrustZone 技术将中央处理器（Central Processing Unit, CPU）的工作状态分为了正常世界状态（Normal World Status, NWS）和安全世界状态（Secure World Status, SWS）。支持 TrustZone 技术的芯片提供了对外围硬件资源的硬件级别的保护和安全隔离。当 CPU 处于正常世界状态时，任何应用都无法访问安全硬件设备，也无法访问属于安全世界状态下的内存、缓存（Cache）以及其他外围安全硬件设备。

有了两个世界的划分，我们就可以将前面的 REE 和 SE 的模式结合起来。一般的操作系统（如Linux、Android、Windows等）以及应用运行在正常世界状态中，提供丰富的系统资源，即为 REE 。而将 SE 实现的安全功能放到安全世界中，但并不需要增加额外的硬件，而是通过可信任的操作系统以及上层的可信应用（Trusted Application, TA）实现。可信任的操作系统以及上层的可信应用（Trusted Application, TA）运行于安全世界状态，也就是本文要探讨的可信执行环境（TEE）。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202104/images/tee_intro_04.png)

需要指出的是，根据 GlobalPlatform 组织的抽象和标准化，TEE 与具体的技术实现无关，并不和 ARM 的TrustZone 绑定。

可信执行环境带来的有点显而易见：

1. 解决了安全元件（SE）的性能问题，因为 TEE 运行在智能设备的 CPU 上，并且在安全世界状态下，独占 CPU， 充分使用CPU的全部性能。
2. 受硬件机制保护， TEE 隔离于 REE， REE 只能通过特定的入口与 TEE 通信。这样处于正常世界状态中的 Linux 即使被 root 也无法访问安全世界状态中的任何资源，包括操作安全设备、访问安全内存数据、获取缓存数据等。这很像一个保险箱，不管保险箱的外在环境是否安全，其内部的物件都有足够的安全性。
3. TEE 中可以同时运行多个 Trusted Application（TA），相对于完全元件（SE）只提供单一功能，通用性比较好。
4. 快速通信机制， TEE 可以访问 REE 的内存，反之则不行。

在真实环境中，可以将用户的敏感数据保存到 TEE 中，并由可信应用（TrustedApplication, TA）使用重要算法和处理逻辑来完成对数据的处理。当需要使用用户的敏感数据做身份验证时，则通过在 REE 侧定义具体的请求编号（IDentity, ID）从 TEE 侧获取验证结果。验证的整个过程中用户的敏感数据始终处于 TEE 中， REE 侧无法查看到任何 TEE 中的数据。对于 REE 而言， TEE 中的 TA 相当于一个黑盒，只会接受有限且提前定义好的合法调用，而至于这些合法调用到底是什么作用，会使用哪些数据，做哪些操作在 REE 侧是无法知晓的。如果在 REE 侧发送的调用请求是非法请求， TEE 内的 TA 是不会有任何的响应或是仅返回错误代码，并不会暴露任何数据给 REE 侧。

TEE 提供介于普通 REE 和 SE 之间的安全性的框架。某些小额的支付、DRM、企业 VPN 等，所需要的安全保护强度并不高，还不需要一个单独的 SE 来保护，但是又不能直接放在 REE 中，因为后者的开放性使其很容易被攻击。所以对于这类应用， TEE 则提供了合适的保护强度，并且平衡了成本和易开发性。

#### TEE 的实现

目前支持TEE的嵌入式硬件技术包含如下：

（1）AMD的PSP（Platform Security Processor）处理器

（2）ARM TrustZone技术（支持TrustZone的所有ARM处理器）

（3）Intel x86-64指令集：SGX Software Guard Extensions

（4）MIPS：虚拟化技术Virtualization

几个TEE的软件实现（提供开源工具或者基于TEE开发的SDK）如下：

（1）Trustonic公司的t-base，是一个商业产品，已经得到 GlobalPlatform 的授权认可。详情可访问：https://www.trustonic.com/products-services/trusted-execution-environment/

（2）OP-TEE，开源实现，来自STMicroelectronics，BSD授权支持下的开源，得到众多手机厂商的支持。开源地址：https://github.com/OP-TEE

（3）Open TEE，开源实现，来自芬兰赫尔辛基大学的一个研究项目，由Intel提供支持，在Apache授权下提供支持。开源地址：https://github.com/Open-TEE/project

（4）SierraTEE，来自Sierraware公司的实现，拥有双重授权模式，一半开源一半商业性质。地址：http://www.sierraware.com/open-source-ARM-TrustZone.html

（5）Trusty TEE，开源实现，由 Google 开发，主要用于 Android 系统，详情可访问：https://source.android.google.cn/security/trusty?hl=zh-cn

#### Android Trusty TEE

Trusty 是一种安全的操作系统 (OS)，可为 Android 提供可信执行环境 (TEE)。Trusty 操作系统与 Android 操作系统在同一处理器上运行，但 Trusty 通过硬件和软件与系统的其余组件隔离开来。Trusty 与 Android 彼此并行运行。Trusty 可以访问设备主处理器和内存的全部功能，但完全隔离。隔离可以保护 Trusty 免受用户安装的恶意应用以及可能在 Android 中发现的潜在漏洞的侵害。

Trusty 与 ARM 和 Intel 处理器兼容。在 ARM 系统中，Trusty 使用 ARM 的 Trustzone™ 虚拟化主处理器，并创建安全的可信执行环境。使用 Intel 虚拟化技术的 Intel x86 平台也提供类似的支持。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202104/images/tee_intro_06.png)

Trusty 包含以下组件：

* 由 Little Kernel 衍生的小型操作系统内核
* Linux 内核驱动程序，用于在安全环境和 Android 之间传输数据
* Android 用户空间库，用于通过内核驱动程序与可信应用（即安全任务/服务）通信

Android 支持各种 TEE 实现，每一种 TEE 操作系统都会通过一种独特的方式部署可信应用。对试图确保应用能够在所有 Android 设备上正常运行的可信应用开发者来说，这种不统一性可能是一个问题。 使用 Trusty 作为标准，可以帮助应用开发者轻松地创建和部署应用，而不用考虑有多个 TEE 系统的不统一性。借助 Trusty TEE，开发者和合作伙伴能够实现透明化、进行协作、检查代码并轻松地进行调试。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)