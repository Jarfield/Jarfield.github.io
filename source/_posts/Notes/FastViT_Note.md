---
title: "FastViT: A Fast Hybrid Vision Transformer  using Structural Reparameterization 阅读笔记"
cover: /images/Covers/flowers.jpg
date: 2025-11-1 21:42:11
tags:
  - 高效模型
  - 视觉Transformer
  - 网络压缩
categories:
  - 论文解读
mathjax: true
---

**金培晟**	*Jarfield*

## 0 摘要

近年来 **Transformer** 和卷积架构的结合促使视觉模型在准确率和效率上稳步提升。在这项工作中，作者提出了 **FastViT**，这是一种混合型视觉Transformer架构，在**时延-准确率折中**（latency-accuracy trade-off）上达到当前最佳水平。为此，作者设计了一种新颖的令牌混合（token mixing）算子 **RepMixer** 作为 FastViT 的基本模块。RepMixer 通过结构重参数化（structural reparameterization）移除网络中的跳跃连接（skip-connections），显著降低**内存访问开销**。此外，作者在训练期间引入线性过参数化（train-time overparameterization）和大卷积核（large kernel）卷积来提高模型准确率，并从实验证明这些改动对推理时延几乎没有负面影响。实验结果表明：在 ImageNet 数据集上，以相同准确率比较，FastViT 模型在移动设备上推理速度比最新的混合Transformer架构 CMT 快 **3.5 倍**，比 EfficientNet 快 **4.9 倍**，比 ConvNeXt 快 **1.9 倍**；在相近推理时延下，FastViT 的ImageNet Top-1准确率比 MobileOne 高出 **4.2%**。在图像分类、目标检测、语义分割和3D网格回归等多个任务上，FastViT 相比其他架构均取得更高的准确率且大幅降低推理时延（包括移动设备和桌面GPU）。同时，FastViT 对分布外样本和图像扰动具有更高的**鲁棒性**（robustness），优于现有的抗扰动模型。代码和预训练模型已开放至：[FastViT](https://github.com/apple/ml-fastvit "FastViT代码仓库")。

<!-- more -->

## 1 Introduction

近年来 **Vision Transformer (ViT)** 模型在图像分类、目标检测、语义分割等任务上取得了卓越表现，但其计算开销一直较高。为降低ViT的计算和内存需求，不少研究提出了高效Transformer改进。尤其是**混合架构**（hybrid architecture）的视觉Transformer成功融合了卷积神经网络（CNN）的局部建模优势和Transformer的全局建模能力，能够在众多视觉任务上取得极具竞争力的效果。FastViT 的目标是在保证精度的前提下，将推理**时延**（latency）显著降低，取得**最佳的时延-准确率折中**。

值得注意的是，许多近期视觉Transformer和混合架构都遵循 MetaFormer框架，即模块由“令牌混合+跳跃连接”以及“前馈网络（FFN）+跳跃连接”组成。然而，这些跳跃连接会显著增加推理延迟，因为它们带来了额外的**内存访问**成本。为解决这一瓶颈，作者引入**RepMixer** 模块——一种完全可重参数化的令牌混合算子，可在推理时重构网络结构以**移除跳跃连接**。RepMixer 模块内部采用深度卷积（depthwise convolution）对空间信息进行混合，灵感来自 ConvMixer；但与ConvMixer不同的是，RepMixer **不含非线性激活**且在推理时能够融合分支，使其等效为单一卷积层，从而减少内存访问开销。

此外，为进一步提升效率（降低参数量和 FLOPs），FastViT 将网络中的所有 $k\times k$ 密集卷积替换为分解形式（depthwise卷积 + 逐点卷积），这一策略在MobileNet等高效架构中很常见。然而，简单地因式分解卷积会降低模型容量，影响准确率（见*(table1)*第一行与第三行比较）。为此，作者借鉴 MobileOne等工作的**线性训练时过参数化**方法，在训练阶段为部分卷积分支添加额外参数分支以提升容量（推理时再融合移除）。这些额外分支仅在训练时存在，在推理时通过重参数化融入主干，不增加推理开销。

最后，作者在网络中引入**大卷积核**卷积。原因在于，尽管自注意力（self-attention）可以有效建模长程依赖提升精度，但其在硬件上计算延迟高。相比之下，大卷积核能够扩大感受野且计算高效。因此，FastViT 在 FFN 层和补丁嵌入层中引入了大卷积核的深度卷积，提高性能的同时对总体时延影响很小。

<img src="/images/Notes/FastViT/Figure1.png" alt="Figure1" style="zoom:67%;" />

综上，FastViT 基于三大设计原则构建：（i）使用 RepMixer 模块移除跳跃连接；（ii）在训练时对卷积层进行线性过参数化以提升容量；（iii）在模型中引入大卷积核卷积以扩大感受野但避免高昂的注意力计算。凭借这些设计，FastViT 在移动设备和桌面GPU上均实现了**高效的通用视觉Transformer**。**Figure1** 展示了不同模型在ImageNet精度和推理时延上的折线对比，FastViT 在移动端与GPU端均取得**最佳的准确率-时延曲线**，显著领先于此前的高效CNN和Transformer模型。

**贡献总结：**

- **提出FastViT架构**：设计混合Transformer并通过结构重参数化移除跳跃连接，达到当前**最优的精度-时延折中**。
- **极快的推理速度**：在两种常用平台上（移动设备和桌面GPU），FastViT 都实现了同精度下的**最低延迟**。
- **多任务泛化能力**：FastViT 在图像分类、目标检测、语义分割和3D手部网格回归等多个任务上均取得**领先性能**，证明其通用性。
- **鲁棒性强**：FastViT 对图像腐败和分布外数据具有高鲁棒性，在显著提速的同时超越了现有强调鲁棒性的模型。

## 2 Related Work

过去十年中，卷积神经网络（CNN）一直是视觉模型的标准架构，如 ResNet 系列在分类和检测中表现优异。然而近年**Transformer** 技术兴起，使模型具备建模长程依赖的能力，在视觉任务中也取得巨大成功。Transformer 的全局自注意力提供了CNN不具备的全局信息，但代价是计算复杂度高、速度较慢。为此，众多工作尝试在视觉Transformer中降低自注意力的计算代价，如限制注意力范围或引入稀疏/低秩注意力等。

**混合视觉Transformer：** 为兼顾效率与精度，近期一些架构采用**混合设计**，将**卷积与Transformer结合**来同时捕获局部和全局特征。一些模型使用卷积**改进ViT的补丁提取阶段**或**在网络前几层引入卷积**提取低级特征；也有模型采用**窗口化注意力**在局部范围内计算自注意力以降低复杂度。此外，一系列显式混合架构被提出，通过**交替或并行**地使用卷积模块和Transformer模块来交换信息。值得注意的是，MetaFormer 架构表明，即使用简单的池化算子替代自注意力（如 PoolFormer 模型），也能获得不错的性能，提示令牌混合方式的多样化潜力。然而，大多数混合Transformer的令牌混合仍以自注意力为主，低效的全局操作限制了推理速度。

**结构重参数化：** 近年来，一些研究表明可以通过**重参数化**的技巧将训练时的复杂结构简化为推理时的高效结构，从而**降低实际部署时的开销**。例如 RepVGG和 MobileOne展示了将训练时的多分支卷积结构在推理时融合为单一卷积层的可行性，特别是移除**跳跃连接**能够减少推理时的内存访问成本。在 FastViT 中，作者借鉴这一思想，提出了可重参数化的RepMixer模块，在推理时去除MetaFormer中的跳跃连接，从而降低延迟。

此外，为提高效率，很多高效模型使用了**卷积因式分解**（如 depthwise + pointwise 卷积）来降低FLOPs。但这往往减少了参数量，带来模型容量下降的问题。**线性过参数化**技术近期被用于补偿这一缺陷：在**训练阶段为卷积层添加线性分支**增加参数容量，推理时再折叠融合。FastViT 在因式分解卷积的基础上应用了该方法，提高模型容量的同时保持高效的推理结构。

综上所述，据作者所知，此前尚无混合Transformer架构同时采用**跳跃连接重参数化**和**线性过参数化**来优化延迟与容量的工作。FastViT 将这些思想相结合，在保持或提升准确率的同时，大幅降低了不同硬件环境下的推理延迟。

## 3 Architecture

### 3.1 Overview

FastViT 是一种卷积-Transformer混合架构，由四个分辨率逐渐降低的阶段组成，如**Figure2**所示；各 FastViT 模型的详细配置参见**table2**。整个网络遵循 MetaFormer 框架：每个阶段由一个令牌混合模块（Token Mixer）和一个前馈网络（FFN）堆叠而成，但与标准MetaFormer不同的是，FastViT 对这些模块的结构做了精心设计以优化速度。

<img src="/images/Notes/FastViT/Figure2.png" alt="Figure2" style="zoom:67%;" />

如**Figure2**所示：

- **RepMixer 模块：** FastViT 的令牌混合模块采用 **RepMixer**，通过结构重参数化移除了常规MetaFormer中令牌混合后的跳跃连接，从而降低**内存访问成本**。此外，RepMixer利用深度卷积在空间维度混合信息（类似 ConvMixer），既保证局部建模能力又便于在推理时融合分支。
- **分解卷积 + 过参数化：** 为提升效率和性能，FastViT 将网络中的 $k\times k$ 卷积（如Stem层和Patch Embedding层中的卷积）替换为**因式分解卷积**（depthwise + 1×1卷积）。单纯分解虽可减少参数和FLOPs，但可能降低模型表示能力。因此，作者对这些卷积层应用**线性训练时过参数化**（MobileOne风格）：在训练阶段添加平行的卷积分支增大容量，而在推理时将其折叠合并。
- **大卷积核卷积：** FastViT 用**大核深度卷积**替代了部分自注意力，以扩大感受野且避免高计算延迟。具体地，在每个FFN层和Patch Embedding层内加入大卷积核的深度卷积（例如$7\times7$卷积），提升局部模块的感受野。由于这些位置处于网络早期或非主干，全局来看对时延影响不大但能带来准确率提升。

<img src="/images/Notes/FastViT/Table1.png" alt="Table1" style="zoom:67%;" />

**Table1**展示了作者从 PoolFormer-S12 基础模型逐步引入上述改动得到 FastViT-S12 的性能变化：将输入分辨率由224增大到256略微提升准确率至77.6%；用RepMixer取代池化令牌混合几乎不增加FLOPs却显著降低延迟，并将Top-1准确率提高到78.5%；将密集卷积替换为因式分解卷积可减少约27%参数和6% FLOPs，但Top-1略有下降至78.0%；训练时对这些卷积添加过参数化分支在不改变推理成本的前提下将Top-1提升回78.9%；最后，在FFN和Patch Embedding中加入大核卷积带来额外+0.9%的准确率增益，使FastViT-S12达到79.8%的ImageNet Top-1准确率，同时移动端延迟仅约1.4ms。

值得注意的是，标准的自注意力令牌混合在高分辨率输入下计算量巨大、速度缓慢。虽然一些工作提出了更高效的注意力变体以减轻此问题，FastViT 选择用**大卷积核卷积**作为更高效的替代，以提升网络早期层次的感受野而几乎不增加延迟。**Table1** 中对比了是否使用大核卷积对精度和速度的影响：对FastViT-S12而言，在FFN和Patch Embedding中加入$7\times7$大核深度卷积使Top-1从78.9%提升到79.8%，移动端时延从1.26ms小幅增至1.40ms，可见以**极小的延迟代价换来了0.9%的准确率提升**。下一节将详细介绍FastViT各组件的设计细节。

### 3.2 FastViT

#### 3.2.1 Reparameterizing Skip Connections

**RepMixer 模块** 视觉Transformer中的令牌混合通常通过自注意力实现，也有工作探索纯卷积的混合方式。例如 ConvMixer采用深度卷积进行令牌混合，并使用跳跃连接融合输入，如：

$$
  Y = \mathrm{BN}\big(\sigma(\mathrm{DWConv}(X))\big) + X \tag{1}
$$

 其中 $\sigma$ 表示非线性激活函数，BN表示批归一化（Batch Normalization），DWConv表示深度卷积。ConvMixer 结构虽然有效，但其模式仍需要在推理时保留加法分支。为此，FastViT 提出将操作顺序重排并移除非线性激活：

$$
  Y = \mathrm{DWConv}\big(\mathrm{BN}(X) + X\big) \tag{2}
$$

 这样设计的主要好处在于，该结构可在推理时**重参数化**为单一的深度卷积层，将加法分支融合进去，如下式所示：

$$
  Y = \mathrm{DWConv}(X) \tag{3}
$$

<img src="/images/Notes/FastViT/Figure2d.ong" alt="Figure2d" />

**Figure2d** 展示了这种重参数化过程：训练时RepMixer含有批归一化和残差分支，而推理时这些操作可并入卷积核权重，使整个RepMixer模块“折叠”成一个等效的深度卷积层。这种移除跳跃连接的设计大幅降低了推理时的内存访问开销。

**位置编码** FastViT 采用**条件位置编码**（conditional positional encoding）来替代传统固定或可学习位置编码。具体实现上，通过一个Depthwise卷积根据每个令牌的局部邻域动态生成位置编码，并将其加到Patch Embedding的输出上。由于此过程不含任何非线性激活，整个位置编码模块也可以与相邻层一并重参数化融入模型（类似**Figure2a** 中的融合示意)。总之，RepMixer 模块配合条件位置编码，在保持模型表达能力的同时，实现了**无跳跃连接**的高效令牌混合。

