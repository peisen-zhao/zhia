

# HR-Net 论文解读（high-resolution

## 摘要

**通过聚合多个平行特征层卷积的方法保证图像的高分辨率**，并应用在现有的RCNN网络中，均取得了不错的优化，大幅度提升小目标检测性能。

## **Architecture:**

![](img\HR-Net.png)

> The multi-resolution group convolution is a simple extension of the group convolution, which divides the input channels into several subsets of channels and performs a regular convolution over each subset over different spatial resolutions separately.

将输入通道划分为多个子集，并将这些子集在不同空间分辨率下进行正则卷积（regular convolution)

**multi-resolution convolution**

![multi-resolution convolution](img\multi-resolution convolution.png)

>The input channels are divided into several subsets, and the output channels are also divided into several subsets. **The input and output subsets are connected in a fully-connected fashion, and each connection is a regular convolution**. *Each subset of output channels is a summation of the outputs of the convolutions over each subset of input channels*

卷积的输入和输出以全连接的方式连接

## 差异

- 每个subset都是在不同分辨率的情况下
- 输入与输出之间的全连接需要处理，下采样通过s=2,k=3的卷积实现，上采样通过线性插值的方法实现

