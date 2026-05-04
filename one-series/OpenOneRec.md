# OpenOneRec Technical Report: A Foundation Model for Generative Recommendation

**论文链接**: [arXiv:2512.24762](https://arxiv.org/abs/2512.24762)  
**机构**: 快手 (KuaiShou)  
**时间**: 2025年12月

---

## 背景：从专用模型到通用基础模型

OneRec 系列前几版均针对**快手短视频推荐**特定场景优化。OpenOneRec 的目标是：

- 将 OneRec 技术栈**开源**，向社区开放；
- 构建一个具有**通用性**的推荐基础模型（Foundation Model for Recommendation）；
- 支持跨领域（cross-domain）推荐能力迁移；
- 向开放推荐通用智能（Open Recommendation General Intelligence）迈进。

---

## 核心框架

### 统一的推荐基础模型

OpenOneRec 将推荐任务建模为**自回归序列生成**问题，统一处理多类型推荐任务：

```
Pre-Training
├── Itemic-Text Alignment（物品-文本对齐预训练）
└── 推荐领域数据 + 通用领域数据联合训练

Post-Training
├── SFT（监督微调，解锁多种下游任务）
└── RL（交替：通用蒸馏 + 强化学习）
    ├── 保持通用推理能力
    └── 增强推荐专项能力
```

### 物品 Token 化：RQ-Kmeans

延续 OneRec 系列的 `RQ-Kmeans` 方案，将 item metadata 的语义 embedding 离散化为 discrete codes（`<|item_begin|><item_a_5028><item_b_6733><item_c_2559><|item_end|>`）。

### RecIF-Bench：4 层评估体系

OpenOneRec 提出专用 benchmark **RecIF-Bench**，分为 4 层、8 类任务：

| 层次 | 能力 | 任务 |
|------|------|------|
| Layer 0 | 语义对齐 | Item描述↔ItemToken 双向理解 |
| Layer 1 | 基础推荐 | 捕捉用户偏好，预测交互行为 |
| Layer 2 | 指令遵循 | 自然语言推荐任务的指令适应 |
| Layer 3 | 推理能力 | 生成自然语言推荐理由 |

### 训练数据：三类推荐领域数据

1. **Itemic Dense Caption Data**：增强模型对 item 的语义理解；
2. **Sequential User Behavior Data**：基础推荐能力核心语料；
3. **Interleaved User Persona Grounding Data**：构建量化空间的语义 grounding。

---

## 开源意义

1. **技术普惠**：将工业级生成式推荐技术向学术界和中小企业开放；
2. **基准建立**：RecIF-Bench 为社区提供统一评估框架；
3. **推动范式转变**：从"ID-based 推荐"向"LLM-based 生成式推荐"演进的公共基础；
4. **社区共建**：吸引研究者在 OpenOneRec 基础上探索推理增强、多模态融合等方向。

---

## 相关论文

- [OneRec (arXiv:2502.18965)](https://arxiv.org/abs/2502.18965) — 工业版起点
- [OneRec-V2 (arXiv:2508.20900)](https://arxiv.org/abs/2508.20900) — Decoder-Only 架构
- [OneRec-Think (arXiv:2510.11639)](https://arxiv.org/abs/2510.11639) — 推理增强版
