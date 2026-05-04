# OneSearch: A Preliminary Exploration of the Unified End-to-End Generative Framework for E-commerce Search

**论文链接**: [arXiv:2509.03236](https://arxiv.org/abs/2509.03236)  
**机构**: 快手 (KuaiShou)  
**时间**: 2025年9月

---

## 核心问题：电商搜索的范式革新

传统电商搜索系统的局限：

1. **多阶段级联**：召回→粗排→精排的多阶段流程，各阶段目标割裂；
2. **文本匹配主导**：传统 BM25/ANN 方法依赖关键词匹配，语义理解能力有限；
3. **个性化搜索不足**：用户个性化偏好难以在多阶段系统中全程贯穿。

---

## 核心方案：OneSearch

将电商搜索建模为**端到端生成式检索**任务，直接从用户查询+行为序列生成目标商品的 Semantic ID。

### 统一生成框架

```
用户搜索 Query + 历史行为
        ↓ Encoder
    语义表示
        ↓ Decoder（自回归）
目标商品 Semantic ID（直接生成）
```

- 消除传统召回-排序-重排的多阶段流程；
- 将搜索任务与推荐任务在同一生成式框架内统一；
- 支持商品语义 ID（Semantic Item ID，SID）直接检索。

### Semantic ID 方案

延续 OneRec 系列的 Residual K-Means 量化方案，为电商商品生成多级语义 ID：

- 融合商品标题、类目、品牌、价格等多维属性；
- 生成具有电商语义的层级 Semantic ID；
- 支持从查询到 ID 的自回归生成。

### 搜索与推荐的统一

OneSearch 的一个重要探索方向：**搜索和推荐能否共享一个生成式模型？**

- 搜索任务：从 query 生成目标商品 ID；
- 推荐任务：从用户行为序列生成下一个感兴趣商品 ID；
- 两类任务在 Encoder-Decoder 框架内通过**任务标识符（task token）**区分；
- 共享的 item 语义空间使得搜索信号和推荐信号相互增强。

---

## 演进：OneSearch-V2（2026）

OneSearch 发表后，快手进一步提出 **OneSearch-V2**：

- **Latent Reasoning Enhanced Self-distillation**：引入隐空间推理增强；
- **自蒸馏框架**：模型自我提升，减少信息茧房和长尾稀疏问题；
- 有效缓解搜索系统中的"信息茧房"和"长尾稀疏"问题，**不增加推理延迟**。

---

## 与 One系列 的关系

OneSearch 将 One系列 的端到端生成范式扩展到**搜索领域**，与 OneSug 构成快手电商搜索的完整生成式解决方案：

| 系统 | 场景 | 任务 |
|------|------|------|
| OneSug | 电商搜索 | Query 推荐（输入补全） |
| OneSearch | 电商搜索 | 商品检索（从 query 到商品） |
| OneMall | 电商推荐 | 商品推荐（从行为到商品） |

---

## 相关论文

- [OneRec (arXiv:2502.18965)](https://arxiv.org/abs/2502.18965) — One系列基础
- [OneSug (AAAI 2026)](https://arxiv.org/abs/2506.06233) — 电商查询推荐
- [OneMall (arXiv:2601.21770)](https://arxiv.org/abs/2601.21770) — 电商多场景推荐
