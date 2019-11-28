# 双向业务中的视频播放

这里的双向业务指的是在浏览器中运行的业务，也就是通过编写网页实现的业务逻辑。本文将介绍双向业务中的视频播放实现方式，不涉及技术细节，供大家做一个基本的了解，在下一篇文章中再针对具体的点展开技术细节。

在OCN的双向业务中，视频播放的实现有多种，有的是HTML规范，有的是NGB-H规范，还有的是OCN自定义的规范。

## HTML Video

这是HTML 5标准规范中定义的方法，通过HTML video标签实现：

```html
<!DOCTYPE HTML>
<html>
<body>

<video src="/i/movie.ogg" width="400" height="300" controls="controls">
your browser does not support the video tag
</video>

</body>
</html>
```

显示效果如下：

![HTML video视频播放](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_work/images/media_play_in_browser_01.png)

OCN双向业务中使用这种视频播放方式的不多，但其实这是最标准的一种方式，主流的Android浏览器、IOS浏览器、桌面浏览器都支持。关于HTML video视频播放，需要理解以下几点：

* 最终是走到系统框架层的MediaPlayer进行播放，对于Android系统来说，就是Android API中的MediaPlayer对象。这种方式所支持的视频格式、协议，取决于MediaPlayer。一般来说会支持HTTP视频流，通常支持的视频格式有mp4。
* 上图中的播放有关UI由video属性控制，由浏览器绘制。但有的网页中并没有给controls属性，也可以自行设计播放UI，比如优酷之类的视频网站，长得和上面的不太一样，就是自行设计的。
* 可以只用指定宽高，视频窗口的位置由浏览器排版决定。因为HTML video也是一个HTML元素，所以也可以通过CSS进行精确定位，比如：
  
```html
  <video id="v1" src="" style="position:absolute; z-index:99; left:750px; top:300px; width:450px; height:300px;">
  </video>
```

* 由于目前我们的TVOS/COS系统框架中的MediaPlayer不支持DVB播放，所以与直播有关的业务都没走这个路径。

## NGB-H MediaPlayer

这是NGB-H规范中定义的方法，通过扩展的MediaPlayer JavaScript对象进行播放控制。在NGB-H规范中，定义了一系列针对视频播放的JS接口，其中几个主要的接口有：

* setMediaSource
* setVideoDisplayMode
* setVideoDisplayArea
* play
* stop

下面是一段示例代码：

```javascript
var mp = new MediaPlayer();
var id = mp.getPlayerInstanceID();
mp.bindPlayerInstance(id);
mp.setMediaSource("http://10.27.104.104/html/webpagetestPlus/res/video/1.mp4");
mp.setVideoDisplayMode(0);
var rect = new Rectangle();
rect.left = 80;
rect.top = 50;
rect.width = 200;
rect.height = 200;
mp.setVideoDisplayArea(rect); //设置小窗口播放
mp.refresh();
mp.setVideoDisplayMode(1); //设置全屏播放
mp.refresh();
mp.stop(); //停止播放
mp.unbindPlayerInstance(id);
```

关于NGB-H MediaPlayer播放，需要理解以下几点：

* 视频窗口的位置通过js指定，包括左上角坐标及宽、高。默认只有一个视频窗口，并没有界面UI，如果需要界面UI，需要自行设计网页。比如我们设计的点播页面，通过按键进行控制。
* 媒体播放走的是TvosMediaPlayer，TvosMediaPlayer只是一个封装层，最终走向哪个Player，TvosMediaPlayer可以根据协议或其它方式控制，比如DVB播放，最后走的是DvbPlayer。具体能够支持哪种协议、哪种格式，依然由最底层的Player决定。
* 目前TvosMediaPlayer封装了http、RTSP、dvb、netdvb的播放支持，对于媒体播放支持最全面。但这个NGB-H MediaPlayer是我们自行扩展的对象，所以针对这种播放方式设计的网页，用标准的Android浏览器、桌面浏览器是无法正常工作的。

![NGB-H视频播放界面](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_work/images/media_play_in_browser_02.png)

## VOD

这是OCN定义的HTML扩展，通过vod以及vodpara标签实现。浏览器本身并不会处理播放，仅仅解析vod和vodpara标签及属性，然后启动另一个APP：VODPlayer，将参数传给它。后面的处理和浏览器无关。

