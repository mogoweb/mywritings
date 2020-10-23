# Android Studio新特性：使用TFLite模型更简单

Android Studio仍然在疯狂更新中，隔一段时间打开Android Studio，就会提示有新版本，对此我已经见怪不怪。一般而言，我会顺手点击一下升级。今天我又点击了升级，粗略看了一下新版本4.1的特性说明，其中有一项是：**使用TensorFlow Lite模型**。出于对机器学习的兴趣，于是就研究了一番这个新特性。

TensorFlow Lite是最受欢迎的编写移动端机器学习模型的开发库，在我之前的文章中也写过如何在Android程序中使用TFLite模型。有了TFLite模型后，我们需要模型开发者提供模型的输入、输出等信息，然后编写封装类，对图片进行预处理（比如裁剪、规范化等等），这对于开发者而言，枯燥而且容易出错。而在Android Studio 4.1中，这个开发过程得到了简化，导入模型后，Android Studio会生成辅助类，我们只需编写极少的代码即可运行模型，而且还提升了类型安全性。

我们先说说如何导入TFLite模型并使用，然后再来解释是如何做到的。

#### 导入模型文件

按照如下步骤即可导入TFLite模型：

1. 新建或打开现有Android项目工程。
2. 通过菜单项 **File > New > Other > TensorFlow Lite Model** 打开TFLite模型导入对话框。

![Android Studio菜单](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202010/images/android_studio_tflite_01.png)

3. 选择后缀名为.tflite的模型文件。模型文件可以从网上下载或自行训练。

![导入模型](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202010/images/android_studio_tflite_02.png)

4. 点击对话框上的 **Finish**。

导入的模型文件位于工程的 **ml/** 文件夹：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202010/images/android_studio_tflite_03.png)

可以看到，除了多了 **ml/** 文件夹下的模型文件外，似乎代码并没有什么变化。如果仅仅是做这点工作的话，那肯定谈不上什么了不得的新特性，让我们继续往下看。

#### 查看模型元数据（metadata）和用法

在Android Studio中双击 **ml/** 文件夹下的模型文件，可以看到模型的详细信息，比如我所使用的 **mobilenet_v1_0.25_160_quantized_1_metadata_1.tflite** 模型，信息如下：

![模型信息](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202010/images/android_studio_tflite_04.png)

主要包括如下三种信息：

* 模型：包括模型名称、描述、版本、作者等等。
* 张量：输入和输出张量。在以往的开发中，这个非常重要，比如图片需要预先处理成合适的尺寸，才能进行推理。
* 示例代码：说明在应用中如何调用模型，包括Java和Kotlin代码。

可以看到，要调用模型，代码相当简单，不需要进行复杂的图片预处理，不需要构建张量，也不需要在张量：

```java
try {
    MobilenetV1025160Quantized1Metadata1 model = MobilenetV1025160Quantized1Metadata1.newInstance(context);

    // Creates inputs for reference.
    TensorImage image = TensorImage.fromBitmap(bitmap);

    // Runs model inference and gets result.
    MobilenetV1025160Quantized1Metadata1.Outputs outputs = model.process(image);
    List<Category> probability = outputs.getProbabilityAsCategoryList();

    // Releases model resources if no longer used.
    model.close();
} catch (IOException e) {
    // TODO Handle the exception
}
```

代码中的 **MobilenetV1025160Quantized1Metadata1** 实现在哪儿呢？这个是自动生成的，点击Android Stuido的 **Build > Make Project** ，在generated目录下就可以看到生成的代码：

```java
package com.mogoweb.tensorflow.example.modelmetadata.ml;

import android.content.Context;
import androidx.annotation.NonNull;
import java.io.IOException;
import java.lang.Integer;
import java.lang.Object;
import java.lang.String;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import org.tensorflow.lite.DataType;
import org.tensorflow.lite.support.common.FileUtil;
import org.tensorflow.lite.support.common.TensorProcessor;
import org.tensorflow.lite.support.common.ops.CastOp;
import org.tensorflow.lite.support.common.ops.DequantizeOp;
import org.tensorflow.lite.support.common.ops.NormalizeOp;
import org.tensorflow.lite.support.common.ops.QuantizeOp;
import org.tensorflow.lite.support.image.ImageProcessor;
import org.tensorflow.lite.support.image.TensorImage;
import org.tensorflow.lite.support.image.ops.ResizeOp;
import org.tensorflow.lite.support.image.ops.ResizeOp.ResizeMethod;
import org.tensorflow.lite.support.label.Category;
import org.tensorflow.lite.support.label.TensorLabel;
import org.tensorflow.lite.support.metadata.MetadataExtractor;
import org.tensorflow.lite.support.model.Model;
import org.tensorflow.lite.support.tensorbuffer.TensorBuffer;

/**
 * Identify the most prominent object in the image from a set of 1,001 categories such as trees, animals, food, vehicles, person etc. */
