# 深度学习的JavaScript基础：从浏览器中提取数据

> 最近在读一本《基于浏览器的深度学习》，书比较薄，但是涉及的内容很多，因此在读的过程中不得不再查阅一些资料，以加深理解。我目前从事的本职工作就是浏览器研发，对于前端技术并不陌生。但是从前段时间开发微信小程序**识狗君**的过程来看，对JavaScript还是掌握得太少，特别是对一些前端框架以及一些比较新的JavaScript语法和编程模型，了解的不够。在修改tfjs-core源码时，就体会到这种痛苦。好吧，既然无法避开，那就正面刚吧。

在python语言中，通过文件、摄像头获取数据，并不是什么难事。但对于浏览器来说，出于安全的考虑，并不能直接访问本地文件，至于访问摄像头、麦克风这样的硬件设备，只是从HTML5才开始得到支持。本文就如果获取数据展开讨论，看看在浏览器中提取数据有哪些方法。

### 加载图像数据

图像分类、对象目标检测等是机器学习方面的重要应用，这离不开图像数据。为了将图像作为机器学习算法的输入，必须事先提取图像的像素值。

#### 从图像中提取像素值

熟悉HTML的朋友肯定知道，要在浏览器中显示一幅图像，通常通过HTML img标签：

```html
<img src="images/cat.jpg" id="img_cat"></img>
```

现在我们可以使用全局DOM API document.getElementById('img_cat')访问图像元素。问题是这样获得的HTMLImageElement类型，并没有相关的API来提取像素值。此外还需要注意的是，这里用到的DOM API只在浏览器中可用，在Node.js这样没有DOM的JavaScript运行时中不可用。

庆幸的是，从HTML 5开始，现代浏览器提供了Canvas API，可以用编程的方式将像素绘制到屏幕上，也有相应的API提取像素值。

为了从Canvas元素中提取数据，我们首先需要创建画布上下文，在此上下文中，我们可以将图像内容绘制到画布上，然后访问并返回画布像素数据。

```javascript
function loadRgbDataFromImage(img) {
    // 创建canvas元素
    const canvas = document.createElement('canvas');

    // 将canvas尺寸设置为图像大小
    canvas.width = img.width;
    canvas.height = img.height;

    // 创建2D渲染上下文
    const ctx = canvas.getContext('2d');

    // 将图像渲染到canvas上下文的坐上角坐标(0, 0)
    ctx.drawImage(img, 0, 0, img.width, img.height);

    // 提取RGB数据
    const imgData = ctx.getImageData(0, 0, canvas.width, canvas.height);

    // 将数据转换为int32数组
    return new Int32Array(imgData.data);
}
```

在上面的代码中，ctx.getImagedata函数返回ImageData类型的数据，这是一个包含width, height和data属性的对象。data属性值的存储格式为类型化数组Uint8ClampedArray。

需要注意的是，图像是异步加载的，因此我们只有在浏览器完全加载了图像才能提取像素值，这可以在onload事件中完成。

```javascript
const img = document.getElementById('img_cat');
img.onload = () => {
    const data = loadRgbdataFromImage(img);
    ...
}
```

#### 加载远程资源

图像数据不仅可以是本服务器上的图片，还可以是其它远程服务器上的资源，以URL的形式提供。

```html
<img src="https://<other_server>/cat.jpg" crossOrigin="anonymous" id="img_cat"></img>
```

在加载其它远程服务器上的资源时，需要了解跨域资源共享(Cross-Origin Resource Sharing, CORS)的概念。出于安全的考虑，浏览器会自动阻止对当前连接之外的不同域、协议或端口的cross-site请求。而CORS策略允许浏览器通过设置附加的HTTP头来执行对资源的跨域HTTP请求。比如上面代码中，使用crossOrigin属性，并将其设置为anonymouse，显式地允许该元素加载cross-site资源。

我们也可以通过JavaScript，以编程方式完成上述代码的功能。需要注意加载图像资源是异步行为，我们返回Promise，而不是已经加载的资源。

```javascript
function loadImage(url) {
    return new Promise((resolve, reject) => {
        const img = new Image();
        img.crossOrigin = "anonymous";
        img.src = url;
        img.onload = () => resolv(img);
        img.onerror = reject;
    });
}
```

