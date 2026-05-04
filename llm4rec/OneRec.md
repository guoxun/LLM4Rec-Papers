# OneRec: Unifying Retrieve and Rank with Generative Recommender and Iterative Preference Alignment

**论文链接**: [arXiv:2502.18965](https://arxiv.org/abs/2502.18965)  
**机构**: 快手 (KuaiShou)  
**作者**: Jiaxin Deng*, Shiyao Wang*, Kuo Cai*, Lejian Ren*, Qigen Hu*, Weifeng Ding*, Qiang Luo*, Guorui Zhou*†  
**时间**: 2025年2月

---

## 核心问题

传统推荐系统采用多阶段级联架构（召回→粗排→精排），存在两个根本性问题：

1. **前链路制约后链路**：前序阶段过滤掉的 item 无法被后续阶段召回，即使其推荐价值更高；
2. **Point-wise 建模忽略 session 内交互**：同一次请求内展示的多个 item 彼此独立建模，缺乏序列一致性。

---

## 核心方案

OneRec 用一个**统一的端到端生成式模型**替代整个多阶段级联系统。

### 模型架构：Encoder-Decoder + MoE

- **Encoder**：对用户历史行为序列（多尺度：静态特征 + 短期行为 + 有效观看序列 + 终身序列）进行编码；
- **Decoder**：基于 MoE（Mixture of Experts）架构，逐点生成 session 内推荐结果；
- 采用稀疏 MoE 扩展模型容量，计算 FLOPs 不随参数量等比增长。

### 物品 Token 化：RQ-Kmeans 分层语义 ID

传统 RQ-VAE 存在 codebook collapse（"沙漏现象"）问题。OneRec 改用多级残差 K-Means 量化：

- 每级强制每个 cluster 包含相同数量的 item（平衡分配）；
- 逐级编码残差，形成从粗到细的层级语义 ID（3层）；
- 融合多模态信号（标题、标签、ASR、图像识别）+ 协同过滤信号。

### Session-wise 生成（核心创新）

与传统 next-item prediction（逐点生成）不同，OneRec 直接生成一个完整的 **session**（一次推荐请求的全部结果），使 session 内 item 之间具备上下文连贯性。

训练目标：

$$\mathcal{L}_{\text{NTP}} = -\sum_{i=1}^{m}\sum_{j=1}^{L}\log P(s_i^{j+1} \mid [\bar{\mathcal{S}}_{<i}, s_i^1, \ldots, s_i^j]; \Theta)$$

### 迭代偏好对齐（IPA）：改进版 DPO

推荐场景的特殊性：每次请求只有一次曝光机会，无法同时获取正负样本对。

解决方案：
1. **训练 Reward Model**（多任务：CTR、VTR、WTR、LTR）；
2. 用 beam search 生成多个候选 session，RM 打分；
3. 取最优/最差得分构建偏好对，用 DPO 优化模型；
4. **迭代**执行上述过程，逐步对齐用户偏好。

---

## 工程优化（MFU）

传统推荐模型 MFU 长期处于个位数。OneRec 通过以下方式大幅提升：

- 关键算子数量压缩 92%（1,200 个）；
- 训练 MFU 达 **23.7%**，推理 MFU 达 **28.6%**；
- 与 LLM 社区 MFU 水平基本接近。

---

## 在线实验结果（快手主站 + 极速版）

| 指标 | OneRec | OneRec w/ RM Selection |
|------|--------|------------------------|
| 停留时长（快手）| 基线对齐 | +0.54% |
| 停留时长（极速版）| 基线对齐 | +1.24% |
| 7日用户生命周期 LT7 | +0.05% | +0.08% |
| 点赞/关注/评论 | 全正向 | 全正向 |

> 快手体系中，0.1% 停留时长或 0.01% LT7 提升即具统计显著性。

**当前部署状态**：快手主站已全量，承担约 **25% QPS**；本地生活场景 **GMV +21.01%**，已 100% 全量切换。

---

## 核心贡献总结

1. 首个在真实大规模场景中**显著超越**传统多阶段级联推荐系统的端到端生成式模型；
2. Session-wise 生成范式优于 point-wise 生成；
3. IPA 迭代偏好对齐优于普通 DPO；
4. 推荐系统中找到了 **Scaling Law**；
5. LLM 社区技术（RL、MoE、MFU 优化）可直接迁移应用。

---

## 相关论文

- [OneRec Technical Report (arXiv:2506.13695)](https://arxiv.org/abs/2506.13695) — 完整技术报告
- [OneRec-V2 (arXiv:2508.20900)](https://arxiv.org/abs/2508.20900) — 第二版，解决计算分配不均 + RL 增强
- [OneRec-Think (arXiv:2510.11639)](https://arxiv.org/abs/2510.11639) — 引入显式推理链
- [OpenOneRec (arXiv:2512.24762)](https://arxiv.org/abs/2512.24762) — 开源版本
