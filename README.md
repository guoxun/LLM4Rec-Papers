# LLM4Rec 必读论文解读：One系列 & 工业精排

快手 **One系列**端到端生成式推荐 + 相关工业推荐论文解读，同步自知乎专栏 [@GuoXun](https://www.zhihu.com/people/guo-xun-16)。

---

## 写给不同背景的读者

> **刚入门的同学**：先读「[背景：推荐系统的困境](#背景推荐系统的困境)」，再看「[五分钟搞懂 One系列](#五分钟搞懂-one系列)」，之后按兴趣读具体论文。
>
> **有经验的算法 RD**：直接看「[论文列表](#论文列表)」和「[核心技术对比](#核心技术对比)」，按需深入。
>
> **技术 Leader / 架构师**：重点看「[两条技术路线的对比](#两条技术路线：生成式-vs-判别式)」和每篇论文的在线实验结果。

---

## 背景：推荐系统的困境

### 现状：多阶段级联架构

过去十年，几乎所有大厂的推荐系统都长这个样子：

```
海量候选（亿级）
    ↓ 召回（百万→万级）：向量检索、协同过滤
    ↓ 粗排（万→千级）：轻量模型打分
    ↓ 精排（千→百级）：重模型精细打分
    ↓ 重排（百→十级）：多样性/策略调整
    ↓ 最终展示（十条）
```

这套架构在过去很好用，但随着 AI 技术的发展，四个问题越来越突出：

| 问题 | 具体表现 |
|------|---------|
| **优化目标割裂** | 各阶段独立优化不同目标，全局最优无从保证 |
| **前链路制约上限** | 召回阶段漏掉的 item，精排再强也无法挽回 |
| **GPU 算力浪费** | 大量 CPU 时代遗留的特征工程模块，MFU 仅 ~5% |
| **技术代差拉大** | LLM 的 Scaling Law、RL、MoE 等突破，难以迁移进来 |

### 工业界的两条破局路线

2024-2025 年，快手和抖音分别给出了自己的答案：

```
快手（One系列）：颠覆式
    → 用一个端到端生成式模型，直接替换整个多阶段系统

抖音（RankMixer）：渐进式
    → 重新设计精排模型架构，让判别式模型也能 Scaling
```

---

## 五分钟搞懂 One系列

### 核心思想：把推荐变成"写作文"

传统推荐：给候选 item 打分，选分最高的。

**OneRec 的做法**：把推荐系统变成一个语言模型——

```
输入：用户最近看过的视频序列（token 化）
输出：直接"写出"用户接下来最想看的视频列表
```

就像语言模型根据上文预测下一个词，OneRec 根据用户历史行为预测下一批推荐内容。

### 三个关键技术点

**1. 视频怎么变成 token？**

把每个视频用 3 个数字（层级 token）来代表——类似汉字的部首+偏旁+笔画：

```
视频 A = [token_12, token_847, token_3291]
视频 B = [token_12, token_851, token_4107]  ← token_12 相同，说明 A/B 是同类视频
```

这套编码方式（RQ-Kmeans）同时融合了视频内容（多模态）和用户行为（协同过滤）信息。

**2. 生成一个列表 vs 逐个预测**

传统生成式推荐：逐个预测 item_1，再预测 item_2，... 各自独立。

OneRec：一次性生成一个完整的"session"（用户一次刷视频的完整列表），列表内各 item 相互感知，更自然更合理。

**3. 用 RL 对齐用户真实偏好**

NTP 预训练只能学历史分布，无法超越过去推荐系统的天花板。OneRec 用强化学习（类似 ChatGPT 的 RLHF）让模型直接优化用户满意度指标（停留时长、LT7 等）。

---

## 论文列表

### One系列核心技术演进（快手）

| 论文 | arXiv | 时间 | 一句话概括 |
|------|-------|------|-----------|
| [QARM](llm4rec/QARM.md) | [2411.11739](https://arxiv.org/abs/2411.11739) | 2024.11 | 解决多模态表示与推荐目标不对齐的问题，广告 Revenue +9.7% |
| QARM V2 | [2602.08559](https://arxiv.org/abs/2602.08559) | 2026.02 | 引入推理增强的用户序列建模，短视频和直播场景进一步提升 |
| [OneRec](llm4rec/OneRec.md) | [2502.18965](https://arxiv.org/abs/2502.18965) | 2025.02 | 首个工业级端到端生成式推荐，替换整个多阶段 Pipeline |
| [OneRec Technical Report](llm4rec/OneRec_TechReport.md) | [2506.13695](https://arxiv.org/abs/2506.13695) | 2025.06 | 完整工程报告：10× FLOPs、Scaling Law、MFU 23.7%/28.8% |
| [OneRec-V2](llm4rec/OneRec_V2.md) | [2508.20900](https://arxiv.org/abs/2508.20900) | 2025.08 | Lazy Decoder-Only 架构，计算量减少 94%，参数扩至 8B |
| [OneRec-Think](llm4rec/OneRec_Think.md) | [2510.11639](https://arxiv.org/abs/2510.11639) | 2025.10 | 引入 CoT 推理链，推荐结果可解释，支持对话式交互 |
| [OpenOneRec](llm4rec/OpenOneRec.md) | [2512.24762](https://arxiv.org/abs/2512.24762) | 2025.12 | 开源基础模型 + RecIF-Bench 评估体系 |

### One系列多场景扩展（快手）

| 论文 | arXiv | 时间 | 一句话概括 |
|------|-------|------|-----------|
| [OneLoc](llm4rec/OneLoc.md) | [2508.14646](https://arxiv.org/abs/2508.14646) | 2025.08 | 本地生活推荐，三层地理感知设计，Recall@5 +13.46% |
| [OneMall](llm4rec/OneMall.md) | [2601.21770](https://arxiv.org/abs/2601.21770) | 2026.01 | 电商三大场景统一（商品卡+短视频+直播间）|
| [OneSug](llm4rec/OneSug.md) | [AAAI 2026](https://arxiv.org/abs/2506.06233) | 2025/2026 | 电商 Query 推荐，业界首个全量 Gen 系统，延迟 -43.2% |
| [OneSearch](llm4rec/OneSearch.md) | [2509.03236](https://arxiv.org/abs/2509.03236) | 2025.09 | 电商商品搜索，搜索与推荐统一生成框架 |

### 工业精排 Scaling（抖音）

| 论文 | arXiv | 时间 | 一句话概括 |
|------|-------|------|-----------|
| [RankMixer](llm4rec/RankMixer.md) | [2507.15551](https://arxiv.org/abs/2507.15551) | 2025.07 | 判别式精排的 Scaling 方案，MFU 从 4.5% → 45%，1B 参数全量部署 |

---

## 核心技术对比

### MFU（GPU 算力利用率）对比

MFU 是衡量"我们是否在白白浪费 GPU"的核心指标。越高越好，LLM 训练通常在 30-60%。

```
传统推荐精排（DIN/DCN/DLRM 等）：约 4.5%
                    ████░░░░░░░░░░░░░░░░░░░░░░░░░░ 4.5%

OneRec 训练：23.7%
                    ████████████░░░░░░░░░░░░░░░░░░ 23.7%（快手）

OneRec 推理：28.8%
                    ██████████████░░░░░░░░░░░░░░░░ 28.8%（快手）

RankMixer：约 45%
                    ████████████████████████░░░░░░ 45%（抖音）
```

### 在线业务指标汇总

| 系统 | 指标 | 数值 | 说明 |
|------|------|------|------|
| OneRec (快手) | App 停留时长 | +0.54% / +1.24% | 快手/极速版，0.1% 具统计显著性 |
| OneRec (快手) | 7日用户生命周期 LT7 | +0.05% / +0.08% | 长期留存提升 |
| OneRec (快手本地生活) | GMV | +21.01% | 已100%切流 |
| OneLoc (快手) | Recall@5 | +13.46% | 本地生活离线 |
| OneSug (快手电商) | 延迟 | -43.2% | 替换多阶段 Pipeline |
| QARM (快手广告) | Revenue | +9.704% | 广告收入 |
| QARM (快手电商) | GMV | +2.296% | 电商成交额 |
| RankMixer (抖音) | 活跃天数 | +0.3% | 数千万用户级 |
| RankMixer (抖音) | App 使用时长 | +1.08% | Feed 推荐 |
| RankMixer (抖音广告) | ADVV | +3.90% | 广告收入价值 |

---

## 核心技术深度解读

### 1. Semantic ID：如何把视频变成 token

这是 One系列的基础。思路类比：

> 传统推荐：每个视频有一个唯一 ID（类似身份证号），ID 之间没有语义关系  
> Semantic ID：视频的 ID 由其内容决定，相似内容的视频有相近的 ID

**QARM（2024）→ OneRec 系列 的演进**：

```
QARM: MLLM 微调（推荐对齐）+ K-Means VQ/RQ 量化
         ↓ 技术升级
OneRec: RQ-Kmeans（多级残差 K-Means）+ 多模态融合 + 协同信号

量化层级（以 OneRec 为例）：
Level 1（粗粒度）: [0-511]  → 划分 512 个大类（如：搞笑类/美食类）
Level 2（中粒度）: [0-511]  → 在大类内细分
Level 3（细粒度）: [0-511]  → 精细区分相似内容
视频 = [L1_token, L2_token, L3_token]  = 3 个整数
```

**为什么不用 RQ-VAE（传统方案）**：RQ-VAE 存在"沙漏现象"（codebook collapse）——大量 token 从未被使用，实际有效码本极小。RQ-Kmeans 通过强制每个 cluster 包含等量 item 来解决此问题。

### 2. 生成式推荐 vs 判别式推荐

**判别式（传统，RankMixer 优化的方向）**：
```
问题形式：给定用户 U 和物品 I，预测 P(click | U, I)
优化目标：二分类 / 回归
特点：每个 item 独立打分，逐一比较
```

**生成式（OneRec 的方向）**：
```
问题形式：给定用户历史 H，直接生成推荐序列 [I_1, I_2, ..., I_k]
优化目标：Next Token Prediction（序列生成）
特点：session 内 item 相互感知，整体最优
```

### 3. RL 偏好对齐：三代演进

| 版本 | 方法 | 核心思路 |
|------|------|---------|
| OneRec v1 | IPA（迭代 DPO） | RM 打分 + 构造偏好对 + DPO 优化 |
| OneRec-V2 | ECPO（Early-Clipped GRPO）| 对负优势样本严格截断，训练更稳定 |
| OneRec-Think | GRPO/DAPO/VAPO | 优化推理行为，生成 CoT 推理链 |

**为什么推荐场景的 RL 特别难**：NLP 的 RLHF 可以让人类对比两个回答打分，但推荐系统每次请求只有一次曝光机会——没有对照组，无法直接获取偏好数据。One系列用奖励模型（RM）来构造"虚拟的偏好对"，再用 RL 优化。

### 4. Scaling：推荐系统的规模定律

**什么是 Scaling Law**：在 LLM 中，"数据量、参数量、计算量越大，效果越好"是一个可量化的规律。

**推荐系统的挑战**：传统架构（DLRM/DIN 等）参数量翻 10 倍，推理成本也翻 10 倍，MFU 还是 5%，完全没有"规模红利"。

**One系列/RankMixer 的解法**：

```
OneRec（快手）的三维 Scaling：
├── Feature Scaling：更多、更丰富的特征
├── Codebook Scaling：更大的 Semantic ID 码本
└── Infer Scaling：推理阶段用更大的 beam size

RankMixer（抖音）的 Scaling：
├── Dense Scaling：Token Mixing + Per-Token FFN 支持等比扩参
└── Sparse MoE Scaling：8× 参数量，推理成本不变
```

---

## 两条技术路线：生成式 vs 判别式

| 对比维度 | 生成式（OneRec，快手）| 判别式（RankMixer，抖音）|
|---------|----------------------|------------------------|
| **核心假设** | 端到端生成能超越多阶段 | 精排模型本身就能 Scaling |
| **对现有系统影响** | 替换整个 Pipeline（大改）| 只换精排模型（小改）|
| **优点** | 全局最优，LLM 技术可直迁 | 迁移成本低，风险小 |
| **难点** | 工程复杂，需重构整个链路 | Scaling 收益有上限（召回仍是瓶颈）|
| **适合场景** | 底层架构重构、新业务 | 现有系统迭代优化 |

**两条路线不是非此即彼**：一个成熟的推荐系统可以同时在不同阶段采用两种思路。

---

## 快速阅读指南

### 如果你只有 30 分钟

1. 读 [OneRec Technical Report](llm4rec/OneRec_TechReport.md) 的"核心发现"部分
2. 看 [RankMixer](llm4rec/RankMixer.md) 的"两大障碍"和"在线结果"
3. 了解 [QARM](llm4rec/QARM.md) 的"核心问题"——理解为什么多模态推荐不那么简单

### 如果你想深入生成式推荐

按顺序读：QARM → OneRec → OneRec Technical Report → OneRec-V2 → OneRec-Think → OpenOneRec

### 如果你负责精排/CTR 模型

重点读：RankMixer（判别式 Scaling） + OneRec Technical Report（理解生成式方向）

### 如果你关注多场景推荐

读：OneLoc（地理场景）→ OneMall（电商多场景）→ OneSug（查询推荐）→ OneSearch（搜索）

---

## 延伸阅读

- [2025年生成式推荐论文一览（知乎）](https://zhuanlan.zhihu.com/p/1983522398591554332)
- [生成式推荐 Survey（recsys-frontier）](https://www.recsys-frontier.com/article/generative-recommendation-survey) — 详细梳理从 TIGER 到 OneRec 的完整发展脉络
- [快手 AAAI 2026 论文集锦](https://juejin.cn/post/7594047593967452198) — 含 One系列相关工作全列表
- [OpenOneRec 解读（arthurchiao）](https://arthurchiao.art/blog/openonerec-tech-report-notes-zh/)

---

## 关于本仓库

- 论文解读以 Markdown 格式存档，参考 RSPapers 体系；
- 内容聚焦快手 One系列 + 相关工业大厂顶级工作；
- 欢迎补充 PR 或在 [Issue](https://github.com/guoxun/LLM4Rec-Papers/issues) 讨论；
- 如有引用或转载，请注明原始来源。
