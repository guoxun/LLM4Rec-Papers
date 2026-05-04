# OneRec-Think: In-Text Reasoning for Generative Recommendation

**论文链接**: [arXiv:2510.11639](https://arxiv.org/abs/2510.11639)  
**机构**: 快手 (KuaiShou)  
**作者**: Zhanyun Liu, Shiyao Wang, Guorui Zhou 等  
**时间**: 2025年10月

---

## 核心问题

现有生成式推荐模型（如 OneRec）本质上是**隐式预测器**：

- 直接从用户历史行为序列生成推荐结果；
- 缺乏可验证的显式推理路径（reasoning pathway）；
- 用户无法理解推荐逻辑，系统可信度低；
- 无法动态适应用户的特定约束或对话式交互需求。

目标：将 DeepSeek-R1、Seed-1.5 等 LLM 推理技术引入推荐系统。

---

## 核心方案：OneRec-Think

统一集成**对话**、**显式推理链**与**个性化生成推荐**的单一模型框架。

### 三大能力

1. **显式 in-text 推理**：生成高质量、可解释的文本推理路径（CoT），推理过程透明可验证；
2. **对话式交互**：支持多轮对话，动态根据用户表达的约束调整推荐；
3. **个性化生成推荐**：在推理框架下保持高精度的推荐结果生成。

### Think-Ahead 架构（工业部署关键）

为解决推理生成带来的**延迟瓶颈**，提出 Think-Ahead 两阶段推理架构：

```
阶段1（离线/预计算）: 用户历史行为 → 生成推理摘要 / 兴趣画像
阶段2（在线）: 推理摘要 + 当前请求 → 快速生成推荐结果
```

- 将计算密集的推理部分移至离线，在线请求仅执行轻量解码；
- 实现生产级实时延迟要求（production-grade latency）。

### 训练策略

借鉴 DeepSeek-R1 等工作，使用 GRPO / DAPO / VAPO 等 RL 技术：

- 优化推理行为（reasoning behavior），鼓励模型生成有益的中间思考步骤；
- 通过奖励信号对齐推理质量与最终推荐准确率；
- 确保生成的推理路径与推荐结果之间的语义一致性。

---

## 意义与定位

OneRec-Think 代表了 One系列 的关键演进方向：

```
OneRec（端到端生成）
    ↓
OneRec-V2（RL + Scaling 增强）
    ↓
OneRec-Think（显式推理 + 对话）
    ↓
OpenOneRec（开源 + 通用化）
```

- 弥合 LLM 推理能力与推荐系统精度之间的鸿沟；
- 为推荐系统引入 Chain-of-Thought 推理范式；
- 为 Agentic Recommendation（智能体推荐）奠定基础。

---

## 相关论文

- [OneRec (arXiv:2502.18965)](https://arxiv.org/abs/2502.18965) — 基础版本
- [OneRec-V2 (arXiv:2508.20900)](https://arxiv.org/abs/2508.20900) — RL 增强版
- [OpenOneRec (arXiv:2512.24762)](https://arxiv.org/abs/2512.24762) — 开源通用版