```html
<html>
<head>
</head>

<body>

<vod index="305" id="305" ServiceName="FOD-TV" ServiceType="SVOD" Preview="0" backstep="1">

    <vodpara name="passthru_ip" value="10.27.65.50" id="passthru_ip"/>
    <vodpara name="mod_app_ip" value="10.27.65.50" id="mod_app_ip"/>
    <vodpara name="srm_ip" value="10.27.65.8" id="srm_ip"/>
    <vodpara name="poster_server_ip" value="10.27.65.50" id="poster_server_ip"/>
    <vodpara name="lsc_comm_proxy_ip" value="10.27.65.50" id="lsc_comm_proxy_ip"/>
    <vodpara name="session_gateway_ip" value="10.27.65.50" id="session_gateway_ip"/>

    <vodpara name="freq1" value="562000" id="freq1"/>
    <vodpara name="sym_rate1" value="6900" id="sym_rate1"/>
    <vodpara name="qam_mode1" value="64" id="qam_mode1"/>
    <vodpara name="freq2" value="834000" id="freq2"/>
    <vodpara name="sym_rate2" value="6900" id="sym_rate2"/>
    <vodpara name="qam_mode2" value="64" id="qam_mode2"/>

    <vodpara name="ApplicationType" value="vod" id="ApplicationType"/>
    <vodpara name="run_time" value="02:25:07" id="run_time"/>
    <vodpara name="content_name" value="T015090600000502-movie.mpg" id="content_name"/>
    <vodpara name="ServiceCode" value="00003" id="ServiceCode"/>

    <vodpara name="category" value="Music" id="category"/>
    <vodpara name="provider" value="SiTV" id="provider"/>

    <vodpara name="protocol_type" value="rtsp" id="protocol_type"/>

    <vodpara name="rtsp" value="10.27.63.108:6050" id="rtsp"/>
    <vodpara name="rtsp_vendor" value="OCN.RTSP" id="rtsp_vendor"/>
    <vodpara name="purchaseType" value="1" id="purchaseType"/>

    <vodpara name="streamTransMode" value="hfc_stream" id="streamTransMode"/>

</vod>

</body>
```

关于VOD播放，需要理解以下几点：

* 浏览器只是起一个中间桥梁的作用，如何播放、视频窗口位置等等，由VODPlayer决定。
* 目前支持两种流，一种是ip流，一种是HFC流
* 需要联彤浏览器以及VODPlayer协同工作，才能正常运转。

## tv标签

这是OCN定义的HTML扩展，用于视频直播，主要的用途就是直播小窗口，无UI、无播放控制，常见的写法如下：

```html
<!-- //下面两个hr之间为测试区域，可自行设置显示风格 -->
<div id=result>

    <tv id="tv1" coords="305,197,679,440" networkid="2184" casystemid="0" ecmpid="0" videopid="0x0200"  
    videotype="0x2" audiopid="0x028A" audiotype="0x04" pcrpid="0x1FFE" freq="650000000" sym="6875000" 
    mod="64" play="all" />

    <tv id="tv2" coords="305,197,679,440" networkid="2184" casystemid="0" ecmpid="0" videopid="0x0201"  
    videotype="0x2" audiopid="0x0294" audiotype="0x04" pcrpid="0x1FFE" freq="650000000" sym="6875000" 
    mod="64" play="all" />

    <tv id="tv3" coords="305,197,679,440" networkid="2184" casystemid="0" ecmpid="0" videopid="0x0202"  
    videotype="0x2" audiopid="0x029E" audiotype="0x04" pcrpid="0x1FFE" freq="650000000" sym="6875000" 
    mod="64" play="all" />
    <!-- 模拟直播环境不可以播放的tv 标签  -->

</div>
```

关于tv标签直播，需要理解以下几点：

* 播放底层走的是TvosMediaPlayer
* 视频窗口的位置由coords属性值决定，其它参数为直播有关的诸如频点、频率之类的参数。
* 未定义播放控制接口，所以动作只有两种，打开页面进行直播，关闭页面结束直播。

## 小结

上述4种视频播放中，HTML Video是得到广泛支持的标准，设计的网页可以在桌面浏览器、标准手机浏览器上验证，但不支持直播。NGB-H MediaPlayer并非业界通用的标准，而是TVOS工作组提出的一组扩展接口，适应面广，点播、直播都可以（所支持的协议和格式，取决于TvosMediaPlayer），HTML Video能够做到的，NGB-H MediaPlayer都能做到。但使用NGB-H MediaPlayer设计的网页，必须使用我们的盒子访问，调试起来不太方便。至于VOD和tv，完全是OCN整出来的一个扩展，只用于某些特定的场景，适应面比较窄。