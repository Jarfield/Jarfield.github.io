---
title: Dense Retrieval
layout: page
description: Dense Retrieval 专题
---

# Dense Retrieval

Dense Retrieval 是 Embedding 最直接的应用场景之一。  
它关心的不是“表示是否优雅”，而是“检索系统是否真能把相关内容找回来”。

## 1. 它要解决什么问题

- 用语义表示替代关键词匹配的第一跳召回。
- 让 query 和 document 在同一空间里可比较。
- 在召回质量、索引速度和计算成本之间找到平衡。

## 2. 核心矛盾

- 单向量表示很高效，但细粒度匹配能力有限。
- 交互更充分的方法效果更强，但在线成本更高。
- 训练数据和负样本策略往往比模型结构本身更影响结果。

## 3. 常见方法线索

### Bi-encoder

最适合大规模检索，因为文档可以提前编码并进入向量索引。

### Hard Negative Mining

很多检索改进来自更困难、更接近真实干扰项的负样本设计。

### Late Interaction

例如 ColBERT 保留 token 级交互，试图在效果和在线成本之间做新的折中。

### Asymmetric Retrieval

查询侧轻、文档侧重，是很现实的一条工程路径。  
这也是我当前最关注的方向之一。

## 4. 进入这个专题的阅读顺序

1. DPR，理解双塔检索的基本范式。
2. BEIR，理解为什么评测集设计会影响我们对模型的判断。
3. ColBERT，理解单向量之外的 late interaction 思路。
4. 再去看效率问题和非对称检索。

## 5. 站内相关内容

- [MAR 阅读笔记](/2025/10/26/Notes/MAR_Note/)
- [Resources](/resources/) 中的 DPR、ColBERT、BEIR、MTEB

## 6. 我当前关心的问题

- 在真实部署里，query 端到底该多轻，document 端又可以多重？
- late interaction 是否会成为 dense embedding 的长期补充，而不是短期技巧？
- benchmark 分数提升是否真的对应检索系统收益？