<img src="/images/Notes/FastViT/Figure3.png" alt="Figure3" style="zoom:67%;" />

**实验结果** 作者通过实验验证移除跳跃连接的收益。在一个MetaFormer S12架构中分别使用池化和RepMixer作为令牌混合模块（两者FLOPs均约1.8G），测试不同输入分辨率下的推理延迟。**Figure3**展示了两者在 iPhone 12 Pro 上的延迟对比曲线：随着分辨率升高，RepMixer的优势愈发明显。在$384\times384$输入时，使用RepMixer比池化将延迟降低约**25.1%**，而在超高分辨率如$1024\times1024$时延迟降低达**43.9%**。这证明**移除跳跃连接**对于高分辨率输入的效率提升至关重要，而RepMixer模块成功在不损失精度的情况下实现了这一点。

#### 3.2.2 Linear Train-time Overparameterization

将卷积分解为 Depthwise 和 Pointwise 虽有效降低了参数和FLOPs，但也削弱了模型容量和准确率。为此，FastViT 在**训练阶段**对这些因式分解卷积引入**线性过参数化**分支，以提高其拟合能力。具体做法是：对于每个用因式分解卷积分解的层（如Stem层、Patch Embedding层以及投影层），在训练时额外添加若干并行的$k\times k$卷积分支（通常使用$1\times1$卷积实现**线性**增加，不改变非线性特征），并将其输出累加到主干分支上。这些附加卷积分支的参数在训练中学习，以弥补因式分解带来的容量下降。在推理阶段，这些并行卷积可以与主卷积核权重相加合并，完全**移除**额外分支，不增加任何推理成本。

