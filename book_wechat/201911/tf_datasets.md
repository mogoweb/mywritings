公开数据集助长了机器学习研究的热潮（h / t Andrew Ng），但是将这些数据集简单地带入机器学习管道仍然很难。每个研究人员都需要编写一次性脚本来下载和准备他们使用的每个数据集，而每个数据集都有不同的源代码格式和复杂性，这让他们感到痛苦。不再。

今天，我们很高兴推出TensorFlow数据集（GitHub），该数据集将公共研究数据集公开为tf.data.Datasets和NumPy数组。它完成了获取源数据并将其准备为磁盘上通用格式的所有繁琐工作，并且使用tf.data API来构建高性能的输入管道，这些输入管道已支持TensorFlow 2.0，可与tf一起使用。 keras模型。我们将推出29种流行的研究数据集，例如MNIST，街景门牌号码，10亿字语言模型基准和大型电影评论数据集，并将在未来几个月内增加更多内容；我们希望您自己加入并添加数据集。

```
# Install: pip install tensorflow-datasets
import tensorflow_datasets as tfds
mnist_data = tfds.load("mnist")
mnist_train, mnist_test = mnist_data["train"], mnist_data["test"]
assert isinstance(mnist_train, tf.data.Dataset)
```

#### tfds.load和DatasetBuilder

每个数据集都作为DatasetBuilder公开，它知道：

* 从何处下载数据以及如何提取数据并将其写入标准格式（DatasetBuilder.download_and_prepare）。
* 如何从磁盘加载（DatasetBuilder.as_dataset）。
* 以及有关数据集的所有信息，例如所有要素的名称，类型和形状，每个拆分中的记录数，源URL，数据集或相关纸张的引用等（DatasetBuilder.info）。

您可以直接实例化任何DatasetBuilder或使用tfds.builder通过字符串获取它们：

```
import tensorflow_datasets as tfds

# Fetch the dataset directly
mnist = tfds.image.MNIST()
# or by string name
mnist = tfds.builder('mnist')

# Describe the dataset with DatasetInfo
assert mnist.info.features['image'].shape == (28, 28, 1)
assert mnist.info.features['label'].num_classes == 10
assert mnist.info.splits['train'].num_examples == 60000

# Download the data, prepare it, and write it to disk
mnist.download_and_prepare()

# Load data from disk as tf.data.Datasets
datasets = mnist.as_dataset()
train_dataset, test_dataset = datasets['train'], datasets['test']
assert isinstance(train_dataset, tf.data.Dataset)

# And convert the Dataset to NumPy arrays if you'd like
for example in tfds.as_numpy(train_dataset):
  image, label = example['image'], example['label']
  assert isinstance(image, np.array)
```
as_dataset（）接受batch_size参数，该参数将为您提供一批示例，而不是一次提供一个示例。 对于适合内存的小型数据集，您可以传递batch_size = -1作为tf.Tensor一次获取整个数据集。 使用tfds.as_numpy（）可以轻松将所有tf.data.Datasets转换为NumPy数组的可迭代对象。

为方便起见，您可以使用tfds.load来完成上述所有操作，tfds.load将按名称获取DatasetBuilder，调用download_and_prepare（），并调用as_dataset（）。

```
import tensorflow_datasets as tfds

datasets = tfds.load("mnist")
train_dataset, test_dataset = datasets["train"], datasets["test"]
assert isinstance(train_dataset, tf.data.Dataset)
```

您还可以通过传递with_info = True轻松地从tfds.load获取DatasetInfo对象。 有关所有选项，请参见API文档。

#### 数据集版本控制

每个数据集都有版本（builder.info.version），因此您可以放心，您下面的数据不会更改，并且结果可重复。 目前，我们保证如果数据更改，版本将增加。

请注意，虽然我们保证给定相同版本的数据值和拆分是相同的，但我们目前不保证相同版本的记录顺序。

#### 数据集配置

具有不同变体的数据集使用名为BuilderConfigs进行配置。 例如，大电影审阅数据集（tfds.text.IMDBReviews）可以对输入文本具有不同的编码（例如，纯文本，字符编码或子词编码）。 内置配置随数据集文档一起列出，可以通过字符串进行寻址，也可以传入自己的配置。

```
# See the built-in configs
configs = tfds.text.IMDBReviews.builder_configs
assert "bytes" in configs

# Address a built-in config with tfds.builder
imdb = tfds.builder("imdb_reviews/bytes")
# or when constructing the builder directly
imdb = tfds.text.IMDBReviews(config="bytes")
# or use your own custom configuration
my_encoder = tfds.features.text.ByteTextEncoder(additional_tokens=['hello'])
my_config = tfds.text.IMDBReviewsConfig(
    name="my_config",
    version="1.0.0",
    text_encoder_config=tfds.features.text.TextEncoderConfig(encoder=my_encoder),
)
imdb = tfds.text.IMDBReviews(config=my_config)
```

