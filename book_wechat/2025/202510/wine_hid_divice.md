# 信创系统支持 USB HID 设备的方案

随着信创产业的快速发展，党政机关的信创化已基本完成，目前正在全力推动行业系统的信创化。在这一过程中，面临的最大挑战之一是如何支持各种外设，例如税控机、读卡器、打印机等。这些外设缺乏统一标准，适配起来较为复杂。经过不断努力，目前 UOS 系统对各种打印机已有较好的支持，而其他外设仍在逐步适配中。

HID（Human Interface Device，人机接口设备）是一种标准化的设备类别，用于统一人机交互类设备（如鼠标、键盘、游戏手柄、触控板、扫描枪、笔输入设备等）的通信方式。相较于传统的串口通信设备，HID 设备具有传输速率更高、标准化支持更强、可热插拔以及免驱动的优势。随着越来越多的外设逐步转向 HID 标准，这里探讨一下 HID 设备在 UOS 系统中的支持方案。

尽管 HID 是标准化设备类别，但这并不意味着插上设备就能直接使用。不同厂商的 HID 设备仍需针对性适配。

Windows / Linux / macOS 都内置 HID 驱动，但这些驱动主要支持几种“通用用途”：

* 键盘（Boot Keyboard）

* 鼠标（Boot Mouse）

* 游戏手柄（Generic Gamepad）

* 消费控制（音量、播放控制）

而更多的工业设备、医疗设备、控制面板、条码枪、测量仪器等，都属于 自定义 HID（Custom HID）。

这类设备虽然仍属于 HID 类，但：

* 操作系统不会自动识别它的用途；

* 系统不会为其创建特定输入事件；

* 应用层必须通过 hidraw / hidapi / IOHIDManager 自行解析数据。

所以开发时仍然需要针对设备写“用户态解析逻辑”。

当然 HID 设备比较好的一点是，通常不需要专门的驱动，这样在信创系统上可以原生支持。前提是应用程序需要进行适配。

## 方案一：应用程序适配 HID 设备

在 Linux 下，要支持 HID 设备，推荐使用如下几个库：

### 1. hidapi

简介：跨平台的 HID 接口库，支持 Linux、Windows、macOS。

特点：

* 封装了 /dev/hidraw* 和 libusb 接口；

* 简单易用，只需 VID/PID 即可打开设备；

* 支持读写 Input/Output/Feature 报告。

安装：

```
sudo apt install libhidapi-dev
```

这是一个跨平台库，我以前在开发语音输入时就使用过，结合 QT 的跨平台性，在开发不同平台版本的应用时，可以复用一套代码。当然，这个库由于封装了 /dev/hidraw 和 libusb 接口，有时会遇到一些问题，这个时候可能需要直接读写 /dev/hidraw。

### 2. libusb

简介：通用 USB 访问库，HID 是 USB 设备的一类。

特点：

* 可以访问 HID 以及非 HID USB 设备；

* 更底层，需要自己解析 HID 报告；

* 更灵活，可以实现特定控制请求。

安装：

```
sudo apt install libusb-1.0-0-dev
```

使用场景：

当 hidapi 无法满足特殊控制命令；或者需要对 HID 进行固件升级、发送特定 Feature Report。

### 3. hidraw 接口

简介：Linux 内核直接提供的字符设备接口 /dev/hidraw*。

特点：

* 最底层，直接访问 HID 报告字节；

* 不经过通用 HID 层解析；

* 可结合 ioctl() 获取 HID 报告描述符、发送 Feature Report 等。

虽然直接控制灵活度比较高，但使用起来比较麻烦，所以除非在某些特殊场景，一般不推荐直接使用。

### 权限问题

借助于 hidapi 这样的库，应用中支持 HID 设备并不难，不过需要注意的是，如果应用程序需要访问 HID 设备，需要获取 root 权限。

通常应用程序由用户启动，通常不会有 root 权限，这个时候就需要解决权限问题。虽然可以使用 sudo 命令启动应用程序，或者在启动时提权，但这对普通用户来说，不是很友好。提权时，需要用户输入密码，这会让用户感到不安。

对于这种情况，有两种方案：

