---
title: Efficient Embedding
layout: page
description: Efficient Embedding 专题
---

# Efficient Embedding

Efficient Embedding 不是“模型做完以后再优化”的附属主题。  
它本身就是研究问题，因为在线检索系统最终要面对的是延迟、显存、索引大小和吞吐。

## 1. 这个专题真正讨论什么

- 降低向量维度或存储成本。
- 在保持召回质量的同时压缩表示。
- 把训练目标和部署目标放在一起考虑，而不是分开处理。

## 2. 典型抓手

- Distillation
- Quantization
- Low-dimensional embeddings
- Matryoshka Representation Learning (MRL)
- Sparse / hybrid representation
- Asymmetric retrieval

## 3. 为什么它会越来越重要

- embedding 正在从学术任务走向真实系统。
- 越大的基础模型，越需要一个便宜、稳定、可缓存的前置表示层。
- RAG、搜索、推荐与企业知识库都在持续放大这一需求。

## 4. 理解这个方向时不要只盯着压缩率

真正需要同时看的是：

- 召回质量下降多少
- 索引和存储节省多少
- 在线吞吐提升多少
- 是否适合 ANN 系统
- 是否仍然保持跨任务泛化

## 5. 站内相关内容

- [Dense Retrieval](/topics/dense-retrieval/)
- [MAR 阅读笔记](/2025/10/26/Notes/MAR_Note/)
- [Resources](/resources/) 里的 E5、MTEB 与 benchmark 入口

## 6. 我对这个专题的当前判断

- 未来很多工作不会只比“更高分”，而是比“在同样预算下更优”。
- 非对称检索、低维向量和评测体系会越来越紧密地绑在一起。
- 单纯压缩 embedding 本身不够，必须连同数据、训练目标和索引方案一起看。