<img src="/images/Notes/FastViT/Table3.png" alt="Table3" style="zoom:67%;" />

**Table3** 对比了开启和关闭训练时过参数化对FastViT模型精度和训练时间的影响。例如，对于FastViT-SA12，在ImageNet-1k上不使用过参数化训练的Top-1准确率为80.0%，总训练时长31.3小时；加入过参数化分支训练后，准确率提高到80.6%，训练耗时增加到33.4小时（约+6.7%）。类似地，较大的FastViT-SA36准确率从83.3%提升至83.6%，训练耗时增加约4.4%。由此可见，**训练时过参数化**可以以很小的额外训练代价换取**+0.5%~0.6%**的准确率提升。值得注意的是，FastViT 仅对那些由密集卷积替换为因式分解卷积的层应用过参数化（如前文提到的Stem、Patch Embedding和Projection层）。这些层在整个网络中计算成本占比较低，因此对它们进行过参数化不会显著拖慢训练。例如，在相同设置下，应用过参数化的FastViT-S12训练时间比原模型仅增加约6.7%，FastViT-SA36增加约4.4%。因此，**线性过参数化**策略有效提升了模型容量和准确率，同时对推理效率无任何影响，对训练开销的影响也在可接受范围内。

#### 3.2.3 Large Kernel Convolutions

