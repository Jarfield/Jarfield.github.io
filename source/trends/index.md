---
title: Trends
layout: page
description: 趋势与前沿
---

# Trends

这页不是“最新论文清单”，而是记录当前研究正在往哪里移动。

## 1. Embedding efficiency 正在从工程问题变成研究问题

过去很多工作默认在线成本不是主要限制。  
现在情况不同了：RAG、企业检索、知识库问答都要求低延迟、高吞吐、可缓存的表示层。

因此，量化、低维表示、MRL、蒸馏和 ANN 协同不再只是部署附录，而是模型设计的一部分。

## 2. Dense embedding 不再是唯一答案

单向量检索仍然重要，但越来越多工作在探索：

- token-level late interaction
- hybrid sparse + dense retrieval
- reranker / retriever 协同
- query-side light, document-side heavy 的非对称路径

这意味着后续研究不会只问“向量是不是更好”，而会问“在什么约束下它更合适”。

## 3. Evaluation 正在成为瓶颈

MTEB 和 BEIR 提醒我们一件事：  
一个 embedding 模型在单个任务上更强，不代表它在更广泛场景中更稳。

后续真正重要的问题是：

- 评测是否覆盖真实检索任务？
- 是否覆盖不同语言、长度和领域？
- 榜单增益是否能转化为系统收益？

## 4. Domain-specific embedding 仍有很大空间

医学、法律、代码、企业文档等场景都表明：

- 通用模型的零样本能力并不总是足够。
- 领域数据、任务格式和负样本构造会显著影响效果。
- 但领域模型也会带来更重的部署和维护成本。

这类权衡会长期存在，不会因为更大的基础模型出现而消失。

## 5. 我接下来会重点跟踪的主题

- Asymmetric retrieval
- Efficient embedding for RAG
- Benchmark 与真实系统收益之间的关系
- 长文档与任务感知 embedding

如果你想看结构化入口，回到 [Topics](/topics/)；如果想看正在写的内容，去 [Notes](/notes/)。
