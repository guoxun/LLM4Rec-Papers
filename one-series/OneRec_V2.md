# OneRec-V2 Technical Report

**论文链接**: [arXiv:2508.20900](https://arxiv.org/abs/2508.20900)  
**机构**: 快手 (KuaiShou)  
**时间**: 2025年8月

---

## 背景：OneRec v1 的局限

OneRec 第一版（arXiv:2502.18965）证明了端到端生成式推荐的可行性，但存在以下待改进问题：

1. **推理阶段计算分配不均**：Infer 阶段 step 的 Scaling up 效果不明显；
2. **强化学习能力受限**：RL 对推理过程的影响路径有限；
3. **架构从 Encoder-Decoder 演进为 Decoder-Only**。

---

## 核心改进

### 架构升级：Encoder-Decoder → Decoder-Only

OneRec-V2 将模型架构从 Encoder-Decoder 升级为 **Decoder-Only** 结构，更贴近主流 LLM 技术栈，便于与 LLM 社区的推理增强技术（如 Test-Time Scaling）对接。

### RL 强化：ECPO（Early-Clipped GRPO）

针对推荐场景的稀疏奖励问题，提出 ECPO 优化策略：

- 基于 GRPO 改进，对**负优势（A < 0）样本**进行更严格的策略梯度截断；
- 保留样本的同时防止梯度爆炸，使训练更加稳定；
- 结合 P-Score 奖励显著提升 App 使用时长。

### 计算资源分配优化

- 解决 Infer 阶段计算不均衡问题；
- 提升模型在推理时 Scaling 的实际增益；
- 进一步优化 MFU（模型浮点运算利用率）。

---

## 关键技术细节

### Semantic ID：RQ-Kmeans 继承与改进

延续 OneRec v1 的 Residual K-Means Quantization（残差 K-Means 量化）方案：
- 多级残差量化，每级使用 Balanced K-Means Clustering；
- 避免 RQ-VAE 的 codebook collapse 问题；
- 保留多模态融合的语义 ID 生成方式。

### 训练范式：三段式对齐

```
Pre-training → SFT → RL（ECPO）
```

1. **Pre-training**：Next Token Prediction 基础推荐能力；
2. **SFT**：监督微调，解锁多样化下游任务；
3. **RL**：ECPO 偏好对齐，与用户实际行为对齐。

---

## 与 OneRec v1 的对比

| 维度 | OneRec v1 | OneRec-V2 |
|------|-----------|-----------|
| 架构 | Encoder-Decoder | Decoder-Only |
| 偏好对齐 | IPA（迭代 DPO） | ECPO（改进 GRPO） |
| Scaling | 有限 | 推理阶段 Scaling 增强 |
| RL 能力 | 初步 | 更强、更稳定 |

---

## 相关论文

- [OneRec (arXiv:2502.18965)](https://arxiv.org/abs/2502.18965) — 第一版
- [OneRec-Think (arXiv:2510.11639)](https://arxiv.org/abs/2510.11639) — 显式推理链增强版
- [OpenOneRec (arXiv:2512.24762)](https://arxiv.org/abs/2512.24762) — 开源版本
