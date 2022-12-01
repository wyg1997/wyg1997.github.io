---
layout: post
title:  "GPT论文笔记"
date:   2022-11-30 20:05:21 +0800
tags: deep-learning
color: rgb(255,90,90)
cover: '../assets/imgs/GPT.png'
subtitle: 'GPT论文笔记'
---

# GPT 论文解读

GPT 系列使用 transformer 中的解码器，使用无标号的文本做预训练，再到相关的子任务上做 finetune。

## GPT

### transformer 和 RNN 的选择

transformer 中有更结构化的记忆，可以处理更长的文本信息，从而可以抽取到更好的句子层面和段落层面的信息。

### 半监督（自监督）预训练

$$ L_1(u)=\sum_{i}logP(u_i|u_{i-k},...,u_{i-1};\theta) $$

> NOTE：$\theta$表示模型参数，k 表示序列长度。最大似然估计 k 个词后一个词的出现概率。

### 使用 transformer 解码器

解码器对第 i 个元素抽特征时，只能看到第 i 个元素前的元素（后面的元素被 mask 掉），而编码器可以看到序列中的所有元素。所以这里的标准语言模型中只能使用解码器。

### 有监督 fine-tuning

$$ P(y|x^1,...,x^m)=softmax(h_{l}^mW_y) $$

$$L_2(C)=\sum_{x,y}logP(y|x^1,...,x^m)$$

> NOTE：y 表示标签，h表示这个序列的最后一层的特征，W 表示一个分类的全连接层，通过 softmax 得到最后的预测概率。
>
> C 表示要 finetune 的数据集，L2 loss 就是整个句子在 label 上的预测损失。

$$ L_3=L_2(C)+\lambda*L_1(C) $$

> NOTE：把具体任务上的训练 loss 和文本预测 loss 放一起加权 finetune，可以得到更好的效果。

### NLP 中常见的应用

1. 分类（Classification）：输入一句话，得到最后的预测概率。
2. 蕴含（Entailment）：给出两段文本，前面是条件，后面是推理，判断前者是否支持后者的结论，三分类问题（支持、不支持、无关联）。
3. 相似（Similarity）：给出两段文本，判断两段文字是否相似。
4. 多选（Multiple Choice）：给出一个问题和多个回答，分别用问题和回答构造出一个句子，把所有结果做 softmax。

## GPT-2

Zero-Shot：不在子任务上做 finetune。

模型上相对于 GPT 有少量的结构上的改动，和 Sparse Transformer 类似。

在单一任务的训练效果不如 Bert，这篇论文从另一个角度出发，训练一个更为通用的语言模型。在各方面达到还不错的效果，都不惊艳，但都可以做。

因为要训练更为通用的模型，则不能引入一些特殊的 token。作者通过利用模型的语言理解能力，使用类似语言提示的东西来迁移到下游任务。但也需要更大的数据来提升模型的语言理解能力。

## GPT-3

few-shot：可以使用少量子任务数据。但不做微调和梯度更新（因为参数量非常大，微调成本也很高）。 

模型和 GPT-2 相同。

### Fine-tuning & Zero-shot & One-shot & Few-shot

- Fine-tuning：使用子任务数据在预训练权重上训练，并更新梯度。
- Zero-shot：使用原预训练模型，在句子中加入任务描述。
- One-shot：使用原预训练模型，在句子中加入任务描述的同时加入子任务的那个例子。
- Few-shot：使用原预训练模型，在句子中加入任务描述的同时加入子任务的少量例子。