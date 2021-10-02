## 9.14 TPH-YOLOV5

作者单位:  北京航空航天大学

论文链接：https://arxiv.org/abs/2108.11539

笔记参考公众号【集智书童】: https://mp.weixin.qq.com/s/1itG2bVUbuLC0g7mX5VLGA

### 1. Data Augmentation:

#### 1.1 全局数据增强

- **Photometric**：随机亮度、对比度、色调、饱和度、噪声
- **Geometric**: 随机扩展、裁剪、翻转

#### 1.2 特殊数据增强

- **Mixup**：随机两张图片加权求和，标签也对应加权求和
- **CutMix**： 使用另一个图像的区域覆盖被遮挡的区域
- **Mosaic**：拼接4幅图像

### 2.Multi-Model Ensemble Method

由于权重更新的随机性，每次训练都会带来一定偶然性，使得模型有了一个**高方差**。

减少模型方差的一个成功方法是训练多个模型而不是单一模型，并结合这些模型的预测。

针对不同的目标检测模型，有3种不同的**ensemble boxes**方法:

- NMS:非极大值抑制，对于重合的框给定重合度阈值IOU，大于阈值取最高置信度。
- Soft-NMS: 它根据IoU值对相邻边界box的置信度设置衰减函数
- Weighted Boxed Fusion(WBF):将所有框合并形成最终结果。

### 3. Object Detection

- BackBone： VGG,ResNet,DenseNet,MobileNet,EfficientNet,CSPDarkNet53,**Swin-Transformer**
- Neck: FPN,PANet,NaS-FPN,BiFPN,ASFF,SAM,SPP,ASPP,RFB,CBAM
- Head: One-Stage Two-Stage

### 4. Details

![](img\THP-YOLOV5.jpg)

架构：

![](img\THP-YOLOV5 arch.jpg)

#### 增加微小物体检测头

（head 1)由高分辨feature-map生成

#### Transformer encoder block

与CSPDarknet53中原有的bottleneck blocks相比，作者认为Transformer encoder block可以捕获全局信息和丰富的上下文信息。

每个Transformer encoder block包含2个子层。第1子层为multi-head attention layer，第2子层(MLP)为全连接层。每个子层之间使用残差连接。Transformer encoder block增加了捕获不同局部信息的能力。它还可以利用自注意力机制来挖掘特征表征潜能。

#### Convolutional block attention module (CBAM)

#### Self-trained classifier

通过裁剪ground-truth边界框并将每个图像patch的大小调整为6464来构建训练集。然后选择ResNet18作为分类器网络。

注：添加检测头增加计算量，采用transformer encoder blocks减少计算量。

