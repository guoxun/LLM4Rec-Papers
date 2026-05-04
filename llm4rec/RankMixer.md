# RankMixer: Scaling Up Ranking Models in Industrial Recommenders

**论文链接**: [arXiv:2507.15551](https://arxiv.org/abs/2507.15551)  
**机构**: 抖音 / 字节跳动 (ByteDance)  
**作者**: Jie Zhu, Zhifang Fan, Xiaoxie Zhu, Yuchen Jiang, Hangyu Wang, Xintian Han, Haoran Ding, Xinmin Wang, Wenlin Zhao, Zhen Gong, Huizhi Yang, Zheng Chai, Zhe Chen, Yuchao Zheng, Qiwei Chen†, Feng Zhang, Xun Zhou, Peng Xu, Xiao Yang, Di Wu, Zuotao Liu  
**时间**: 2025年7月

---

## 定位：判别式推荐的 Scaling Law 解法

与 OneRec 系列（**生成式**推荐）不同，RankMixer 走的是**判别式**路线——仍然是传统的 CTR 预估/精排模型，但通过重新设计架构，使得推荐排序模型也能像 LLM 一样 Scaling。

```
         生成式推荐                 判别式推荐（精排）
快手：OneRec 系列            抖音：RankMixer（本文）
（替换整个 Pipeline）        （优化精排模型的 Scaling）
```

---

## 核心问题：为什么传统精排模型无法 Scaling？

### 两大障碍

**障碍1：训练/推理成本受约束**

- 工业推荐精排需要严格满足延迟（< 50ms）和高 QPS 要求；
- 不能无限增加参数量，否则推理延迟超标。

**障碍2：CPU 时代遗留架构**

传统 DLRM 架构（如 DeepFM、DCN）的问题：

```
传统特征交叉模块（如 DCN、FM）
├── 继承自 CPU 时代，大量 memory-bound 算子
├── 无法充分利用 GPU 高并行度
├── MFU 通常仅 4.5%（浪费 95% 的 GPU 算力）
└── 参数量扩大后，计算成本近似线性增长
```

**根本矛盾**：想 Scaling，但架构决定了加参数只会等比增加成本，Scaling Law 的收益无法实现。

---

## 核心方案：RankMixer

**核心思路**：保留 Transformer 的高并行度，用两个轻量模块替换低效的自注意力和 FFN。

### 架构总览

```
输入：异构特征 Embedding（用户、物品、上下文等）
        ↓ Feature Tokenization（特征离散化为 Token）
  [token_1, token_2, ..., token_N]
        ↓ Multi-Head Token Mixing（跨特征交互）
  [mixed_token_1, ..., mixed_token_N]
        ↓ Per-Token FFN（独立特征子空间建模）
  [refined_token_1, ..., refined_token_N]
        ↓ ... × L 层堆叠
        ↓ Output Head
   CTR 预测值
```

### 模块1：Multi-Head Token Mixing（多头 Token 混合）

**替代**传统自注意力（Self-Attention）的交叉特征模块：

```
自注意力：Q·K^T → O(N²) 复杂度，GPU 并行度受限
Token Mixing：共享参数的线性混合，O(N) 复杂度，无额外参数
```

- **无参数**跨特征交互：仅用固定的混合矩阵（learnable）对所有 token 进行线性组合；
- 类似 MLP-Mixer 的思路，但专为推荐系统异构特征设计；
- 多头设计捕捉不同粒度的特征交互，保留表达能力；
- **GPU 友好**：全矩阵乘法，高并行度，MFU 大幅提升。

### 模块2：Per-Token FFN（逐特征独立 FFN）

**替代**传统共享 FFN：

```
传统 FFN：所有 token 共享同一个 FFN（参数利用率低）
Per-Token FFN：每个 token 有独立的 FFN 参数
```

- 每个特征 token 有**独立的 FFN 参数空间**，针对性建模各特征子空间；
- 参数量随特征数量（N）线性增长，但 GPU 可以并行处理所有特征；
- 在不增加推理 FLOPs 的前提下，大幅提升模型容量。

### Sparse MoE 扩展（关键 Scaling 手段）

为了进一步 Scaling 参数量（从 100M → 1B），引入 Sparse MoE：

**两个关键创新**：

1. **DTSI（Dynamic Top-k Sparse Index）**：
   - 动态决定每个 token 激活哪些 expert；
   - 解决 expert 训练不足（部分 expert 几乎从不被激活）问题；

2. **ReLU Routing**：
   - 用 ReLU 代替 Softmax 作为路由函数；
   - 天然稀疏（ReLU 会产生大量 0），不需要手动限制激活 expert 数量；
   - 解决 expert 路由不均衡问题。

```
传统 MoE 问题：
├── Softmax 路由 → 所有 expert 都有非零权重 → 实际不稀疏
└── Expert 不均衡 → 部分 expert 过载，部分 expert 欠训练

DTSI + ReLU 路由：
├── ReLU → 天然稀疏，只有少数 expert 激活
└── DTSI → 动态激活，确保每个 expert 充分训练
```

**效果**：参数量扩大 **8×** 时 AUC 几乎无损，同时推理吞吐量提升 **>50%**。

---

## Scaling Law 验证

在抖音万亿级数据上的实验：

| 参数规模 | AUC | 业务提升 |
|---------|-----|---------|
| 16M（基线）| 基线 | - |
| 100M    | 显著提升 | - |
| 1B（RankMixer）| 明显领先 | 全量上线 |

**Scaling Law 成立**：在 RankMixer 架构上，模型参数增大带来持续的效果提升，且增量可预测。

---

## MFU 对比

| 系统 | MFU |
|------|-----|
| 传统 DLRM/DIN/DCN 等 | **4.5%** |
| RankMixer-1B | **~45%** |

MFU 提升近 **10×**，达到与 OneRec（23.7%/28.8%）相近的量级，彻底改变了判别式推荐的算力利用效率。

---

## 在线实验结果（抖音全量部署）

**推荐信息流（Feed Recommendation）**：

| 指标 | 提升 |
|------|------|
| 活跃天数（Active Days）| **+0.3%** |
| App 使用时长 | **+1.08%** |

> 抖音体量下，0.1% 活跃天数提升代表数千万用户级别的影响。

**广告（Advertising）**：

| 指标 | 提升 |
|------|------|
| AUC | +0.73% |
| ADVV（广告收入价值）| **+3.90%** |

**推理成本**：RankMixer-1B 的推理延迟与 16M 基线**持平**（通过高 MFU + 低 FLOPs/Param 实现）。

---

## RankMixer vs OneRec：两种路径的对比

| 维度 | RankMixer（抖音）| OneRec（快手）|
|------|-----------------|-------------|
| 推荐范式 | **判别式**（CTR 预估/精排）| **生成式**（直接生成推荐列表）|
| 架构风格 | Token Mixing + Per-Token FFN | Encoder-Decoder / Decoder-Only |
| 对现有系统影响 | 替换精排模型，兼容现有 Pipeline | 替换整个多阶段 Pipeline |
| MFU | ~45%（训练）| 23.7%（训练），28.8%（推理）|
| Scaling 方式 | Dense + Sparse MoE | 参数扩展 + 推理 Scaling |
| 核心挑战解决 | GPU 算力利用效率 | 多阶段优化目标割裂 |

**两种方案各有侧重**，代表了工业界推荐系统 Scaling 的两条技术路线：
- **渐进式**（RankMixer）：在保持兼容性的前提下，大幅提升精排模型的规模和效率；
- **颠覆式**（OneRec）：重构整个推荐系统架构，实现端到端优化。

---

## 相关论文

- [OneRec Technical Report (arXiv:2506.13695)](https://arxiv.org/abs/2506.13695) — 生成式推荐的 Scaling 方案
- [HSTU (Meta, ICML 2024)](https://arxiv.org/abs/2402.17152) — 另一条生成式推荐 Scaling 路线
- [Wukong (Meta)](https://arxiv.org/abs/2305.08192) — 判别式推荐 Scaling 的另一方案
