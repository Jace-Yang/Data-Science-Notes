## RNN模型

### 语句表示

对于一个序列数据 x，用符号 x⟨t⟩来表示这个数据中的第 tt个元素，用 y⟨t⟩来表示第 t个标签，用 Tx和 Ty来表示输入和输出的长度。对于一段音频，元素可能是其中的几帧；对于一句话，元素可能是一到多个单词。

想要表示一个词语，需要先建立一个**词汇表（Vocabulary）**，或者叫**字典（Dictionary）**。将需要表示的所有词语变为一个列向量，可以根据字母顺序排列，然后根据单词在向量中的位置，用 **one-hot 向量（one-hot vector）**来表示该单词的标签：将每个单词编码成一个 R|V|×1向量，其中 |V|是词汇表中单词的数量。一个单词在词汇表中的索引在该向量对应的元素为 1，其余元素均为 0。one-hot 向量是最简单的词向量。它的**缺点**是，由于每个单词被表示为完全独立的个体，因此单词间的相似度无法体现。

### 正向传播

$$
W_a=[W_{aa},W_{ax}]
$$

$$
a^{<t>}=g_1(W_a[a^{<t-1>};x^{<t>}]+b_a)
$$

$$
\hat y^{<t>}=g_2(W_{ya}a^{<t>}+b_y)
$$

### 反向传播

单个位置上（或者说单个时间步上）某个单词的预测值的损失函数采用**交叉熵损失函数**，将单个位置上的损失函数相加，得到整个序列的成本函数