请参阅有关添加数据集的文档中有关数据集配置的部分。

#### 文本数据集和词汇表

由于编码和词汇文件不同，使用文本数据集通常会很痛苦。 tensorflow-datasets使它变得更加容易。 它附带了许多文本任务，并包括三种TextEncoder，它们均支持Unicode：

* ByteTextEncoder，用于字节/字符级编码
* TokenTextEncoder用于基于词汇文件的单词级编码
* SubwordTextEncoder，用于子字词级编码（以及构建调整为特定文本语料库的子字词词汇表的能力），并具有字节级的后备功能，使其完全可逆。 例如，“ hello world”可以分为[“ he”，“ llo”，“，”，“ wor”，“ ld”]，然后进行整数编码。 子词是介于单词级和字节级编码之间的一种快乐媒介，在某些自然语言研究项目中很流行。

编码器及其词汇量可以通过DatasetInfo访问：

```
imdb = tfds.builder("imdb_reviews/subwords8k")

# Get the TextEncoder from DatasetInfo
encoder = imdb.info.features["text"].encoder
assert isinstance(encoder, tfds.features.text.SubwordTextEncoder)

# Encode, decode
ids = encoder.encode("Hello world")
assert encoder.decode(ids) == "Hello world"

# Get the vocabulary size
vocab_size = encoder.vocab_size
```

TensorFlow和TensorFlow数据集都将在将来进一步改善文本支持。

pip install tensorflow-datasets

>>> tfds.list_builders()
['abstract_reasoning', 'aeslc', 'aflw2k3d', 'amazon_us_reviews', 'bair_robot_pushing_small', 'big_patent', 'bigearthnet', 'billsum', 'binarized_mnist', 'binary_alpha_digits', 'c4', 'caltech101', 'caltech_birds2010', 'caltech_birds2011', 'cars196', 'cassava', 'cats_vs_dogs', 'celeb_a', 'celeb_a_hq', 'chexpert', 'cifar10', 'cifar100', 'cifar10_1', 'cifar10_corrupted', 'clevr', 'cmaterdb', 'cnn_dailymail', 'coco', 'coco2014', 'coil100', 'colorectal_histology', 'colorectal_histology_large', 'curated_breast_imaging_ddsm', 'cycle_gan', 'deep_weeds', 'definite_pronoun_resolution', 'diabetic_retinopathy_detection', 'downsampled_imagenet', 'dsprites', 'dtd', 'dummy_dataset_shared_generator', 'dummy_mnist', 'emnist', 'eurosat', 'fashion_mnist', 'flores', 'food101', 'gap', 'gigaword', 'glue', 'groove', 'higgs', 'horses_or_humans', 'image_label_folder', 'imagenet2012', 'imagenet2012_corrupted', 'imagenet_resized', 'imdb_reviews', 'iris', 'kitti', 'kmnist', 'lfw', 'lm1b', 'lsun', 'malaria', 'mnist', 'mnist_corrupted', 'moving_mnist', 'multi_news', 'multi_nli', 'multi_nli_mismatch', 'newsroom', 'nsynth', 'omniglot', 'open_images_v4', 'oxford_flowers102', 'oxford_iiit_pet', 'para_crawl', 'patch_camelyon', 'pet_finder', 'places365_small', 'quickdraw_bitmap', 'reddit_tifu', 'resisc45', 'rock_paper_scissors', 'rock_you', 'scene_parse150', 'scientific_papers', 'shapes3d', 'smallnorb', 'snli', 'so2sat', 'squad', 'stanford_dogs', 'stanford_online_products', 'starcraft_video', 'sun397', 'super_glue', 'svhn_cropped', 'ted_hrlr_translate', 'ted_multi_translate', 'tf_flowers', 'the300w_lp', 'titanic', 'trivia_qa', 'uc_merced', 'ucf101', 'visual_domain_decathlon', 'voc', 'wider_face', 'wikihow', 'wikipedia', 'wmt14_translate', 'wmt15_translate', 'wmt16_translate', 'wmt17_translate', 'wmt18_translate', 'wmt19_translate', 'wmt_t2t_translate', 'wmt_translate', 'xnli', 'xsum']


tfds.load is a convenience method that's the simplest way to build and load a tf.data.Dataset.

Instructions for updating:
Use eager execution and: 
`tf.data.TFRecordDataset(path)`
Dataset mnist downloaded and prepared to /home/alex/tensorflow_datasets/mnist/1.0.0. Subsequent calls will reuse this data. 

https://adventuresinmachinelearning.com/tensorflow-dataset-tutorial/