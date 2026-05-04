# OneSug: The Unified End-to-End Generative Framework for E-commerce Query Suggestion

**论文链接**: [AAAI 2026](https://ojs.aaai.org/index.php/AAAI/article/download/38497/42459) | [arXiv](https://arxiv.org/pdf/2506.06233)  
**机构**: 快手 (KuaiShou) + 中国科学技术大学  
**作者**: Xian Guo, Ben Chen*, Siyuan Wang, Ying Yang, Mingyue Cheng, Chenyi Lei, Yuqing Ding, Han Li  
**会议**: AAAI 2026  
**时间**: 2025年/2026年

---

## 核心问题：查询推荐（Query Suggestion）的级联困境

电商搜索场景中，Query Suggestion（Sug）系统帮助用户快速定位购买意图（如输入"苹果"→推荐"苹果手机"或"苹果水果"）。

传统多阶段级联架构（MCA）的问题：

1. **各阶段目标不一致**：召回、粗排、精排各自优化不同目标，系统整体效率受限；
2. **长尾 query 召回困难**：冷门 query 在早期阶段被过滤，无法进入精排；
3. **迭代成本高**：多阶段分别维护和更新，工程负担重。

---

## 核心方案：OneSug

**业界首个在电商场景全量部署的端到端生成式 Query 推荐系统**（快手电商搜索，稳定承载全流量超半年）。

### 三大核心模块

#### 1. Prefix-Query 表征增强模块

用户输入的 prefix 往往短且意图模糊（如"苹果"可指水果或品牌）：

- **语义与业务空间对齐**：融合商品语义信息与用户交互历史信号；
- **层次化语义 ID 生成**：将 query prefix 映射到多粒度语义表示空间；
- 充分挖掘用户的真实搜索意图。

#### 2. 统一 Enc-Dec 生成架构

```
用户 Prefix + 行为历史
        ↓ Encoder
    上下文表示
        ↓ Decoder（自回归）
推荐 Query（直接生成最可能点击的 query）
```

- 端到端直接生成用户最可能点击的 Query；
- 完全绕过传统召回-粗排-精排三阶段；
- 同样适用于 Decoder-Only 架构（离线指标更高，待在线验证）。

#### 3. 用户行为偏好对齐（RWR：Reward-Weighted Reinforcement）

区别于 NLP 中的 RLHF，推荐场景的奖励信号来源于用户真实点击行为：

- **正负样本区分策略**：区分"点击"与"曝光"比区分"点击"与"随机负样本"难度更高，更有学习价值；
- **行为分档**：对用户行为按质量分级（高质量点击 vs 普通曝光 vs 随机）；
- **奖励加权优化**：根据正负样本间奖励差距加权梯度，精准刻画个性化偏好。

---

## 在线实验结果（快手电商搜索）

**在线 A/B 测试**（与传统多阶段系统对比）：

| 指标 | 提升 |
|------|------|
| CTR | 显著提升 |
| 订单量 | 显著提升 |
| GMV | 显著提升 |
| 人工 GSB 评测 | 大幅提升 |

**推理效率**：
- 在线流程完全取代召回-粗排-精排；
- **平均耗时降低 43.2%**；
- 为后续优化提供充足空间。

**模型鲁棒性**：随时间增量更新（仅用近3天数据），模型效果衰减显著低于传统方案（-0.6% vs -1.1%）。

---

## 与 One系列 的关系

OneSug 将 One系列 的端到端生成范式扩展到**搜索 Query 推荐**任务，与 OneSearch（商品搜索）共同构建快手电商搜索的生成式推荐矩阵：

```
OneRec（短视频推荐）
OneLoc（本地生活）
OneMall（电商多场景推荐）
OneSug（电商查询推荐）← 本文
OneSearch（电商商品搜索）
```

---

## 相关论文

- [OneRec (arXiv:2502.18965)](https://arxiv.org/abs/2502.18965) — One系列基础
- [OneSearch (arXiv:2509.03236)](https://arxiv.org/abs/2509.03236) — 电商搜索
- [OneMall (arXiv:2601.21770)](https://arxiv.org/abs/2601.21770) — 电商推荐
