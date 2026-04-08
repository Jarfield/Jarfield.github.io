---
title: Resources
layout: page
description: 资料库入口
---

# Resources

这页不按时间倒序。  
它只做一件事：把真正值得反复回看的入口资源集中在一起。

## Surveys and Foundations

<div class="resource-list">
  <article class="resource-item">
    <h3>Sentence-BERT</h3>
    <p><span class="chip">Foundation</span><span class="chip">Sentence Embedding</span></p>
    <p>句向量真正走向实用的代表性起点，适合建立“单向量表示”直觉。</p>
    <p><a href="https://arxiv.org/abs/1908.10084" target="_blank" rel="noopener">Open resource</a></p>
  </article>
  <article class="resource-item">
    <h3>SimCSE</h3>
    <p><span class="chip">Contrastive Learning</span><span class="chip">Text Embedding</span></p>
    <p>用非常简洁的对比学习方法推动句向量训练，适合看训练目标如何改变表征空间。</p>
    <p><a href="https://arxiv.org/abs/2104.08821" target="_blank" rel="noopener">Open resource</a></p>
  </article>
  <article class="resource-item">
    <h3>DPR</h3>
    <p><span class="chip">Dense Retrieval</span><span class="chip">Open-domain QA</span></p>
    <p>双塔检索进入主流的关键工作之一，适合建立 retrieval 视角。</p>
    <p><a href="https://arxiv.org/abs/2004.04906" target="_blank" rel="noopener">Open resource</a></p>
  </article>
</div>

## Retrieval and Interaction

<div class="resource-list">
  <article class="resource-item">
    <h3>ColBERT</h3>
    <p><span class="chip">Late Interaction</span><span class="chip">Retrieval</span></p>
    <p>如果你想理解为什么单向量之外仍然有持续空间，这篇几乎绕不开。</p>
    <p><a href="https://arxiv.org/abs/2004.12832" target="_blank" rel="noopener">Open resource</a></p>
  </article>
  <article class="resource-item">
    <h3>BEIR</h3>
    <p><span class="chip">Benchmark</span><span class="chip">Zero-shot IR</span></p>
    <p>帮助区分“模型在单一数据集上更高分”和“模型在异构检索场景里更稳”。</p>
    <p><a href="https://openreview.net/forum?id=wCu6T5xFjeJ" target="_blank" rel="noopener">Open resource</a></p>
  </article>
  <article class="resource-item">
    <h3>MTEB</h3>
    <p><span class="chip">Benchmark</span><span class="chip">Evaluation</span></p>
    <p>把 embedding 评测从少数任务扩到更完整任务集合，是理解“通用 embedding”最好的入口之一。</p>
    <p><a href="https://arxiv.org/abs/2210.07316" target="_blank" rel="noopener">Open resource</a></p>
  </article>
</div>

## Recent and Practical Directions

<div class="resource-list">
  <article class="resource-item">
    <h3>E5</h3>
    <p><span class="chip">General-purpose</span><span class="chip">Retrieval</span></p>
    <p>很适合观察统一训练数据、任务提示和零样本检索之间的关系。</p>
    <p><a href="https://arxiv.org/abs/2212.03533" target="_blank" rel="noopener">Open resource</a></p>
  </article>
  <article class="resource-item">
    <h3>MAR Reading Note</h3>
    <p><span class="chip">Internal Note</span><span class="chip">Asymmetric Retrieval</span></p>
    <p>站内一篇已经成形的非对称检索笔记，适合作为“研究型笔记”的示例页。</p>
    <p><a href="/2025/10/26/Notes/MAR_Note/">Open note</a></p>
  </article>
  <article class="resource-item">
    <h3>Notes Stream</h3>
    <p><span class="chip">Internal</span><span class="chip">Incremental Updates</span></p>
    <p>如果你想看最近新增的内容，而不是按专题浏览，可以直接进入时间流。</p>
    <p><a href="/notes/">Open notes stream</a></p>
  </article>
</div>

## 如何使用这页

- 想快速入门：先看 Sentence-BERT、DPR、MTEB。
- 想理解检索系统结构：继续看 ColBERT、BEIR、MAR。
- 想关注更实用的统一 embedding：再看 E5 以及后续效率问题。