public final class MobilenetV1025160Quantized1Metadata1 {
  @NonNull
  private final ImageProcessor imageProcessor;

  @NonNull
  private final List<String> labels;

  @NonNull
  private final TensorProcessor probabilityPostProcessor;

  @NonNull
  private final Model model;

  private MobilenetV1025160Quantized1Metadata1(@NonNull Context context,
      @NonNull Model.Options options) throws IOException {
    model = Model.createModel(context, "mobilenet_v1_0.25_160_quantized_1_metadata_1.tflite", options);
    MetadataExtractor extractor = new MetadataExtractor(model.getData());
    ImageProcessor.Builder imageProcessorBuilder = new ImageProcessor.Builder()
      .add(new ResizeOp(160, 160, ResizeMethod.NEAREST_NEIGHBOR))
      .add(new NormalizeOp(new float[] {127.5f}, new float[] {127.5f}))
      .add(new QuantizeOp(128f, 0.0078125f))
      .add(new CastOp(DataType.UINT8));
    imageProcessor = imageProcessorBuilder.build();
    TensorProcessor.Builder probabilityPostProcessorBuilder = new TensorProcessor.Builder()
      .add(new DequantizeOp((float)0, (float)0.00390625))
      .add(new NormalizeOp(new float[] {0.0f}, new float[] {1.0f}));
    probabilityPostProcessor = probabilityPostProcessorBuilder.build();
    labels = FileUtil.loadLabels(extractor.getAssociatedFile("labels.txt"));
  }

  @NonNull
  public static MobilenetV1025160Quantized1Metadata1 newInstance(@NonNull Context context) throws
      IOException {
    return new MobilenetV1025160Quantized1Metadata1(context, (new Model.Options.Builder()).build());
  }

  @NonNull
  public static MobilenetV1025160Quantized1Metadata1 newInstance(@NonNull Context context,
      @NonNull Model.Options options) throws IOException {
    return new MobilenetV1025160Quantized1Metadata1(context, options);
  }

  @NonNull
  public Outputs process(@NonNull TensorImage image) {
    TensorImage processedimage = imageProcessor.process(image);
    Outputs outputs = new Outputs(model);
    model.run(new Object[] {processedimage.getBuffer()}, outputs.getBuffer());
    return outputs;
  }

  public void close() {
    model.close();
  }

  @NonNull
  public Outputs process(@NonNull TensorBuffer image) {
    TensorBuffer processedimage = image;
    Outputs outputs = new Outputs(model);
    model.run(new Object[] {processedimage.getBuffer()}, outputs.getBuffer());
    return outputs;
  }

  public class Outputs {
    private TensorBuffer probability;

    private Outputs(Model model) {
      this.probability = TensorBuffer.createFixedSize(model.getOutputTensorShape(0), DataType.UINT8);
    }

    @NonNull
    public List<Category> getProbabilityAsCategoryList() {
      return new TensorLabel(labels, probabilityPostProcessor.process(probability)).getCategoryList();
    }

    @NonNull
    public TensorBuffer getProbabilityAsTensorBuffer() {
      return probabilityPostProcessor.process(probability);
    }

    @NonNull
    private Map<Integer, Object> getBuffer() {
      Map<Integer, Object> outputs = new HashMap<>();
      outputs.put(0, probability.getBuffer());
      return outputs;
    }
  }
}
```

从代码可以看到，ImageProcess类完成了图片的预处理工作，我们甚至不用关心图片预处理的细节。

#### 不足之处

当然，作为新开发的特性，并不是所有的tflite模型都能通过这种方式导入，目前这种使用方法还存在如下几种限制：

1. tflite模型必须包含元数据。如果你希望得到包含元数据的模型，一种方法是前往**TensorFlow Hub**下载模型，一种方法是自行为tflite模型添加元数据。这里有一篇指导说明如何为TFLite模型添加元数据：

> https://tensorflow.google.cn/lite/convert/metadata

2. 目前进支持图片分类和风格迁移类的模型，当然随着开发进程，这个会扩展到更多的模型。
3. 目前输入输出的数据类型仅支持DataType.UINT8和DataType.FLOAT32。

目前看来，这项新特性还完成的比较粗糙，但也可以看出谷歌的目标，将机器学习扩展到终端，让机器学习应用程序开发越来越简单。你觉得Android Studio的这项新特性有用吗？欢迎交流！

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)