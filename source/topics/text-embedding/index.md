---
title: Text Embedding
layout: page
description: Text Embedding 专题
---

# Text Embedding

Text Embedding 关心的是：怎样把一句话、一段文本或一个文档压缩成足够稳定、足够有迁移性的表示。

## 1. 为什么这个专题是起点

- 很多下游任务都可以从单向量表示开始。
- 它是从通用语义表征走向检索、聚类和分类的桥梁。
- 很多后续检索和效率问题，都是在这个基础上继续演化出来的。

## 2. 核心问题

- 单向量表示能保留多少语义结构？
- 训练目标是更偏 STS，还是更偏检索与任务迁移？
- 模型要追求“通用性”还是“领域适配性”？
- 输入前缀、任务指令和数据构造成果到底有多大？

## 3. 一条实用的发展线

- 早期从平均词向量、InferSent、Universal Sentence Encoder 这类工作起步。
- Sentence-BERT 让句向量真正成为可直接落地的工具。
- SimCSE 把对比学习做得更简洁，也推动了统一句向量训练范式。
- E5、BGE 一类工作则把“检索效果、任务提示和统一评测”放到更中心的位置。

## 4. 阅读这个专题时要重点看什么

- 训练目标和负样本策略，而不只是 backbone。
- 输入模板、query/document 前缀和任务定义。
- 评测集是否覆盖 retrieval、STS、clustering、classification 等不同用途。

## 5. 建议先读

- [Resources](/resources/) 中的 Sentence-BERT、SimCSE、E5、MTEB。
- 如果你更关心搜索系统，下一页直接去 [Dense Retrieval](/topics/dense-retrieval/)。

## 6. 我目前更关心的部分

- 通用文本表示如何迁移到检索场景，而不是只在 STS 上看起来好。
- 当模型变大以后，是否真的还需要同样高维的在线表示。
- 对中文、领域文本和长文档，统一训练范式是否仍然有效。