由于 RepMixer 等卷积型令牌混合提供的感受野是**局部**的，相比全局自注意力可能限制模型捕获长程依赖的能力。为兼顾效率和全局感受野，FastViT 提出在不采用自注意力的阶段引入**大卷积核**的深度卷积来拓展感受野。这一想法计算上非常高效，可显著提升网络早期层的感受野和特征表达能力，而不会像自注意力那样带来高昂的计算和内存代价。具体而言，FastViT 在每个 FFN 模块中和每次降采样的 Patch Embedding 模块中加入了 **$7\times7$ 深度卷积核**（可视作ConvNeXt样式的卷积FFN改进）。

<img src="/images/Notes/FastViT/Table4.png" alt="Table4" style="zoom:67%;" />

**Table4** 对比了使用大核卷积与插入自注意力层在早期阶段的影响：如 **V4 vs V3**，当将所有阶段都用RepMixer（无自注意力）但加大核卷积（V4）对比前三阶段含自注意力的模型（V3），V4模型Top-1仅低0.6%，但参数减少11%、**推理延迟降低**约2.3倍。类似地，**V2 vs V4**：V2（含部分自注意力）比V4（无注意力，大卷积）参数多20%、延迟高7%，准确率反而相近。这表明**大核卷积**能够在无需自注意力的情况下取得与注意力模型相当的精度，却能显著降低计算延时，是提升早期阶段性能的高效方案。在**Table1** 的消融实验最后两行也可以看到：在FFN中使用大卷积核可提升FastViT-S12准确率约0.5%，在Patch Embedding中再使用大卷积核再增益0.4%，两者合计**+0.9% Top-1**，移动端延迟仅增加约0.1-0.2ms。

