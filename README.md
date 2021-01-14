[toc]

# 细粒度特征学习的多交互注意力网络MIAN

## 背景

> ctr预估中常利用户历史行为序列来挖掘用户兴趣，建模用户历史行为序列中物品与候选物品之间的关系，从而给用户进行推荐，如阿里提出的DIN和DIEN。但这些方法着重于用户的历史行为序列，很少关注上下文信息，这样可能存在以下问题：
>
> 1）大多数现有的方法主要从用户历史行为中挖掘用户的兴趣，但用户经常会有与过往行为不同的新的需求出现；
>  2）部分用户的历史行为大都发生在很久以前，而近期行为较少，仅仅依赖于历史行为建模容易推荐出与用户当前兴趣不符的“过时”物品；
>  3）上下文因素同样重要，如夏天相较于冬天，T恤更容易被进行推荐。因此上下文因素和候选物品的特征交互同样需要考虑。
>
> 上述所说的三点问题可以由下图表示：

![image](https://github.com/ShaoQiBNU/CTR_MIAN/blob/main/img/1.png)

> 在推荐系统中，有众多的用户特征和上下文特征可以挖掘，使用这些特征可以有效缓解上述的问题，特别是当用户历史行为较少的情况下。如上图中，如果候选物品是机械键盘，这与用户当前的职业“程序员”可能较为匹配，但从用户历史行为中可能难以发现这一点。

## 模型结构

### 整体结构

> MIAN的整体架构如图所示，主要包含三部分，分别是Embedding Layer、Multi-Interactive Layer和Prediction Layer，接下来将分别进行介绍。

![image](https://github.com/ShaoQiBNU/CTR_MIAN/blob/main/img/2.png)

### embedding layer

> 模型的输入特征主要包含四部分，分别是候选物品特征、用户历史行为序列、用户特征和上下文特征。原始输入特征主要是离散特征，经过Embedding layer转换为对应的embedding。经过转换后，候选物品特征、用户历史行为序列、用户特征和上下文特征分别表示为：

![image](https://github.com/ShaoQiBNU/CTR_MIAN/blob/main/img/3.png)

### Multi-Interactive Layer

> Multi-Interactive Layer包含四个单独的模块，分别是IBIM、IUIM、ICIM和GIM，具体介绍如下：

#### Item-Behaviors Interaction Module (IBIM)

![image](https://github.com/ShaoQiBNU/CTR_MIAN/blob/main/img/4.png)

> IBIM模块主要包含两部分：
>
> 1. Pre-LN Transformer
>
>    将用户历史行为序列中的每个物品向量转换为对应的hidden state，类似于DIEN中对于用户兴趣的提取，但是Transformer的并行计算可以在一定程度上降低计算耗时。Pre-LN Transformer的block与原始Transformer的block的区别主要在于将Layer Normalization的计算放在Multi-Head Self-Attention之前。具体参考：
>
> 2. Attention
>
>    用于计算每一个hidden state与候选物品之间的相关性，并进行加权求和得到IBIM模块的输出

![image](https://github.com/ShaoQiBNU/CTR_MIAN/blob/main/img/4.png)

#### Item-User Interaction Module (IUIM)

> 如果仅建模历史行为序列和候选物品的关系，当用户行为序列较为少时，难以获得较为准确的推荐结果。因此论文近一步显式建模了用户特征中每一个field的特征与候选物品的关系。例如一个新来的用户，如果是女性，则化妆品可能有更高的被推荐概率，如果是男性，则球鞋等有更高的被推荐可能性。IUIM模块与下述的ICIM模块在一定程度上也能解决冷启动的问题。

![image](https://github.com/ShaoQiBNU/CTR_MIAN/blob/main/img/5.png)

#### Item-Context Interaction Module (ICIM)

> ICIM模块与IUIM模块结构相同，用于显式建模上下文特征中每一个field的特征与候选物品的相关性，如下：

![image](https://github.com/ShaoQiBNU/CTR_MIAN/blob/main/img/6.png)

#### Global Interaction Module (GIM)

> 经过上述三个模块得到7部分的特征表示，分别是候选物品特征表示、用户历史行为序列特征表示、用户特征表示和上下文特征表示，以及IBIM、IUIM和ICIM三个模块输出的特征表示。
>
> 前四个可以看作是低阶特征表示，后三个可以看作是高阶特征表示。
>
> DCN论文中表明，显式建模低阶特征和高阶特征的交互，可以有效提升CTR预估效果。因此借鉴此思想，论文近一步增加了GIM模块，来显式建模低阶特征和高阶特征的关系，结构如下：

![image](https://github.com/ShaoQiBNU/CTR_MIAN/blob/main/img/7.png)

### Prediction Layer

> Prediction Layer是多层全连接神经网络，最终的损失函数为交叉熵损失函数。

![image](https://github.com/ShaoQiBNU/CTR_MIAN/blob/main/img/8.png)

## 实验效果

### 公开数据集实验效果

> 论文对比了MIAN和部分baseline模型在公开数据集和工业数据集上的离线效果，以及MIAN的线上效果，具体结果如下：

### 模型性能

> 论文对比了MIAN和DCN的性能，具体结果如下：

### Ablation study and Visualization Study

#### Effectiveness of Multi-Interactive Layer

> 论文对模型Multi-Interactive Layer的4个子模块分别做了ablation study，证明各个子模块的作用，具体如下：

![image](https://github.com/ShaoQiBNU/CTR_MIAN/blob/main/img/9.png)

> 从图中可以看出，4个子模块的移除都会对模型的效果产生影响。
>
> 移除IBIM模块的影响最大，这也证明了MIAN能够有效的利用序列行为来提取用户的兴趣。
>
> 移除IUIM和ICIM模块对模型影响也很大，这也证明了MIAN能够充分利用用户特征和上下文特征。此外，在这个设置下，MIAN的性能也比最佳基准DIEN稍好，这说明IBIM模块可以充分有效地利用用户的历史行为特征。

#### Effectiveness of Pre-LN Transformer

> 论文比较了不同提取用户序列行为模型的效果，具体如下：

![image](https://github.com/ShaoQiBNU/CTR_MIAN/blob/main/img/10.png)

> 从图中可以看出，没有transformer，模型的效果会很差，因为它忽略了用户历史行为之间的依赖。
>
> 用原始的transformer模型，模型效果有提升，这说明提取用户历史序列行为之间关联的重要性。
>
> 用Pre-LN transformer，模型效果比transformer有所提升，但是训练速度和稳定性则非常占优势。

#### Visualization of Global-attention

> 论文随机从Amazon数据集里筛选了14个case，可视化了GIM模块attention的权重，如图所示：

![image](https://github.com/ShaoQiBNU/CTR_MIAN/blob/main/img/11.png)

> 从图中可以看出，interactive feature representations的权重均高于原始特征，可视化结果不仅证明了细粒度特征学习的重要性，而且表明MIAN能够学习候选项目与多种细粒度信息之间的深度交互关联。