* 开发一个 service 服务，在后台运行，服务以 root 运行，所以有权限访问 HID 设备。应用程序和服务器程序进行通信，应用程序访问 HID 设备均通过服务程序。服务程序 crash 后有重启机制，避免由于服务程序 crash 导致应用程序无法和 HID 设备通信；
* 配置 udev 规则，让特定的设备对其它组有访问权限。这对于 Android 应用开发者应该比较熟悉，为了让普通用户 adb 能访问到手机，可以通过配置 uodev 规则：
比如新建文件 /etc/udev/rules.d/99-usb-permissions.rules
```
SUBSYSTEM=="usb", ATTR{idVendor}=="141c", ATTR{idProduct}=="0a12", MODE="0666"
```

由于 udev 规则是系统级配置，有些操作系统可能管控比较严格，应用程序上架会被拒绝，这点需要注意。

## 方案二：通过 Wine 运行

某些遗留系统，可能没有人力或者没有意愿进行系统适配，这个时候可以考虑通过 Wine 运行之前的 Windows 应用程序。Wine 是一个让 Windows 应用程序在 Linux 上运行的兼容层，也支持 HID 设备。

Wine通过一个多层架构将Windows的HID设备API调用转换到Linux的HID设备。整个流程涉及以下几个关键组件：

### 1. Linux端设备检测（winebus.sys）

winebus.sys 是Wine的平台原生总线驱动程序，负责桥接Windows HID API到Linux设备。

在Linux上，winebus通过 udev 和 hidraw 接口检测USB HID设备：
* 使用libudev监控 /dev/hidraw*（原始HID设备）和 /dev/input/event*（Linux输入事件设备）
* 当检测到设备时，udev_add_device() 函数会调用 hidraw_device_create() 创建hidraw设备实例 
* 初始设备枚举通过扫描 /dev/hidraw* 和 /dev/input/event* 完成

### 2. 设备描述符获取

创建设备时，通过Linux的ioctl接口获取HID报告描述符：

* 使用 HIDIOCGRDESCSIZE ioctl获取描述符大小
* 使用 HIDIOCGRDESC ioctl获取完整的报告描述符

### 3. 设备创建事件处理

当Linux设备被检测到后，winebus主线程处理设备创建事件：

* bus_main_thread() 接收 BUS_EVENT_TYPE_DEVICE_CREATED 事件
* 调用 bus_create_hid_device() 创建Windows设备对象

bus_create_hid_device() 函数创建Windows风格的设备对象，并将其添加到设备列表：

### 4. Windows HID类驱动（hidclass.sys）

hidclass.sys 是Wine实现的Windows HID类驱动，位于winebus和应用程序之间：

* winebus通过 HidRegisterMinidriver() 注册为HID微驱动程序
* hidclass.sys管理设备堆栈，创建FDO（功能设备对象）和PDO（物理设备对象）
* 为每个HID集合创建PDO，并生成设备ID、硬件ID等

### 5. 数据传输流程

读取输入报告：

* Linux设备的数据通过 read() 系统调用读取
* 报告被加入事件队列，主线程通过 process_hid_report() 处理
* Windows应用程序通过 IOCTL_HID_READ_REPORT 请求数据

写入输出报告和功能报告：

* 应用程序的写入请求通过 IOCTL_HID_WRITE_REPORT 或 IOCTL_HID_SET_OUTPUT_REPORT 发送
* 最终通过Linux的 write() 系统调用发送到hidraw设备

功能报告获取：

* 使用 HIDIOCGFEATURE 和 HIDIOCSFEATURE ioctl在Linux层面获取/设置功能报告

总体架构图如下：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202510/images/wine_hid_device_01.png)

在 wine 上运行，同样需要注意权限问题，可以参考方案一。

## 小结

信创系统在 USB HID 设备支持上已具备较为成熟的方案：通过应用层原生适配效果最佳，同时也能通过 Wine 兼容遗留 Windows 应用。

展望未来，随着行业系统信创化的推进，更多专业外设（如税控机、医疗设备、工业控制仪器等）将逐步标准化为 HID 或统一接口模式，UOS 等信创系统在外设支持上的覆盖面将进一步扩大。应用开发者将能更便捷地实现跨平台适配，信创生态下的行业软件兼容性和可扩展性将显著提升，推动政务、金融、医疗、工业等领域的信息化水平持续升级。