FastViT 的 FFN 和补丁嵌入层的具体结构如 **Figure2** 所示，其模式与 ConvNeXt 块类似但有关键区别：（1）使用 **Batch Normalization** 替代 Layer Normalization，这是因为 BN 能与前一层融合，在推理时无额外开销，而LayerNorm在ConvNeXt实现中需要张量维度变换，不利于部署效率。（2）大卷积核增加了FFN块的卷积特性。**卷积型FFN**模块被认为比原始全连接FFN对扰动更加鲁棒。因此在FFN中引入大核卷积不仅扩大全局感受野，也**提升了模型鲁棒性**。总体而言，引入大卷积核是一种**高效提升模型性能与稳健性的手段**，在FastViT中发挥了重要作用。

## 4 Experiments

### 4.1 Image Classification

作者首先在 **ImageNet-1K** 图像分类数据集上验证了 FastViT 的性能。

- ImageNet-1K 包含约128万张训练图像和5万张验证图像，涵盖1000个类别。
- 所有 FastViT 模型均按照标准训练配置进行训练：训练周期300个epoch，优化器使用AdamW，权重衰减0.05，批量大小1024，余弦学习率调度（预热5个epoch，峰值学习率 $10^{-3}$）。
- 采用 PyTorch 的 *timm* 库实现训练并使用8张 NVIDIA A100 GPU。
- 对于分辨率提升至384×384的变体，作者在224模型基础上额外微调30个epoch（学习率$5\times10^{-5}$，权重衰减$10^{-8}$，batch size 512）。
- 推理延迟测量方面：移动端在 iPhone 12 Pro Max 上使用 **Core ML Tools** 将模型转为CoreML格式执行，batch size=1，并取100次运行的中位数延迟；GPU端使用 **TensorRT** 将模型导出后在NVIDIA RTX-2080Ti上测试(batch size=8取中位数)。

<img src="/images/Notes/FastViT/Table5.png" alt="Table5" style="zoom:67%;" />

