---
title: Overview
layout: page
description: Embedding 总览
---

# Embedding Overview

Embedding 不只是“把文本映射成向量”。  
它真正关心的是：**如何把语义、任务相关性和可部署性压缩进一个可比较的表示空间里。**

## 1. 它到底要解决什么问题

- 把对象表示成可用于检索、匹配、聚类、分类和重排的向量。
- 让“相似”的内容在空间里更接近，而不是只靠词面重叠。
- 在下游系统里做到可扩展、可缓存、可部署，而不是只能离线分析。

> 如果把生成模型看成“按 token 继续写”，那么 embedding 更像是“把内容压缩成一个能被搜索和比较的索引入口”。

## 2. 为什么它重要

Embedding 处在很多系统的前面一层：

- 检索系统需要它做第一跳召回。
- RAG 需要它把问题快速映射到候选知识。
- 聚类、去重、推荐、召回和排序，都依赖稳定的表征空间。
- 当模型越来越大时，**低延迟、低成本的表示层**反而变得更关键。

## 3. 一条足够实用的发展脉络

### 早期分布式表征

- word2vec / GloVe 主要解决词级表征。
- 核心问题是如何让“共现”转成“几何关系”。

### 预训练模型时代

- BERT 类模型让上下文感知表征成为主流。
- 问题从“词向量是否有效”变成“句子与段落表征如何稳定获得”。

### Sentence Embedding 与 Dense Retrieval

- 关注点转向句向量、段落向量和双塔检索。
- 核心任务从 STS 延伸到检索、重排和问答召回。

### Task-aware 与 Instruction-aware Embedding

- 开始强调不同任务的意图差异。
- embedding 不再只是通用语义表示，而是和任务指令、输入格式、负样本策略耦合。

### Efficiency / Multilingual / Long Context / Domain-specific

- 研究重点逐步从“只看效果”转向“效果 + 成本 + 泛化 + 可迁移”。
- 这也是当前最值得系统跟踪的一组问题。

## 4. 现在最核心的研究问题

- 如何在不显著增加在线成本的前提下提升检索质量？
- 如何让 embedding 对不同任务与不同领域仍然有效？
- 如何处理长文本、跨语言和多模态情形？
- 如何平衡向量维度、存储成本、索引速度和检索准确率？
- 如何评估一个 embedding 模型，而不是只看某一个榜单分数？

## 5. 当前主流范式

### Contrastive Learning

通过正负对构建表征空间，是现代 embedding 的默认起点。

### Bi-encoder / Dual-encoder

在检索里最实用，因为文档向量可离线缓存，查询端只做一跳编码。

### Hard Negative Mining

很多性能提升并不来自结构变化，而来自更难、更贴近真实干扰项的负样本。

### Late Interaction

当单向量不够表达细粒度匹配时，ColBERT 这类方式保留更多 token 级交互。

### Efficiency-oriented Design

量化、MRL、压缩、蒸馏、低维表示和 ANN 协同，不再是部署细节，而是研究主体之一。

## 6. 推荐从哪里切入

如果你是第一次系统接触这个方向，可以按下面顺序：

1. 先看 [Topics](/topics/) 页面，明确有哪些稳定专题。
2. 再看 [Resources](/resources/) 里的综述与 benchmark。
3. 之后读 [Dense Retrieval](/topics/dense-retrieval/) 和 [Efficient Embedding](/topics/efficient-embedding/)。
4. 最后去 [Notes](/notes/) 看具体论文笔记和阶段理解。

## 7. 这个总览页的作用

这页不会追求覆盖所有工作。  
它更像一个长期更新的总目录，用来回答下面这类问题：

- 这个方向现在的主战场在哪里？
- 某一篇新工作到底是在解决什么旧问题？
- 我应该把时间放在模型结构、数据构造、训练目标，还是评测与部署上？

下一步建议直接进入 [Topics](/topics/)。
