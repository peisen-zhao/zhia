# 屠榜的Transformer

---

7.17

## Transformer简介

1. 应用了自注意力机制来建模全局的上下文信息，缺点：计算量大，改进方法通常为轻量化的注意力，分治法

2. 架构灵活，很难应用在小规模数据集上进行训练，缺乏泛化性能

3. 模型自适应性

原生Transformer模型由编码器和解码器组成的序列到序列模型解码器端的自注意力模块作用是用于避免当前位置关注到后续位置的信息。

Transformer采用了带有**Query-Key-Value**(QKV)的自**注意力模块**。其中，Q指的是查询向量，K指的是关键字向量，而V指的是内容向量。Q与K点积可得到关联矩阵，经过**SoftMax**得到权重图，权重图叠加到V可得到不同区域加权。

采取多头注意力，多头注意力结果拼接作为MSA最终的输出，

- **Self-attention**. 在Transformer的编码器中，我们设置，这里为上一层的输出。
- **Masked Self-attention**. 在Transformer的解码器中，自注意力受到限制，使得每个位置的查询只能关注到包括该位置及之前位置的所有键值对。常规做法是将掩码矩阵(mask matrix)添加到注意力分数上，其中非法位置采用进行遮挡。这一类注意力方法也经常被称为自回归(autogressive)或者因果(causal)注意力。
- **Cross-attetnion**. 查询是从前一个（解码器）层的输出投影所获得的，而键和值是使用编码器的输出投影得到的。

**位置编码**

>由于Tranformer完全摒弃了RNN和CNN结构，导致它对位置信息是一无所知的（尤其是编码器）。因此，需要额外的位置表示来对Token的排序进行建模。

Transformer可使用以下三种方式

- Encoder-Deocder

- Encoder only

- Decoder only

  影响Transformer的复杂度的两个因素为隐藏层D的维度和输入序列长度T

---

7.20

## VIsion Transformer（屠榜视觉领域）

### **self-attention**

![self-attention](img\self-attention1.jpg)

Xn 乘一个矩阵得到embedding ai，在将ai 乘三个不同的Wq,Wk,Wv得到q，k，v，q与v做attention即qk点积除以维度的根号

![self-attention](img\self-attention2.jpg)

进行softmax操作

![self-attention](img\self-attention3.jpg)

与vi相乘得到bi，此时bi用到了整个sequence的信息

>如果要考虑**local**的information，则只需要学习出相应的a1,i = 0，b1 就不再带有那个对应分支的信息了；如果要考虑**global**的information，则只需要学习出相应的a1,i != 0，b1 就带有全部的对应分支的信息了。

self-attention就是一堆矩阵乘法，可以实现GPU加速

处理Sequence:RNN可以考虑到全局，但不能做到并行化

cnn可以并行化，但不能考虑全局，堆叠filter可以解决

一种新的layer，叫**self-attention**，它的输入和输出和RNN是一模一样的，输入一个sequence，输出一个sequence，它的每一个输出都看过了整个的输入sequence，这一点与bi-directional RNN相同。但是神奇的地方是：它的每一个输出可以并行化计算。

Embedding 是一个将离散变量转为连续向量表示的一个方式。有以下三个主要目的

1. 在 embedding 空间中**查找最近邻**，这可以很好的用于根据用户的兴趣来进行推荐。
2. 作为监督性学习任务的**输入**。
3. 用于**可视化**不同离散变量之间的关系。

与one-hot对比：one-hot由N-1个0与单个1组成的vector，所显现的缺点有

1. 对于具有非常多类型的类别变量，变换后的向量维数过于巨大，且过于稀疏。
2. 映射之间完全独立，并不能表示出不同类别之间的关系。

考虑到这两个问题，表示类别变量的理想解决方案则是我们是否可以通过较少的维度表示出每个类别，并且还可以一定的表现出不同类别变量之间的关系，这也就是 embedding 出现的目的。

embedding最酷的一点是可视化，当然要我们能够观察，需要通过降维技术来达到 2 维或 3 维。最流行的降维技术是：t-Distributed Stochastic Neighbor Embedding (TSNE)。

### **Multi-head self-attention**

![图片](img\Mutli self-attention.jpg)

### **Positional Encoding：** 位置编码

给每一个位置规定一个表示位置信息的向量ei，让它与ai加在一起之后作为新的ai参与后面的运算过程，但是这个向量ei是由人工设定的，而不是神经网络学习出来的。每一个位置都有一个不同的ei。

### **Transformer**

![transformer](img\transformer.jpg)

Masked Multi-Head Self-attention，masked的意思是使attention只会attend on已经产生的sequence，

## Dynamic ViT ：动态Token稀疏化ViT 论文解读



论文地址: Transformer Interpretability Beyond Attention Visualization

[源码地址](https://github.com/hila-chefer/Transformer-Explainability)

 **point:** 动态token稀疏化视觉transformer， 分层剪枝 VIT

待解决：交叉验证，mix up， 