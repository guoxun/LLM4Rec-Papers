# OneLoc: Geo-Aware Generative Recommender Systems for Local Life Service

**论文链接**: [arXiv:2508.14646](https://arxiv.org/abs/2508.14646)  
**机构**: 快手 (KuaiShou)  
**作者**: Zichao Wei, Kuo Cai, Jiqing She, Jieming Chen, Mingchen Ma, Yafang Zhu, Qiang Luo, Weilin Zeng, Rongqian Tang, Kun Gai 等  
**时间**: 2025年8月

---

## 核心问题：本地生活推荐的地理感知

本地生活（外卖、到店、周边娱乐）推荐与短视频推荐的本质区别：

1. **地理位置约束**：用户只对附近的商家/服务感兴趣，距离是硬约束；
2. **用户兴趣与地理的耦合**：同一用户在不同位置有不同偏好；
3. **传统方案的局限**：地理信息仅作为独立 ID 特征，无法深度融合进生成过程。

快手平台服务 **4亿 DAU**，本地生活服务是快手重要的商业化场景。

---

## 核心方案：OneLoc（One Model for Local Life Service）

端到端生成式推荐模型，专为本地生活场景设计，将地理信息深度融入生成框架的三个层次。

### 三层地理感知设计

#### 1. Geo-Aware Semantic IDs（表示层）

在生成 Semantic ID 时融入地理信息：

- 在多模态 item embedding 中引入地理位置（经纬度、行政区划等）；
- 生成的 Semantic ID 天然包含地理语义；
- 相邻地理位置的商家在 ID 空间中也具有相似性。

#### 2. Geo-Aware Self-Attention（Encoder 注意力层）

修改 Encoder 的自注意力机制：

- 为用户历史行为序列中的每个 item 引入地理位置 bias；
- 使模型在编码用户兴趣时自动感知行为的地理分布；
- 捕捉用户在不同区域的差异化偏好。

#### 3. Neighbor-Aware Prompt（Decoder 提示层）

在 Decoder 生成阶段引入地理邻域信息：

- 将用户当前位置的邻域商家作为 prompt 输入；
- 约束 Decoder 仅在地理可达范围内生成推荐；
- 避免推荐距离过远的商家。

### 两阶段训练范式

```
Pre-training（NTP 基础推荐能力）
    ↓
Post-training（RL 业务价值对齐）
```

强化学习阶段优化商业目标（GMV、订单量等），而非仅仅点击率。

---

## 在线实验结果

**离线（KuaiLLSR 数据集）**：

| 指标 | 提升 |
|------|------|
| Recall@5 | +13.46% |
| Recall@20 | +10.44% |
| NDCG@5 | +14.47% |
| NDCG@20 | +12.97% |

**Foursquare 数据集**（跨平台泛化）：
- Recall@5 平均提升 +13.18%
- NDCG@5 平均提升 +16.34%

**线上业务指标**：GMV 显著提升，订单量提升，新用户获取效率提升。

---

## 与 One系列 的关系

OneLoc 是 OneRec 在**本地生活场景**的专项扩展，证明了 One系列 架构的**场景可迁移性**：

```
OneRec（短视频）→ OneLoc（本地生活）→ OneMall（电商）→ ...
```

地理感知设计为后续多场景统一（如 OneMall 的多场景 token）提供了参考。

---

## 相关论文

- [OneRec (arXiv:2502.18965)](https://arxiv.org/abs/2502.18965) — One系列基础版
- [OneMall (arXiv:2601.21770)](https://arxiv.org/abs/2601.21770) — 电商多场景
- [OneSearch (arXiv:2509.03236)](https://arxiv.org/abs/2509.03236) — 电商搜索
