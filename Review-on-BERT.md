---
title: Review on BERT
date: 2021-09-15 16:49:28
update: 2021-09-15 16:49:28
tags:
    - BERT
    - NLP
    - Pre-trained Language Model
categories: Paper Review
---

BERT是一种双向编码器表征的Transformer模型(Bidirectional Encoder Repersention from Transformer)。主要利用掩码语言模型(Masked Language Model, MLM)训练的一个双向Transformer编码器，可将将文本序列编码成上下文相关的表征。这种表征再添加额外的输出层并在下游任务微调，可以取得STOA的结果。


## 模型架构

BERT的基础架构是一个双向多层Transformer编码器，其预训练和微调的过程如图：

![BERT](Review-on-BERT/Bert)

### 输入输出表征

针对不同的下游任务需要，BERT可以清晰地将一个单独的句子或一个句子对表示为一个序列。其中，句子可以指一段文本中的任务跨度，而不仅仅是实际的语言句子。一个序列指输入到BERT的token序列，可以是一个单独的句子或两个句子打包在一起。

每一个序列中的第一个token总是一个特殊的分类token：[CLS]，对应的最终隐藏状态用于分类任务中作为序列的总体类型表示。对于打包在一起的句子对，使用两种方法对其进行区分：1. 可以使用一个特殊的token [SEP]作为分段标志。2. 为每个token添加一个可学习的embedding来指示这个token属于句子A或句子B

假设输入embedding是 $E$，特殊token [CLS]的最终隐藏向量是 $C\in \mathbb{R}^H$，第 $i$ 个输入token的最终隐藏向量为 $T_i\in \mathbb{R}^H$

对于一个输入token，它的表征由对应的token embedding, 分段embedding和位置embedding的加和构成。

![Input](Review-on-BERT/input)

## 预训练任务

BERT的预训练使用两种无监督任务：

**Masked LM** : 直觉上来说，深度双向模型比单向模型或者两个单向模型连接的效果更好。但是，标准的条件语言模型只能是单向的。这是因为标准语言模型是以预测当前词为目标，而双向模型将允许每个词“看到自己”，这样模型就可以在多层上下文中直接给出目标词。

为了训练双向表征，BERT使用MLM：

在输入序列中随机抽取tokens，然后将抽取到的tokens替换为特殊的token ${[MASK]}$。

MLM目标是对被遮罩的token进行预测的交叉熵。

掩码token最终的隐藏向量被馈送到词典上的输出softmax中。

**这种训练目标带来一个问题：在预训练和微调之间形成了割裂——在微调中不存在所谓的掩码**。针对这个问题，我们并不是总是将掩码位置的token替换为[MASK]，而是以下述规则进行掩码操作：

BERT选取输入序列中15%的tokens做替换，对于被选中的tokens：

- 其中的80%被替换为 $[MASK]$
- 其中的10%保持不变
- 其中的10%被替换为词典中的随机词

然后， $T_i$ 被用于使用交叉熵预测原始的token

**Next Sentence Prediction, NSP** (后续的工作认为该任务无效)：

NSP目标是预测两个句子在原始文档中是否是连续句子的二元分类损失。

正样本从文本中提取连续句子构造，负样本从不同文档中提取句子对构造，正负样本等概率采样。

具体地说，选择两个句子A，B作为一个预训练样本。50%的时间B是A的下一句(标记为 $\text{IsNext}$)，另外50%的时间不是(标记为 $\text{NotNext}$)。上文所述的对应于特殊token [CLS]的最终隐藏向量用于NSP任务，其值经过softmax得到NSP的预测概率。

## 微调

BERT作为编码器，为下游任务提供语言表征。在额外添加输出层后，可以直接端到端微调。