![img](https://i.loli.net/2021/08/04/VMPcZnrig6opvI1.png)

通过RNN来预测一个序列的值通常需要两步，第一步是构建一个语言模型，第二部是通过采样来构建序列

### 语言模型

建立语言模型所采用的训练集是一个大型的**语料库（Corpus）**，指数量众多的句子组成的文本。建立过程的第一步是标记化，即建立字典；然后将语料库中的每个词表示为对应的 one-hot 向量。第二步是将标志化后的训练集用于训练 RNN

### 采样

在训练好一个语言模型后，可以通过**采样（Sample）**新的序列来了解这个模型中都学习到了一些什么

在第一个时间步输入 a⟨0⟩a⟨0⟩和 x⟨1⟩x⟨1⟩为零向量，输出预测出的字典中每个词作为第一个词出现的概率，根据 softmax 的分布进行随机采样（`np.random.choice`），将采样得到的 ŷ ⟨1⟩作为第二个时间步的输入 x⟨2⟩。以此类推，直到采样到 EOS，最后模型会自动生成一些句子，从这些句子中可以发现模型通过语料库学习到的知识。

## 改进RNN模型

传统的RNN存在梯度消失问题，不善于捕捉语句长期的依赖关系，通过GRU与LSTM模型可以解决长期依赖的问题

### GRU

GRU 单元有一个新的变量称为 c，代表记忆细胞，其作用是提供记忆的能力，记住例如前文主语是单数还是复数等信息。

更新门$\Gamma_u$代表是否要更新参数，在0-1取值；相关门$\Gamma_r$表示 c̃ ⟨t⟩和 c⟨t⟩的相关性
$$
\Gamma_u=\sigma(W_u[c^{<t-1>},x^{<t>}]+b_u)
$$

$$
\Gamma_r=\sigma(W_r[c^{<t-1>},x^{<t>}]+b_r)
$$

$$
\tilde c^{<t>}=tanh(W_c[\Gamma_r*c^{<t-1>},x^{<t>}]+b_c)
$$

$$
a^{<t>}=c^{<t>}=\Gamma_u * \tilde c^{<t-1>}+(1-\Gamma_u)*c^{<t-1>}
$$

![img](https://i.loli.net/2021/08/04/cDJTWHUgOPkS2ua.png)

### LSTM

相比于GRU，LSTM新引入了遗忘门$\Gamma_f$和输出门$\Gamma_o$，去掉了相关门$\Gamma_r$
$$
\Gamma_u=\sigma(W_u[a^{<t-1>},x^{<t>}]+b_u)
$$

$$
\Gamma_f=\sigma(W_f[a^{<t-1>},x^{<t>}]+b_f)
$$

$$
\Gamma_o=\sigma(W_o[a^{<t-1>},x^{<t>}]+b_o)
$$

$$
\tilde c^{<t>}=tanh(W_c[\Gamma_r*c^{<t-1>},x^{<t>}]+b_c)
$$

$$
c^{<t>}=\Gamma_u * \tilde c^{<t-1>}+\Gamma_f*c^{<t-1>}
$$

$$
a^{<t>}=\Gamma_o*tanh(c^{<t>})
$$

![ST](https://i.loli.net/2021/08/04/3atNUyejpIVchEF.png)

### BRNN双向循环神经网络

**双向循环神经网络（Bidirectional RNN，BRNN）**可以在序列的任意位置使用之前和之后的数据。**缺点**是需要完整的序列数据，才能预测任意位置的结果。

![img](https://i.loli.net/2021/08/04/lQj8AhFHwJYkEoV.png)

### DRNN深度循环神经网络

循环神经网络的每个时间步上也可以包含多个隐藏层

![img](https://i.loli.net/2021/08/04/NHiFXYjkBSVM49L.png)

## 词嵌入

### 概念

使用OneHot离散编码训练的RNN模型有两个缺点，分别是参数量大以及不同词之间正交，因此引入了词嵌入机制，将OneHot向量映射到一个低位空间上。

**词嵌入**是 NLP 中语言模型与表征学习技术的统称，概念上而言，它是指把一个维数为所有词的数量的高维空间（one-hot 形式表示的词）“嵌入”到一个维数低得多的连续向量空间中，每个单词或词组被映射为实数域上的向量。

![img](https://i.loli.net/2021/08/04/Bp8QcoewPVYhF9C.png)

用词嵌入做迁移学习可以降低学习成本，提高效率。其步骤如下：

1. 从大量的文本集中学习词嵌入，或者下载网上开源的、预训练好的词嵌入模型；
2. 将这些词嵌入模型迁移到新的、只有少量标注训练集的任务中；
3. 可以选择是否微调词嵌入。当标记数据集不是很大时可以省下这一步。

训练过程：

将语料库中的某些词作为目标词，以目标词的部分上下文作为输入，Softmax 输出的预测结果为目标词。嵌入矩阵 E 和 w、b 为需要通过训练得到的参数。这样，在得到嵌入矩阵后，就可以得到词嵌入后生成的词向量。

### word2vec

![img](https://i.loli.net/2021/08/04/CbJ2PA1qd4skVv9.png)

word2vec的目标是通过一个语料库来学习词嵌入矩阵，通常有两种Skip-gram和CBOW两种方法。word2vec模型背后的基本思想是对出现在上下文环境里的词进行预测。对于每一条输入文本，我们选取一个上下文窗口和一个中心词，并基于这个中心词去预测窗口里其他词出现的概率。因此，word2vec模型可以方便地从新增语料中学习到新增词的向量表达，是一种高效的在线学习算法

**CBOW**是从原始语句推测目标字词；而**Skip-Gram**正好相反，是从目标字词推测出原始语句。**CBOW**对小型数据库比较合适，而**Skip-Gram**在大型语料中表现更好。

• Skip-gram：根据词预测目标上下文

从左到右是 One-hot 向量，乘以 center word 的矩阵 W 于是找到词向量，乘以另一个 context word 的矩阵 W′ 得到对每个词语的“相似度”，对相似度取 Softmax 得到概率，与答案对比计算损失

<img src="https://i.loli.net/2021/08/04/SGHnm1TwRgBVNyF.png" alt="img" style="zoom:60%;" />

• CBOW：根据上下文预测目标词

CBOW与Skip-gram相反，使用上下文的词来预测目标词，对输出相似度取 Softmax 得到概率，并计算loss

<img src="https://i.loli.net/2021/08/04/nvJEei4abws3Rkr.jpg" alt="img" style="zoom:20%;" />

由于在OneHot向量的纬度可能很大，使用Softmax输出结果是要计算所有指数项的和，使用分层softmax和负采样的技术可以加速

• 分层Softmax：将 N 分类问题变成 log(N)次二分类

在实践中分级**softmax**分类器不会使用一棵完美平衡的分类树或者说一棵左边和右边分支的词数相同的对称树。实际上，分级的**softmax**分类器会被构造成常用词在顶部，然而不常用的词像**durian**会在树的更深处，因为更常见的词会更频繁，所以可能只需要少量检索就可以获得常用单词像**the**和**of**。

• 负采样：预测总体类别的一个子集

生成数据的方式是我们选择一个上下文词，再选一个目标词，这就是正样本，给定标签为1；然后我们将用相同的上下文词，再从字典中选取随机的词，并标记0，这些就会成为负样本

![img](https://i.loli.net/2021/08/04/5SEPxn9AGVq2J3w.png)

### 情感分类

情感分类是指分析一段文本对某个对象的情感是正面的还是负面的，实际应用包括舆情分析、民意调查、产品意见调查等等。情感分类的问题之一是标记好的训练数据不足。但是有了词嵌入得到的词向量，中等规模的标记训练数据也能构建出一个效果不错的情感分类器。

使用 RNN 能够实现一个效果更好的情感分类器，相比于普通的基于平均值的情感分类器，RNN的情感分类器考虑了单词时序

![img](https://i.loli.net/2021/08/04/FAwhLygaZbrdlq1.png)

## 序列模型

**Seq2Seq（Sequence-to-Sequence）**模型能够应用于机器翻译、语音识别等各种序列到序列的转换问题。一个 Seq2Seq 模型包含**编码器（Encoder）**和**解码器（Decoder）**两部分，它们通常是两个不同的 RNN，将编码器的输出作为解码器的输入，由解码器负责输出正确的翻译结果。

![img](https://i.loli.net/2021/08/04/yHekzjfKh6taDw2.png)

这种编码器-解码器的结构也可以用于图像描述。将 AlexNet 作为编码器，最后一层的 Softmax 换成一个 RNN 作为解码器，网络的输出序列就是对图像的一个描述。

![img](https://i.loli.net/2021/08/04/tTBlm68bDk2Vc93.png)

• 贪心搜索Greedy Search：生成第一个词的分布以后，它将会根据你的条件语言模型挑选出最有可能的第一个词进入你的机器翻译模型中，在挑选出第一个词之后它将会继续挑选出最有可能的第二个词，然后继续挑选第三个最有可能的词

• 集束搜索Beam Search：考虑每个时间步多个可能的选择。设定一个**集束宽（Beam Width）**B，代表了解码器中每个时间步的预选单词数量。例如 B=3B=3，则将第一个时间步最可能的三个预选单词及其概率值 P(ŷ ⟨1⟩|x)保存到计算机内存，以待后续使用。第二步中，分别将三个预选词作为第二个时间步的输入，得到 P(ŷ ⟨2⟩|x,ŷ ⟨1⟩)P(y^⟨2⟩|x,y^⟨1⟩)。在每次迭代时选取集束宽B个结果，依次迭代。

在做Beam Search时，多个条件概率相乘可能造成数值下溢，通过取对数方法缓解

Beam Search倾向于选择句长更短的句子，我们可以做长度标准化来解决这个问题

![img](https://i.loli.net/2021/08/04/JPOM4bnYzKqX9DG.png)

### Attention机制

注意力模型的一个示例网络结构如下图所示。其中，底层是一个双向循环神经网络（BRNN），该网络中每个时间步的激活都包含前向传播和反向传播产生的激活。顶层是一个“多对多”结构的循环神经网络，第 t 个时间步的输入包含该网络中前一个时间步的激活 s⟨t−1⟩、输出 y⟨t−1⟩以及底层的 BRNN 中多个时间步的激活 c

![img](https://i.loli.net/2021/08/04/BfT2ScuOlW3DY1V.png)

![img](https://i.loli.net/2021/08/04/8TaCWgzYcHkMu1x.png)

### 语音识别

在语音识别任务中，输入是一段以时间为横轴的音频片段，输出是文本。

语音识别系统可以用注意力模型来构建，在输入音频的不同时间帧上，可以用一个注意力模型，来输出文本描述

![img](https://i.loli.net/2021/08/04/IbzHhrMq9UwGd25.png)

**CTC**损失函数

由于输入是音频数据，使用 RNN 所建立的系统含有很多个时间步，且输出数量往往小于输入。因此，不是每一个时间步都有对应的输出。CTC 允许 RNN 生成下图蓝字所示的输出，并将两个空白符中重复的字符折叠起来，再将空白符去掉，得到最终的输出文本。

![img](https://i.loli.net/2021/08/04/dg2W549oVLIzB6r.png)