**与SOTA模型比较：** **Table5** 汇总了 FastViT 与近期代表模型在ImageNet-1K上的性能和效率对比。为公平起见，作者对ConvNeXt的实现进行了优化（移除了一些不必要的reshape操作）以提升其部署效率，同时排除了一些无法成功导出到CoreML或TensorRT的模型（表中以“-”标注）。从表中可以看到，在**桌面GPU**和**移动设备**两种平台下，FastViT 相较同时代模型都取得了**最佳的准确率-延迟折中**。【例如，FastViT-MA36 模型（256×256输入）在 Top-1准确率 83.9% 时，参数量42.7M、FLOPs 7.9G，GPU延迟6.7ms、手机延迟4.5ms，分别比 ConvNeXt-B 提高了**1.9×**（手机）和 **2.0×**（GPU）的速度】。在相似准确率84.9%的情况下，FastViT-MA36 的GPU延迟与 NFNet-F1【1】相当，但参数量仅为其33.3%、FLOPs仅为其49.9%、移动端推理更是快**42.8%**。对于小模型，FastViT-S12（79.8% Top-1）相比MobileOne-S4【57】在iPhone上快**26.3%**，在2080Ti上快**26.9%**，但准确率略低0.4%。总体来看，无论是低延迟还是高准确率区间，FastViT 系列均全面覆盖并超越了之前的 CNN 和 Transformer 模型。

<img src="/images/Notes/FastViT/Table6.png" alt="Table6" style="zoom:67%;" />

**Table6**展示了使用**知识蒸馏**（distillation）训练时，各模型在ImageNet上的性能对比。作者按照 DeiT提出的硬蒸馏方案进行训练：教师模型选用 RegNet-16GF（即16GFLOPs的RegNet），将教师的硬分类结果作为伪标签。所有 FastViT 模型蒸馏训练同样跑满300个epoch，并**未**像部分方法那样额外添加一个蒸馏用的分类头，而是直接利用蒸馏标签训练原有头。结果显示 FastViT 在蒸馏设置下进一步提升了性能，并超越了最新高效模型 EfficientFormer等。【例如 FastViT-SA24 蒸馏后Top-1达到83.4%，与EfficientFormer-L7相当，但FastViT-SA24参数量仅为后者的1/3.8（20.6M vs 82.1M），FLOPs仅为其约37%（3.8G vs 10.2G），推理延迟也低2.7×以上】。可见，在蒸馏增强下FastViT的小模型取得了接近大模型的准确率，同时保持了明显的效率优势。

### 4.2 Robustness Evaluation

作者进一步评估了 FastViT 在分布外数据和图像扰动下的性能。具体采用了四个常用的鲁棒性基准：

- (i) **ImageNet-A**：收集了使ResNet易出错的真实图像；
- (ii) **ImageNet-R**：ImageNet类别的艺术化和卡通渲染图像集；
- (iii) **ImageNet-Sketch**：ImageNet类别对应的手绘黑白线稿集；
- (iv) **ImageNet-C**：对ImageNet验证集施加高斯噪声、模糊等一系列合成扰动得到的腐败集（有15种失真×5个强度）。

评估指标方面，对ImageNet-C使用平均**腐败误差** (mean corruption error, mCE，值越低越好)，对其余三个数据集报告Top-1准确率 (值越高越好)。所有模型均使用开源实现在这些基准上测试，并确保只使用ImageNet-1K预训练（无其他数据）以公平比较。

<img src="/images/Notes/FastViT/Table7.png">

**Table7**给出了不同模型在以上鲁棒性测试的结果（按FLOPs分组）。可以看到，FastViT 系列在**鲁棒性-效率**上表现出色：相比纯CNN模型，其对腐败和分布外数据更稳健，而与强调Transformer鲁棒性的近期模型（如 ConvNeXt、RVT等）相比，FastViT 明显更快且精度相当甚至更优。例如，在相近FLOPs组中，FastViT-SA36 的ImageNet-C误差为**51.8**（越低越好），显著优于ResNet-50 (65.5)和ConvNeXt-T (53.2)；FastViT-SA36 在ImageNet-A上准确率**32.3%**，仅略低于参数更大的ConvNeXt-S (31.2%) 和EffNet-B4 (26.3%)，在ImageNet-R和Sketch上则达到与ConvNeXt-S相当的48.1%和35.8%。FastViT-MA36（83.9% ImageNet精度）在各鲁棒性指标上甚至全面媲美ConvNeXt-S（83.1%精度，参数多出6M，FLOPs多10%），表现出**更高的腐败鲁棒性和相近的分布外鲁棒性**。作者将FastViT的高鲁棒性归功于架构设计上的选择：例如§3.2.3提到的大卷积核卷积与自注意力的结合，能够提升模型对纹理变化和噪声的适应性。此外，FastViT 使用的卷积型FFN模块本身也较标准FFN更稳健。综上，FastViT 在不牺牲速度的前提下，实现了**优于同级模型**的鲁棒性，在实际应用中能更好地应对不可预测的输入扰动。

