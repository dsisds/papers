# RNN基本概念

## 一 什么是RNN呢?

RNN主要是利用序列信息。在传统神经网络中的输入是彼此独立的（输出也是）。但这种独立对于很多任务来说是不利的。如果你想预测一个句子中的下一个单词是什么，最好是知道这个单词之前有哪些单词。RNN中的R是recurrent, 是周期性, 反复的意思, 表示会对句子中的每一个单词用相同的方法处理，让输出基于之前的计算结果。理论上RNN可以利用任意长度的句子，但实际中我们只关注最近的几步。一个典型的RNN示意图如下:

![rnn](images/rnn.jpg)

<!-- more -->

上图是将左侧的RNN展开成完整的网络。展开的意思就是把需要关注的序列对应的网络写出来。比如对于一个句子，我们只关心5个单词，那么RNN网络就被展开成一个5层的神经网络，每一层对应序列中的一个单词。网络中的一些规则定义如下:

* $x_t$表示在t时刻(step)的输入。比如$x_1$是句子中第二个单词对应的one-hot向量(向量中只有1个1，其余都是0)。
* $s_t$是t时刻(step)的隐状态，是网络中作为"记忆"的存在。$s_t$是基于前一个隐状态和当前的输入计算出来的: $s_t=f(Ux_t+W{s_{t-1}})$ 。$f()$通常是非线性函数（如$tanh$, $ReLU$)。初始隐状态$s_{-1}$用于计算第一个隐状态，通常$s_{-1}$被初始化为全0。
* $o_t$是t时刻(step)的输出。比如要预测句子的下一个单词是什么，$o_t$就我们词典中每一个单词出现的概率，$o_t=softmax(Vs_t)$

有些地方需要注意下:

* $s_t$可以被理解为网络的记忆。$s_t$用于捕捉之前的所有时刻发生了啥。本时刻的输出$o_t$完全取决于t时刻的记忆。在实际中会很复杂，因为$s_t$不能捕捉太多之前的步骤。
* 不同于传统的深度神经网络（每层拥有不同的参数），RNN在所有层间（又叫step，跟上面的时刻对应)共享参数$(U, V, W)$ 。实际上我们是在各层执行相同的事情，只是输入不同罢了。这就极大的减少了我们需要学习的参数个数。
* 上面的图中每个step都有输出，但是在实际中不一定每个step都需要输出。比如我们想预测一个句子的感情色彩，我们只关系这个句子的最后输出。同样的，也不是每个step上都需要有输入。RNN的主要特点就在于其隐状态，是用于捕获一些具有顺序特点的信息而已。

## 二 RNN可以做什么呢？

很多NLP任务都可以利用RNN。RNN中最常用的是LSTMs，这个比原始RNNs在捕获有长依赖信息时更好用。LSTMs与RNN从本质上相同，只是在计算隐状态时不同而已。下面是一些RNN的应用例子。

### 1 语言模型以及生成文字

给出一系列的词语，我们希望预测在给定前一个词语时下一个词语的概率。语言模型给了我们可以评估一个句子确实是一个句子的尺子，这在机器翻译中很重要，因为高概率的句子通常是正确的结果。预测下一个词的副产品就是我们得到了一个生成模型，即允许我们生成新的文本。并且根据我们喂给模型的训练数据，几乎可以生成任何相应特点的新句子。在语言模型中个，输入一般是一系列的词（编码成one-hot形式的向量），输出就是预测的词。在训练时，我们设置$o_t=x_{t+1}$，因为在t step时的真正输出是$x_{t+1}$。

### 2 机器翻译

![机器翻译](images/language_translation.png)

机器翻译跟语言模型很相似，输入都是词语的序列，比如原文是德文，我们希望输出是英文。关键的不同是输出是在我们见过了所有的输入后才有的，因为翻译后的句子的第一个词是需要了解了整个句子原文的。

### 3 语音识别(Speech recognition)

输入声音信号，预测如何根据语音将他们转换成文字。

### 4 生成图片描述

与CNN一起使用, RNN模型作为整体模型的一部分生成未标注图片的描述。组合而成的整体模型可以将从图片里面找到的物品和文字一一对应起来。

![captioning](images/captioning.png)

图片来源是neuraltalk的大牛Karpathy那里借的:  
<http://cs.stanford.edu/people/karpathy/deepimagesent/>

## 三 训练RNN

训练RNN与训练传统NN类似。同样使用BP算法, 但有一些小变化。因为网络中各层（time steps）共用相同的参数，因此每层输出的梯度不仅与当前的层有关，还有之前的各层。比如要计算t＝4的梯度，我们需要前面3层的梯度，把他们加起来。这个叫做BPTT`Backpropagation Through Time`。但是原始RNN使用BPTT训练长依赖关系时时效果很不好，因为著名的`消失的梯度`问题。LSTM就是解决这个训练长依赖时的难题被设计出来的。

后面会有训练RNN的具体方法及代码。

## 四 RNN的扩展

经年累月，伟大的研究者开发了更加复杂的RNN去解决原生RNN的短处。简单介绍下先。

### 1 双向RNN

主体思想是t时刻输出不仅仅依赖于之前的步骤，还与未来有关。比如想要预测一句话中缺失的单词，考虑句子的上下文显然是更加有效的。双向RNN很简单，仅仅是两个RNN摞在一起，而输出就是基于两个RNN的隐状态。

![](images/bidirectional-rnn.png)

### 2 深度双向RNN

与双向RNN类似，区别仅仅是在每层中有多个子层而已。实践中这个模型学习能力更好，但也需要更多的训练数据。

![](images/deep_bi_rnn.png)

### 3 LSTM网络

LSTM最近大热。它与RNN并没有本质的区别，仅仅区别于计算隐状态的方法。LSTM中的记忆体被称作cells，可以被理解为一个黑盒，输入是前一个状态$h_{t-1}$和当前的输入$x_t$。cells内部决定记住哪些（以及删掉哪些）。因此它可以将之前的状态，当前的记忆，以及当前的输入合并在一起。在实践中，这样的cells可以很好的捕获长依赖关系。





## 参考资料

[1] <http://www.wildml.com/2015/09/recurrent-neural-networks-tutorial-part-1-introduction-to-rnns/>  
