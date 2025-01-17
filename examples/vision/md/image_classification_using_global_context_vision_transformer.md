# Image Classification using Global Context Vision Transformer

**Author:** Md Awsafur Rahman<br>
**Date created:** 2023/10/30<br>
**Last modified:** 2023/10/30<br>
**Description:** Implementation and fine-tuning of Global Context Vision Transformer for image classification.


<img class="k-inline-icon" src="https://colab.research.google.com/img/colab_favicon.ico"/> [**View in Colab**](https://colab.research.google.com/github/keras-team/keras-io/blob/master/examples/vision/ipynb/image_classification_using_global_context_vision_transformer.ipynb)  <span class="k-dot">•</span><img class="k-inline-icon" src="https://github.com/favicon.ico"/> [**GitHub source**](https://github.com/keras-team/keras-io/blob/master/examples/vision/image_classification_using_global_context_vision_transformer.py)



# Setup


```python
!pip install --upgrade keras_cv tensorflow
!pip install --upgrade keras
```

```python
import keras
from keras_cv.layers import DropPath
from keras import ops
from keras import layers

import tensorflow as tf  # only for dataloader
import tensorflow_datasets as tfds  # for flower dataset

from skimage.data import chelsea
import matplotlib.pyplot as plt
import numpy as np
```
<div class="k-default-codeblock">
```
Requirement already satisfied: keras_cv in /usr/local/lib/python3.10/dist-packages (0.7.2)
Requirement already satisfied: tensorflow in /usr/local/lib/python3.10/dist-packages (2.15.0)
Collecting tensorflow
  Downloading tensorflow-2.15.0.post1-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (475.2 MB)
[2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 475.2/475.2 MB [31m1.9 MB/s eta [36m0:00:00
[?25hRequirement already satisfied: packaging in /usr/local/lib/python3.10/dist-packages (from keras_cv) (23.2)
Requirement already satisfied: absl-py in /usr/local/lib/python3.10/dist-packages (from keras_cv) (1.4.0)
Requirement already satisfied: regex in /usr/local/lib/python3.10/dist-packages (from keras_cv) (2023.6.3)
Requirement already satisfied: tensorflow-datasets in /usr/local/lib/python3.10/dist-packages (from keras_cv) (4.9.4)
Requirement already satisfied: keras-core in /usr/local/lib/python3.10/dist-packages (from keras_cv) (0.1.7)
Requirement already satisfied: astunparse>=1.6.0 in /usr/local/lib/python3.10/dist-packages (from tensorflow) (1.6.3)
Requirement already satisfied: flatbuffers>=23.5.26 in /usr/local/lib/python3.10/dist-packages (from tensorflow) (23.5.26)
Requirement already satisfied: gast!=0.5.0,!=0.5.1,!=0.5.2,>=0.2.1 in /usr/local/lib/python3.10/dist-packages (from tensorflow) (0.5.4)
Requirement already satisfied: google-pasta>=0.1.1 in /usr/local/lib/python3.10/dist-packages (from tensorflow) (0.2.0)
Requirement already satisfied: h5py>=2.9.0 in /usr/local/lib/python3.10/dist-packages (from tensorflow) (3.9.0)
Requirement already satisfied: libclang>=13.0.0 in /usr/local/lib/python3.10/dist-packages (from tensorflow) (16.0.6)
Requirement already satisfied: ml-dtypes~=0.2.0 in /usr/local/lib/python3.10/dist-packages (from tensorflow) (0.2.0)
Requirement already satisfied: numpy<2.0.0,>=1.23.5 in /usr/local/lib/python3.10/dist-packages (from tensorflow) (1.23.5)
Requirement already satisfied: opt-einsum>=2.3.2 in /usr/local/lib/python3.10/dist-packages (from tensorflow) (3.3.0)
Requirement already satisfied: protobuf!=4.21.0,!=4.21.1,!=4.21.2,!=4.21.3,!=4.21.4,!=4.21.5,<5.0.0dev,>=3.20.3 in /usr/local/lib/python3.10/dist-packages (from tensorflow) (3.20.3)
Requirement already satisfied: setuptools in /usr/local/lib/python3.10/dist-packages (from tensorflow) (67.7.2)
Requirement already satisfied: six>=1.12.0 in /usr/local/lib/python3.10/dist-packages (from tensorflow) (1.16.0)
Requirement already satisfied: termcolor>=1.1.0 in /usr/local/lib/python3.10/dist-packages (from tensorflow) (2.4.0)
Requirement already satisfied: typing-extensions>=3.6.6 in /usr/local/lib/python3.10/dist-packages (from tensorflow) (4.5.0)
Requirement already satisfied: wrapt<1.15,>=1.11.0 in /usr/local/lib/python3.10/dist-packages (from tensorflow) (1.14.1)
Requirement already satisfied: tensorflow-io-gcs-filesystem>=0.23.1 in /usr/local/lib/python3.10/dist-packages (from tensorflow) (0.34.0)
Requirement already satisfied: grpcio<2.0,>=1.24.3 in /usr/local/lib/python3.10/dist-packages (from tensorflow) (1.60.0)
Requirement already satisfied: tensorboard<2.16,>=2.15 in /usr/local/lib/python3.10/dist-packages (from tensorflow) (2.15.1)
Requirement already satisfied: tensorflow-estimator<2.16,>=2.15.0 in /usr/local/lib/python3.10/dist-packages (from tensorflow) (2.15.0)
Requirement already satisfied: keras<2.16,>=2.15.0 in /usr/local/lib/python3.10/dist-packages (from tensorflow) (2.15.0)
Requirement already satisfied: wheel<1.0,>=0.23.0 in /usr/local/lib/python3.10/dist-packages (from astunparse>=1.6.0->tensorflow) (0.42.0)
Requirement already satisfied: google-auth<3,>=1.6.3 in /usr/local/lib/python3.10/dist-packages (from tensorboard<2.16,>=2.15->tensorflow) (2.17.3)
Requirement already satisfied: google-auth-oauthlib<2,>=0.5 in /usr/local/lib/python3.10/dist-packages (from tensorboard<2.16,>=2.15->tensorflow) (1.2.0)
Requirement already satisfied: markdown>=2.6.8 in /usr/local/lib/python3.10/dist-packages (from tensorboard<2.16,>=2.15->tensorflow) (3.5.1)
Requirement already satisfied: requests<3,>=2.21.0 in /usr/local/lib/python3.10/dist-packages (from tensorboard<2.16,>=2.15->tensorflow) (2.31.0)
Requirement already satisfied: tensorboard-data-server<0.8.0,>=0.7.0 in /usr/local/lib/python3.10/dist-packages (from tensorboard<2.16,>=2.15->tensorflow) (0.7.2)
Requirement already satisfied: werkzeug>=1.0.1 in /usr/local/lib/python3.10/dist-packages (from tensorboard<2.16,>=2.15->tensorflow) (3.0.1)
Requirement already satisfied: rich in /usr/local/lib/python3.10/dist-packages (from keras-core->keras_cv) (13.7.0)
Requirement already satisfied: namex in /usr/local/lib/python3.10/dist-packages (from keras-core->keras_cv) (0.0.7)
Requirement already satisfied: dm-tree in /usr/local/lib/python3.10/dist-packages (from keras-core->keras_cv) (0.1.8)
Requirement already satisfied: click in /usr/local/lib/python3.10/dist-packages (from tensorflow-datasets->keras_cv) (8.1.7)
Requirement already satisfied: etils[enp,epath,etree]>=0.9.0 in /usr/local/lib/python3.10/dist-packages (from tensorflow-datasets->keras_cv) (1.6.0)
Requirement already satisfied: promise in /usr/local/lib/python3.10/dist-packages (from tensorflow-datasets->keras_cv) (2.3)
Requirement already satisfied: psutil in /usr/local/lib/python3.10/dist-packages (from tensorflow-datasets->keras_cv) (5.9.5)
Requirement already satisfied: tensorflow-metadata in /usr/local/lib/python3.10/dist-packages (from tensorflow-datasets->keras_cv) (1.14.0)
Requirement already satisfied: toml in /usr/local/lib/python3.10/dist-packages (from tensorflow-datasets->keras_cv) (0.10.2)
Requirement already satisfied: tqdm in /usr/local/lib/python3.10/dist-packages (from tensorflow-datasets->keras_cv) (4.66.1)
Requirement already satisfied: array-record>=0.5.0 in /usr/local/lib/python3.10/dist-packages (from tensorflow-datasets->keras_cv) (0.5.0)
Requirement already satisfied: fsspec in /usr/local/lib/python3.10/dist-packages (from etils[enp,epath,etree]>=0.9.0->tensorflow-datasets->keras_cv) (2023.6.0)
Requirement already satisfied: importlib_resources in /usr/local/lib/python3.10/dist-packages (from etils[enp,epath,etree]>=0.9.0->tensorflow-datasets->keras_cv) (6.1.1)
Requirement already satisfied: zipp in /usr/local/lib/python3.10/dist-packages (from etils[enp,epath,etree]>=0.9.0->tensorflow-datasets->keras_cv) (3.17.0)
Requirement already satisfied: cachetools<6.0,>=2.0.0 in /usr/local/lib/python3.10/dist-packages (from google-auth<3,>=1.6.3->tensorboard<2.16,>=2.15->tensorflow) (5.3.2)
Requirement already satisfied: pyasn1-modules>=0.2.1 in /usr/local/lib/python3.10/dist-packages (from google-auth<3,>=1.6.3->tensorboard<2.16,>=2.15->tensorflow) (0.3.0)
Requirement already satisfied: rsa<5,>=3.1.4 in /usr/local/lib/python3.10/dist-packages (from google-auth<3,>=1.6.3->tensorboard<2.16,>=2.15->tensorflow) (4.9)
Requirement already satisfied: requests-oauthlib>=0.7.0 in /usr/local/lib/python3.10/dist-packages (from google-auth-oauthlib<2,>=0.5->tensorboard<2.16,>=2.15->tensorflow) (1.3.1)
Requirement already satisfied: charset-normalizer<4,>=2 in /usr/local/lib/python3.10/dist-packages (from requests<3,>=2.21.0->tensorboard<2.16,>=2.15->tensorflow) (3.3.2)
Requirement already satisfied: idna<4,>=2.5 in /usr/local/lib/python3.10/dist-packages (from requests<3,>=2.21.0->tensorboard<2.16,>=2.15->tensorflow) (3.6)
Requirement already satisfied: urllib3<3,>=1.21.1 in /usr/local/lib/python3.10/dist-packages (from requests<3,>=2.21.0->tensorboard<2.16,>=2.15->tensorflow) (2.0.7)
Requirement already satisfied: certifi>=2017.4.17 in /usr/local/lib/python3.10/dist-packages (from requests<3,>=2.21.0->tensorboard<2.16,>=2.15->tensorflow) (2023.11.17)
Requirement already satisfied: MarkupSafe>=2.1.1 in /usr/local/lib/python3.10/dist-packages (from werkzeug>=1.0.1->tensorboard<2.16,>=2.15->tensorflow) (2.1.3)
Requirement already satisfied: markdown-it-py>=2.2.0 in /usr/local/lib/python3.10/dist-packages (from rich->keras-core->keras_cv) (3.0.0)
Requirement already satisfied: pygments<3.0.0,>=2.13.0 in /usr/local/lib/python3.10/dist-packages (from rich->keras-core->keras_cv) (2.16.1)
Requirement already satisfied: googleapis-common-protos<2,>=1.52.0 in /usr/local/lib/python3.10/dist-packages (from tensorflow-metadata->tensorflow-datasets->keras_cv) (1.62.0)
Requirement already satisfied: mdurl~=0.1 in /usr/local/lib/python3.10/dist-packages (from markdown-it-py>=2.2.0->rich->keras-core->keras_cv) (0.1.2)
Requirement already satisfied: pyasn1<0.6.0,>=0.4.6 in /usr/local/lib/python3.10/dist-packages (from pyasn1-modules>=0.2.1->google-auth<3,>=1.6.3->tensorboard<2.16,>=2.15->tensorflow) (0.5.1)
Requirement already satisfied: oauthlib>=3.0.0 in /usr/local/lib/python3.10/dist-packages (from requests-oauthlib>=0.7.0->google-auth-oauthlib<2,>=0.5->tensorboard<2.16,>=2.15->tensorflow) (3.2.2)
Installing collected packages: tensorflow
  Attempting uninstall: tensorflow
    Found existing installation: tensorflow 2.15.0
    Uninstalling tensorflow-2.15.0:
      Successfully uninstalled tensorflow-2.15.0
Successfully installed tensorflow-2.15.0.post1
Requirement already satisfied: keras in /usr/local/lib/python3.10/dist-packages (2.15.0)
Collecting keras
  Downloading keras-3.0.2-py3-none-any.whl (1.0 MB)
[2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 1.0/1.0 MB [31m12.2 MB/s eta [36m0:00:00
[?25hRequirement already satisfied: absl-py in /usr/local/lib/python3.10/dist-packages (from keras) (1.4.0)
Requirement already satisfied: numpy in /usr/local/lib/python3.10/dist-packages (from keras) (1.23.5)
Requirement already satisfied: rich in /usr/local/lib/python3.10/dist-packages (from keras) (13.7.0)
Requirement already satisfied: namex in /usr/local/lib/python3.10/dist-packages (from keras) (0.0.7)
Requirement already satisfied: h5py in /usr/local/lib/python3.10/dist-packages (from keras) (3.9.0)
Requirement already satisfied: dm-tree in /usr/local/lib/python3.10/dist-packages (from keras) (0.1.8)
Requirement already satisfied: markdown-it-py>=2.2.0 in /usr/local/lib/python3.10/dist-packages (from rich->keras) (3.0.0)
Requirement already satisfied: pygments<3.0.0,>=2.13.0 in /usr/local/lib/python3.10/dist-packages (from rich->keras) (2.16.1)
Requirement already satisfied: mdurl~=0.1 in /usr/local/lib/python3.10/dist-packages (from markdown-it-py>=2.2.0->rich->keras) (0.1.2)
Installing collected packages: keras
  Attempting uninstall: keras
    Found existing installation: keras 2.15.0
    Uninstalling keras-2.15.0:
      Successfully uninstalled keras-2.15.0
[31mERROR: pip's dependency resolver does not currently take into account all the packages that are installed. This behaviour is the source of the following dependency conflicts.
tensorflow 2.15.0.post1 requires keras<2.16,>=2.15.0, but you have keras 3.0.2 which is incompatible.[31m
Successfully installed keras-3.0.2

```
</div>
---
## Introduction

In this notebook, we will utilize multi-backend Keras 3.0 to implement the
[**GCViT: Global Context Vision Transformer**](https://arxiv.org/abs/2206.09959) paper,
presented at ICML 2023 by A Hatamizadeh et al. The, we will fine-tune the model on the
Flower dataset for image classification task, leveraging the official ImageNet pre-trained
weights. A highlight of this notebook is its compatibility with multiple backends:
TensorFlow, PyTorch, and JAX, showcasing the true potential of multi-backend Keras.

---
## Motivation

> **Note:** In this section we'll learn about the backstory of GCViT and try to
understand why it is proposed.

* During recent years, **Transformers** have achieved dominance in **Natural Language
Processing (NLP)** tasks and with the **self-attention** mechanism which allows for
capturing both long and short-range information.
* Following this trend, **Vision Transformer (ViT)** proposed to utilize image patches as
tokens in a gigantic architecture similar to encoder of the original Transformer.
* Despite the historic dominance of **Convolutional Neural Network (CNN)** in computer
vision, **ViT-based** models have shown **SOTA or competitive performance** in various
computer vision tasks.
<img src="https://raw.githubusercontent.com/awsaf49/gcvit-tf/main/image/vit_gif.gif"
width=600>

* However, **quadratic [`O(n^2)`] computational complexity** of self-attention and **lack
of multi-scale information** makes it difficult for **ViT** to be considered as
general-purpose architecture for Compute Vision tasks like **segmentation and object
detection** where it requires **dense prediction at the pixel level**.
* Swin Transformer has attempted to address the issues of **ViT** by proposing
**multi-resolution/hierarchical** architectures in which the self-attention is computed
in **local windows** and cross-window connections such as **window shifting** are used
for modeling the interactions across different regions. But the **limited receptive field
of local windows** can not capture long-range information, and cross-window-connection
schemes such as **window-shifting only cover a small neighborhood** in the vicinity of
each window. Also, it lacks **inductive-bias** that encourages certain translation
invariance is still preferable for general-purpose visual modeling, particularly for the
dense prediction tasks of object detection and semantic segmentation.

<img src="https://raw.githubusercontent.com/awsaf49/gcvit-tf/main/image/swin_vs_vit.JPG"
width=400>     <img
src="https://raw.githubusercontent.com/awsaf49/gcvit-tf/main/image/shifted_window.JPG"
width=400>
<img src="https://raw.githubusercontent.com/awsaf49/gcvit-tf/main/image/swin_arch.JPG"
width=800>

* To address above limitations, **Global Context (GC) ViT** network is proposed.

---
## Architecture

Let's have a quick **overview** of our key components,
1. `Stem/PatchEmbed:` A stem/patchify layer processes images at the network’s beginning.
For this network, it creates **patches/tokens** and converts them into **embeddings**.
2. `Level:` It is the repetitive building block that extracts features using different
blocks.
3. `Global Token Gen./FeatureExtraction:` It generates **global tokens/patches** with
**Depthwise-CNN**, **SqueezeAndExcitation (Squeeze-Excitation)**, **CNN** and
**MaxPooling**. So basically
it's a Feature Extractor.
4. `Block:` It is the repetitive module that applies attention to the features and
projects them to a certain dimension.
        1. `Local-MSA:` Local Multi head Self Attention.
        2. `Global-MSA:` Global Multi head Self Attention.
        3. `MLP:` Linear layer that projects a vector to another dimension.
5. `Downsample/ReduceSize:` It is very similar to **Global Token Gen.** module except it
uses **CNN** instead of **MaxPooling** to downsample with additional **Layer
Normalization** modules.
6. `Head:` It is the module responsible for the classification task.
    1. `Pooling:` It converts `N x 2D` features to `N x 1D` features.
    2. `Classifier:` It processes `N x 1D` features to make a decision about class.

I've annotated the architecture figure to make it easier to digest,
<img src="https://raw.githubusercontent.com/awsaf49/gcvit-tf/main/image/arch_annot.png">

### Unit Blocks

> **Note:** This blocks are used to build other modules throughout the paper. Most of the
blocks are either borrowed from other work or modified version old work.

1. `SqueezeAndExcitation`: **Squeeze-Excitation (SE)** aka **Bottleneck** module acts sd
kind of **channel
attention**. It consits of **AvgPooling**, **Dense/FullyConnected (FC)/Linear** ,
**GELU** and **Sigmoid** module.
<img src="https://raw.githubusercontent.com/awsaf49/gcvit-tf/main/image/se_annot.png"
width=400>

2. `Fused-MBConv:` This is similar to the one used in **EfficientNetV2**. It uses
**Depthwise-Conv**, **GELU**, **SqueezeAndExcitation**, **Conv**, to extract feature with
a resiudal
connection. Note that, no new module is declared for this one, we simply applied
corresponding modules directly.
<img src="https://raw.githubusercontent.com/awsaf49/gcvit-tf/main/image/fmb_annot.png"
width=350>

3. `ReduceSize`: It is a **CNN** based **downsample** module which abvobe mentioned
`Fused-MBConv` module to extract feature, **Strided Conv** to simultaneously reduce
spatial dimension and increse channelwise dimention of the features and finally
**LayerNormalization** module to normalize features. In the paper/figure this module is
referred as **downsample** module. I think it is mention worthy that **SwniTransformer**
used `PatchMerging` module instead of `ReduceSize` to reduce the spatial dimention and
increase channelwise dimension which uses **fully-connected/dense/linear** module.
According to the **GCViT** paper, one of the purposes of using `ReduceSize` is to add
inductive bias through **CNN** module.
<img src="https://raw.githubusercontent.com/awsaf49/gcvit-tf/main/image/down_annot.png"
width=300>

4. `MLP:` This is our very own **Multi Layer Perceptron** module. This a
feed-forward/fully-connected/linear module which simply projects input to an arbitary
dimension.


```python

class SqueezeAndExcitation(layers.Layer):
    """Squeeze and excitation block.

    Args:
        output_dim: output features dimension, if `None` use same dim as input.
        expansion: expansion ratio.
    """

    def __init__(self, output_dim=None, expansion=0.25, **kwargs):
        super().__init__(**kwargs)
        self.expansion = expansion
        self.output_dim = output_dim

    def build(self, input_shape):
        inp = input_shape[-1]
        self.output_dim = self.output_dim or inp
        self.avg_pool = layers.GlobalAvgPool2D(keepdims=True, name="avg_pool")
        self.fc = [
            layers.Dense(int(inp * self.expansion), use_bias=False, name="fc_0"),
            layers.Activation("gelu", name="fc_1"),
            layers.Dense(self.output_dim, use_bias=False, name="fc_2"),
            layers.Activation("sigmoid", name="fc_3"),
        ]
        super().build(input_shape)

    def call(self, inputs, **kwargs):
        x = self.avg_pool(inputs)
        for layer in self.fc:
            x = layer(x)
        return x * inputs


class ReduceSize(layers.Layer):
    """Down-sampling block.

    Args:
        keepdims: if False spatial dim is reduced and channel dim is increased
    """

    def __init__(self, keepdims=False, **kwargs):
        super().__init__(**kwargs)
        self.keepdims = keepdims

    def build(self, input_shape):
        embed_dim = input_shape[-1]
        dim_out = embed_dim if self.keepdims else 2 * embed_dim
        self.pad1 = layers.ZeroPadding2D(1, name="pad1")
        self.pad2 = layers.ZeroPadding2D(1, name="pad2")
        self.conv = [
            layers.DepthwiseConv2D(
                kernel_size=3, strides=1, padding="valid", use_bias=False, name="conv_0"
            ),
            layers.Activation("gelu", name="conv_1"),
            SqueezeAndExcitation(name="conv_2"),
            layers.Conv2D(
                embed_dim,
                kernel_size=1,
                strides=1,
                padding="valid",
                use_bias=False,
                name="conv_3",
            ),
        ]
        self.reduction = layers.Conv2D(
            dim_out,
            kernel_size=3,
            strides=2,
            padding="valid",
            use_bias=False,
            name="reduction",
        )
        self.norm1 = layers.LayerNormalization(
            -1, 1e-05, name="norm1"
        )  # eps like PyTorch
        self.norm2 = layers.LayerNormalization(-1, 1e-05, name="norm2")

    def call(self, inputs, **kwargs):
        x = self.norm1(inputs)
        xr = self.pad1(x)
        for layer in self.conv:
            xr = layer(xr)
        x = x + xr
        x = self.pad2(x)
        x = self.reduction(x)
        x = self.norm2(x)
        return x


class MLP(layers.Layer):
    """Multi-Layer Perceptron (MLP) block.

    Args:
        hidden_features: hidden features dimension.
        out_features: output features dimension.
        activation: activation function.
        dropout: dropout rate.
    """

    def __init__(
        self,
        hidden_features=None,
        out_features=None,
        activation="gelu",
        dropout=0.0,
        **kwargs,
    ):
        super().__init__(**kwargs)
        self.hidden_features = hidden_features
        self.out_features = out_features
        self.activation = activation
        self.dropout = dropout

    def build(self, input_shape):
        self.in_features = input_shape[-1]
        self.hidden_features = self.hidden_features or self.in_features
        self.out_features = self.out_features or self.in_features
        self.fc1 = layers.Dense(self.hidden_features, name="fc1")
        self.act = layers.Activation(self.activation, name="act")
        self.fc2 = layers.Dense(self.out_features, name="fc2")
        self.drop1 = layers.Dropout(self.dropout, name="drop1")
        self.drop2 = layers.Dropout(self.dropout, name="drop2")

    def call(self, inputs, **kwargs):
        x = self.fc1(inputs)
        x = self.act(x)
        x = self.drop1(x)
        x = self.fc2(x)
        x = self.drop2(x)
        return x

```

### Stem

> **Notes**: In the code, this module is referred to as **PatchEmbed** but on paper, it
is referred to as **Stem**.

In the model, we have first used `patch_embed` module. Let's try to understand this
module. As we can see from the `call` method,
1. This module first **pads** input
2. Then uses **convolutions** to extract patches with embeddings.
3. Finally, uses `ReduceSize` module to first extract features with **convolution** but
neither reduces spatial dimension nor increases spatial dimension.
4. One important point to notice, unlike **ViT** or **SwinTransformer**, **GCViT**
creates **overlapping patches**. We can notice that from the code,
`Conv2D(self.embed_dim, kernel_size=3, strides=2, name='proj')`. If we wanted
**non-overlapping** patches then we would've used the same `kernel_size` and `stride`.
5. This module reduces the spatial dimension of input by `4x`.
> Summary: image → padding → convolution →
(feature_extract + downsample)


```python

class PatchEmbed(layers.Layer):
    """Patch embedding block.

    Args:
        embed_dim: feature size dimension.
    """

    def __init__(self, embed_dim, **kwargs):
        super().__init__(**kwargs)
        self.embed_dim = embed_dim

    def build(self, input_shape):
        self.pad = layers.ZeroPadding2D(1, name="pad")
        self.proj = layers.Conv2D(self.embed_dim, 3, 2, name="proj")
        self.conv_down = ReduceSize(keepdims=True, name="conv_down")

    def call(self, inputs, **kwargs):
        x = self.pad(inputs)
        x = self.proj(x)
        x = self.conv_down(x)
        return x

```

### Global Token Gen.

> **Notes:** It is one of the two **CNN** modules that is used to imppose inductive bias.

As we can see from above cell, in the `level` we have first used `to_q_global/Global
Token Gen./FeatureExtraction`. Let's try to understand how it works,

* This module is series of `FeatureExtract` module, according to paper we need to
repeat this module `K` times, where `K = log2(H/h)`, `H = feature_map_height`,
`W = feature_map_width`.
* `FeatureExtraction:` This layer is very similar to `ReduceSize` module except it uses
**MaxPooling** module to reduce the dimension, it doesn't increse feature dimension
(channelsie) and it doesn't uses **LayerNormalizaton**. This module is used to in
`Generate Token Gen.` module repeatedly to generte **global tokens** for
**global-context-attention**.
* One important point to notice from the figure is that, **global tokens** is shared
across the whole image which means we use only **one global window** for **all local
tokens** in a image. This makes the computation very efficient.
* For input feature map with shape `(B, H, W, C)`, we'll get output shape `(B, h, w, C)`.
If we copy these global tokens for total `M` local windows in an image where,
`M = (H x W)/(h x w) = num_window`, then output shape: `(B * M, h, w, C)`."

> Summary: This module is used to `resize` the image to fit window.

<img
src="https://raw.githubusercontent.com/awsaf49/gcvit-tf/main/image/global_token_annot.png"
width=800>


```python

class FeatureExtraction(layers.Layer):
    """Feature extraction block.

    Args:
        keepdims: bool argument for maintaining the resolution.
    """

    def __init__(self, keepdims=False, **kwargs):
        super().__init__(**kwargs)
        self.keepdims = keepdims

    def build(self, input_shape):
        embed_dim = input_shape[-1]
        self.pad1 = layers.ZeroPadding2D(1, name="pad1")
        self.pad2 = layers.ZeroPadding2D(1, name="pad2")
        self.conv = [
            layers.DepthwiseConv2D(3, 1, use_bias=False, name="conv_0"),
            layers.Activation("gelu", name="conv_1"),
            SqueezeAndExcitation(name="conv_2"),
            layers.Conv2D(embed_dim, 1, 1, use_bias=False, name="conv_3"),
        ]
        if not self.keepdims:
            self.pool = layers.MaxPool2D(3, 2, name="pool")
        super().build(input_shape)

    def call(self, inputs, **kwargs):
        x = inputs
        xr = self.pad1(x)
        for layer in self.conv:
            xr = layer(xr)
        x = x + xr
        if not self.keepdims:
            x = self.pool(self.pad2(x))
        return x


class GlobalQueryGenerator(layers.Layer):
    """Global query generator.

    Args:
        keepdims: to keep the dimension of FeatureExtraction layer.
        For instance, repeating log(56/7) = 3 blocks, with input
        window dimension 56 and output window dimension 7 at down-sampling
        ratio 2. Please check Fig.5 of GC ViT paper for details.
    """

    def __init__(self, keepdims=False, **kwargs):
        super().__init__(**kwargs)
        self.keepdims = keepdims

    def build(self, input_shape):
        self.to_q_global = [
            FeatureExtraction(keepdims, name=f"to_q_global_{i}")
            for i, keepdims in enumerate(self.keepdims)
        ]
        super().build(input_shape)

    def call(self, inputs, **kwargs):
        x = inputs
        for layer in self.to_q_global:
            x = layer(x)
        return x

```

### Attention

> **Notes:** This is the core contribution of the paper.

As we can see from the `call` method,
1. `WindowAttention` module applies both **local** and **global** window attention
depending on `global_query` parameter.

2. First it converts input features into `query, key, value` for local attention and
`key, value` for global attention. For global attention, it takes global query from
`Global Token Gen.`. One thing to notice from the code is that we divide the **features
or embed_dim** among all the **heads of Transformer** to reduce the computation.
`qkv = tf.reshape(qkv, [B_, N, self.qkv_size, self.num_heads, C // self.num_heads])`
3. Before sending query, key and value for attention, **global token** goes through an
important process. Same global tokens or one global window gets copied for all the local
windows to increase efficiency.
`q_global = tf.repeat(q_global, repeats=B_//B, axis=0)`, here `B_//B` means `num_windows`
in a image.
4. Then simply applies `local-window-self-attention` or `global-window-attention`
depending on `global_query` parameter. One thing to notice from the code is that we are
adding **relative-positional-embedding** with the **attention mask** instead of the
**patch embedding**.
`attn = attn + relative_position_bias[tf.newaxis,]`
<img src="https://raw.githubusercontent.com/awsaf49/gcvit-tf/main/image/lvg_msa.PNG"
width=800>
5. Now, let's think for a bit and try to understand what is happening here. Let's focus
on the figure below. We can see from the left, that in the **local-attention** the
**query is local** and it's **limited to the local window** (red square border) hence we
don't have access to long-range information. But on the right that due to **global
query** we're now **not limited to local-windows** (blue square border) and we have
access to long-range information.
<img src="https://raw.githubusercontent.com/awsaf49/gcvit-tf/main/image/lvg_arch.PNG"
width=800>
6. In **ViT** we compare (attention) image-tokens with image-tokens, in
**SwinTransformer** we compare window-tokens with window-tokens but in **GCViT** we
compare image-tokens with window-tokens. But now you may ask, how can compare(attention)
image-tokens with window-tokens even after image-tokens have larger dimensions than
window-tokens? (from above figure image-tokens have shape `(1, 8, 8, 3)` and
window-tokens have shape `(1, 4, 4, 3)`). Yes, you are right we can't directly compare
them hence we resize image-tokens to fit window-tokens with `Global Token
Gen./FeatureExtraction` **CNN** module. The following table should give you a clear
comparison,

| Model            | Query Tokens    | Key-Value Tokens  | Attention Type            | Attention Coverage |
|------------------|-----------------|-------------------|---------------------------|--------------------|
| ViT              | image           | image             | self-attention            | global             |
| SwinTransformer  | window          | window            | self-attention            | local              |
| **GCViT**        | **resized-image** | **window**     | **image-window attention** | **global**        |


```python

class WindowAttention(layers.Layer):
    """Local window attention.

    This implementation was proposed by
    [Liu et al., 2021](https://arxiv.org/abs/2103.14030) in SwinTransformer.

    Args:
        window_size: window size.
        num_heads: number of attention head.
        global_query: if the input contains global_query
        qkv_bias: bool argument for query, key, value learnable bias.
        qk_scale: bool argument to scaling query, key.
        attention_dropout: attention dropout rate.
        projection_dropout: output dropout rate.
    """

    def __init__(
        self,
        window_size,
        num_heads,
        global_query,
        qkv_bias=True,
        qk_scale=None,
        attention_dropout=0.0,
        projection_dropout=0.0,
        **kwargs,
    ):
        super().__init__(**kwargs)
        window_size = (window_size, window_size)
        self.window_size = window_size
        self.num_heads = num_heads
        self.global_query = global_query
        self.qkv_bias = qkv_bias
        self.qk_scale = qk_scale
        self.attention_dropout = attention_dropout
        self.projection_dropout = projection_dropout

    def build(self, input_shape):
        embed_dim = input_shape[0][-1]
        head_dim = embed_dim // self.num_heads
        self.scale = self.qk_scale or head_dim**-0.5
        self.qkv_size = 3 - int(self.global_query)
        self.qkv = layers.Dense(
            embed_dim * self.qkv_size, use_bias=self.qkv_bias, name="qkv"
        )
        self.relative_position_bias_table = self.add_weight(
            name="relative_position_bias_table",
            shape=[
                (2 * self.window_size[0] - 1) * (2 * self.window_size[1] - 1),
                self.num_heads,
            ],
            initializer=keras.initializers.TruncatedNormal(stddev=0.02),
            trainable=True,
            dtype=self.dtype,
        )
        self.attn_drop = layers.Dropout(self.attention_dropout, name="attn_drop")
        self.proj = layers.Dense(embed_dim, name="proj")
        self.proj_drop = layers.Dropout(self.projection_dropout, name="proj_drop")
        self.softmax = layers.Activation("softmax", name="softmax")
        super().build(input_shape)

    def get_relative_position_index(self):
        coords_h = ops.arange(self.window_size[0])
        coords_w = ops.arange(self.window_size[1])
        coords = ops.stack(ops.meshgrid(coords_h, coords_w, indexing="ij"), axis=0)
        coords_flatten = ops.reshape(coords, [2, -1])
        relative_coords = coords_flatten[:, :, None] - coords_flatten[:, None, :]
        relative_coords = ops.transpose(relative_coords, axes=[1, 2, 0])
        relative_coords_xx = relative_coords[:, :, 0] + self.window_size[0] - 1
        relative_coords_yy = relative_coords[:, :, 1] + self.window_size[1] - 1
        relative_coords_xx = relative_coords_xx * (2 * self.window_size[1] - 1)
        relative_position_index = relative_coords_xx + relative_coords_yy
        return relative_position_index

    def call(self, inputs, **kwargs):
        if self.global_query:
            inputs, q_global = inputs
            B = ops.shape(q_global)[0]  # B, N, C
        else:
            inputs = inputs[0]
        B_, N, C = ops.shape(inputs)  # B*num_window, num_tokens, channels
        qkv = self.qkv(inputs)
        qkv = ops.reshape(
            qkv, [B_, N, self.qkv_size, self.num_heads, C // self.num_heads]
        )
        qkv = ops.transpose(qkv, [2, 0, 3, 1, 4])
        if self.global_query:
            k, v = ops.split(
                qkv, indices_or_sections=2, axis=0
            )  # for unknown shame num=None will throw error
            q_global = ops.repeat(
                q_global, repeats=B_ // B, axis=0
            )  # num_windows = B_//B => q_global same for all windows in a img
            q = ops.reshape(
                q_global, new_shape=[B_, N, self.num_heads, C // self.num_heads]
            )
            q = ops.transpose(q, axes=[0, 2, 1, 3])
        else:
            q, k, v = ops.split(qkv, indices_or_sections=3, axis=0)
            q = ops.squeeze(q, axis=0)

        k = ops.squeeze(k, axis=0)
        v = ops.squeeze(v, axis=0)

        q = q * self.scale
        attn = q @ ops.transpose(k, axes=[0, 1, 3, 2])
        relative_position_bias = ops.take(
            self.relative_position_bias_table,
            ops.reshape(self.get_relative_position_index(), new_shape=[-1]),
        )
        relative_position_bias = ops.reshape(
            relative_position_bias,
            new_shape=[
                self.window_size[0] * self.window_size[1],
                self.window_size[0] * self.window_size[1],
                -1,
            ],
        )
        relative_position_bias = ops.transpose(relative_position_bias, axes=[2, 0, 1])
        attn = attn + relative_position_bias[None,]
        attn = self.softmax(attn)
        attn = self.attn_drop(attn)

        x = ops.transpose((attn @ v), axes=[0, 2, 1, 3])
        x = ops.reshape(x, new_shape=[B_, N, C])
        x = self.proj_drop(self.proj(x))
        return x

```

### Block

> **Notes:** This module doesn't have any Convolutional module.

In the `level` second module that we have used is `block`. Let's try to understand how it
works. As we can see from the `call` method,
1. `Block` module takes either only feature_maps for local attention or additional global
query for global attention.
2. Before sending feature maps for attention, this module converts **batch feature maps**
to **batch windows** as we'll be applying **Window Attention**.
3. Then we send batch **batch windows** for attention.
4. After attention has been applied we revert **batch windows** to **batch feature maps**.
5. Before sending the attention to applied features for output, this module applies
**Stochastic Depth** regularization in the residual connection. Also, before applying
**Stochastic Depth** it rescales the input with trainable parameters. Note that, this
**Stochastic Depth** block hasn't been shown in the figure of the paper.

<img src="https://raw.githubusercontent.com/awsaf49/gcvit-tf/main/image/block2.JPG"
width=400>


### Window
In the `block` module, we have created **windows** before and after applying attention.
Let's try to understand how we're creating windows,
* Following module converts feature maps `(B, H, W, C)` to stacked windows
`(B x H/h x W/w, h, w, C)` → `(num_windows_batch, window_size, window_size, channel)`
* This module uses `reshape` & `transpose` to create these windows out of image instead
of iterating over them.


```python

class Block(layers.Layer):
    """GCViT block.

    Args:
        window_size: window size.
        num_heads: number of attention head.
        global_query: apply global window attention
        mlp_ratio: MLP ratio.
        qkv_bias: bool argument for query, key, value learnable bias.
        qk_scale: bool argument to scaling query, key.
        drop: dropout rate.
        attention_dropout: attention dropout rate.
        path_drop: drop path rate.
        activation: activation function.
        layer_scale: layer scaling coefficient.
    """

    def __init__(
        self,
        window_size,
        num_heads,
        global_query,
        mlp_ratio=4.0,
        qkv_bias=True,
        qk_scale=None,
        dropout=0.0,
        attention_dropout=0.0,
        path_drop=0.0,
        activation="gelu",
        layer_scale=None,
        **kwargs,
    ):
        super().__init__(**kwargs)
        self.window_size = window_size
        self.num_heads = num_heads
        self.global_query = global_query
        self.mlp_ratio = mlp_ratio
        self.qkv_bias = qkv_bias
        self.qk_scale = qk_scale
        self.dropout = dropout
        self.attention_dropout = attention_dropout
        self.path_drop = path_drop
        self.activation = activation
        self.layer_scale = layer_scale

    def build(self, input_shape):
        B, H, W, C = input_shape[0]
        self.norm1 = layers.LayerNormalization(-1, 1e-05, name="norm1")
        self.attn = WindowAttention(
            window_size=self.window_size,
            num_heads=self.num_heads,
            global_query=self.global_query,
            qkv_bias=self.qkv_bias,
            qk_scale=self.qk_scale,
            attention_dropout=self.attention_dropout,
            projection_dropout=self.dropout,
            name="attn",
        )
        self.drop_path1 = DropPath(self.path_drop)
        self.drop_path2 = DropPath(self.path_drop)
        self.norm2 = layers.LayerNormalization(-1, 1e-05, name="norm2")
        self.mlp = MLP(
            hidden_features=int(C * self.mlp_ratio),
            dropout=self.dropout,
            activation=self.activation,
            name="mlp",
        )
        if self.layer_scale is not None:
            self.gamma1 = self.add_weight(
                name="gamma1",
                shape=[C],
                initializer=keras.initializers.Constant(self.layer_scale),
                trainable=True,
                dtype=self.dtype,
            )
            self.gamma2 = self.add_weight(
                name="gamma2",
                shape=[C],
                initializer=keras.initializers.Constant(self.layer_scale),
                trainable=True,
                dtype=self.dtype,
            )
        else:
            self.gamma1 = 1.0
            self.gamma2 = 1.0
        self.num_windows = int(H // self.window_size) * int(W // self.window_size)
        super().build(input_shape)

    def call(self, inputs, **kwargs):
        if self.global_query:
            inputs, q_global = inputs
        else:
            inputs = inputs[0]
        B, H, W, C = ops.shape(inputs)
        x = self.norm1(inputs)
        # create windows and concat them in batch axis
        x = self.window_partition(x, self.window_size)  # (B_, win_h, win_w, C)
        # flatten patch
        x = ops.reshape(x, new_shape=[-1, self.window_size * self.window_size, C])
        # attention
        if self.global_query:
            x = self.attn([x, q_global])
        else:
            x = self.attn([x])
        # reverse window partition
        x = self.window_reverse(x, self.window_size, H, W, C)
        # FFN
        x = inputs + self.drop_path1(x * self.gamma1)
        x = x + self.drop_path2(self.gamma2 * self.mlp(self.norm2(x)))
        return x

    def window_partition(self, x, window_size):
        """
        Args:
            x: (B, H, W, C)
            window_size: window size
        Returns:
            local window features (num_windows*B, window_size, window_size, C)
        """
        B, H, W, C = ops.shape(x)
        x = ops.reshape(
            x,
            new_shape=[
                -1,
                H // window_size,
                window_size,
                W // window_size,
                window_size,
                C,
            ],
        )
        x = ops.transpose(x, axes=[0, 1, 3, 2, 4, 5])
        windows = ops.reshape(x, new_shape=[-1, window_size, window_size, C])
        return windows

    def window_reverse(self, windows, window_size, H, W, C):
        """
        Args:
            windows: local window features (num_windows*B, window_size, window_size, C)
            window_size: Window size
            H: Height of image
            W: Width of image
            C: Channel of image
        Returns:
            x: (B, H, W, C)
        """
        x = ops.reshape(
            windows,
            new_shape=[
                -1,
                H // window_size,
                W // window_size,
                window_size,
                window_size,
                C,
            ],
        )
        x = ops.transpose(x, axes=[0, 1, 3, 2, 4, 5])
        x = ops.reshape(x, new_shape=[-1, H, W, C])
        return x

```

### Level

> **Note:** This module has both Transformer and CNN modules.

In the model, the second module that we have used is `level`. Let's try to understand
this module. As we can see from the `call` method,
1. First it creates **global_token** with a series of `FeatureExtraction` modules. As
we'll see
later that `FeatureExtraction` is nothing but a simple **CNN** based module.
2. Then it uses series of`Block` modules to apply **local or global window attention**
depending on depth level.
3. Finally, it uses `ReduceSize` to reduce the dimension of **contextualized features**.

> Summary: feature_map → global_token → local/global window
attention → dowsample

<img src="https://raw.githubusercontent.com/awsaf49/gcvit-tf/main/image/level.png"
width=400>


```python

class Level(layers.Layer):
    """GCViT level.

    Args:
        depth: number of layers in each stage.
        num_heads: number of heads in each stage.
        window_size: window size in each stage.
        keepdims: dims to keep in FeatureExtraction.
        downsample: bool argument for down-sampling.
        mlp_ratio: MLP ratio.
        qkv_bias: bool argument for query, key, value learnable bias.
        qk_scale: bool argument to scaling query, key.
        drop: dropout rate.
        attention_dropout: attention dropout rate.
        path_drop: drop path rate.
        layer_scale: layer scaling coefficient.
    """

    def __init__(
        self,
        depth,
        num_heads,
        window_size,
        keepdims,
        downsample=True,
        mlp_ratio=4.0,
        qkv_bias=True,
        qk_scale=None,
        dropout=0.0,
        attention_dropout=0.0,
        path_drop=0.0,
        layer_scale=None,
        **kwargs,
    ):
        super().__init__(**kwargs)
        self.depth = depth
        self.num_heads = num_heads
        self.window_size = window_size
        self.keepdims = keepdims
        self.downsample = downsample
        self.mlp_ratio = mlp_ratio
        self.qkv_bias = qkv_bias
        self.qk_scale = qk_scale
        self.dropout = dropout
        self.attention_dropout = attention_dropout
        self.path_drop = path_drop
        self.layer_scale = layer_scale

    def build(self, input_shape):
        path_drop = (
            [self.path_drop] * self.depth
            if not isinstance(self.path_drop, list)
            else self.path_drop
        )
        self.blocks = [
            Block(
                window_size=self.window_size,
                num_heads=self.num_heads,
                global_query=bool(i % 2),
                mlp_ratio=self.mlp_ratio,
                qkv_bias=self.qkv_bias,
                qk_scale=self.qk_scale,
                dropout=self.dropout,
                attention_dropout=self.attention_dropout,
                path_drop=path_drop[i],
                layer_scale=self.layer_scale,
                name=f"blocks_{i}",
            )
            for i in range(self.depth)
        ]
        self.down = ReduceSize(keepdims=False, name="downsample")
        self.q_global_gen = GlobalQueryGenerator(self.keepdims, name="q_global_gen")
        super().build(input_shape)

    def call(self, inputs, **kwargs):
        x = inputs
        q_global = self.q_global_gen(x)  # shape: (B, win_size, win_size, C)
        for i, blk in enumerate(self.blocks):
            if i % 2:
                x = blk([x, q_global])  # shape: (B, H, W, C)
            else:
                x = blk([x])  # shape: (B, H, W, C)
        if self.downsample:
            x = self.down(x)  # shape: (B, H//2, W//2, 2*C)
        return x

```

### Model

Let's directly jump to the model. As we can see from the `call` method,
1. It creates patch embeddings from an image. This layer doesn't flattens these
embeddings which means output of this module will be
`(batch, height/window_size, width/window_size, embed_dim)` instead of
`(batch, height x width/window_size^2, embed_dim)`.
2. Then it applies `Dropout` module which randomly sets input units to 0.
3. It passes these embeddings to series of `Level` modules which we are calling `level`
where,
    1. Global token is generated
    1. Both local & global attention is applied
    1. Finally downsample is applied.
4. So, output after `n` number of **levels**, shape: `(batch, width/window_size x 2^{n-1},
width/window_size x 2^{n-1}, embed_dim x 2^{n-1})`. In the last layer,
paper doesn't use **downsample** and increase **channels**.
5. Output of above layer is normalized using `LayerNormalization` module.
6. In the head, 2D features are converted to 1D features with `Pooling` module. Output
shape after this module is `(batch, embed_dim x 2^{n-1})`
7. Finally, pooled features are sent to `Dense/Linear` module for classification.

> Sumamry: image → (patchs + embedding) → dropout
→ (attention + feature extraction) → normalizaion →
pooling → classify


```python

class GCViT(keras.Model):
    """GCViT model.

    Args:
        window_size: window size in each stage.
        embed_dim: feature size dimension.
        depths: number of layers in each stage.
        num_heads: number of heads in each stage.
        drop_rate: dropout rate.
        mlp_ratio: MLP ratio.
        qkv_bias: bool argument for query, key, value learnable bias.
        qk_scale: bool argument to scaling query, key.
        attention_dropout: attention dropout rate.
        path_drop: drop path rate.
        layer_scale: layer scaling coefficient.
        num_classes: number of classes.
        head_activation: activation function for head.
    """

    def __init__(
        self,
        window_size,
        embed_dim,
        depths,
        num_heads,
        drop_rate=0.0,
        mlp_ratio=3.0,
        qkv_bias=True,
        qk_scale=None,
        attention_dropout=0.0,
        path_drop=0.1,
        layer_scale=None,
        num_classes=1000,
        head_activation="softmax",
        **kwargs,
    ):
        super().__init__(**kwargs)
        self.window_size = window_size
        self.embed_dim = embed_dim
        self.depths = depths
        self.num_heads = num_heads
        self.drop_rate = drop_rate
        self.mlp_ratio = mlp_ratio
        self.qkv_bias = qkv_bias
        self.qk_scale = qk_scale
        self.attention_dropout = attention_dropout
        self.path_drop = path_drop
        self.layer_scale = layer_scale
        self.num_classes = num_classes
        self.head_activation = head_activation

        self.patch_embed = PatchEmbed(embed_dim=embed_dim, name="patch_embed")
        self.pos_drop = layers.Dropout(drop_rate, name="pos_drop")
        path_drops = np.linspace(0.0, path_drop, sum(depths))
        keepdims = [(0, 0, 0), (0, 0), (1,), (1,)]
        self.levels = []
        for i in range(len(depths)):
            path_drop = path_drops[sum(depths[:i]) : sum(depths[: i + 1])].tolist()
            level = Level(
                depth=depths[i],
                num_heads=num_heads[i],
                window_size=window_size[i],
                keepdims=keepdims[i],
                downsample=(i < len(depths) - 1),
                mlp_ratio=mlp_ratio,
                qkv_bias=qkv_bias,
                qk_scale=qk_scale,
                dropout=drop_rate,
                attention_dropout=attention_dropout,
                path_drop=path_drop,
                layer_scale=layer_scale,
                name=f"levels_{i}",
            )
            self.levels.append(level)
        self.norm = layers.LayerNormalization(axis=-1, epsilon=1e-05, name="norm")
        self.pool = layers.GlobalAvgPool2D(name="pool")
        self.head = layers.Dense(num_classes, name="head", activation=head_activation)

    def build(self, input_shape):
        super().build(input_shape)
        self.built = True

    def call(self, inputs, **kwargs):
        x = self.patch_embed(inputs)  # shape: (B, H, W, C)
        x = self.pos_drop(x)
        for level in self.levels:
            x = level(x)  # shape: (B, H_, W_, C_)
        x = self.norm(x)
        x = self.pool(x)  # shape: (B, C__)
        x = self.head(x)
        return x

    def build_graph(self, input_shape=(224, 224, 3)):
        """
        ref: https://www.kaggle.com/code/ipythonx/tf-hybrid-efficientnet-swin-transformer-gradcam
        """
        x = keras.Input(shape=input_shape)
        return keras.Model(inputs=[x], outputs=self.call(x), name=self.name)

    def summary(self, input_shape=(224, 224, 3)):
        return self.build_graph(input_shape).summary()

```

---
## Build Model

* Let's build a complete model with all the modules that we've explained above. We'll
build **GCViT-XXTiny** model with the configuration mentioned in the paper.
* Also we'll load the ported official **pre-trained** weights and try for some
predictions.


```python
# Model Configs
config = {
    "window_size": (7, 7, 14, 7),
    "embed_dim": 64,
    "depths": (2, 2, 6, 2),
    "num_heads": (2, 4, 8, 16),
    "mlp_ratio": 3.0,
    "path_drop": 0.2,
}
ckpt_link = (
    "https://github.com/awsaf49/gcvit-tf/releases/download/v1.1.6/gcvitxxtiny.keras"
)

# Build Model
model = GCViT(**config)
inp = ops.array(np.random.uniform(size=(1, 224, 224, 3)))
out = model(inp)

# Load Weights
ckpt_path = keras.utils.get_file(ckpt_link.split("/")[-1], ckpt_link)
model.load_weights(ckpt_path)

# Summary
model.summary((224, 224, 3))
```

<div class="k-default-codeblock">
```
Downloading data from https://github.com/awsaf49/gcvit-tf/releases/download/v1.1.6/gcvitxxtiny.keras
 48767519/48767519 ━━━━━━━━━━━━━━━━━━━━ 0s 0us/step

```
</div>
<pre style="white-space:pre;overflow-x:auto;line-height:normal;font-family:Menlo,'DejaVu Sans Mono',consolas,'Courier New',monospace"><span style="font-weight: bold">Model: "gc_vi_t"</span>
</pre>




<pre style="white-space:pre;overflow-x:auto;line-height:normal;font-family:Menlo,'DejaVu Sans Mono',consolas,'Courier New',monospace">┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━┓
┃<span style="font-weight: bold"> Layer (type)                       </span>┃<span style="font-weight: bold"> Output Shape                  </span>┃<span style="font-weight: bold">     Param # </span>┃
┡━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━┩
│ input_layer (<span style="color: #0087ff; text-decoration-color: #0087ff">InputLayer</span>)           │ (<span style="color: #00d7ff; text-decoration-color: #00d7ff">None</span>, <span style="color: #00af00; text-decoration-color: #00af00">224</span>, <span style="color: #00af00; text-decoration-color: #00af00">224</span>, <span style="color: #00af00; text-decoration-color: #00af00">3</span>)           │           <span style="color: #00af00; text-decoration-color: #00af00">0</span> │
├────────────────────────────────────┼───────────────────────────────┼─────────────┤
│ patch_embed (<span style="color: #0087ff; text-decoration-color: #0087ff">PatchEmbed</span>)           │ (<span style="color: #00d7ff; text-decoration-color: #00d7ff">None</span>, <span style="color: #00af00; text-decoration-color: #00af00">56</span>, <span style="color: #00af00; text-decoration-color: #00af00">56</span>, <span style="color: #00af00; text-decoration-color: #00af00">64</span>)            │      <span style="color: #00af00; text-decoration-color: #00af00">45,632</span> │
├────────────────────────────────────┼───────────────────────────────┼─────────────┤
│ pos_drop (<span style="color: #0087ff; text-decoration-color: #0087ff">Dropout</span>)                 │ (<span style="color: #00d7ff; text-decoration-color: #00d7ff">None</span>, <span style="color: #00af00; text-decoration-color: #00af00">56</span>, <span style="color: #00af00; text-decoration-color: #00af00">56</span>, <span style="color: #00af00; text-decoration-color: #00af00">64</span>)            │           <span style="color: #00af00; text-decoration-color: #00af00">0</span> │
├────────────────────────────────────┼───────────────────────────────┼─────────────┤
│ levels_0 (<span style="color: #0087ff; text-decoration-color: #0087ff">Level</span>)                   │ (<span style="color: #00d7ff; text-decoration-color: #00d7ff">None</span>, <span style="color: #00af00; text-decoration-color: #00af00">28</span>, <span style="color: #00af00; text-decoration-color: #00af00">28</span>, <span style="color: #00af00; text-decoration-color: #00af00">128</span>)           │     <span style="color: #00af00; text-decoration-color: #00af00">180,964</span> │
├────────────────────────────────────┼───────────────────────────────┼─────────────┤
│ levels_1 (<span style="color: #0087ff; text-decoration-color: #0087ff">Level</span>)                   │ (<span style="color: #00d7ff; text-decoration-color: #00d7ff">None</span>, <span style="color: #00af00; text-decoration-color: #00af00">14</span>, <span style="color: #00af00; text-decoration-color: #00af00">14</span>, <span style="color: #00af00; text-decoration-color: #00af00">256</span>)           │     <span style="color: #00af00; text-decoration-color: #00af00">688,456</span> │
├────────────────────────────────────┼───────────────────────────────┼─────────────┤
│ levels_2 (<span style="color: #0087ff; text-decoration-color: #0087ff">Level</span>)                   │ (<span style="color: #00d7ff; text-decoration-color: #00d7ff">None</span>, <span style="color: #00af00; text-decoration-color: #00af00">7</span>, <span style="color: #00af00; text-decoration-color: #00af00">7</span>, <span style="color: #00af00; text-decoration-color: #00af00">512</span>)             │   <span style="color: #00af00; text-decoration-color: #00af00">5,170,608</span> │
├────────────────────────────────────┼───────────────────────────────┼─────────────┤
│ levels_3 (<span style="color: #0087ff; text-decoration-color: #0087ff">Level</span>)                   │ (<span style="color: #00d7ff; text-decoration-color: #00d7ff">None</span>, <span style="color: #00af00; text-decoration-color: #00af00">7</span>, <span style="color: #00af00; text-decoration-color: #00af00">7</span>, <span style="color: #00af00; text-decoration-color: #00af00">512</span>)             │   <span style="color: #00af00; text-decoration-color: #00af00">5,395,744</span> │
├────────────────────────────────────┼───────────────────────────────┼─────────────┤
│ norm (<span style="color: #0087ff; text-decoration-color: #0087ff">LayerNormalization</span>)          │ (<span style="color: #00d7ff; text-decoration-color: #00d7ff">None</span>, <span style="color: #00af00; text-decoration-color: #00af00">7</span>, <span style="color: #00af00; text-decoration-color: #00af00">7</span>, <span style="color: #00af00; text-decoration-color: #00af00">512</span>)             │       <span style="color: #00af00; text-decoration-color: #00af00">1,024</span> │
├────────────────────────────────────┼───────────────────────────────┼─────────────┤
│ pool (<span style="color: #0087ff; text-decoration-color: #0087ff">GlobalAveragePooling2D</span>)      │ (<span style="color: #00d7ff; text-decoration-color: #00d7ff">None</span>, <span style="color: #00af00; text-decoration-color: #00af00">512</span>)                   │           <span style="color: #00af00; text-decoration-color: #00af00">0</span> │
├────────────────────────────────────┼───────────────────────────────┼─────────────┤
│ head (<span style="color: #0087ff; text-decoration-color: #0087ff">Dense</span>)                       │ (<span style="color: #00d7ff; text-decoration-color: #00d7ff">None</span>, <span style="color: #00af00; text-decoration-color: #00af00">1000</span>)                  │     <span style="color: #00af00; text-decoration-color: #00af00">513,000</span> │
└────────────────────────────────────┴───────────────────────────────┴─────────────┘
</pre>




<pre style="white-space:pre;overflow-x:auto;line-height:normal;font-family:Menlo,'DejaVu Sans Mono',consolas,'Courier New',monospace"><span style="font-weight: bold"> Total params: </span><span style="color: #00af00; text-decoration-color: #00af00">11,995,428</span> (45.76 MB)
</pre>




<pre style="white-space:pre;overflow-x:auto;line-height:normal;font-family:Menlo,'DejaVu Sans Mono',consolas,'Courier New',monospace"><span style="font-weight: bold"> Trainable params: </span><span style="color: #00af00; text-decoration-color: #00af00">11,995,428</span> (45.76 MB)
</pre>




<pre style="white-space:pre;overflow-x:auto;line-height:normal;font-family:Menlo,'DejaVu Sans Mono',consolas,'Courier New',monospace"><span style="font-weight: bold"> Non-trainable params: </span><span style="color: #00af00; text-decoration-color: #00af00">0</span> (0.00 B)
</pre>



---
## Sanity check for Pre-Trained Weights


```python
img = keras.applications.imagenet_utils.preprocess_input(
    chelsea(), mode="torch"
)  # Chelsea the cat
img = ops.image.resize(img, (224, 224))[None,]  # resize & create batch
pred = model(img)
pred_dec = keras.applications.imagenet_utils.decode_predictions(pred)[0]

print("\n# Image:")
plt.figure(figsize=(6, 6))
plt.imshow(chelsea())
plt.show()
print()

print("# Prediction (Top 5):")
for i in range(5):
    print("{:<12} : {:0.2f}".format(pred_dec[i][1], pred_dec[i][2]))
```

<div class="k-default-codeblock">
```
Downloading data from https://storage.googleapis.com/download.tensorflow.org/data/imagenet_class_index.json
 35363/35363 ━━━━━━━━━━━━━━━━━━━━ 0s 0us/step
```
</div>
    
<div class="k-default-codeblock">
```
# Image:

```
</div>
    
![png](/img/examples/vision/image_classification_using_global_context_vision_transformer/image_classification_using_global_context_vision_transformer_24_1.png)
    


    
<div class="k-default-codeblock">
```
# Prediction (Top 5):
Egyptian_cat : 0.72
tiger_cat    : 0.04
tabby        : 0.03
crossword_puzzle : 0.01
panpipe      : 0.00

```
</div>
# Fine-tune **GCViT** Model

In the following cells, we will fine-tune **GCViT** model on Flower Dataset which
consists `104` classes.

### Configs


```python
# Model
IMAGE_SIZE = (224, 224)

# Hyper Params
BATCH_SIZE = 32
EPOCHS = 5

# Dataset
CLASSES = [
    "dandelion",
    "daisy",
    "tulips",
    "sunflowers",
    "roses",
]  # don't change the order

# Other constants
MEAN = 255 * np.array([0.485, 0.456, 0.406], dtype="float32")  # imagenet mean
STD = 255 * np.array([0.229, 0.224, 0.225], dtype="float32")  # imagenet std
AUTO = tf.data.AUTOTUNE
```

---
## Data Loader


```python

def make_dataset(dataset: tf.data.Dataset, train: bool, image_size: int = IMAGE_SIZE):
    def preprocess(image, label):
        # for training, do augmentation
        if train:
            if tf.random.uniform(shape=[]) > 0.5:
                image = tf.image.flip_left_right(image)
        image = tf.image.resize(image, size=image_size, method="bicubic")
        image = (image - MEAN) / STD  # normalization
        return image, label

    if train:
        dataset = dataset.shuffle(BATCH_SIZE * 10)

    return dataset.map(preprocess, AUTO).batch(BATCH_SIZE).prefetch(AUTO)

```

### Flower Dataset


```python
train_dataset, val_dataset = tfds.load(
    "tf_flowers",
    split=["train[:90%]", "train[90%:]"],
    as_supervised=True,
    try_gcs=False,  # gcs_path is necessary for tpu,
)

train_dataset = make_dataset(train_dataset, True)
val_dataset = make_dataset(val_dataset, False)
```

<div class="k-default-codeblock">
```
Downloading and preparing dataset 218.21 MiB (download: 218.21 MiB, generated: 221.83 MiB, total: 440.05 MiB) to /root/tensorflow_datasets/tf_flowers/3.0.1...

Dl Completed...:   0%|          | 0/5 [00:00<?, ? file/s]

Dataset tf_flowers downloaded and prepared to /root/tensorflow_datasets/tf_flowers/3.0.1. Subsequent calls will reuse this data.

```
</div>
### Re-Build Model for Flower Dataset


```python
# Re-Build Model
model = GCViT(**config, num_classes=104)
inp = ops.array(np.random.uniform(size=(1, 224, 224, 3)))
out = model(inp)

# Load Weights
ckpt_path = keras.utils.get_file(ckpt_link.split("/")[-1], ckpt_link)
model.load_weights(ckpt_path, skip_mismatch=True)

model.compile(
    loss="sparse_categorical_crossentropy", optimizer="adam", metrics=["accuracy"]
)
```

<div class="k-default-codeblock">
```
/usr/local/lib/python3.10/dist-packages/keras/src/saving/saving_lib.py:269: UserWarning: A total of 1 objects could not be loaded. Example error message for object <Dense name=head, built=True>:
```
</div>
    
<div class="k-default-codeblock">
```
Layer 'head' expected 2 variables, but received 0 variables during loading. Expected: ['kernel', 'bias']
```
</div>
    
<div class="k-default-codeblock">
```
List of objects that could not be loaded:
[<Dense name=head, built=True>]
  warnings.warn(msg)

```
</div>
### Training


```python
history = model.fit(
    train_dataset, validation_data=val_dataset, epochs=EPOCHS, verbose=1
)
```

<div class="k-default-codeblock">
```
Epoch 1/5
 104/104 ━━━━━━━━━━━━━━━━━━━━ 153s 581ms/step - accuracy: 0.5140 - loss: 1.4615 - val_accuracy: 0.8828 - val_loss: 0.3485
Epoch 2/5
 104/104 ━━━━━━━━━━━━━━━━━━━━ 7s 69ms/step - accuracy: 0.8775 - loss: 0.3437 - val_accuracy: 0.8828 - val_loss: 0.3508
Epoch 3/5
 104/104 ━━━━━━━━━━━━━━━━━━━━ 7s 68ms/step - accuracy: 0.8937 - loss: 0.2918 - val_accuracy: 0.9019 - val_loss: 0.2953
Epoch 4/5
 104/104 ━━━━━━━━━━━━━━━━━━━━ 7s 68ms/step - accuracy: 0.9232 - loss: 0.2397 - val_accuracy: 0.9183 - val_loss: 0.2212
Epoch 5/5
 104/104 ━━━━━━━━━━━━━━━━━━━━ 7s 68ms/step - accuracy: 0.9456 - loss: 0.1645 - val_accuracy: 0.9210 - val_loss: 0.2897

```
</div>
---
## Reference

* [gcvit-tf - A Python library for GCViT with TF2.0](https://github.com/awsaf49/gcvit-tf)
* [gcvit - Official codebase for GCViT](https://github.com/NVlabs/GCVit)