### 加载二进制块

经过训练的模型，模型权重、参数等数据，通常以二进制块的形式保存，所以在浏览器中使用机器学习模型，一定会面临二进制块的加载问题。好在JavaScript是一种非常通用的语言，内置了对类型化数组和数组缓冲区的支持，这使得在浏览器中使用二进制数据非常方便。

相比文本表示格式（如csv或JSON），二进制数据文件更小，加载速度更快（不需要解析），这使得在JavaScript中加载较大规模的模型权重成为可能。

假如我们有一个二进制块rand.bin，可以创建一个函数来获取二进制块作为数组缓冲区。

```javascript
async function loadBinaryDataFromUrl(url) {
    const req = new Request(url);
    const res = await fetch(req);
    if (!res.ok) {
        throw Error(res.statusText);
    }

    return res.arrayBuffer();
}
```

### 访问外设数据

随着移动终端的普及，以前很多需要电脑上完成的工作，都可以在移动终端上完成，而移动终端丰富且使用方便的外设（相机、麦克风、重力感应器等）提供了多种玩法。早期的浏览器访问设备的能力几乎没有，但从HTML5开始，增加了硬件访问能力，提供了Device API，借助于Device API，通过JS和HTML页面访问终端的应该成为可能。

#### 从网络摄像头获取图像

浏览器的MediaDevices API允许用户访问视频和音频设备，例如相机、麦克风和扬声器。它是更通用的WebRTC API的一部分。

我们可以使用MediaDevices::getUserMedia()函数启动视频流，该函数将返回包含MediaStream对象的promise。

```javascript
navigator.mediaDevices.getUserMedia({ video: true, audio: false})
    .then((stream) => { ... });
```

为了从MediaStream中提取数据，需要将流附加到HTML video元素。我们可以通过代码创建一个这样的元素，并将流提供给播放器。

```javascript
const player = document.createElement('video');

navigator.mediaDevices.getUserMedia({ video: true, audio: false})
    .then((stream) => { player.srcObject = stream; });
```

最后，我们可以从video元素中提取内容，将图像渲染到画布，然后提取画布中的像素。

```javascript
const data = loadRgbDataFromImage(player, width, height);
```

是的，这个地方没有看错，player也可以传递给loadRgbDataFromImage。查看drawImage函数的原型，对于img参数的说明为：

> img： Specifies the image, canvas, or video element to use

也就是说这里传递image、canvas或video都是可以的。

还有一种更高端用法，就是从WebGL中的video元素访问，而无须使用画布，有兴趣的可以查阅相关资料。

#### 用麦克风录音

访问麦克风同样通过MediaDevices API，处理数据则通过WebAudio API，这是一个非常灵活的基于图的音频处理API。

首先使用MediaDevices::getUserMedia()函数检索音频流。

```javascript
navigator.mediaDevices.getUserMedia( { audio: true, video: false })
    .then(onStream);
```
接下来，设置一个非常简单的音频图，包括输入、简单处理器和默认输出。我们还需定义处理器的属性，包括输入和输出通道的数量以及音频块的缓冲区大小。

```javascript
const audioContext = new AudioContext();

const bufferSize = 4096;
const numInputChannels = 1;
const numOutputChannels = 3;

function onStream(stream) {
    const source = audioContex.createMediaStreamSource(stream);
    const processor = audioContext.createScriptProcessor(
        bufferSize, numInputChannels, numOutputChannels);
    source.connect(processor);
    processor.connect(audioContext.destination);
    processor.onaudioprocess = onProcess;
}
```

然后，在onProcess中执行处理。

```javascript
function onProcess(e) {
    const data = e.inputBuffer.getChannelData(0);
    ...
}
```

AudioBuffer.getChannelData()函数返回Float32Array(bufferSize)的数据，还可以通过duration、sampleRate和numberOfChannels属性获得其它音频信息。

现在我们可以使用这些数据进一步处理或直接送给模型。

### 小结

本文探讨如何在浏览器中获取数据的几种方法，包括图像数据、音频数据，现代浏览器具备原来越丰富的设备访问能力，配合移动终端方便易用的外设，必将产生越来越多的有趣的机器学习应用。

![images](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)