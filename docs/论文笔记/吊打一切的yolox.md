## 7.20 公众号阅读：吊打一切yolo的YOLOX

### abstract

相对于yolov3的调整：添加了**EMA**权值更新、**cosine学习率机制**、**IoU损失**、**IoU感知分支**。采用**BCE损失**训练cls与obj分支，**IoU损失**训练reg分支。这些广义的训练技巧与YOLOX的关键改进是正交的

**yolov3与yolox**

![yolov3](img\yolov3.jpg)

### improvement

改进一：检测头解耦

改进二：采取Mosaic 与 Mixup 两种数据增广方法 （由于采取了更强的数据增广，我们发现ImageNet的预训练毫无意义，因此采用从头训练）

改进三：Anchor-free  将每个位置的预测从3下降为1并直接*预测四个值*，即**两个offset和高宽**。参考[**FCOS**](), 将每个目标中心定位正样本并预定义一个尺度范围以便于对每个目标指派**FPN**

anchor的缺点：

>1. anchor集合存在数据相关性，泛化性能较差。
>2. anchor机制提高了检测头的复杂性

改进四：Multi positives：参考**FCOS**   *对每个目标赋予多个正样本？？？*，本文简单的赋予中心3x3区域为正样本。

改进五：SimOTA 先进的标签分配 **OTA研究？** 选择OTA为候选标签分配策略 （将其简化为动态top-k策略以得到一个近似解“SimOTA” 这不仅可以降低训练时间，同时可以避免额外的超参问题。

改进六（但未包含）：参考**PSS**添加额外的卷积层，one-to-one 标签分配和stop gradient,使其进化成端到端形式

使用**深度卷积** 构建YOLOX-nano,训练小模型时移除MixUp,弱化Mosaic可提高模型性能