### 4.3 3D Hand Mesh Estimation

FastViT 架构在 **3D手部网格回归** 任务中同样表现优异。当前实时3D手部重建方法往往在CNN主干后附加复杂的网格回归模块（如基于Graph或Transformer的姿态回归层）。常用主干包括ResNet或HRNet系列，这些CNN在大部分硬件上都有良好优化，但沉重的回归头使整体延迟仍然较高。作者提出利用 FastViT 作为高效主干，同时尽量简化回归头：在FastViT提取的特征上直接用一个轻量级MLP回归 **MANO** 手部参数（不引入图形模型等额外计算）。在训练策略上，只使用ImageNet-1K预训练权重并**仅在 FreiHAND** 数据集上微调，不借助额外的合成数据，以突出模型自身的效果。

![image-20251101212139452](/images/Notes/FastViT/Table8.png)

**Table8**列出了 FreiHAND 基准测试上的指标对比。在“实时”方法中（例如MobileHand、MobRecon等都是追求实时速度的轻量模型），FastViT 提供的方案达到**最优的精度-速度**权衡：它在关键指标（如顶点位置误差、命中率等）上略优于之前的最佳模型，同时在推理速度上远超对手——在相同硬件上，FastViT 模型比MobileHand快**1.9倍**，比最新的MobRecon快**2.8倍**。这充分说明了FastViT在三维重建任务中的潜力：通过减少主干延迟和简化回归头，可实现在几乎**不损失精度**的情况下大幅提升运行效率，非常适合嵌入式或移动端的实时3D感知应用。

### 4.4 Semantic Segmentation and Object Detection

**语义分割：** 作者在 **ADE20K** 数据集上评估了FastViT作为分割模型主干的效果。ADE20K包含2万张训练图像和2千张验证图像，涵盖150个语义类别。实验采用 **Semantic FPN** 作为分割解码头，并使用与PoolFormer论文相同的训练配置（如80k步数、学习率等）。所有模型的主干都初始化为各自ImageNet-1K预训练权重。由于分割需处理高分辨率输入，这里统一在 $512\times512$ 图像裁块上评估各模型主干的 FLOPs 和延迟（GPU上使用batch=2来模拟高分辨率推理并获取稳定延迟）。

![image-20251101212435433](/images/Notes/FastViT/Table9.png)

**Table9** 列出了若干主干在ADE20K上的分割结果和性能指标。可以看到，FastViT 系列相比同尺寸的CNN或Transformer主干取得了更高的 **mIoU** 和更低的推理延迟。例如，FastViT-MA36 主干在分割任务中达到 **44.6%** mIoU，较同级的 PoolFormer-M36 提高了**5.2%**，并且**移动端延迟降低约1.5倍**（24.8ms降至16.3ms）；在GPU上FastViT-MA36的主干延迟仅8.2ms，显著小于ResNet-101（4.6ms）或PoolFormer-M36（41.4ms）等。这说明FastViT不仅在分类，就算作为分割模型的特征提取主干也能提供**更优的精度**和**更快的推理**，有效提升分割模型的整体速度。

**目标检测：** 作者还在 **MS COCO** 检测数据集上验证了FastViT用于检测的性能。COCO包含118k训练图像和5k验证图像（80类目标）。实验采用经典的 **Mask R-CNN**框架，并使用1x调度（12个训练epoch）。各模型主干均用各自ImageNet预训练权重初始化。在评估时，同样将各模型在 $512\times512$ 输入下的主干延迟在GPU(batch=2)和移动端进行测量。

![image-20251101212650208](/images/Notes/FastViT/Table10.png)

