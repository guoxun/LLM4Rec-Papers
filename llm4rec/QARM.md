# QARM: Quantitative Alignment Multi-Modal Recommendation at Kuaishou

**论文链接**: [arXiv:2411.11739](https://arxiv.org/abs/2411.11739)  
**机构**: 快手 (KuaiShou)  
**作者**: Xinchen Luo, Jingcai Cao, Tao Sun, Jie Yu, Rui Huang, Wenrui Yuan, Houkai Lin, Yafei Zheng, Shiyao Wang, Qigen Hu, Caizhi Qiu, Jiashu Zhang, Xuanhua Zhang, Zhuoran Yan, Jian Zhang, Sen Zhang, Mingwei Wen, Zhenbang Liu, Kun Gai, Guorui Zhou  
**时间**: 2024年11月

---

## 为什么看这篇？与 OneRec 的关系

QARM 解决的是 **OneRec 系列的"上游"问题**：如何让多模态大模型（MLLM）的表示真正对推荐任务有用？

```
多模态内容（视频/图文）
        ↓ MLLM 编码
   多模态 Embedding
        ↓ QARM（本文）：对齐 + 量化
   Semantic ID（可学习、与推荐目标对齐）
        ↓ 输入
   OneRec / 传统推荐模型
```

QARM 生成的量化表示，正是 OneRec 系列 RQ-Kmeans 语义 ID 方案的**前身和基础**。

---

## 核心问题：多模态表示的两大困境

### 问题1：Representation Unmatching（表示不匹配）

- MLLM 用 NLP/CV 任务（图文匹配、字幕生成等）预训练；
- 推荐模型用真实用户-物品交互监督；
- **两种任务目标根本不同**，导致 MLLM 表示与推荐需求脱节；
- 例如：MLLM 认为"苹果手机"和"苹果水果"相似（语义相似），但推荐系统中用户行为表明二者完全不同。

### 问题2：Representation Unlearning（表示无法学习）

- 传统方案将多模态表示**存入缓存**，作为推荐模型的固定输入；
- 推荐模型的梯度**无法反向传播**到多模态模型；
- 多模态表示永远固定，无法根据推荐目标持续优化；
- 导致"用了多模态但提升有限"的工业常见困境。

---

## 核心方案：两步走

### 第一步：Item Alignment（物品对齐）

**目标**：让 MLLM 的表示空间与推荐任务对齐。

**方法**：挖掘推荐场景下的高质量**相似 item pair**，用对比学习微调 MLLM：

```
相似 item pair 来源：
├── U2I 检索模型：为每个用户的正向点击 item 找最相似的 item
└── I2I 检索模型：为每个 item 找高相似度的 item

对比学习损失：
L_align = BatchContrastive(M_trigger, M_target, B)
```

- `M_trigger`：触发 item 的 MLLM 表示；
- `M_target`：目标 item 的 MLLM 表示；
- 批量对比学习，使推荐相关的 item 在表示空间中靠近。

**本质**：用**推荐系统积累的协同过滤知识**来"教"MLLM 什么叫做推荐意义上的"相似"。

### 第二步：Quantitative Code（量化编码）

**目标**：将对齐后的连续向量量化为**可学习的离散 Semantic ID**，使推荐模型能端到端更新多模态表示。

**方法**：不使用 RQ-VAE（训练复杂），而是直接在对齐后的 embedding 上用 K-Means 聚类：

**VQ（Vector Quantization，向量量化）**：
```
v = NearestCode(R, m)
R ∈ R^{K×d}  # K 个聚类中心构成码本
v ∈ {1, ..., K}  # 最近邻码本索引
```

**RQ（Residual Quantization，残差量化）**：
```
r_1 = NearestCode(R^1, m, 1)
r_2 = NearestCode(R^2, m^1, 1)  # m^1 = m - R^1_{r_1}（残差）
...
# L 级码本，从粗到细
```

**关键创新**：码本通过 K-Means 启发式生成（而非 VAE 训练），因为 MLLM 已经过对齐微调，表示质量有保证，无需复杂的 VAE 解码器。

---

## 端到端训练的实现

量化后的 Semantic ID 作为推荐模型的**可学习输入**：

- 推荐模型（CTR/精排）的梯度可通过 **Straight-Through Estimator（STE）** 近似反传回多模态表示；
- 多模态 Embedding 因此可以随着推荐任务持续更新；
- 彻底解决"Representation Unlearning"问题。

---

## 在线实验结果（快手平台）

| 场景 | 指标 | 提升 |
|------|------|------|
| 广告 | Revenue | **+9.704%** |
| 电商 | GMV | **+2.296%** |

> 广告 Revenue +9.7% 在工业推荐系统中属于极大提升，通常超过 1% 就被认为极具显著性。

---

## 技术传承：QARM → OneRec 系列

QARM 的核心思想直接影响了 OneRec 系列的设计：

| 技术 | QARM | OneRec 系列 |
|------|------|------------|
| 量化方案 | K-Means VQ/RQ | RQ-Kmeans（多级残差 K-Means）|
| 对齐思路 | Item-Item 对比学习 | 协同感知多模态 Tokenizer |
| 端到端训练 | STE 梯度传播 | 原生端到端 NTP 训练 |
| 目标 | 多模态 Embedding 可学习化 | 统一 Semantic ID 生成 |

QARM 是快手从"多模态辅助 ID 推荐"过渡到"纯生成式推荐（OneRec）"的关键技术桥梁。

---

## 后续演进：QARM V2（arXiv:2602.08559）

QARM V2 在 V1 基础上引入**推理增强的用户序列建模**：

- 更强的用户行为序列理解；
- 融合多模态内容的推理能力；
- 在快手短视频和直播场景均取得进一步提升。

---

## 相关论文

- [OneRec (arXiv:2502.18965)](https://arxiv.org/abs/2502.18965) — QARM 的下游受益者
- [OneRec Technical Report (arXiv:2506.13695)](https://arxiv.org/abs/2506.13695) — Semantic ID 的完整生产方案
- [QARM V2 (arXiv:2602.08559)](https://arxiv.org/abs/2602.08559) — 推理增强升级版
