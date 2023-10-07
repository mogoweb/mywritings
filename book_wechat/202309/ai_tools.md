# 在创业公司，我靠它续命 ...

不知不觉就在新公司工作了一周，没有想象中那么难受。创业公司里没有复杂的人际关系，也没有无聊的会议，也没有复杂的流程。每天上班第一件事就是开个小会，可能是站着开，也可能是连麦开。大家简单过一下前一天的进度，以及今天的安排，然后就是各忙各的。

创业公司因为人比较少，不像大公司分工那么细，所以对个人综合能力要求比较高一些，因为碰到一些问题，不能像大公司那样互相推诿。进公司一周的时间，还没有来得及梳理代码，就忙着解决各种 BUG。

我已经很久都没有做 Windows 下的开发，之前也没有用过 QT，要快速修复或实现一些新功能，确实有点难。好在我使用了一款新工具: Cursor，一款针对程序员的 AI 辅助工具。Cursor 功能很强大，但我目前只用了对话功能。

Cursor 运行后，主界面如下：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202309/images/cursor_using_01.png)

右上角往输入框输入文字，就可以问问题。

效果怎么样？试试看。

我现在有一个需求，语音转文字，通常都是通过麦克风或录音笔之类的设备，但我希望模拟硬件输入，通过读取文件的方式。看了一下代码，发现 QT 中有一个 QAudioInput 类用于从系统声音设备读取数据，于是我提出了第一个问题：

> 写一段代码模拟QAudioInput的实现，从一个音频源文件读取

Cursor 很快给了一个答案：

> 以下是一个简单的Python代码示例，使用pyaudio库模拟QAudioInput的实现，从一个音频源文件读取：

```
import pyaudio
import wave

class AudioInput:
    def __init__(self, file_path):
        self.file_path = file_path
        self.wf = wave.open(self.file_path, 'rb')
        self.p = pyaudio.PyAudio()

    def start(self):
        stream = self.p.open(format=self.p.get_format_from_width(self.wf.getsampwidth()),
                             channels=self.wf.getnchannels(),
                             rate=self.wf.getframerate(),
                             output=True)

        data = self.wf.readframes(1024)

        while data != '':
            stream.write(data)
            data = self.wf.readframes(1024)

        stream.stop_stream()
        stream.close()

        self.p.terminate()

# 使用
audio_input = AudioInput('your_audio_file.wav')
audio_input.start()
```
> 这段代码首先打开一个音频文件，然后创建一个pyaudio.PyAudio对象。然后，它使用音频文件的参数（如采样宽度、通道数和帧率）来打开一个音频流。最后，它读取音频文件的帧并写入音频流，直到音频文件读取完毕。
> 
> 请注意，这只是一个基本的实现，可能需要根据你的具体需求进行修改。

忘记限定语言了，接着提问：

> 请使用C++，完全模拟QAudioInput的各接口

这回结果还比较靠谱：

> 在Qt中，你可以使用QBuffer类来模拟音频设备。QBuffer类提供了一个接口，使得你可以从内存中的字节数组读取和写入数据，就像它是一个设备一样。以下是一个使用QBuffer和QAudioOutput的例子：

```
#include <QAudioOutput>
#include <QBuffer>
#include <QFile>

class MyAudioInput {
private:
    QAudioOutput* audio; // QAudioOutput对象
    QBuffer audioBuffer; // 音频缓冲区

public:
    MyAudioInput() {
        // 初始化QAudioFormat对象
        QAudioFormat format;
        format.setSampleRate(8000);
        format.setChannelCount(1);
        format.setSampleSize(8);
        format.setCodec("audio/pcm");
        format.setByteOrder(QAudioFormat::LittleEndian);
        format.setSampleType(QAudioFormat::UnSignedInt);

        QAudioDeviceInfo info = QAudioDeviceInfo::defaultOutputDevice();
        if (!info.isFormatSupported(format)) {
            qWarning() << "Default format not supported, trying to use the nearest.";
            format = info.nearestFormat(format);
        }

        audio = new QAudioOutput(format, this);
    }

    void start() {
        QFile sourceFile;   // 选择你的.raw文件
        sourceFile.setFileName("/tmp/test.raw");
        sourceFile.open(QIODevice::ReadOnly);

        audioBuffer.setData(sourceFile.readAll()); // 读取文件内容到缓冲区
        audioBuffer.open(QIODevice::ReadOnly); // 打开缓冲区

        audio->start(&audioBuffer); // 开始播放
    }

    void stop() {
        audio->stop();
        audioBuffer.close();
        delete audio;
    }
};
```
> 在这个例子中，我们首先从一个.raw文件中读取数据，然后将这些数据写入QBuffer对象。然后，我们使用QAudioOutput对象来播放这些数据，就像它们是从音频设备读取的一样。

我们还可以进一步细化自己的需求，让 Cursor 给出的代码更加完善，由于篇幅的原因，就不过多展示了。

除了帮助写代码，还可以像 Cursor 问一些技术上的问题，比如：

> Q: 可以传一个模拟设备给QAudioInput吗？
>
> A: 不幸的是，QAudioInput类的设计是直接与音频硬件设备进行交互的，它没有提供一个接口来接受一个模拟的音频设备。你不能直接将一个模拟设备（如一个文件或一个缓冲区）传递给QAudioInput。
>
> 然而，你可以创建一个自定义的类来模拟QAudioInput的行为，如我之前所示的那样。你可以从一个文件或一个缓冲区读取数据，然后提供类似于QAudioInput的API，如start(), stop(), read(), stateChanged信号等。这样，你的应用程序可以像使用QAudioInput一样使用你的模拟类，而不需要知道数据实际上是从一个模拟设备而不是一个真实的音频设备读取的。
>
> Q: 我希望上层调用者不用关心是实际设备还是模拟设备，可以从QAudioInput继承吗？
>
> A: 在C++中，你可以从QAudioInput类继承来创建你自己的类。然而，QAudioInput类并没有设计为一个可以被继承和重写的类。它的大部分方法都不是虚函数，这意味着你不能在子类中重写它们。此外，QAudioInput类的构造函数需要一个QAudioFormat对象和一个QAudioDevice对象，这使得在子类中模拟音频设备变得困难。
>
> 因此，虽然从技术上讲，你可以从QAudioInput类继承，但这并不是一个好的选择。相反，你应该创建一个新的类，提供与QAudioInput类相同的API，但在内部使用你自己的模拟音频设备。这样，你的上层调用者可以像使用QAudioInput一样使用你的类，而不需要知道它实际上是一个模拟的音频设备。

Cursor 后台连接的是 OpenAI 的 ChatGPT 4.0，我们可以使用非常自然的语言将问题描述出来，返回的也是英文解答，相当优化。更神奇的是，写出的代码还有注释，代码后面还有对这段代码的说明，真的是贴心到家了。

在工作中，借助一些 AI 工具，无疑能大大提供我们工作的效率。但要说 AI 能取代程序员，还为时尚早。不信你提一个需求，写一个微信，看看 ChatGPT 能不能帮你搞定？

你在工作中使用到 AI 了吗？
