### BN踩坑记--谈一下Batch Normalization的优缺点和适用场景

这个问题没有定论，很多人都在探索，所以只是聊一下我自己的理解，顺便为讲 layer-norm做个引子。

BN的理解重点在于它是针对整个Batch中的样本在同一维度特征在做处理。

在MLP中，比如我们有10行5列数据。5列代表特征，10行代表10个样本。是对第一个特征这一列（对应10个样本）做一次处理，第二个特征（同样是一列）做一次处理，依次类推。

在CNN中扩展，我们的数据是N·C·H·W。其中N为样本数量也就是batch_size，C为通道数，H为高，W为宽，BN保留C通道数，在N,H,W上做操作。比如说把第一个样本的第一个通道的数据，第二个样本第一个通道的数据.....第N个样本第一个通道的数据作为原始数据，处理得到相应的均值和方差。

#### BN有两个优点。

第一个就是可以解决内部协变量偏移，简单来说训练过程中，各层分布不同，增大了学习难度，BN缓解了这个问题。当然后来也有论文证明BN有作用和这个没关系，而是可以使损失平面更加的平滑，从而加快的收敛速度。

第二个优点就是缓解了梯度饱和问题（如果使用sigmoid激活函数的话），加快收敛。

#### BN的缺点：
第一个，batch_size较小的时候，效果差。这一点很容易理解。BN的过程，使用 整个batch中样本的均值和方差来模拟全部数据的均值和方差，在batch_size 较小的时候，效果肯定不好。

第二个缺点就是 BN 在RNN中效果比较差。这一点和第一点原因很类似，不过我单挑出来说。

首先我们要意识到一点，就是RNN的输入是长度是动态的，就是说每个样本的长度是不一样的。

举个最简单的例子，比如 batch_size 为10，也就是我有10个样本，其中9个样本长度为5，第10个样本长度为20。

那么问题来了，前五个单词的均值和方差都可以在这个batch中求出来从而模型真实均值和方差。但是第6个单词到底20个单词怎么办？

只用这一个样本进行模型的话，不就是回到了第一点，batch太小，导致效果很差。

第三个缺点就是在测试阶段的问题，分三部分说。

首先测试的时候，我们可以在队列里拉一个batch进去进行计算，但是也有情况是来一个必须尽快出来一个，也就是batch为1，这个时候均值和方差怎么办？

这个一般是在训练的时候就把均值和方差保存下来，测试的时候直接用就可以。那么选取效果好的均值和方差就是个问题。

其次在测试的时候，遇到一个样本长度为1000的样本，在训练的时候最大长度为600，那么后面400个单词的均值和方差在训练数据没碰到过，这个时候怎么办？

这个问题我们一般是在数据处理的时候就会做截断。

还有一个问题就是就是训练集和测试集的均值和方差相差比较大，那么训练集的均值和方差就不能很好的反应你测试数据特性，效果就回差。这个时候就和你的数据处理有关系了。

#### BN使用场景

对于使用场景来说，BN在MLP和CNN上使用的效果都比较好，在RNN这种动态文本模型上使用的比较差。至于为啥NLP领域BN效果会差，Layer norm 效果会好，下一个文章会详细聊聊我的理解。



列一下参考资料：

模型优化之Batch Normalization - 大师兄的文章 - 知乎 https://zhuanlan.zhihu.com/p/54171297

这个文章写的很好，推荐，从BN的特点（ICS/梯度饱和），训练，测试以及损失函数平滑都讲了一下。

李宏毅- Batch Normalization  https://www.bilibili.com/video/av16540598/

大佬的讲解视频，不解释，推荐

各种Normalization - Mr.Y的文章 - 知乎 https://zhuanlan.zhihu.com/p/86765356

这个文章关于BN在CNN中使用的讲解很好，推荐一下。