结果如**Table10**：FastViT 系列在不同延迟/精度等级下均达到**SOTA级别**。特别地，FastViT-MA36 主干在检测中取得了 **45.1** 的AP<sub>bbox</sub>（边界框平均精度）和 **40.5** 的AP<sub>mask</sub>（实例分割平均精度），与同等设定下当前最佳的CMT-S主干（AP<sub>b</sub>=44.6, AP<sub>m</sub>=40.7）性能非常接近，但FastViT-MA36在推理速度上对CMT-S形成碾压：**GPU端快2.4倍，移动端快4.3倍**（CMT-S主干GPU延时≈19.9ms vs FastViT≈8.2ms；移动端≈70.9ms vs 16.3ms）。同时我们看到，FastViT中等尺寸模型（如SA24、SA36）在Mask R-CNN上的检测精度也全面超过了同级别的ResNet-50/101和PoolFormer等主干。综上，在高水平下，FastViT主干可以**以显著更低的延迟**实现与最先进CNN/Transformer相当甚至更优的检测和分割性能，使高精度视觉模型更贴近实际部署要求。

## 5. Conclusion

作者提出了一种通用的混合视觉Transformer **FastViT**，能在移动设备和桌面GPU等多种平台上都实现**高效推理**。通过结构重参数化、训练时过参数化和大卷积核等策略，FastViT 在保持模型容量和准确率的同时，将推理时不必要的开销降至最低，达到了目前视觉模型中**顶尖的精度-效率组合**。实验结果证明，FastViT 不仅在标准ImageNet分类上超越已有模型，在检测、分割、3D重建等多任务上也展现出**强大的性能**与**泛化性**，且具备优异的鲁棒性。未来，作者的工作为高效视觉Transformer的设计提供了新思路，有望激发更多在模型重参数化和混合架构方面的研究和应用。

## 个人思考

这篇论文通过一系列巧妙的结构改进，在不牺牲性能的前提下大幅提升了视觉Transformer的推理效率，让我们看到**架构设计对实际部署的重要性**。FastViT 的 **RepMixer** 模块令人印象深刻——它抓住了跳跃连接带来的内存访问瓶颈，通过重参数化彻底将其移除，实现了Transformer架构在移动设备上的**极致加速**。这一思路不局限于视觉Transformer，未来或许也能迁移到其它模型（如NLP中的Transformer、语音模型等）中，通过移除或重构冗余结构来优化推理延迟。

另一个启发点在于**训练与推理分离的设计**。FastViT 的一些结构在训练时“加冕”，在推理时“隐去”（例如训练时过参数化分支、BatchNorm层等），充分利用了训练阶段的冗余来提升模型容量，又不增加推理成本。这种“所见非所得”的设计理念可以在其他模型压缩领域探索，例如结合**量化**和**蒸馏**进一步压缩模型，同时保持性能。

FastViT 展示的**大卷积核卷积**在Transformer框架下的应用也值得关注。这表明即使在Transformer大行其道的今天，卷积算子仍有不可替代的价值：大核卷积提供的鲁棒性和局部建模能力与自注意力形成互补。未来模型或许可以**自适应地决定**在哪些层采用卷积、哪些层采用注意力，以充分利用二者优势。

潜在的应用方面，FastViT 的超高速度和良好精度使其非常适合部署在对时延敏感的场景，如**移动端实时AR**、**视频实时处理**、**无人机或自动驾驶**中的视觉模块等。在这些场景中，每毫秒的节省都十分宝贵，FastViT 提供了一个用小模型实现大作为的范例。

当然，FastViT 也有值得改进之处。例如，在极致追求速度时，FastViT 小模型的准确率仍落后于更大模型，如何在**不大幅增加推理延迟**的情况下进一步提升精度是一个挑战。此外，FastViT 的重参数化策略使模型训练相对复杂一些（比如需要精心调试融合分支的训练），未来或许可以探索**自动化的结构重参数化**方法，降低设计和调参成本。

总的来说，FastViT 体现了通过**结构创新**优化模型的巨大潜力。它让我们意识到，不仅要关注算法层面的改进，**模型结构设计**和**训练-推理策略**的融合也能带来革命性的提升。相信随着这一方向的深入研究，会有更多既高速又高精度的模型涌现，推动视觉AI在资源受限设备上的应用发展。