# One系列论文解读

快手 (KuaiShou) **One系列**端到端生成式推荐论文精选解读，同步自知乎专栏 [@郭珣](https://www.zhihu.com/people/guoxun)。

> **One系列**是快手在生成式推荐领域的系统性研究成果，核心理念：**用一个统一的端到端生成式模型，替代传统多阶段级联架构，实现"一个模型覆盖更多场景"。**

---

## 背景：为什么是 One系列？

传统推荐系统采用多阶段级联架构（召回→粗排→精排），存在根本性局限：

- **优化目标割裂**：各阶段独立优化，全局最优无从保证；
- **计算碎片化**：无法充分利用 GPU/TPU 算力，MFU 长期处于个位数；
- **技术代差拉大**：LLM 领域的 Scaling Law、强化学习等突破难以迁移；
- **前链路制约上限**：前序过滤的 item 无法被后续阶段召回。

One系列以快手 **OneRec** 为起点，首次在真实大规模工业场景中**显著超越**传统级联推荐系统，开启了端到端生成式推荐的工业化新纪元。

---

## One系列论文全景

### 核心模型演进路线

```
OneRec（2025.02）→ OneRec-V2（2025.08）→ OneRec-Think（2025.10）→ OpenOneRec（2025.12）
```

### 多场景扩展路线

```
OneRec（短视频推荐）
    ├── OneLoc（本地生活）
    ├── OneMall（电商多场景）
    ├── OneSug（电商查询推荐）
    └── OneSearch（电商商品搜索）
```

---

## 论文列表

### 核心技术演进

| 论文 | arXiv/会议 | 场景 | 核心亮点 |
|------|-----------|------|---------|
| [OneRec](one-series/OneRec.md) | arXiv:2502.18965 | 短视频推荐 | 首个工业级端到端生成式推荐，停留时长 +0.54%~1.24% |
| [OneRec-V2](one-series/OneRec_V2.md) | arXiv:2508.20900 | 短视频推荐 | Decoder-Only 架构 + ECPO RL 增强 |
| [OneRec-Think](one-series/OneRec_Think.md) | arXiv:2510.11639 | 短视频推荐 | 显式 in-text 推理链 + 对话式交互 |
| [OpenOneRec](one-series/OpenOneRec.md) | arXiv:2512.24762 | 通用 | 开源基础模型 + RecIF-Bench 评估体系 |

### 多场景扩展

| 论文 | arXiv/会议 | 场景 | 核心亮点 |
|------|-----------|------|---------|
| [OneLoc](one-series/OneLoc.md) | arXiv:2508.14646 | 本地生活 | 三层地理感知设计，Recall@5 +13.46% |
| [OneMall](one-series/OneMall.md) | arXiv:2601.21770 | 电商多场景 | 商品卡+短视频+直播间三场景统一 |
| [OneSug](one-series/OneSug.md) | AAAI 2026 | 电商查询推荐 | 全量部署 6 个月，延迟降低 43.2% |
| [OneSearch](one-series/OneSearch.md) | arXiv:2509.03236 | 电商搜索 | 搜索与推荐统一生成框架 |

---

## One系列的核心技术体系

### 1. Semantic ID：物品的语言

One系列用 **Semantic ID（SID）** 将推荐问题转化为语言模型可理解的 token 序列：

- **RQ-Kmeans（Residual K-Means）**：多级残差量化，解决 RQ-VAE 的 codebook collapse 问题；
- **多模态融合**：标题、标签、ASR、图像 + 协同过滤信号；
- **层级语义结构**：从粗到细的 3 级量化，保留语义层次。

### 2. Session-wise 生成 vs Point-wise 生成

传统生成式推荐（如 TIGER）逐个预测下一个 item。OneRec 直接**生成一个完整的 session**：

```
Point-wise：预测 item_1，再预测 item_2，...（独立）
Session-wise：一次性生成 [item_1, item_2, ..., item_k]（上下文连贯）
```

### 3. 偏好对齐：从 DPO 到 ECPO

推荐场景无法直接获取正负样本对（不像 NLP 可以让人类对比两个回答）：

| 方法 | 特点 |
|------|------|
| IPA（OneRec v1）| RM 打分 + 迭代 DPO |
| ECPO（OneRec-V2）| 改进 GRPO，对负优势样本严格截断 |

### 4. MFU 革命

One系列将推荐系统 MFU 从个位数提升至与 LLM 接近的水平：

| 系统 | MFU |
|------|-----|
| 传统推荐模型 | < 5% |
| OneRec 训练 | **23.7%** |
| OneRec 推理 | **28.6%** |

---

## 延伸阅读

- [快手 AAAI 2026 论文集锦（含 One系列相关工作）](https://juejin.cn/post/7594047593967452198)
- [生成式推荐综述 | recsys-frontier.com](https://www.recsys-frontier.com/article/generative-recommendation-survey)
- [从 Tokenization 视角看生成式推荐发展（arthurchiao）](https://arthurchiao.art/blog/large-generative-recommendation-tokenization-perspective-notes-zh/)

---

## 关于本仓库

- 论文解读均以 Markdown 格式存档，便于离线阅读与检索；
- 内容聚焦快手 One系列端到端生成式推荐方向；
- 如有引用或转载，请注明原始来源。
