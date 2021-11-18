---
title: "ERICA: Improving Entity and Relation Understanding for Pre-trained Language Models via Contrastive Learning"
tags:
    - NLP
    - PLM
    - Knowledge Embedding
    - Knowledge Graph
categories: Paper Note
---
# ERICA

<!--more-->

## Methodology

### 数据构建

ERICA 在大规模无标签的语料上训练，并由外部知识图谱 $\mathcal{K}$ 进行远程监督。

首先将数据以文档为单位分批， $\mathcal{D} = \{d_i\}_{i=1}^{|\mathcal{D}|}$ 是一个包含若干文档的一个 batch。

$\mathcal{E}_i = \{e_{ij}\}_{j=1}^{|\mathcal{E}_i|}$ 是文档 $d_i$ 中出现的所有实体的集合，其中 $e_{ij}$ 是文档 $d_i$ 中的第 $j$ 个实体。

对于文档 $d_i$, 作者枚举出其中的所有实体对 $(e_{ij}, e_{ik})$, 并将它们以 $\mathcal{K}$ 中的关系 $r_{jk}^i$ 连接起来，这样就得到了一个元组集 $\mathcal{T}_i = \{t_{jk}^i=(d_i, e_{ij}, r_{jk}^i, e_{ik})|j\neq k\}$. 对于 $\mathcal{K}$ 中不存在关系的实体对(out-of-KG 问题), 将其关系设置为 `no_relation`. 

对于整个批次，将其中每个文档的元组集拼接得到整个批次的元组集 $\mathcal{T} = \mathcal{T}_i\cup\mathcal{T}_2\cup...\cup\mathcal{T}_{|\mathcal{D}|}$.

进一步，作者通过去除 $\mathcal{T}$ 中所有关系为 `no_relation` 的元素构建了阳性元组集 $\mathcal{T}^+$. $\mathcal{T}^+$ 中包含句内实体对和句间实体对。

### 实体和关系的表征方法

对于每个文档 $d_i$ ，作者首先使用 PLM 对齐进行编码，计算出每个 token 的表征 $\{\mathbf{h}_1, \mathbf{h}_2, ..., \mathbf{h}_{|d_i|}\}$, 然后对于文档中的每个实体 $e_{ij}$, 作者通过对这个实体所包含的所有 token 的表征进行均值池化(*mean pooling*) 获得这个实体本次出现的局部表征。由于一个实体在文档中可能多次出现，其第 $k$ 次出现的局部表征记为：

$$\mathbf{m}_{e_{ij}}^k = \text{MeanPool}(\mathbf{h}_{n_{start}^k}, ..., \mathbf{h}_{n_{end}^k})$$

其中，$n_{start}^k$ 和 $n_{end}^k$ 分别是实体 $e_{ij}$ 所包含的 token 在文档 $d_i$ 中第 $k$ 次出现的起始位置和终止位置。

同时作者将文档中 $e_{ij}$ 的所有局部表征取均值作为其在文档中的全局特征 $\mathbf{e}_{ij}$，**作者认为$\mathbf{e}_{ij}$中包含该实体在该文档中的全部信息**

对于文档中两个实体 $e_{ij_1}, e_{ij_2}$ 之间的关系，作者将其全局表征拼接起来作为其关系的表征 $\mathbf{r}_{j_1j_2}^i = [\mathbf{e}_{ij_1}; \mathbf{e}_{ij_2}]$